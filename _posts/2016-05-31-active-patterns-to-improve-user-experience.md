---
layout: post
title: "Active patterns to improve user experience"
description: ""
category: 
tags: []
---
{% include JB/setup %}

### A little bit about UI and UX

User interface and user experience is a very complex topic. There are many things which influence how user perceivesS UI. Many small pieces which aren't part of the core domain, but sometimes have huge impact at
final result. If everything is fine, then user even won't notice that, but if something is wrong with this little piece then it will stand out like a sore thumb.
One example of that is correct noun flexion. Especially in language like Polish, because we have rather than complex flexion rules for nouns based on count. In order to provide proper user experience in application we should try to present content in correct grammar form.
Let see how we can use `F#` to help us with that task.

### Identifying rules
To keep example simple, we will take a look on words in masculine. Yes, the rules are sometimes a bit more complicated ;) 
So, below in table, for one example is listed how it should be declined:

| Count|Word flexion|
|---|---|
|0|wydatków
|1|wydatek
|2-4,22-24,32-34,...|wydatki
|5-9,15-19,25-29,...|wydatków
|10-14,20-21,30-31,...|wydatków

<br/>
We can use feature from F# to name those rules and make easier for creating functions that will be using those rules. That feature is called `Active Patterns`.

### Active patterns

They are used to state frequent patterns that we use in pattern matching. But they also can act as a way for make complex patterns shorter.
We can leverage active pattern feature to express our rules in following way:

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> (|<span class="p">RuleForZeroAndFromFiveToNineAndTens</span>|<span class="p">RuleForOne</span>|<span class="p">RuleForTwoThreeAndFour</span>|<span class="p">RuleFromElevenToNineteen</span>|) <span onmouseout="hideTip(event, 'fs1', 1)" onmouseover="showTip(event, 'fs1', 1)" class="i">n</span> <span class="o">=</span>
    <span class="k">match</span> <span onmouseout="hideTip(event, 'fs1', 2)" onmouseover="showTip(event, 'fs1', 2)" class="i">n</span> <span class="k">with</span>
    | <span class="n">0</span> <span class="k">-&gt;</span> <span class="p">RuleForZeroAndFromFiveToNineAndTens</span>
    | <span class="n">1</span> <span class="k">-&gt;</span> <span class="p">RuleForOne</span>
    | <span onmouseout="hideTip(event, 'fs2', 3)" onmouseover="showTip(event, 'fs2', 3)" class="i">i</span> <span class="k">when</span> <span onmouseout="hideTip(event, 'fs2', 4)" onmouseover="showTip(event, 'fs2', 4)" class="i">i</span> <span class="o">&gt;</span><span class="o">=</span> <span class="n">2</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 5)" onmouseover="showTip(event, 'fs2', 5)" class="i">i</span> <span class="o">&lt;=</span> <span class="n">4</span> <span class="k">-&gt;</span> <span class="p">RuleForTwoThreeAndFour</span>
    | <span onmouseout="hideTip(event, 'fs2', 6)" onmouseover="showTip(event, 'fs2', 6)" class="i">i</span> <span class="k">when</span> (<span onmouseout="hideTip(event, 'fs2', 7)" onmouseover="showTip(event, 'fs2', 7)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">=</span> <span class="n">1</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 8)" onmouseover="showTip(event, 'fs2', 8)" class="i">i</span> <span class="o">%</span> <span class="n">100</span> <span class="o">&lt;&gt;</span> <span class="n">11</span>) <span class="k">-&gt;</span> <span class="p">RuleForZeroAndFromFiveToNineAndTens</span>
    | <span onmouseout="hideTip(event, 'fs2', 9)" onmouseover="showTip(event, 'fs2', 9)" class="i">i</span> <span class="k">when</span> (<span onmouseout="hideTip(event, 'fs2', 10)" onmouseover="showTip(event, 'fs2', 10)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">=</span> <span class="n">2</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 11)" onmouseover="showTip(event, 'fs2', 11)" class="i">i</span> <span class="o">%</span> <span class="n">100</span> <span class="o">&lt;&gt;</span> <span class="n">12</span>) <span class="k">-&gt;</span> <span class="p">RuleForTwoThreeAndFour</span>
    | <span onmouseout="hideTip(event, 'fs2', 12)" onmouseover="showTip(event, 'fs2', 12)" class="i">i</span> <span class="k">when</span> (<span onmouseout="hideTip(event, 'fs2', 13)" onmouseover="showTip(event, 'fs2', 13)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">=</span> <span class="n">3</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 14)" onmouseover="showTip(event, 'fs2', 14)" class="i">i</span> <span class="o">%</span> <span class="n">100</span> <span class="o">&lt;&gt;</span> <span class="n">13</span>) <span class="k">-&gt;</span> <span class="p">RuleForTwoThreeAndFour</span>
    | <span onmouseout="hideTip(event, 'fs2', 15)" onmouseover="showTip(event, 'fs2', 15)" class="i">i</span> <span class="k">when</span> (<span onmouseout="hideTip(event, 'fs2', 16)" onmouseover="showTip(event, 'fs2', 16)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">=</span> <span class="n">4</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 17)" onmouseover="showTip(event, 'fs2', 17)" class="i">i</span> <span class="o">%</span> <span class="n">100</span> <span class="o">&lt;&gt;</span> <span class="n">14</span>) <span class="k">-&gt;</span> <span class="p">RuleForTwoThreeAndFour</span>
    | <span onmouseout="hideTip(event, 'fs2', 18)" onmouseover="showTip(event, 'fs2', 18)" class="i">i</span> <span class="k">when</span> <span onmouseout="hideTip(event, 'fs2', 19)" onmouseover="showTip(event, 'fs2', 19)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">&gt;</span><span class="o">=</span><span class="n">5</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 20)" onmouseover="showTip(event, 'fs2', 20)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">&lt;=</span><span class="n">9</span> <span class="k">-&gt;</span> <span class="p">RuleForZeroAndFromFiveToNineAndTens</span>
    | <span onmouseout="hideTip(event, 'fs2', 21)" onmouseover="showTip(event, 'fs2', 21)" class="i">i</span> <span class="k">when</span> <span onmouseout="hideTip(event, 'fs2', 22)" onmouseover="showTip(event, 'fs2', 22)" class="i">i</span> <span class="o">&gt;</span> <span class="n">10</span> <span class="o">&amp;&amp;</span> <span onmouseout="hideTip(event, 'fs2', 23)" onmouseover="showTip(event, 'fs2', 23)" class="i">i</span> <span class="o">&lt;</span> <span class="n">20</span> <span class="k">-&gt;</span> <span class="p">RuleFromElevenToNineteen</span>
    | <span onmouseout="hideTip(event, 'fs2', 24)" onmouseover="showTip(event, 'fs2', 24)" class="i">i</span> <span class="k">when</span> <span onmouseout="hideTip(event, 'fs2', 25)" onmouseover="showTip(event, 'fs2', 25)" class="i">i</span> <span class="o">%</span> <span class="n">10</span> <span class="o">=</span> <span class="n">0</span> <span class="k">-&gt;</span> <span class="p">RuleForZeroAndFromFiveToNineAndTens</span>
</code></pre>

I could squash some rules together but I think there's sense to leave it in such a way because it will be easier to maintain. It is simple as it looks, based on given count we compute which rule we should apply.

### Using active pattern

Having stated our rules as active pattern, we can easily create function which will help us to display proper form on the UI. Let's see how it can be done:

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span onmouseout="hideTip(event, 'fs3', 26)" onmouseover="showTip(event, 'fs3', 26)" class="f">wordDeclination</span> <span onmouseout="hideTip(event, 'fs4', 27)" onmouseover="showTip(event, 'fs4', 27)" class="i">wordForZeroAndFromFiveToNineAndTens</span> <span onmouseout="hideTip(event, 'fs5', 28)" onmouseover="showTip(event, 'fs5', 28)" class="i">wordForOne</span> <span onmouseout="hideTip(event, 'fs6', 29)" onmouseover="showTip(event, 'fs6', 29)" class="i">wordForTwoThreeAndFour</span> <span onmouseout="hideTip(event, 'fs7', 30)" onmouseover="showTip(event, 'fs7', 30)" class="i">wordForElevenToNineteen</span> <span onmouseout="hideTip(event, 'fs1', 31)" onmouseover="showTip(event, 'fs1', 31)" class="i">n</span> <span class="o">=</span>
    <span class="k">match</span> <span onmouseout="hideTip(event, 'fs1', 32)" onmouseover="showTip(event, 'fs1', 32)" class="i">n</span> <span class="k">with</span>
    | <span onmouseout="hideTip(event, 'fs8', 33)" onmouseover="showTip(event, 'fs8', 33)" class="p">RuleForZeroAndFromFiveToNineAndTens</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs4', 34)" onmouseover="showTip(event, 'fs4', 34)" class="i">wordForZeroAndFromFiveToNineAndTens</span>
    | <span onmouseout="hideTip(event, 'fs9', 35)" onmouseover="showTip(event, 'fs9', 35)" class="p">RuleForOne</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs5', 36)" onmouseover="showTip(event, 'fs5', 36)" class="i">wordForOne</span>
    | <span onmouseout="hideTip(event, 'fs10', 37)" onmouseover="showTip(event, 'fs10', 37)" class="p">RuleForTwoThreeAndFour</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs6', 38)" onmouseover="showTip(event, 'fs6', 38)" class="i">wordForTwoThreeAndFour</span>
    | <span onmouseout="hideTip(event, 'fs11', 39)" onmouseover="showTip(event, 'fs11', 39)" class="p">RuleFromElevenToNineteen</span> <span class="k">-&gt;</span> <span onmouseout="hideTip(event, 'fs7', 40)" onmouseover="showTip(event, 'fs7', 40)" class="i">wordForElevenToNineteen</span>
</code></pre>

Thanks to active patterns we do not have to convolute method with logic that is responsible for selecting appropriate word flexion. Entire core logic is hiden in active pattern declaration.
Using function `wordDeclination` we can further create small helper functions which will give a word in correct form. There is one important thing to be aware, here arguments order matters, because take a look if we will move `n` argument at the beginning then we won't be able to apply currying to `n` argument. It should be clear if you look how we can use function `wordDeclination`:

<pre class="fssnip highlighted"><code lang="fsharp"><span class="k">let</span> <span onmouseout="hideTip(event, 'fs12', 41)" onmouseover="showTip(event, 'fs12', 41)" class="f">forUIWydatekDeclination</span> <span class="o">=</span> <span onmouseout="hideTip(event, 'fs3', 42)" onmouseover="showTip(event, 'fs3', 42)" class="f">wordDeclination</span> <span class="s">&quot;wydatk&#243;w&quot;</span> <span class="s">&quot;wydatek&quot;</span> <span class="s">&quot;wydatki&quot;</span> <span class="s">&quot;wydatk&#243;w&quot;</span>
</code></pre>

Here, we partially applied `wordDeclination` function in `forUIWydatekDeclination` function. As a result `forUIWydatekDeclination` takes one argument and it is count of items. Looking again at function `wordDeclination`, if we didn't move the `n` argument at the end, then we would have to provide also `n` argument for `forUIWydatekDeclination` function. We didn't do that, so `n` argument is curried. We can use `forUIWydatekDeclination` in following way:

<pre class="fssnip highlighted"><code lang="fsharp"><span onmouseout="hideTip(event, 'fs12', 43)" onmouseover="showTip(event, 'fs12', 43)" class="f">forUIWydatekDeclination</span> <span class="n">22</span>
</code></pre>


### Summary
You may think that this is not rocket science, but as I said earlier those things influence overall user perception about application. Of course you could do that using different manner.
But I think this is neat and clean use case for active patterns.