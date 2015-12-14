---
layout: post
title : "LINQ Except method explanation"
tags : [LINQ, .NET]
---
{% include JB/setup %}

`LINQ` has been in .NET from 3.5 version. It simplifies a lot many tasks, allowing expressing data processing in more declarative manner. You probably had an occasion to use `Except` method. But, do you know exact workings of `Except` method?

At [MSDN](https://msdn.microsoft.com/library/bb300779%28v=vs.100%29.aspx) we can find that :

><div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">static</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">Except</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;(</span>
  <span class="k">this</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">first</span><span class="p">,</span>
  <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">second</span>
<span class="p">)</span>
</code></pre></div>

> Produces the set difference of two sequences by using the default equality comparer to compare values.<br/>

Having above information, do you have any idea, what would be written into the console when following code is executed?

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">var</span> <span class="n">invalidChars</span> <span class="p">=</span> <span class="nc">Path</span><span class="p">.</span><span class="n">GetInvalidFileNameChars</span><span class="p">();</span>
<span class="k">var</span> <span class="n">fileName</span> <span class="p">=</span> <span class="s">&quot;Availability&quot;</span><span class="p">;</span>
<span class="k">var</span> <span class="n">sanitizedFileName</span> <span class="p">=</span> <span class="k">new</span> <span class="kt">string</span><span class="p">(</span><span class="n">fileName</span><span class="p">.</span><span class="n">Except</span><span class="p">(</span><span class="n">invalidChars</span><span class="p">).</span><span class="n">ToArray</span><span class="p">());</span>
<span class="nc">Console</span><span class="p">.</span><span class="n">WriteLine</span><span class="p">(</span><span class="n">sanitizedFileName</span><span class="p">);</span>
</code></pre></div>

If you initially thought that output will be *Availability* you are wrong. This is also I thought at first. But there is subtle point in `MSDN` explanation of `Except` method. It would treat sequences as *sets*. That means there will be no elements in output sequence from second sequence. This is no surprise. But this also means that every element from the first sequence, which appears more than once, will be omitted in result.

To achieve required behavior, we need a method that would produce the difference between two sequences, so we can do:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">static</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">Without</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;(</span>
    <span class="k">this</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">first</span><span class="p">,</span> 
    <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">second</span><span class="p">)</span>
<span class="p">{</span>
    <span class="k">if</span><span class="p">(</span><span class="n">first</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">ArgumentNullException</span><span class="p">(</span><span class="s">&quot;first&quot;</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">if</span><span class="p">(</span><span class="n">second</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">throw</span> <span class="k">new</span> <span class="nc">ArgumentNullException</span><span class="p">(</span><span class="s">&quot;second&quot;</span><span class="p">);</span>
    <span class="p">}</span>
    <span class="k">return</span> <span class="nf">WithoutIterator</span><span class="p">(</span><span class="n">first</span><span class="p">,</span> <span class="n">second</span><span class="p">);</span>
<span class="p">}</span>

<span class="k">public</span> <span class="k">static</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="n">TSource</span><span class="p">&gt;</span> <span class="n">WithoutIterator</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;(</span><br/>    <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">first</span><span class="p">,</span><br/>    <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;</span> <span class="n">second</span><span class="p">)</span> 
<span class="p">{</span>
    <span class="k">var</span> <span class="n">withoutElements</span> <span class="p">=</span> <span class="k">new</span> <span class="nc">HashSet</span><span class="p">&lt;</span><span class="nc">TSource</span><span class="p">&gt;(</span><span class="n">second</span><span class="p">);</span>

    <span class="k">foreach</span><span class="p">(</span><span class="k">var</span> <span class="n">element</span> <span class="k">in</span> <span class="n">first</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="k">if</span><span class="p">(!</span><span class="n">withoutElements</span><span class="p">.</span><span class="n">Contains</span><span class="p">(</span><span class="n">element</span><span class="p">))</span>
        <span class="p">{</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="n">element</span><span class="p">;</span>
        <span class="p">}</span>
    <span class="p">}</span>
<span class="p">}</span>
</code></pre></div>

### Summary

`Except` method is little misleading. When we read the example with sanitizing file name, I bet most of us think that the only `invalidChars` will be removed from `fileName`. It is good to know that to avoid further confusion. And if we do not want to treat sequences as sets we could always resort to the method like `Without` which is shown above.