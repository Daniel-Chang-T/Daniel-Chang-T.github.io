---
layout: post
title:  "什麼是Free Monad/自由單子"
date:   2022-08-30 10:40:23 +0800
categories: jekyll update
usemathjax: true
---

## 緣起
之前實習時讀到了很多有趣東西，想說寫下來當作實習成果+推廣這些有趣觀念。
```haskell
fmap::(a->b) -> f a -> f b
fmap g fx = 
```
## 先備知識
- 能閱讀Haskell語法。比較重要的是能理解 Type constructor 、 type class 、currying的概念。這部分可以參考[HASKELL 趣學指南](https://learnyouahaskell.mno2.org/)
- 知道什麼是Functor和Monad，與他們對應的Laws。如果不熟悉同樣可以參考[HASKELL 趣學指南](https://learnyouahaskell.mno2.org/)或[穆信成老師對Monad的說明](https://scm.iis.sinica.edu.tw/ncs/2009/11/a-monad-primer/)
- 有碰過一些範疇論或抽象代數會更好，但沒有也沒關係。
## 兩種 Monad 定義
在大多數講Haskell的參考資料中，Monad 是由 $return$ 和 $>>=$這兩個operator來定義的。但為了等一下比較方便，這裡介紹另一種Monad的定義方式：$return::a\rightarrow m a, fmap$ 和 $join$。

## Free monad 的 "Free" 是什麼意思?
如果你很剛好的修過抽象代數，你八成會遇過一個叫Free group的東西，Free Monad 的"Free" exactly就是 Free group的 "Free"。我的代數助教是這麼說的:「所謂的Free group呢，就是一個很沒有結構的group。那要怎麼描述它很沒有結構這件事呢?就是你隨便給一個group，都能從Free group serjectively的打過去」。