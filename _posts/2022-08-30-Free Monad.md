---
layout: post
title:  "什麼是Free Monad/自由單子"
date:   2022-08-30 10:40:23 +0800
categories: jekyll update
usemathjax: true
---

## 緣起
之前實習時讀到了很多有趣東西，想說寫下來當作實習成果+推廣這些有趣觀念。
## 先備知識
- 能閱讀 Haskell 語法。比較重要的是能理解 Type constructor、type class、currying 的概念。這部分可以參考 [HASKELL 趣學指南](https://learnyouahaskell.mno2.org/)
- 知道什麼是 Functor 和 Monad，與他們對應的 Laws。如果不熟悉同樣可以參考[ HASKELL 趣學指南](https://learnyouahaskell.mno2.org/)或[穆信成老師對 Monad 的說明](https://scm.iis.sinica.edu.tw/ncs/2009/11/a-monad-primer/)
- 高中數學
- 有碰過一些範疇論或抽象代數會更好，但沒有也沒關係。

## Free Structure/自由結構
要了解什麼是 free monad，我們得先知道這個 free 到底是啥意思。如果你有學過抽象代數，對沒錯這就是 free group 的 free，你可以跳去下一節了。對於沒學過的人，我會在這裡介紹一個簡單的 free structure，來讓你感受一下"free"的感覺。
### monoid
monoid 是最簡單的代數結構之一，它的定義包含了一個集合 $M$，一個二元運算子 $\oplus$ ( 或者說一個$M\times M \to M$的 function )，和一個 $M$ 中的元素 $e$。這個運算子如果滿足以下兩條定律，$(M, \oplus, e)$ 就被稱為一個 monoid。
- identity element / 單位元 : $\forall x\in M, x \oplus e = e\oplus x = x$
- associativity / 結合律 : $\forall x,y,z\in M, (x\oplus y)\oplus z = x\oplus(y\oplus z)$

常見的 monoid 例子有 $(Int , + , 0 )$ 、 $(Int , \cdot , 1 )$、  ( $n$ 階方陣 , 矩陣乘法 , 單位矩陣 ) 以及很重要的 $( [a] , ++, [] )$ 。
### monoid morphism
morphism 指的是那些會保持代數結構的，從結構到結構的函數。像一個從 $(M_1, \oplus _1,e_1)$ 到 $(M_2, \oplus _2,e_2)$ 的 monoid morphism f，就該滿足這條式子 $\forall x,y\in M_1 , f(x \oplus _1 y) = f(x) \oplus _2 f(y)$
### $([a] , ++, []  )$ 是 free monoid over $a$
我們先把 type theorist 的嘴堵上，假裝每個 type 都會是一個 set，每個 set 也都是一個 type，好省掉許多麻煩的討論。 $(M, \oplus, e)$ 要被稱為free monoid over $a$ ，對於所有在$a$ 上訂出來的 monoid $(a, \oplus', e')$ ，都要存在從 $(M, \oplus, e)$ 到 $(a, \oplus', e')$ 的 serjective monoid morphism。 $( [a] , ++, [] )$ 滿足這條件嗎? 對這個 case 而言，monoid morphism $f$ 是可以直接被明確定義出來的 : 
$$
f :: [a] \to a\\
f\ [] = e'\\
f\ (x:xs) = x\ \oplus'\ (f xs)
$$
或者也可以寫成
$$
f :: [a] \to a\\
f = foldr\ (\oplus')\ e'
$$
$f$ 的 serjectivity 是顯而易見的，因為 $\forall x \in a, f\ [x] = x \oplus' e' = x$

如果要驗證 $f$ 真的是個 monoid morphism，就要確認 $f\ ([a_1,a_2,...,a_n] ++ [a_{n+1},...,a_m]) = (f\ [a_1,a_2,...,a_n]) \oplus' (f\ [a_{n+1},...,a_m])$ 這條等式真的永遠成立，證明過程自然是對第一個 list 的長度 $n$ 做 induction，base case 跟 induction case 對應到了 $(a,\oplus',e')$ 的單元律與結合律。 ( 詳細證明留給讀者作為習題 XD )

### 所以什麼是 free structure
我想改寫之前聽過的，代數助教的說法來總結 : 「一個 free structure 是只有最基本的結構的東西，所以對於任何更有結構的東西，它都有辦法映射過去。」

## 兩種 Monad 定義
如果前面那段你讀的霧茫茫，這段會岔開一下講別的東西，給你點時間接受 free structure。
在大多數講Haskell的參考資料中，Monad 是由 $return$ 和 $>>=$這兩個operator來定義的。但為了等一下比較方便，這裡介紹另一種Monad的定義方式：$return::a\rightarrow m a, fmap$ 和 $join$。

## Free monad 的 "Free" 是什麼意思?
如果你很剛好的修過抽象代數，你八成會遇過一個叫Free group的東西，Free Monad 的"Free" exactly就是 Free group的 "Free"。我的代數助教是這麼說的:「所謂的Free group呢，就是一個很沒有結構的group。那要怎麼描述它很沒有結構這件事呢?就是你隨便給一個group，都能從Free group serjectively的打過去」。