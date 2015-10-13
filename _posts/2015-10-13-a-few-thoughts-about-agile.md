---
layout: post
title: "A few thoughts about Agile"
description: ""
tags: [Agile]
---
{% include JB/setup %}

These days, Agile lightweight methodologies seem to be used as major software development methods across the world. Even not software development companies are using Agile processes. Scrum is one of the most common implementations of an Agile process. Recently, I’m thinking a lot about agile methodologies and it understanding after fourteen years when the [Agile Manifesto](http://www.agilemanifesto.org/) came out and I want to share my thoughts.
Before you read further, think about what is Agile for you?

**Quick recall**

The other software development process which is often contrasted to Agile methodologies is [waterfall method](https://en.wikipedia.org/wiki/Waterfall_model).
In waterfall process there were phases executed step by step. Input to the next stage was output from the previous. The assumption was, if we spend adequate amount of time on given phase and do it well, there will be no need for changes latter. For example, in design phase there was [Big Design Up Front](https://en.wikipedia.org/wiki/Big_Design_Up_Front), and the argument was if we first design architecture of our system, therefore we don't have to change anything in implementation phase. Unfortunately, that's not true, because we can't foreseen everything and in waterfall we can't go back and change our design (at least easily). There wasn't feedback of our design until the implementation phase was finished. That was the problem with waterfall process - lack of early feedback and difficulty in introducing changes. And that is what Agile changed - possibility of getting early feedback and responding to change even in lately development. The phases from waterfall are repeated in small time boxes and that defines iterative and incremental development. Because phases are repeated, therefore we have iterative development. New value to the software product is added feature by feature, therefore we have incremental development.
Apart from that, you can see, that's no surprise that other methods which gives us early feedback at different levels of creating software product are widely used. That's why we use [test-driven development](https://en.wikipedia.org/wiki/Test-driven_development), pair programming, code reviews and so on.

**What you hear when you ask about Agile?**

Probably when you ask somebody about Agile you will hear about sprint planning, board, cards, sprint retrospective and other artifacts and ceremonies connected with Scrum (because is the most common implementation of Agile).
But they are implementation details. And like when we write code if we focus our system too much on implementation details it makes the system tightly coupled. The same I think focusing too much on implementation details prevent us for understanding what agile is aim for.

Introducing an Agile process into company means that you want to collect feedback early and therefore you welcome changes. But what if at the down level the change is hard to implement? You are agile because you write tasks on cards, estimate it and stick to the board or have daily standups? I've seen teams that doesn't use neither Scrum nor any of the known agile frameworks but their code really embrace change. So, you would say they aren't agile? 

I think in the jungle of Scrum ceremonies and artifacts (or maybe not properly understanding what Agile aims for) we forgot about other [principles that are behind Agile Manifesto](http://www.agilemanifesto.org/principles.html). How many times did you see a piece of code that wasn't good enough and you did nothing about that? The phrase *I don't have time* is a poor explanation for professional software developer. We should be responsible for the code that we work on. If we make our code adaptive to change, but have problems with our software management process it's still better than otherwise.

**Leave the code better than you found it. (The Boy Scout Rule)**

Yes, that's important part of being agile. Carrying over the quality of the software product. And if you wonder how you would know that your change would not mess in other parts of the system. The answer is that you didn't understand Agile idea well, you didn't build your feedback foundation at the code level - you fooled yourself that the artifacts and events is all what you need to claim that you are agile.

**Thank you Scrum inventors!**

I'm not saying the Scrum or other Agile method is unnecessary. It is very helpful and we only could be thankful to inventors of the Scrum framework. But we must also understand that the codebase and it's quality is a huge part of that how the software product is agile (even more bigger part than the process itself). You may listen to as many podcasts about Agile as you want, go to conferences and hear about best Agile practices but that is a waste of time until you build your feedback foundation and make your codebase adaptive to change.

**Summary**

To wrap up the Agile is more about feedback and embracing changing requirements than particular artifacts and ceremonies. 
If you find something interesting in this post, let everybody know, otherwise forget about it.
