---
layout: post
title: "Learning functional programming"
tags: [Functional programming, Software engineering]
---
{% include JB/setup %}

It's been some time since I wrote last post. I hope, now I will be able to write posts more often. Although, it is getting warmer here in Poland, so hiking and cycling will be possible :)

### Functional paradigm everywhere

Nowadays, functional paradigm is quite popular. `Java` has got some features known from functional languages as well as `C#`. There's assumption that functional programming is a way to solve problems we have now in object oriented programming. You can find some blog entry, video tutorial, workshop, presentation everywhere, where someone is talking about how to approach your code by functional programming.
Many people think that functional programming is about immutability. But it is not. You can have immutable types also in `C#` or `Java`. You don't have it out of the box, but it is possible. Even more, immutability appeared many years ago, for example Eric Evans in his blue book was talking about immutable value objects and how they help to build code, which is easy to maintain and work on. Not to even mention about advantages when it comes to parallelism.
Being functional means composing higher order functions of small functions. Those functions are so small that we can test them easily and quickly.

### How we learnt OO ?

It seems that we had similar situation before, when object-oriented programming had its time. It was introduced as a better solution than procedural programming. Do you remember how at university we were first introduced to procedural programming and then to object-oriented programming? At the beginning each of us was writing procedural code using object-oriented language. And still many of us are. They introduced us to object orientation in such a way that we have `Shape` and from `Shape` we can derive `Circle`, `Rectangle` and so on. And we believed that this is the root what object orientation is about. How to define data in reusable way, that they share common characteristics and how to leverage inheritance. However, we suddenly get into problems like [Fragile Base Class](https://en.wikipedia.org/wiki/Fragile_base_class) using this approach. 
It's not possible to write generic code that it will handle every cases, is it?. To be honest, think a while how it's easy to explain such an approach to OO even to your mum. Because it's similar to what we have in our life. Every rectangle is a shape, isn't it? This is the same reason why they explained to us in such a way at university.

### Another view at OO
For a while let's back to Alan Kay (considered by some as father of object-oriented programming) famous sentence:
> It's all about messaging.

There is another view at object-oriented systems. Key is to focus on how objects communicate with each other. What messages they send? Object can receive message, takes parameters and send message to another object or return response. It's easy to understand when you take a look at `Smalltalk` language because there notion of sending messages is explicit. For me it's quite different view from that one focusing on the structure. I think it's better because it makes that we think in terms of objects. And then objects are not only holders for data. They play their own roles, they have their responsibilities. They must have their state encapsulated properly, because inproper messages can corrupt object. Proper naming becomes then much more important. Names of objects must reflect what role objects play and must reflect domain. Then it's easy to reason about it.
When you are looking at OO system using messaging approach, then you realize how important principles like Tell Don't Ask or Law of Demeter are. To be honest, SOLID rules become more clear. And they pretty good fit to this view.

### Learn by one's mistakes

So now I also got a little bit into functional programming. It's very entertaining for somebody so far familiar with OO languages. I like description that functional programming is about defining recipes for our data. And after that composing functions to perform interesting behavior for us. And those functions are so small that we can test them easily and quickly in REPL environment. This is a huge advantage that we can play with functions at REPL. Also thanks to immutability which is built-in into functional language, compilers can make some additional steps to compile such code to executable code. To be honest, I'm amazed by functional programming. I'm trying to play with Haskell, to learn functional programming correctly. I don't want to make the same mistake as with OOP, because it took me some time to understand benefits of it (or that's what I think - as always it is opinionated view). At the end of the day, ultimate goal is to write idiomatic functional code without heritage from OOP.