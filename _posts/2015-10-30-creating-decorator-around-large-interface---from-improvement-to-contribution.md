---
layout: post
title : "Creating decorator around large interface - from improvement to contribution"
tags : [.NET, Productivity, OSS]
---
{% include JB/setup %}

Recently, I've caught myself that I was creating [decorator](https://en.wikipedia.org/wiki/Decorator_pattern) around interface which has quite a few methods. I was interested only in one particular method, whereas for the others I had to delegate a call to the decoratee. Thus, it was tedious. This leads me to searching a way for doing this task automatically. 

Apart from that, we could argue that the interface does not follow [Integrace Segregation Principle](https://en.wikipedia.org/wiki/Interface_segregation_principle) as Robert C. Martin pointed out that
>Clients should not be forced to depend upon methods that they do not use. 

Unfortunately, we often work with code which we can't change and that's the case here.

#### Know your tools

I use *ReSharper*, so at the beginning I started to look if it can help with that. I started with the following code:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">class</span> <span class="nc">CachingRepository</span> <span class="p">:</span> <span class="nc">IRepository</span>
<span class="p">{</span>
<span class="p">}</span>
</code></pre></div>

After couple minutes of investigation, I found that if code generation pop-up is invoked (`Alt+Enter`) there is an option *Delegating members*, but it is unavailable. So, I thought that maybe I should add to my class a field of interface type, which I want to decorate.

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">class</span> <span class="nc">CachingRepository</span> <span class="p">:</span> <span class="nc">IRepository</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="nc">IRepository</span> <span class="n">repository</span><span class="p">;</span>
<span class="p">}</span>
</code></pre></div>

After that step, the option *Delegating members* became available. Moreover, if caret position is on the class name and we press the shortcut for context actions (`Ctrl+Enter`), there is also an action *Delegate implementation to 'repository' field* available.

*Visual Studio 2015*, also offers quick actions, which are indicated by light bulb. We can invoke quick actions by using `Ctrl+.` shortcut. In earlier mentioned code snippet, if we put caret on interface name in type list of decorator, there would be actions offered by *Visual Studio*. One of the available fixes is *Implement interface through 'repository'*, which produces the same results as the *ReSharper* code generation option.

#### More automation

I wondered why there is a requirement to include the field in class for the options to be available. After all, the field could be auto-generated. We can go even further and generate constructor with the decoratee as a parameter. 

Let's assume that we want to create a decorator around `IRepository` interface and we have:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">class</span> <span class="nc">CachingRepository</span> <span class="p">:</span> <span class="nc">IRepository</span>
<span class="p">{</span>
<span class="p">}</span>
</code></pre></div>

Why can't a tool do the work for us and generate following code?

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">class</span> <span class="nc">CachingRepository</span> <span class="p">:</span> <span class="nc">IRepository</span>
<span class="p">{</span>
    <span class="k">private</span> <span class="k">readonly</span> <span class="nc">IRepository</span> <span class="n">repository</span><span class="p">;</span>

    <span class="k">public</span> <span class="nf">CachingRepository</span><span class="p">(</span><span class="nc">IRepository</span> <span class="n">repository</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span> <span class="p">(</span><span class="n">repository</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">throw</span> <span class="k">new</span> <span class="nc">ArgumentNullException</span><span class="p">(</span><span class="k">nameof</span><span class="p">(</span><span class="n">repository</span><span class="p">));</span>
         <span class="p">}</span>
        <span class="k">this</span><span class="p">.</span><span class="n">repository</span> <span class="p">=</span> <span class="n">repository</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Save</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="n">repository</span><span class="p">.</span><span class="n">Save</span><span class="p">();</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">void</span> <span class="nf">Delete</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="n">repository</span><span class="p">.</span><span class="n">Delete</span><span class="p">();</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre></div>

Nowadays, thanks to [Roslyn](https://github.com/dotnet/roslyn) kind of task like that are much easier to implement. I thought about implementing it, but as I contribute to [code-cracker](https://github.com/code-cracker/code-cracker) project I came up with idea that this would be a good place to propose it as a new refactoring. Code-cracker is an analyzer library which provides helpful analysis and refactorings while you are writing C# (or VB) code. I created [new issue](https://github.com/code-cracker/code-cracker/issues/572) in code-cracker project with details how the refactoring should work. Maybe someone else would be eager to implement it, unless I do it by myself.

#### Summary
I'm quite surprised, how the idea of adhering to the [DRY principle](https://en.wikipedia.org/wiki/Don't_repeat_yourself) turned into a contribution to the code-cracker project. In growing OSS world it is valuable to share your ideas because someone else might find it useful. We just need to find appropriate place where the idea could be used.