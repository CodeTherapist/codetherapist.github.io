---
layout: post
title:  "C# 8: indexes and ranges"
date:   2019-04-08
---

<p class="intro">
    <span class="dropcap">I</span>ndex and ranges gives you another way to access elements in an array.
</p>

### Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try out indexes and ranges. We need to modify the .csproj file to enable C# 8.0:

{% highlight xml %}

<LangVersion>8.0</LangVersion>

{% endhighlight %}

### Let's compare how we access specific elements in an array

Access elements is not something new. It's fundamental to any programming language and thus also a concept in C#.
<a href="https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/Index.cs" target="_blank">Index</a> is a new readonly struct added to the Framework.

I like to stick with cars in this post as well as I did in the <a href="https://codetherapist.github.io/blog/csharp8-nullable-ref-types/" target="_blank">previous C# 8 blog post</a>.
This time we have 10 parked cars that we get as an array:

{% highlight csharp %}
    class Program
    {
        static void Main(string[] args)
        {
            Car[] parkedCars = GetAllParkedCars();

            // (1)
            var thirdCar = parkedCars[3];
            var lastCar = parkedCars[parkedCars.Length -1];

            // (2)
            var thirdCarLinq = parkedCars.ElementAt(3);
            var lastCarLinq = parkedCars.Last();

            // (3)
            var thirdCarByIndex = parkedCars[^7];
            var lastCarByIndex = parkedCars[^1];
        }

        private static Car[] GetAllParkedCars()
        {
            var l = new List<Car>();
            foreach (var item in Enumerable.Range(1,10))
            {
                l.Add(new Car($"Car {item}"));
            }
            return l.ToArray();
        }
    }
{% endhighlight %}

The first (1) is a common way how we access elements inside an array.
_LINQ_ (2) is the most natural to read but that comes also with a (minimal) cost - It's slower than the other solutions because it iterates over the elements from the start until the condition is met. But think about that readability is usually in most cases more important than performance.
The last (3) uses the new "hat"-operator to express the new index type:

* [^7] means **from the end count seven elements backwards**.
* [^1] means **from the end the first element**.

Pay attention that [^0] is not valid and will throw an exception - because [^0] is equal to [array.Length - 0] and this throws an _IndexOutOfRangeException_.
The [^7] and [^1] are shorthand versions of this:

{% highlight csharp %}
        // index as type
        var thirdCarByIndex2 = parkedCars[new Index(7, fromEnd: true)];
        var lastCarByIndex2 = parkedCars[new Index(1, fromEnd: true)];
{% endhighlight %}

Usually I want also understand how the compiler is emitting my C# code and how the .NET Runtime is handling it.
You maybe wonder if _[^1]_ is compiled to _[array.Length - 1]_.
It is almost the same. Decompiling the assembly with IL-Spy proves this:

{% highlight csharp %}
	index = new Index(1, true);
	Car car8 = array[index.get_IsFromEnd() ? (array.Length - index.get_Value()) : index.get_Value()];
{% endhighlight %}

The critical IL-Operation is the same as well:

By int:

![array-length-sub-int](/assets/img/csharp8-indices-ranges/array-length-sub-Int.png)

By Index:

![array-length-sub-index](/assets/img/csharp8-indices-ranges/array-length-sub-index.png)

### Access a range of specific elements

Let's combine two indexes to form a range - makes sense, right?

<a href="https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/Range.cs" target="_blank"> Range</a>
 is a readonly struct and a new type. It consists mainly of a <a href="https://github.com/dotnet/corefx/blob/355917229c2e5a35b8ea7f14d5d6de2455c5dc1b/src/Common/src/CoreLib/System/Range.cs#L22" target="_blank">Start</a> and <a href="https://github.com/dotnet/corefx/blob/355917229c2e5a35b8ea7f14d5d6de2455c5dc1b/src/Common/src/CoreLib/System/Range.cs#L25" target="_blank">End</a> property of type <a href="https://github.com/dotnet/corefx/blob/master/src/Common/src/CoreLib/System/Index.cs" target="_blank"> Index</a>.
The following code is usually what we do to get a _range_ out of a string using _Substring()_:

{% highlight csharp %}
    var greeting = "Hello my name is Code Therapist!";
    var helloMyNameIs = greeting.Substring(0, 17);
    var codeTherapist = greeting.Substring(17, 9);
{% endhighlight %}

With ranges you can express exactly the same in a slightly different way:

{% highlight csharp %}
    var helloMyNameIs = greeting[..17];
    var codeTherapist = greeting[17..^1];
{% endhighlight %}

It compiles internally  to a call to _Substring()_.

### Closing words

I'm curious how this feature will change the way I or other developers write code in C#.
In my opinion it's very convenient in some situations - especially the _Range_.
But compared to other features it's not something completely new that you couldn't do with a lower C# version.
Simply because these two operators will be lowered to regular indexer/method calls.
