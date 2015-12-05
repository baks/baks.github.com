---
layout: post
title : "Design patterns koans - koan #1"
tags : [Software design, Design patterns]
---
{% include JB/setup %}

Exercises that are intended to check your knowledge of language or teach something about it are called [Koans](https://en.wikipedia.org/wiki/K%C5%8Dan). For example you can find koans for JavaScript language [here](https://github.com/liammclennan/JavaScript-Koans). 

Often, I find that applying solution to some problem later turns out that it was a design pattern. So, I thought that it would be good to describe the context of a problem and leave it to figure out the solution as an exercise for you, dear reader. As the mentioned koans contain tests which initially fails, I will try to do the same and also create a suite of tests which should tell you if your solution is good.

### Problem context
Let's assume that we use [domain event pattern](http://martinfowler.com/eaaDev/DomainEvent.html) for notifying about events in our system. This is also handy when we want to use [Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html). We have a domain event definition that is simply marker interface and looks like this:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">interface</span> <span class="nc">IDomainEvent</span>
<span class="p">{</span>
<span class="p">}</span>
</code></pre></div>

We also have an interface for event handlers which are used to mark implementation that will handle these events:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">interface</span> <span class="nc">IEventHandler</span><span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="k">void</span> <span class="nf">Handle</span><span class="p">(</span><span class="nc">T</span> <span class="n">domainEvent</span><span class="p">);</span>
<span class="p">}</span>
</code></pre></div>

But, new requirement shows up and we need to introduce a [MassTransit library](https://github.com/MassTransit/MassTransit) because in the near future we want to use queue messaging system to reliably deliver domain events. So we change our `IEventHandler<T>` definition to implement `IConsumer<T>` interface from `MassTransit`.

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">interface</span> <span class="nc">IEventHandler</span><span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span> <span class="p">:</span> <span class="nc">IConsumer</span><span class="p">&lt;</span><span class="nc">T</span><span class="p">&gt;</span> <span class="k">where</span> <span class="nc">T</span> <span class="p">:</span> <span class="k">class</span><span class="p">,</span> <span class="nc">IDomainEvent</span>
<span class="p">{</span>
    <span class="k">void</span> <span class="nf">Handle</span><span class="p">(</span><span class="nc">T</span> <span class="n">domainEvent</span><span class="p">);</span>
<span class="p">}</span>
</code></pre></div>

Now, we can use our event handlers along with `MassTransit`.

You can find entire solution [here](https://github.com/baks/DesignPatternsKoans/tree/master/KoanOne). There is a simple console application that uses `MassTransit` to send messages over bus and shows that everything works. There are also unit tests which initially fails (tests are written using [NUnit 3](https://github.com/nunit/nunit)). The first failing test states that `IEventHandler` for `UserCreated` event should not be assignable from `IConsumer<T>` `MassTransit` interface. The second test states that DomainModel should not have any references to `MassTransit` library.

### Why bother?
In spirit of the Ports and Adapters architecture we should try to make our domain context independent. Using interface that we do not own from external library make the domain polluted with infrastructural concerns. Also that domain is not reusable if we want to switch to another message library. New domain module, that deals with events must also take dependency to `MassTransit`. What we learned in blue book, is that we should isolate domain model from other code.

### Summary
Every craftsmen who would like to become better must practice, as we can read in [Manifesto for Software Craftsmanship](http://manifesto.softwarecraftsmanship.org/). Therefore, I strongly encourage you to [fork](https://github.com/baks/DesignPatternsKoans#fork-destination-box), and try to change the code to make all tests pass. As you know I designed it to apply design pattern to solve the problem. However, this is not only solution, that you may think of. But I designed it having in mind a particular design pattern, so that's why I called it 'design pattern koans'.
