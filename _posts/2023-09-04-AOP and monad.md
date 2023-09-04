---
layout: post
title:  "Aspects are side effects"
date:   2022-09-04 20:40:23 +0800
categories: jekyll update
usemathjax: true
---
First I have to say, I am not an expert in AOP and more , so there's almost surely some misunderstanding in AOP. Very welcome for who interested in this topic sned me email for discussion.

Just came out of the idea walking home. Wrote some java in Spring lately and (maybe unevitable) read some AOP concepts. These concepts are so wiredly familiar, which turns out to be that's because what they designed to do is basically "adding" side effects to funcions. For example, one of the most common example of AOP application is the logger Aspect, which logs everytime specified method is called. This is exactly what writer monad does! 

When talking about side effects, it's nature to discuss how the side effects interact with each other. But for aspects it's not really a big deal since the modularity of the aspect often itself provides the tensor semantics. Interceptors and filters are degeneracy of the continuatoin effect

Forgot to mention that how the operations accompnay with monads implicitly used in AOP. The advices are the actuall operations on the monad, the operation signature are all simple operations paremetrized by:
- the metatdata of join-points. e.g. name of adviced method.
- the adviced controling code block itself as a function.

Write to here, it's more clear that actually advices in aspects are actually reader monads with states(from the language's nature mutability), paremetrized by the metatdata of join-points. And advicing is twisting the environment monad the adived part lives in. To make it more clear, I'll write the signature explicitly below:

$$ Advice:: metadat -> (a->r) -> (a->r) $$