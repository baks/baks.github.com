---
layout: post
title : "Simple pattern matching in C#"
tags : [C#, .NET, Functional programming]
---
{% include JB/setup %}

Nowadays, functional languages are becoming very popular. Few years ago we thought that the object orientation would be the answer for complexity in systems. But now, there is a belief that functional languages are the new solution. We will see what is gonna be the "thing" in the next years. 

Lately, I had an occasion to apply one of feature from functional languages i.e pattern matching to code written in C#. But let's start from the beginning.

### Switch statement case study

Couple days ago, I encountered a piece of code which you can find below. It checks which hint is passed to the method. Then, it returns appropriate provider.

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="nc">IHintProvider</span> <span class="nf">GetHintProvider</span><span class="p">(</span><span class="kc">string</span> <span class="n">hint</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">switch</span> <span class="p">(</span><span class="n">hint</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">case</span> <span class="n">HintOne</span><span class="p">:</span>
            <span class="k">return</span> <span class="k">new</span> <span class="nc">HintOneProvider</span><span class="p">();</span>
        <span class="k">case</span> <span class="n">HintTwo</span><span class="p">:</span>
            <span class="k">return</span> <span class="k">new</span> <span class="nc">HintTwoProvider</span><span class="p">();</span>
        <span class="k">case</span> <span class="n">HintThree</span><span class="p">:</span>
            <span class="k">return</span> <span class="k">new</span> <span class="nc">HintThreeProvider</span><span class="p">();</span>
        <span class="k">case</span> <span class="n">HintFour</span><span class="p">:</span>
            <span class="k">return</span> <span class="k">new</span> <span class="nc">HintFourProvider</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">return</span> <span class="k">new</span> <span class="nc">NullHintProvider</span><span class="p">();</span>
<span class="p">}</span>
</code></pre></div>

It looks like a procedural code and I felt bad about the code, especially that I made it and probably forgot to refactor it. Nevertheless, it is not as worst as it could be, because for example it could have used the `if` and `else if` statements. 

### Table-driven methods

I started to look for better alternatives, whether I can refactor this slightly to reduce the switch statement noise. This led me to a table-driven method. I have known it before from [Code Complete book](http://amzn.com/0735619670 ). Basically it means, that sometimes we can precompute answers for given paths and store it in a table (or in our case in dictionary). Then, given an argument we just do lookup and return stored value. Owing to using this technique, we can now refactor `GetHintProvider` method:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">private</span> <span class="k">readonly</span> <span class="nc">IDictionary</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">,</span> <span class="nc">IHintProvider</span><span class="p">&gt;</span> <span class="n">providers</span> <span class="p">=</span> <span class="k">new</span> <span class="nc">Dictionary</span><span class="p">&lt;</span><span class="kt">string</span><span class="p">,</span> <span class="nc">IHintProvider</span><span class="p">&gt;</span>
<span class="p">{</span>
    <span class="p">{</span><span class="n">HintOne</span><span class="p">,</span> <span class="k">new</span> <span class="nc">HintOneProvider</span><span class="p">()},</span>
    <span class="p">{</span><span class="n">HintTwo</span><span class="p">,</span> <span class="k">new</span> <span class="nc">HintTwoProvider</span><span class="p">()},</span>
    <span class="p">{</span><span class="n">HintThree</span><span class="p">,</span> <span class="k">new</span> <span class="nc">HintThreeProvider</span><span class="p">()},</span>
    <span class="p">{</span><span class="n">HintFour</span><span class="p">,</span> <span class="k">new</span> <span class="nc">HintFourProvider</span><span class="p">()}</span>
<span class="p">};</span>

<span class="k">public</span> <span class="nc">IHintProvider</span> <span class="nf">GetHintProvider</span><span class="p">(</span><span class="kc">string</span> <span class="n">hint</span><span class="p">)</span> <span class="p">=&gt;</span> 
	<span class="n">providers</span><span class="p">.</span><span class="n">ContainsKey</span><span class="p">(</span><span class="n">hint</span><span class="p">)</span> <span class="p">?</span> <span class="n">providers</span><span class="p">[</span><span class="n">hint</span><span class="p">]</span> <span class="p">:</span> <span class="k">new</span> <span class="nc">NullHintProvider</span><span class="p">();</span>
</code></pre></div>

### Simple pattern matching

But, as you can see the syntax of creating dictionary mimics somewhat the pattern matching feature from functional languages. I have been wondering if we are able to use this construction to create a simple pattern matching functionality in `C#`. It turns out, that `C#` compiler in some situations uses kinda [duck-typing](https://en.wikipedia.org/wiki/Duck_typing). If a type implements `IEnumerable`, we can use collection initializer while creating object instance of that type. Instances passed in collection initializer compiler translates to corresponding `Add` methods with matching signature. Given that, we can create `Match` class that stores patterns and associates code with them to execute when pattern matches. With that, we can write `GetHintProvider` method as following:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="nc">IHintProvider</span> <span class="nf">GetHintProvider</span><span class="p">(</span><span class="kc">string</span> <span class="n">hint</span><span class="p">)</span> <span class="p">=&gt;</span>
    <span class="k">new</span> <span class="nc">MatchFunc</span><span class="p">&lt;</span><span class="kc">string</span><span class="p">,</span> <span class="nc">IHintProvider</span><span class="p">&gt;</span>
    <span class="p">{</span>
        <span class="p">{</span><span class="n">HintOne</span><span class="p">,</span> <span class="n">s</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nc">HintOneProvider</span><span class="p">()},</span>
        <span class="p">{</span><span class="n">HintTwo</span><span class="p">,</span> <span class="n">s</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nc">HintTwoProvider</span><span class="p">()},</span>
        <span class="p">{</span><span class="n">HintThree</span><span class="p">,</span> <span class="n">s</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nc">HintThreeProvider</span><span class="p">()},</span>
        <span class="p">{</span><span class="n">HintFour</span><span class="p">,</span> <span class="n">s</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nc">HintFourProvider</span><span class="p">()},</span>
        <span class="p">{</span><span class="nc">Match</span><span class="p">.</span><span class="n">Default</span><span class="p">,</span> <span class="n">s</span> <span class="p">=&gt;</span> <span class="k">new</span> <span class="nc">NullHintProvider</span><span class="p">()</span> <span class="p">}</span>
    <span class="p">}.</span><span class="n">Func</span><span class="p">(</span><span class="n">info</span><span class="p">).</span><span class="n">Single</span><span class="p">();</span>
</code></pre></div>

Now, the code looks nice to me and it is more declarative than the original example. 

If you are interested in, you can check code for `MatchFunc` class [here](https://github.com/baks/PatternMatching/tree/master/src/SimplePatternMatching).

### Summary
Being declarative, expressing intent of code is an advantage of functional languages, that is presented as opposite to imperative code. Actually, I think code is easier to understand and to maintain. I'm curios whether `C#` language engineers have thought that its collection initializers syntactic sugar could be used in such a way. If not, it is interesting how the solution to problem in software field can be used in not foreseen ways later.