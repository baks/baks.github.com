---
layout: post
title : "Testing for equality"
tags : [Unit testing, .NET, AutoFixture]
---
{% include JB/setup %}

Often, while writing code we discover value objects in our application. Further, we also need to perform some operation on that types that involve determining equality of our value objects.
In `.NET` there are established rules how to properly implement equality, but testing equality in value objects involves quite a lot of repetitive work.
Let's see what we can do with that and adhere to [DRY principle](https://en.wikipedia.org/wiki/Don't_repeat_yourself). At first let's review the rules that we should take into account while implementing equality.

#### Things you should remember while implementing equality

According to various resources, we should adhere to the properties of equality implementation, which are mentioned for example at [MSDN](https://msdn.microsoft.com/en-us/library/dd183755.aspx).

So, the implementation of equality should:

* be *reflexive*

`x.Equals(x)` returns `true`

* be *symmetric*

`x.Equals(y) == y.Equals(x)`

* be *transitive*

`x.Equals(y) && y.Equals(z) == true` then `x.Equals(z) == true`

* be *consistent*

Invocations of `x.Equals(y)` return the same value as long as the values in `x` or `y` are not modified or references are not changed

* return `false` when compared against `null`

`x.Equals(null) == false`


We should also override `GetHashCode` method when overriding `Equals` method, even compiler reminds us (as a warning) about that. This is due to implementation of many `.NET` classes (for example `Hashtable` or `Dictionary`), because they rely on the rule that if two objects are equal, they should return the same hash code value.

#### Nice, when you do it when implementing equality

There are also other guidelines which are good to use while implementing equality:

* Overload `==` and `!=` operator

As default the `==` and `!=` operators perform identity check. When we provide our equality implementation we should also overload equality and inequality operators to avoid confusion and bugs while working with our objects.

* Implement `System.IEquatable<T>` interface

This is an interface which defines a strongly typed `Equals` method, which can be used internally by other equality comparison methods. It is worth mentioning that using strongly typed `Equals` method avoids boxing while working with structs.

#### Test-Do list for testing equality

And last but not least, we must put logic for actually performing the equality comparison of our objects. Finally, this is our test-do list for testing equality implementation:

&nbsp;&nbsp;&nbsp;&nbsp;[ ] `Equals` and `GetHashCode` method are overridden<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is reflexive <br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is symmetric<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is transitive<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is consistent<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if returns `false` when compared to `null`<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check whether implementation of equality works according to the comparison logic<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `GetHashCode` implementation produces correct results<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `GetHashCode` implementation is consistent<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `==` and `!=` operators are overloaded<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `==` and `!=` operators produce correct results<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `IEquatable<T>` is implemented<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `IEquatable<T>` produces correct results<br/>

#### Write once, ~~run~~ use everywhere

It is a lot of work, and our goal is to avoid repeating this every time we introduce value object. We know what to test for, to check equality implementation correctness. It would be the best if we can simply say *what* we want to test instead of *how* the equality tests should be done. That reminds me about the **Write once, run everywhere** slogan coined by `Sun` for `Java`. Actually, in our case we should say **Write once, use everywhere**. We can apply it here, write reusable component, which performs test cases and we can reuse it when we introduce new value objects to the system.

Fortunately, there is a library that can help us. It is [AutoFixture](https://github.com/AutoFixture/AutoFixture) (maybe there are other libraries but I don't know any), and its part `AutoFixture.Idioms`. If we use that library, out of the box we can cross four items from test-do list. For the remaining items we can plug into `AutoFixture.Idioms` infrastructure and write assertions that performs other test cases.

So, this is the test-do list we have after using [AutoFixture](https://github.com/AutoFixture/AutoFixture):

&nbsp;&nbsp;&nbsp;&nbsp;[ ] `Equals` and `GetHashCode` method are overridden<br/>
&nbsp;&nbsp;&nbsp;&nbsp;~~[x] Check if implementation is reflexive~~ **[EqualsSelfAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/EqualsSelfAssertion.cs) class from** `AutoFixture.Idioms`<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is symmetric<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check if implementation is transitive<br/>
&nbsp;&nbsp;&nbsp;&nbsp;~~[x] Check if implementation is consistent~~ **[EqualsSuccessiveAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/EqualsSuccessiveAssertion.cs) class from** `AutoFixture.Idioms`<br/>
&nbsp;&nbsp;&nbsp;&nbsp;~~[x] Check if returns false when compare to null~~ **[EqualsNullAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/EqualsNullAssertion.cs) class from** `AutoFixture.Idioms`<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] Check whether implementation of equality works according to the comparison logic <br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `GetHashCode` implementation produces correct results<br/>
&nbsp;&nbsp;&nbsp;&nbsp;~~[x] `GetHashCode` implementation is consistent~~ **[GetHashCodeSuccessiveAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/GetHashCodeSuccessiveAssertion.cs) class from** `AutoFixture.Idioms`<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `==` and `!=` operators are overloaded <br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `==` and `!=` operators produce correct results<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `IEquatable<T>` is implemented<br/>
&nbsp;&nbsp;&nbsp;&nbsp;[ ] `IEquatable<T>` produces correct results<br/>

For clarification, the assertions that will be presented are intended to use with immutable value objects. Data used in equality comparison is provided by arguments passed to constructor. Example of that value object is following:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="k">public</span> <span class="k">class</span> <span class="nc">Point</span>
<span class="p">{</span>
    <span class="k">public</span> <span class="nf">Point</span><span class="p">(</span><span class="kt">int</span> <span class="n">x</span><span class="p">,</span> <span class="kt">int</span> <span class="n">y</span><span class="p">)</span>
    <span class="p">{</span>
        <span class="n">X</span> <span class="p">=</span> <span class="n">x</span><span class="p">;</span>
        <span class="n">Y</span> <span class="p">=</span> <span class="n">y</span><span class="p">;</span>
    <span class="p">}</span>

    <span class="k">public</span> <span class="k">int</span> <span class="n">X</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
    <span class="k">public</span> <span class="k">int</span> <span class="n">Y</span> <span class="p">{</span> <span class="k">get</span><span class="p">;</span> <span class="k">private</span> <span class="k">set</span><span class="p">;</span> <span class="p">}</span>
<span class="p">}</span>
</code></pre></div>

#### Making EqualityOperatorOverloadAssertion 

Goal of the `EqualityOperatorOverloadAssertion` is to check whether given type overloads `==` operator. Assertion derives from [IdiomaticAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/IIdiomaticAssertion.cs) abstract class that is defined in `AutoFixture.Idioms`. `IdiomaticAssertion` provides helpful default implementation for methods from [IIdiomaticAssertion](https://github.com/AutoFixture/AutoFixture/blob/master/Src/Idioms/IIdiomaticAssertion.cs) interface. In our example we are overriding `Verify` method with `Type` parameter as this is the information necessary to perform interesting us assertion. After it turns out that type does not overload `==` operator, we throw meaningful exception which explains what is wrong.
This is the code for `EqualityOperatorOverloadAssertion`:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp">    <span class="k">public</span> <span class="k">class</span> <span class="nc">EqualityOperatorOverloadAssertion</span> <span class="p">:</span> <span class="nc">IdiomaticAssertion</span>
    <span class="p">{</span>
        <span class="k">public</span> <span class="k">override</span> <span class="k">void</span> <span class="nf">Verify</span><span class="p">(</span><span class="n">Type</span> <span class="n">type</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">if</span> <span class="p">(</span><span class="n">type</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="k">throw</span> <span class="k">new</span> <span class="nc">ArgumentNullException</span><span class="p">(</span><span class="s">&quot;type&quot;</span><span class="p">);</span>
            <span class="p">}</span>

            <span class="k">var</span> <span class="n">equalityOperatorOverload</span> <span class="p">=</span> <span class="n">type</span><span class="p">.</span><span class="n">GetEqualityOperatorMethod</span><span class="p">();</span>

            <span class="k">if</span> <span class="p">(</span><span class="n">equalityOperatorOverload</span> <span class="p">==</span> <span class="k">null</span><span class="p">)</span>
            <span class="p">{</span>
                <span class="k">throw</span> <span class="k">new</span> <span class="nc">EqualityOperatorException</span><span class="p">(</span>
                    <span class="k">string</span><span class="p">.</span><span class="n">Format</span><span class="p">(</span><span class="s">&quot;Expected type {0} to overload == operator with parameters of type {0}&quot;</span><span class="p">,</span> <span class="n">type</span><span class="p">.</span><span class="n">Name</span><span class="p">));</span>
            <span class="p">}</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre></div>

Usage of `EqualityOperatorOverloadAssertion`:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="nc">    [Fact]</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">ShouldOverloadEqualityOperator</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="k">new</span> <span class="nc">EqualityOperatorOverload</span><span class="p">().</span><span class="n">Verify</span><span class="p">(</span><span class="k">typeof</span><span class="p">(</span><span class="nc">SomeValueObject</span><span class="p">));</span>
    <span class="p">}</span>
</code></pre></div>

I think that it doesn't make sense to put here implementation of every assertion. Therefore, I decided to pick one simple assertion and explain how it is done. If you are interested how other assertions are implemented, please take a look [here](https://github.com/baks/EqualityTests/tree/master/EqualityTests).

#### Compose all the assertions

To write a reusable component that will execute test cases from our test-do list we need to compose all created assertions. We can use `CompositeIdiomaticAssertion` class from `AutoFixture.Idioms` that allows to supply suite of assertions that will be verified.

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp">    <span class="k">public</span> <span class="k">static</span> <span class="k">class</span> <span class="nc">EqualityTestsFor</span><span class="p">&lt;</span><span class="n">T</span><span class="p">&gt;</span> <span class="k">where</span> <span class="n">T</span> <span class="p">:</span> <span class="k">class</span>
    <span class="p">{</span>
        <span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Assert</span><span class="p">()</span>
        <span class="p">{</span>
            <span class="k">var</span> <span class="n">fixture</span> <span class="p">=</span> <span class="k">new</span> <span class="nc">Fixture</span><span class="p">();</span>
            <span class="k">var</span> <span class="n">compositeAssertion</span> <span class="p">=</span>
                <span class="k">new</span> <span class="nc">CompositeIdiomaticAssertion</span><span class="p">(</span><span class="n">EqualityAssertions</span><span class="p">(</span><span class="n">fixture</span><span class="p">,</span> <span class="k">new</span> <span class="nc">EqualityTestCaseProvider</span><span class="p">(</span><span class="n">fixture</span><span class="p">)));</span>

            <span class="n">VerifyCompositeAssertion</span><span class="p">(</span><span class="n">compositeAssertion</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">public</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">Assert</span><span class="p">(</span><span class="n">Func</span><span class="p">&lt;</span><span class="nc">EqualityTestsConfiguration</span><span class="p">&lt;</span><span class="n">T</span><span class="p">&gt;&gt;</span> <span class="n">configuration</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">var</span> <span class="n">compositeAssertion</span> <span class="p">=</span>
                <span class="k">new</span> <span class="nc">CompositeIdiomaticAssertion</span><span class="p">(</span><span class="n">EqualityAssertions</span><span class="p">(</span><span class="k">new</span> <span class="nc">Fixture</span><span class="p">(),</span> <span class="n">configuration</span><span class="p">()));</span>

            <span class="n">VerifyCompositeAssertion</span><span class="p">(</span><span class="n">compositeAssertion</span><span class="p">);</span>
        <span class="p">}</span>

        <span class="k">private</span> <span class="k">static</span> <span class="k">void</span> <span class="nf">VerifyCompositeAssertion</span><span class="p">(</span><span class="n">CompositeIdiomaticAssertion</span> <span class="n">compositeAssertion</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="n">compositeAssertion</span><span class="p">.</span><span class="n">Verify</span><span class="p">(</span><span class="k">typeof</span><span class="p">(</span><span class="n">T</span><span class="p">));</span>
            <span class="n">compositeAssertion</span><span class="p">.</span><span class="n">Verify</span><span class="p">(</span><span class="k">typeof</span><span class="p">(</span><span class="n">T</span><span class="p">).</span><span class="n">GetEqualsMethod</span><span class="p">());</span>
            <span class="n">compositeAssertion</span><span class="p">.</span><span class="n">Verify</span><span class="p">(</span><span class="k">typeof</span><span class="p">(</span><span class="n">T</span><span class="p">).</span><span class="n">GetMethod</span><span class="p">(</span><span class="s">&quot;GetHashCode&quot;</span><span class="p">));</span>
        <span class="p">}</span>

        <span class="k">private</span> <span class="k">static</span> <span class="nc">IEnumerable</span><span class="p">&lt;</span><span class="nc">IdiomaticAssertion</span><span class="p">&gt;</span> <span class="n">EqualityAssertions</span><span class="p">(</span><span class="nc">ISpecimenBuilder</span> <span class="n">specimenBuilder</span><span class="p">,</span>
            <span class="nc">IEqualityTestCaseProvider</span> <span class="n">equalityTestCaseProvider</span><span class="p">)</span>
        <span class="p">{</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsOverrideAssertion</span><span class="p">();</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">GetHashCodeOverrideAssertion</span><span class="p">();</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsSelfAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsSymmetricAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsTransitiveAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsSuccessiveAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsNullAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualsValueCheckAssertion</span><span class="p">(</span><span class="n">equalityTestCaseProvider</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">GetHashCodeValueCheckAssertion</span><span class="p">(</span><span class="n">equalityTestCaseProvider</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">GetHashCodeSuccessiveAssertion</span><span class="p">(</span><span class="n">specimenBuilder</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualityOperatorOverloadAssertion</span><span class="p">();</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">InequalityOperatorOverloadAssertion</span><span class="p">();</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">EqualityOperatorValueCheckAssertion</span><span class="p">(</span><span class="n">equalityTestCaseProvider</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">InequalityOperatorValueCheckAssertion</span><span class="p">(</span><span class="n">equalityTestCaseProvider</span><span class="p">);</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">IEquatableImplementedAssertion</span><span class="p">();</span>
            <span class="k">yield</span> <span class="k">return</span> <span class="k">new</span> <span class="nc">IEquatableValueCheckAssertion</span><span class="p">(</span><span class="n">equalityTestCaseProvider</span><span class="p">);</span>
        <span class="p">}</span>
    <span class="p">}</span>
</code></pre></div>

If our value object is using only primitive types in constructor, we can rely on default equality test case provider that can handle this situation. It means, that we can write simple unit test for equality like:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="nc">    [Fact]</span>
    <span class="k">public</span> <span class="k">void</span> <span class="nf">ShouldImplementValueEquality</span><span class="p">()</span>
    <span class="p">{</span>
        <span class="nc">EqualityTestsFor</span><span class="p">&lt;</span><span class="nc">Point</span><span class="p">&gt;.</span><span class="n">Assert</span><span class="p">();</span>
    <span class="p">}</span>
</code></pre></div>

But, if we need some more advanced scenarios, we can provide our own instances to describe what the results of equality test cases should be:

<div class="highlight"><pre><code class="language-csharp" data-lang="csharp"><span class="nc">        [Fact]</span>
        <span class="k">public</span> <span class="k">void</span> <span class="nf">ShouldImplementValueEquality</span><span class="p">()</span>
        <span class="p">{</span>
            <span class="nc">EqualityTestsFor</span><span class="p">&lt;</span><span class="nc">Point</span><span class="p">&gt;.</span><span class="n">Assert</span><span class="p">(</span>
                <span class="p">()</span> <span class="p">=&gt;</span>
                    <span class="nc">EqualityTestsConfigurer</span><span class="p">&lt;</span><span class="nc">Point</span><span class="p">&gt;</span>
                    <span class="p">.</span><span class="n">Instance</span><span class="p">(</span><span class="k">new</span> <span class="nc">Point</span><span class="p">(</span><span class="m">1</span><span class="p">,</span> <span class="m">1</span><span class="p">))</span>
                        <span class="p">.</span><span class="n">ShouldBeEqualTo</span><span class="p">(</span><span class="k">new</span> <span class="nc">Point</span><span class="p">(</span><span class="m">1</span><span class="p">,</span> <span class="m">1</span><span class="p">))</span>
                        <span class="p">.</span><span class="n">ShouldNotBeEqualTo</span><span class="p">(</span><span class="k">new</span> <span class="nc">Point</span><span class="p">(</span><span class="m">1</span><span class="p">,</span> <span class="m">2</span><span class="p">))</span>
                        <span class="p">.</span><span class="n">ShouldNotBeEqualTo</span><span class="p">(</span><span class="k">new</span> <span class="nc">Point</span><span class="p">(</span><span class="m">2</span><span class="p">,</span> <span class="m">1</span><span class="p">)));</span>
        <span class="p">}</span>
</code></pre></div>

#### Wrap up

Checking equality implementation in value objects can be repetitive work, hence in this post you can find reusable approach to this problem. [AutoFixture](https://github.com/AutoFixture/AutoFixture) was used as a supporting library. It is heavily using [role interfaces](http://martinfowler.com/bliki/RoleInterface.html). Therefore it is very composable and it is a pleasure to work with design like that.

I'd like to add, if we spot in our solution that we are testing something more than once, it is a sign that maybe there is domain concept which should be made explicit. This is a somewhat term **Test once, use everywhere**. I find it useful, because it avoids duplication of the behavior in the system. We put behavior in one component, test it and we can use it everywhere. It also prevents us for combinatorial explosion of test cases when we have to test the same thing many times.