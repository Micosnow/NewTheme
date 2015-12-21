
##In This Section

C# Keywords
Provides links to information about C# keywords and syntax.
C# Operators
Provides links to information about C# operators and syntax.
C# Preprocessor Directives
Provides links to information about compiler commands for embedding in C# source code.
C# Compiler Options
Includes information about compiler options and how to use them.
C# Compiler Errors
Includes code snippets that demonstrate the cause and correction of C# compiler errors and warnings.
C# Language Specification
Provides pointers to the latest version of the C# Language Specification in Microsoft Word format.


##Related Sections

C# FAQ
Provides a growing list of C# Frequently Asked Questions in the C# Developer Center.
C# KB articles in the Microsoft Knowledge Base
Opens a Microsoft search page for Knowledge Base articles that are available on MSDN.
C#
Provides a portal to Visual C# documentation.
Using the Visual C# Development Environment
Provides links to conceptual and task topics that describe the IDE and Editor.
C# Programming Guide
Includes information about how to use the C# programming language.

<table responsive="true"><tbody><tr><th><p>Title</p></th><th><p>Description</p></th></tr><tr><td data-th="Title"><p><a href="https://int.msdn.microsoft.com/en-us/library/zw4w595w(v=vs.110).aspx">Overview of the .NET Framework</a></p></td><td data-th="Description"><p>Provides detailed information for developers who build applications that target the .NET Framework.</p></td></tr><tr><td data-th="Title"><p><a href="https://int.msdn.microsoft.com/en-us/library/dn151288(v=vs.110).aspx">The .NET Framework and Out-of-Band Releases</a></p></td><td data-th="Description"><p>Describes the .NET Framework out-of-band releases and how to use them in your app.</p></td></tr><tr><td data-th="Title"><p><a href="https://int.msdn.microsoft.com/en-us/library/8z6watww(v=vs.110).aspx">.NET Framework System Requirements</a></p></td><td data-th="Description"><p>Lists the hardware and software requirements for running the .NET Framework.</p></td></tr><tr><td data-th="Title"><p><a href="https://int.msdn.microsoft.com/en-us/library/dn878908(v=vs.110).aspx">.NET Core and Open-Source</a></p></td><td data-th="Description"><p>Describes what .NET Core is in relation to the .NET Framework and how to access the open-source .NET Core projects.</p></td></tr><tr><td data-th="Title"><p><a href="https://int.msdn.microsoft.com/en-us/library/5a4x27ek(v=vs.110).aspx">Installing the .NET Framework</a></p></td><td data-th="Description"><p>Provides information about installing the .NET Framework.</p></td></tr></tbody></table>



<div class="codeSnippetContainer" id="code-snippet-2" xmlns="">
    <div class="codeSnippetContainerTabs">
        <div class="codeSnippetContainerTabSingle" dir="ltr"><a>F#</a></div>
    </div>
    <div class="codeSnippetContainerCodeContainer">
        <div class="codeSnippetToolBar">
            <div class="codeSnippetToolBarText">
                <a name="CodeSnippetCopyLink" title="Copy to clipboard." style="display: block;" href="javascript:if (window.epx.codeSnippet)window.epx.codeSnippet.copyCode('CodeSnippetContainerCode_94902520-9154-4995-a6be-39dc3b0ef6f0');" ms.cmptyp="CodeSnippet">Copy</a>
            </div>
        </div>
        <div class="codeSnippetContainerCode" id="CodeSnippetContainerCode_94902520-9154-4995-a6be-39dc3b0ef6f0" dir="ltr">
            <div style="color: black;"><pre><span style="color: blue;">open</span> System
<span style="color: blue;">open</span> System.Diagnostics

<span style="color: green;">// Represents a stream of IObserver events.</span>
<span style="color: blue;">type</span> ObservableSource&lt;'T&gt;() =

    <span style="color: blue;">let</span> protect function1 =
        <span style="color: blue;">let</span> <span style="color: blue;">mutable</span> ok = <span style="color: blue;">false</span>
        <span style="color: blue;">try</span> 
            function1()
            ok &lt;- <span style="color: blue;">true</span>
        <span style="color: blue;">finally</span>
            Debug.Assert(ok, <span style="color: rgb(163, 21, 21);">"IObserver method threw an exception."</span>)

    <span style="color: blue;">let</span> <span style="color: blue;">mutable</span> key = 0

    <span style="color: green;">// Use a Map, not a Dictionary, because callers might unsubscribe in the OnNext</span>
    <span style="color: green;">// method, so thread-safe snapshots of subscribers to iterate over are needed.</span>
    <span style="color: blue;">let</span> <span style="color: blue;">mutable</span> subscriptions = Map.empty : Map&lt;<span style="color: blue;">int</span>, IObserver&lt;'T&gt;&gt;

    <span style="color: blue;">let</span> next(obs) = 
        subscriptions |&gt; Seq.iter (<span style="color: blue;">fun</span> (KeyValue(_, value)) -&gt; 
            protect (<span style="color: blue;">fun</span> () -&gt; value.OnNext(obs)))

    <span style="color: blue;">let</span> completed() = 
        subscriptions |&gt; Seq.iter (<span style="color: blue;">fun</span> (KeyValue(_, value)) -&gt; 
            protect (<span style="color: blue;">fun</span> () -&gt; value.OnCompleted()))

    <span style="color: blue;">let</span> error(err) = 
        subscriptions |&gt; Seq.iter (<span style="color: blue;">fun</span> (KeyValue(_, value)) -&gt; 
            protect (<span style="color: blue;">fun</span> () -&gt; value.OnError(err)))

    <span style="color: blue;">let</span> thisLock = <span style="color: blue;">new</span> <span style="color: blue;">obj</span>()

    <span style="color: blue;">let</span> obs = 
        { <span style="color: blue;">new</span> IObservable&lt;'T&gt; <span style="color: blue;">with</span>
            <span style="color: blue;">member</span> this.Subscribe(obs) =
                <span style="color: blue;">let</span> key1 =
                    lock thisLock (<span style="color: blue;">fun</span> () -&gt;
                        <span style="color: blue;">let</span> key1 = key
                        key &lt;- key + 1
                        subscriptions &lt;- subscriptions.Add(key1, obs)
                        key1)
                { <span style="color: blue;">new</span> IDisposable <span style="color: blue;">with</span> 
                    <span style="color: blue;">member</span> this.Dispose() = 
                        lock thisLock (<span style="color: blue;">fun</span> () -&gt; 
                            subscriptions &lt;- subscriptions.Remove(key1)) } }

    <span style="color: blue;">let</span> <span style="color: blue;">mutable</span> finished = <span style="color: blue;">false</span>

    <span style="color: green;">// The source ought to call these methods in serialized fashion (from</span>
    <span style="color: green;">// any thread, but serialized and non-reentrant).</span>
    <span style="color: blue;">member</span> this.Next(obs) =
        Debug.Assert(<span style="color: blue;">not</span> finished, <span style="color: rgb(163, 21, 21);">"IObserver is already finished"</span>)
        next obs

    <span style="color: blue;">member</span> this.Completed() =
        Debug.Assert(<span style="color: blue;">not</span> finished, <span style="color: rgb(163, 21, 21);">"IObserver is already finished"</span>)
        finished &lt;- <span style="color: blue;">true</span>
        completed()

    <span style="color: blue;">member</span> this.Error(err) =
        Debug.Assert(<span style="color: blue;">not</span> finished, <span style="color: rgb(163, 21, 21);">"IObserver is already finished"</span>)
        finished &lt;- <span style="color: blue;">true</span>
        error err

    <span style="color: green;">// The IObservable object returned is thread-safe; you can subscribe </span>
    <span style="color: green;">// and unsubscribe (Dispose) concurrently.</span>
    <span style="color: blue;">member</span> this.AsObservable = obs

<span style="color: green;">// Create a source.</span>
<span style="color: blue;">let</span> source = <span style="color: blue;">new</span> ObservableSource&lt;<span style="color: blue;">int</span>&gt;()

<span style="color: green;">// Get an IObservable from the source.</span>
<span style="color: blue;">let</span> obs = source.AsObservable 

<span style="color: green;">// Add a simple subscriber.</span>
<span style="color: blue;">let</span> unsubA = obs |&gt; Observable.subscribe (<span style="color: blue;">fun</span> x -&gt; printfn <span style="color: rgb(163, 21, 21);">"A: %d"</span> x)

<span style="color: green;">// Send some messages from the source.</span>
<span style="color: green;">// Output: A: 1</span>
source.Next(1)
<span style="color: green;">// Output: A: 2</span>
source.Next(2)

<span style="color: green;">// Add another subscriber. This subscriber has a filter.</span>
<span style="color: blue;">let</span> unsubB =
    obs
    |&gt; Observable.filter (<span style="color: blue;">fun</span> num -&gt; num % 2 = 0)
    |&gt; Observable.subscribe (<span style="color: blue;">fun</span> num -&gt; printfn <span style="color: rgb(163, 21, 21);">"B: %d"</span> num)

<span style="color: green;">// Send more messages from the source.</span>
<span style="color: green;">// Output: A: 3</span>
source.Next(3)
<span style="color: green;">// Output: A: 4</span>
<span style="color: green;">//         B: 4</span>
source.Next(4)

<span style="color: green;">// Have subscriber A unsubscribe.</span>
unsubA.Dispose()

<span style="color: green;">// Send more messages from the source.</span>
<span style="color: green;">// No output</span>
source.Next(5)
<span style="color: green;">// Output: B: 6</span>
source.Next(6)

<span style="color: green;">// If you use add, there is no way to unsubscribe from the event.</span>
obs |&gt; Observable.add(<span style="color: blue;">fun</span> x -&gt; printfn <span style="color: rgb(163, 21, 21);">"C: %d"</span> x)

<span style="color: green;">// Now add a subscriber that only does positive numbers and transforms</span>
<span style="color: green;">// the numbers into another type, here a string.</span>
<span style="color: blue;">let</span> unsubD =
    obs |&gt; Observable.choose (<span style="color: blue;">fun</span> int1 -&gt;
             <span style="color: blue;">if</span> int1 &gt;= 0 <span style="color: blue;">then</span> None <span style="color: blue;">else</span> Some(int1.ToString()))
        |&gt; Observable.subscribe(<span style="color: blue;">fun</span> string1 -&gt; printfn <span style="color: rgb(163, 21, 21);">"D: %s"</span> string1)

<span style="color: blue;">let</span> unsubE =
    obs |&gt; Observable.filter (<span style="color: blue;">fun</span> int1 -&gt; int1 &gt;= 0)
        |&gt; Observable.subscribe(<span style="color: blue;">fun</span> int1 -&gt; printfn <span style="color: rgb(163, 21, 21);">"E: %d"</span> int1)

<span style="color: blue;">let</span> unsubF =
    obs |&gt; Observable.map (<span style="color: blue;">fun</span> int1 -&gt; int1.ToString())
        |&gt; Observable.subscribe (<span style="color: blue;">fun</span> string1 -&gt; printfn <span style="color: rgb(163, 21, 21);">"F: %s"</span> string1)

</pre></div>
            
        </div>
    </div>
</div>
