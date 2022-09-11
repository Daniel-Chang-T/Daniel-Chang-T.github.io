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
我們先把 type theorist 的嘴堵上，假裝每個 type 都會是一個 set，每個 set 也都是一個 type，好省掉許多麻煩的討論。 $(M, \oplus, e)$ 要被稱為free monoid over $a$ ，對於所有在$a$ 上訂出來的 monoid $(a, \oplus', e')$ ，都存在唯一一個從 $(M, \oplus, e)$ 到 $(a, \oplus', e')$ 的 serjective monoid morphism。 $( [a] , ++, [] )$ 滿足這條件嗎? 對這個 case 而言，monoid morphism $f$ 是可以直接被明確定義出來的 : 

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

如果要驗證 $f$ 真的是個 monoid morphism，就要確認 $f\ (xs ++\ ys) = (f\ xs) \oplus' (f\ ys)$ 這條等式真的永遠成立，證明過程自然是對 $xs$ 的長度 做 induction，base case 跟 induction case 分別對應 monoid 的單元律與結合律。 ( 詳細證明留給讀者作為習題 XD )

唯一性的部分容我隨便豪小一下：因為 singleton of $a$ 形成的 set 與 $[]$ 透過 $++$ 這 operation 可以生出所有在 $[a]$ 中的元素，他們被 morphism 打過去的值會決定這整個 morphism 長什麼樣子，啊又因為 surjectivity 的關係，他們要被打到能生出整個 monoid 的 set ，再加上 $[a]$ 本身的構造，他們就只能被那樣打過去。合在一起就會得到這個 morphism 唯一的論述。

上面寫的部分有點抽象，我們可以來看更實際例子 : $[1,4,5]$ 會怎麼被打到 $(Int, +, 0)$ 呢? 答案是 $1 + 4 + 5 + 0 = 10$ ，那$(Int, \cdot, 1)$ 呢? 答案是 $1 \cdot 4 \cdot 5 \cdot 1 = 20$。這帶來一個重要的 intuition：free monoid 是把「要被計算的東西」紀錄下來，至於「計算的結果」則等待一個真實的運算(e.g. 上面的$+$ 和 $\cdot$) 出現，才會真的被算出來。這個 intuition 基本上就是 programming 領域對 free structure 的用法 : 紀錄「計算」本身，好等晚點再來真的去算它。

## 兩種 Monad 定義
如果前面那段你讀的霧茫茫，這段會岔開一下講別的東西，給你點時間接受 free structure 的想法。
在大多數講Haskell的參考資料中，Monad 是由 return 和 >>= 這兩個operator來定義的。但為了等一下比較方便，這裡介紹另一種Monad的定義方式：

```haskell
class Monad m where
    fmap    ::  (a -> b) -> m a -> m b 
    pure    ::  a -> m a
    join    ::  m (m a) -> m a
```
它們當然也需要滿足一些 laws :
```haskell
join.join           = join.(fmap pure)      :: m (m (m a)) -> m a
join.pure           = id                    :: m a -> m a
join.(fmap pure  )  = id                    :: m a -> m a

fmap id             = id                    :: m a -> m a
```
Note : 因為有很棒的 parametric theorem a.k.a. theorems for free (之後有機會再介紹)，fmap 其實只需要滿足 idenetity law 就夠了。

這個定義法和 $return,>>=$ 合在一起是等價的，用下面的方式可以互相轉換

```haskell
pure      = return
fmap    ::  (a -> b) -> m a -> m b 
fmap f mx = mx >>= (return.f)
join    ::  m (m a) -> m a
join mmx = mmx >>= id

return   = pure
(>>=)   :: m a -> (a -> m b) ->  m b
mx >>= f = join (fmap f mx)
```

這種定義法可以很清楚的看出「Monad 是 Functor」這件事，更準確的說，Monad 就是「有 pure 跟 join 的 Functor」。有一種很流行的說法是把 monad 看成 value with context，用這說法來看的話， join 定義了兩層的 context 怎麼被壓成一層，可說是決定了 monad 行為的操作。如果稍微拉的遠一些看 Monad laws，會發現這裡的 join 跟 pure 可以約略類似上面 monoid 的 $\oplus$ 和 $e$

```haskell 
join.join           = join.(fmap join)      <-> ∀x,y,z∈M,(x⊕y)⊕z=x⊕(y⊕z)

join.pure           = id                    <-> ∀x∈M,e⊕x=x
join.(fmap pure  )  = id                    <-> ∀x∈M,x⊕e=x
```
這裡的對應有可能是錯的，還請意思意思一下就好，如果想要正確無誤的詳細版本，還請參考[Notions of Computation as Monoids](https://arxiv.org/abs/1406.4823)，讀完以後應該會對 Monad 有更深刻的見解。

## So What is Free monad

總之，我們可以約略的把 join 當成 $\oplus$ ，所以一個 Free monad over Functor f 就會是「對於任意定在 f 上的 pure 和 join ，都存在唯一一個 sejective Monad morphism 」  而 monad morphism 粗略來說就是滿足 g.join = join.(fmap g) 的 g。

仿造剛剛 monoid 的做法， Free Monad 應該要是個把「用 Functor 包起來」這件事記錄下來，晚點再真正去用拿到的 join 把多層 Functor 壓成一層的東西，它要怎麼做呢? 我們直接來看[別人的實作](https://serokell.io/blog/introduction-to-free-monads)
```haskell
data Free f a = Pure a | Free (f (Free f a))
```
長的很像 list
```haskell
data List a  = Nil | Con a (List a)
```
Free f 要怎麼是 monad 呢?
```haskell
instance (Functor f) => Monad (Free f) where
    fmap g (Pure x)     = Pure (g x)
    fmap g (Free fx)    = Free (fmap g fx)

    pure x              = Pure x

    join (Pure x)       = x
    join (Free fx)      = Free (fmap join fx)
```

舉例來看，如果 Functor 是 MyPair
```haskell
data MyPair a = P a a
instance Functor MyPair where
    fmap f (P x y) = P (f x) (f y)
```
Free MyPair 就會是
```haskell
Free MyPair a   = Pure a | Free (MyPair (Free Mypair a))
```
既然已經知道 MyPair 的 Constructor 了，可以把它寫開
```haskell
Free MyPair a   = Pure a | Free (P (Free Mypair a) (Free Mypair a))
```
有沒有似曾相識的感覺? 它的結構跟 External labeled binary tree 一模一樣
```haskell
ExtBin a        = Leaf a | Node (ExtBin a)          (ExtBin a)
```
這並不是巧合，而是 Free 本來就有很類似 External labeled tree 的結構，Pure 跟Leaf 一樣是有值的葉子，Functor Constructor 中 a 出現的次數則決定這個 Node 有幾條分岔，這樣來看 join 是把葉子由樹構成的樹合併成一棵樹的那個自然操作。

## Reference
[stackoverflow , join fmap laws](https://stackoverflow.com/questions/45829110/monad-laws-expressed-in-terms-of-join-instead-of-bind)