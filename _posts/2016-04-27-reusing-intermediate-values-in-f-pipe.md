---
layout: post
title: "Reusing intermediate values in F# pipe"
description: ""
category: 
tags: [Functional programming, F#]
---
{% include JB/setup %}

### Introduction

Since some time, I have been struggling with one problem regarding pipe operator. Sometimes, I'd like to use intermediate values
arising when pipeline is processed. So, previously I was just splitting entire pipeline and create values from those intermediate values using `let` binding.

### Study case

Let's take a look at this simple example. We want to split list at place where the highest number appears. So, my previous approach for that was something like this:

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="i">inputList</span> <span class="o">=</span> [<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="k">let</span> <span onmouseout="hideTip(event, 'fs2', 2)" onmouseover="showTip(event, 'fs2', 2)" class="i">index</span> <span class="o">=</span>
	<span onmouseout="hideTip(event, 'fs1', 3)" onmouseover="showTip(event, 'fs1', 3)" class="i">inputList</span>
	<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 4)" onmouseover="showTip(event, 'fs3', 4)" class="t">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 5)" onmouseover="showTip(event, 'fs4', 5)" class="f">indexed</span>
	<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 6)" onmouseover="showTip(event, 'fs3', 6)" class="t">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 7)" onmouseover="showTip(event, 'fs5', 7)" class="f">maxBy</span> (<span class="k">fun</span> <span onmouseout="hideTip(event, 'fs6', 8)" onmouseover="showTip(event, 'fs6', 8)" class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 9)" onmouseover="showTip(event, 'fs7', 9)" class="f">snd</span> <span onmouseout="hideTip(event, 'fs6', 10)" onmouseover="showTip(event, 'fs6', 10)" class="i">value</span>)
	<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 11)" onmouseover="showTip(event, 'fs8', 11)" class="f">fst</span> 

<span onmouseout="hideTip(event, 'fs1', 12)" onmouseover="showTip(event, 'fs1', 12)" class="i">inputList</span> 
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 13)" onmouseover="showTip(event, 'fs3', 13)" class="t">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 14)" onmouseover="showTip(event, 'fs9', 14)" class="f">splitAt</span> <span onmouseout="hideTip(event, 'fs2', 15)" onmouseover="showTip(event, 'fs2', 15)" class="i">index</span>
</code></pre>

However, recently I had some sort of breakthrough. I've realized that consecutive steps in pipe are just ordinary functions.
Definition of pipe operator follows :

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">val</span> <span class="k">inline</span> ( <span class="o">|&gt;</span> ) <span class="o">:</span> <span class="i">arg</span><span class="o">:</span><span class="o">&#39;</span><span class="i">T1</span> <span class="k">-&gt;</span> <span class="i">func</span><span class="o">:</span>(<span class="o">&#39;</span><span class="i">T1</span> <span class="k">-&gt;</span> <span class="o">&#39;</span><span class="i">U</span>) <span class="k">-&gt;</span> <span class="o">&#39;</span><span class="i">U</span>
</code></pre>

It takes left argument and applies it to right function which takes argument with the same type as left argument of pipeline operator. So, what we are actually passing to pipe operator on the right is function. And actually what we have in our `F#` toolset are lambda functions, and they are just functions without binding name to it.

For instance, we can create function using lambda functions, but assign name for it using `let` binding: 

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="i">identity</span> <span class="o">=</span> <span class="k">fun</span> <span class="i">x</span> <span class="k">-&gt;</span> <span class="i">x</span>
</code></pre>

The same, we can when we are composing our pipeline. However, there are some nuances. Because we don't have to use parentheses to declare lambda function I thought that arguments, which lambda function takes, are immediately available for the rest of pipeline. Unfortunately that's not true. So, that's what I tried first:

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 16)" onmouseover="showTip(event, 'fs1', 16)" class="i">inputList</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 17)" onmouseover="showTip(event, 'fs3', 17)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 18)" onmouseover="showTip(event, 'fs4', 18)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 19)" onmouseover="showTip(event, 'fs1', 19)" class="i">inputList</span>
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 20)" onmouseover="showTip(event, 'fs3', 20)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 21)" onmouseover="showTip(event, 'fs5', 21)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 22)" onmouseover="showTip(event, 'fs7', 22)" class="i">snd</span> <span class="i">value</span>)
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 23)" onmouseover="showTip(event, 'fs8', 23)" class="i">fst</span>
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 24)" onmouseover="showTip(event, 'fs10', 24)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 25)" onmouseover="showTip(event, 'fs3', 25)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 26)" onmouseover="showTip(event, 'fs9', 26)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 27)" onmouseover="showTip(event, 'fs10', 27)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 28)" onmouseover="showTip(event, 'fs1', 28)" class="i">inputList</span>
</code></pre>

But I got an error 

> The value or constructor inputList is not defined. 

That actually makes sense if you know that `F#` uses indentation to mark scope boundaries.
It's better to spot when we add parentheses to lambda function 

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> (<span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 29)" onmouseover="showTip(event, 'fs1', 29)" class="i">inputList</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 30)" onmouseover="showTip(event, 'fs3', 30)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 31)" onmouseover="showTip(event, 'fs4', 31)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 32)" onmouseover="showTip(event, 'fs1', 32)" class="i">inputList</span>)
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 33)" onmouseover="showTip(event, 'fs3', 33)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 34)" onmouseover="showTip(event, 'fs5', 34)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 35)" onmouseover="showTip(event, 'fs7', 35)" class="i">snd</span> <span class="i">value</span>)
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 36)" onmouseover="showTip(event, 'fs8', 36)" class="i">fst</span>
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 37)" onmouseover="showTip(event, 'fs10', 37)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 38)" onmouseover="showTip(event, 'fs3', 38)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 39)" onmouseover="showTip(event, 'fs9', 39)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 40)" onmouseover="showTip(event, 'fs10', 40)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 41)" onmouseover="showTip(event, 'fs1', 41)" class="i">inputList</span>
</code></pre>

Now, we can see that inputList variable is only visible inside lambda function. So, we can add parentheses to subsequent pipeline steps to make them part of this lambda function and allow inputList variable to be visible.

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> (<span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 42)" onmouseover="showTip(event, 'fs1', 42)" class="i">inputList</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 43)" onmouseover="showTip(event, 'fs3', 43)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 44)" onmouseover="showTip(event, 'fs4', 44)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 45)" onmouseover="showTip(event, 'fs1', 45)" class="i">inputList</span>
				  <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 46)" onmouseover="showTip(event, 'fs3', 46)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 47)" onmouseover="showTip(event, 'fs5', 47)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 48)" onmouseover="showTip(event, 'fs7', 48)" class="i">snd</span> <span class="i">value</span>)
				  <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 49)" onmouseover="showTip(event, 'fs8', 49)" class="i">fst</span>
				  <span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 50)" onmouseover="showTip(event, 'fs10', 50)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 51)" onmouseover="showTip(event, 'fs3', 51)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 52)" onmouseover="showTip(event, 'fs9', 52)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 53)" onmouseover="showTip(event, 'fs10', 53)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 54)" onmouseover="showTip(event, 'fs1', 54)" class="i">inputList</span>)
</code></pre>

Hmm, but it's not looking that sexy, because now `List.index inputList` expression puts new identation context at which we must ident subsequent expressions of this function.
However, there's an exception regarding infix operators, you can offset them from the actual offside column by the size of token plus one, so in that case previous code snippet looks like this:

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> (<span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 55)" onmouseover="showTip(event, 'fs1', 55)" class="i">inputList</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 56)" onmouseover="showTip(event, 'fs3', 56)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 57)" onmouseover="showTip(event, 'fs4', 57)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 58)" onmouseover="showTip(event, 'fs1', 58)" class="i">inputList</span>
			       <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 59)" onmouseover="showTip(event, 'fs3', 59)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 60)" onmouseover="showTip(event, 'fs5', 60)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 61)" onmouseover="showTip(event, 'fs7', 61)" class="i">snd</span> <span class="i">value</span>)
			       <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 62)" onmouseover="showTip(event, 'fs8', 62)" class="i">fst</span>
			       <span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 63)" onmouseover="showTip(event, 'fs10', 63)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 64)" onmouseover="showTip(event, 'fs3', 64)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 65)" onmouseover="showTip(event, 'fs9', 65)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 66)" onmouseover="showTip(event, 'fs10', 66)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 67)" onmouseover="showTip(event, 'fs1', 67)" class="i">inputList</span>)
</code></pre>
         
Back to the topic, what we can do to improve our code's appearance and still be able to access intermediate values from pipeline?
We can move first expression of lambda function to newline and that would mean next expressions in that function must be indented the same as that line. Take a look:

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 68)" onmouseover="showTip(event, 'fs1', 68)" class="i">inputList</span> <span class="k">-&gt;</span> 
<span onmouseout="hideTip(event, 'fs3', 69)" onmouseover="showTip(event, 'fs3', 69)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 70)" onmouseover="showTip(event, 'fs4', 70)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 71)" onmouseover="showTip(event, 'fs1', 71)" class="i">inputList</span>
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 72)" onmouseover="showTip(event, 'fs3', 72)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 73)" onmouseover="showTip(event, 'fs5', 73)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 74)" onmouseover="showTip(event, 'fs7', 74)" class="i">snd</span> <span class="i">value</span>)
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 75)" onmouseover="showTip(event, 'fs8', 75)" class="i">fst</span>
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 76)" onmouseover="showTip(event, 'fs10', 76)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 77)" onmouseover="showTip(event, 'fs3', 77)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 78)" onmouseover="showTip(event, 'fs9', 78)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 79)" onmouseover="showTip(event, 'fs10', 79)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 80)" onmouseover="showTip(event, 'fs1', 80)" class="i">inputList</span>
</code></pre>

But now, we break how pipeline looks like, and I feel this is not intuitive. We can take advantage of previously mentioned exception for infix operators. Anyway, I feel it is improved a litte bit but still doesn't solve the issue.

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 81)" onmouseover="showTip(event, 'fs1', 81)" class="i">inputList</span> <span class="k">-&gt;</span> 
   <span onmouseout="hideTip(event, 'fs3', 82)" onmouseover="showTip(event, 'fs3', 82)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 83)" onmouseover="showTip(event, 'fs4', 83)" class="i">indexed</span> <span onmouseout="hideTip(event, 'fs1', 84)" onmouseover="showTip(event, 'fs1', 84)" class="i">inputList</span>
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 85)" onmouseover="showTip(event, 'fs3', 85)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 86)" onmouseover="showTip(event, 'fs5', 86)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 87)" onmouseover="showTip(event, 'fs7', 87)" class="i">snd</span> <span class="i">value</span>)
<span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 88)" onmouseover="showTip(event, 'fs8', 88)" class="i">fst</span>
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 89)" onmouseover="showTip(event, 'fs10', 89)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 90)" onmouseover="showTip(event, 'fs3', 90)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 91)" onmouseover="showTip(event, 'fs9', 91)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 92)" onmouseover="showTip(event, 'fs10', 92)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 93)" onmouseover="showTip(event, 'fs1', 93)" class="i">inputList</span>
</code></pre>
I feel the most comfortable if we ident lambda function expressions a bit to explicitly denote that they belong to that function

<pre class="fssnip highlighted"><code lang="fsharp">[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 94)" onmouseover="showTip(event, 'fs1', 94)" class="i">inputList</span> <span class="k">-&gt;</span> 
    <span onmouseout="hideTip(event, 'fs1', 95)" onmouseover="showTip(event, 'fs1', 95)" class="i">inputList</span> 
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 96)" onmouseover="showTip(event, 'fs3', 96)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 97)" onmouseover="showTip(event, 'fs4', 97)" class="i">indexed</span>
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 98)" onmouseover="showTip(event, 'fs3', 98)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 99)" onmouseover="showTip(event, 'fs5', 99)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 100)" onmouseover="showTip(event, 'fs7', 100)" class="i">snd</span> <span class="i">value</span>)
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 101)" onmouseover="showTip(event, 'fs8', 101)" class="i">fst</span>
    <span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs10', 102)" onmouseover="showTip(event, 'fs10', 102)" class="i">max</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs3', 103)" onmouseover="showTip(event, 'fs3', 103)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 104)" onmouseover="showTip(event, 'fs9', 104)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs10', 105)" onmouseover="showTip(event, 'fs10', 105)" class="i">max</span> <span onmouseout="hideTip(event, 'fs1', 106)" onmouseover="showTip(event, 'fs1', 106)" class="i">inputList</span>
</code></pre>
    
We can do one more thing to get rid of last lambda function, we can define helper function `flip` which does one simple thing. It allows to flip arguments which will be passed to given function. Finally, pipeline looks like this

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span class="k">inline</span> <span class="i">flip</span> <span class="i">f</span> <span class="i">x</span> <span class="i">y</span> <span class="o">=</span> <span class="i">f</span> <span class="i">y</span> <span class="i">x</span>

[<span class="n">5</span>;<span class="n">6</span>;<span class="n">7</span>;<span class="n">8</span>;<span class="n">9</span>;<span class="n">10</span>;<span class="n">4</span>;<span class="n">3</span>;<span class="n">2</span>;<span class="n">1</span>]
<span class="o">|&gt;</span> <span class="k">fun</span> <span onmouseout="hideTip(event, 'fs1', 107)" onmouseover="showTip(event, 'fs1', 107)" class="i">inputList</span> <span class="k">-&gt;</span> 
    <span onmouseout="hideTip(event, 'fs1', 108)" onmouseover="showTip(event, 'fs1', 108)" class="i">inputList</span> 
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 109)" onmouseover="showTip(event, 'fs3', 109)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs4', 110)" onmouseover="showTip(event, 'fs4', 110)" class="i">indexed</span>
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs3', 111)" onmouseover="showTip(event, 'fs3', 111)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs5', 112)" onmouseover="showTip(event, 'fs5', 112)" class="i">maxBy</span> (<span class="k">fun</span> <span class="i">value</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 113)" onmouseover="showTip(event, 'fs7', 113)" class="i">snd</span> <span class="i">value</span>)
    <span class="o">|&gt;</span> <span onmouseout="hideTip(event, 'fs8', 114)" onmouseover="showTip(event, 'fs8', 114)" class="i">fst</span>
    <span class="o">|&gt;</span> <span class="i">flip</span> <span onmouseout="hideTip(event, 'fs3', 115)" onmouseover="showTip(event, 'fs3', 115)" class="i">List</span><span class="o">.</span><span onmouseout="hideTip(event, 'fs9', 116)" onmouseover="showTip(event, 'fs9', 116)" class="i">splitAt</span> <span onmouseout="hideTip(event, 'fs1', 117)" onmouseover="showTip(event, 'fs1', 117)" class="i">inputList</span>
</code></pre>

### Summary

We've seen how we can reuse intermediate values from pipeline using lambda functions.
Of course these are not all possibilities of how you can write that pipeline. I hope you will find something useful here, but just use a way that you feel more comfortable with.