---
layout: post
title:  "C# 8: nullable reference types"
date:   2019-03-20
---

<p class="intro">
    <span class="dropcap">N</span>ullable reference types will change the code we write today.
</p>

### Prerequisites & Setup

To play with this feature you need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0).
For existing code, this feature must be opt-in for making _reference types_ non-nullable.
And this for a good reason: it would break a lot of code because by default every _reference type_ is assumed to be _non-nullable_ (never null).
As soon C# 8.0 is stable, the feature will be enabled by default for new projects.

We need to modify the .csproj file to enable it:

{% highlight xml %}

<LangVersion>8.0</LangVersion>
<NullableContextOptions>enable</NullableContextOptions>

{% endhighlight %}

### Let's try it out

C# has two kind of types: _value types_ (e.g. int, byte, structs) and _reference types_ (e.g. class, interface).
_Reference types_ can be _null_ by design and this introduces additional complexity while writing code.
Especially when the project is fairly large and complex.

Imagine, we have a _CarWashSalon_ class with a method _Wash_ that doesn't accept _null_ as input parameter:

{% highlight csharp %}

public class CarWashSalon
{
	public void Wash(Car car)
	{
		if (car == null)
		{
			throw new ArgumentNullException(nameof(car));
		}

		// wash the car
	}
}

{% endhighlight %}

Looks quite common, right?
There isn't a way to express (at compile time) the intension of our design - _car_ isn't expected to be _null_.
Calling the method with a _null_ reference will end up with an _ArgumentNullException_ at runtime (or even worse with a _NullReferenceException_...):

{% highlight csharp %}

class Program
{
	static void Main(string[] args)
	{
		var carWashSalon = new CarWashSalon();

		Car myCar = null;

		carWashSalon.Wash(myCar);
	}
}

{% endhighlight %}

Wouldn't it be awesome to be able express our design decision and get warnings (or errors) when it is not respected?
This is exactly the purpose of the **nullable reference types** feature.
The compiler ([roslyn](https://github.com/dotnet/roslyn)) analyzes the code and instantly tells us about the _null_ assignment issue:

![non-nullable errors](/assets/img/csharp8-nullable-ref-types/NonNullableErrors.png)

* We can't assign _null_ to the _myCar_ variable because it's non-nullable.
* We can't pass _null_ (in this case _myCar_) to _void CarWashSalon.Wash(Car car)_ because it expects a non-nullable.

Everything is fine when we assign an instance of a _Car_ to _myCar_:

{% highlight csharp %}

Car myCar = new Car();

{% endhighlight %}

But wait! What if _myCar_ could be _null_ (be a nullable reference type) by design?
The new nullable reference type operator is solved elegantly - it's the same as for nullable values types.
By suffixing _Car_ with a **?** it becomes a _nullable reference type_:

{% highlight csharp %}
class Program
{
	static void Main(string[] args)
	{
		var carWashSalon = new CarWashSalon();

		Car? myCar = null;
		
		carWashSalon.Wash(myCar);
	}
}
{% endhighlight %}

Guess what happen? An error is back on our error list - _CarWashSalon.Wash()_ still doesn't accept _null_.

![](/assets/img/csharp8-nullable-ref-types/NonNullableError2.png)

The compiler does control-flow analysis and detect when we try to dereferencing _null_.
To solve the issue we could use a _if_ statement to ensure _null_ is not passed into the _Wash_ method:

{% highlight csharp %}
class Program
{
	static void Main(string[] args)
	{
		var carWashSalon = new CarWashSalon();

		Car? myCar = null;

		if (myCar != null)
		{
			carWashSalon.Wash(myCar);
		}
	}
}
{% endhighlight %}

Otherwise, when we guarantee that _myCar_ is never _null_ the error is gone as well - even _myCar_ is still nullable:

{% highlight csharp %}
	Car? myCar = new Car();
	carWashSalon.Wash(myCar);
{% endhighlight %}

![](/assets/img/csharp8-nullable-ref-types/NoError.png)

Under some conditions we would like to tell the compiler, that we know what we are doing and we are aware of the _null_.
This situation can be expressed with the _null-forgiving operator_ **!**:

{% highlight csharp %}

	Car? myCar = null;
	carWashSalon.Wash(myCar!);

{% endhighlight %}

As an alternative we could change the nullable context to be disabled for a specific code block:

{% highlight csharp %}
Car? myCar = null;
#nullable disable
    carWashSalon.Wash(myCar);
#nullable restore
{% endhighlight %}

### Technical deatails

When you are curious as myself, then you wonder how is a _nullable reference type_ represented at _CIL_ (Common Intermediate Language) level.
There is no added type to the type system, but somehow the tooling is able to distinguish between _nullable_- and _non-nullable reference types_.
A closer look with IL-Spy reveals the secret:

![nullable types CIL](/assets/img/csharp8-nullable-ref-types/NullableTypesCIL.png)

There is a _NullableAttribute_ applied to the members with value 1 (non-nullable) and 2 (nullable).
To avoid external dependencies, the attribute is embedded into the assembly dynamically:

![nullable types CIL](/assets/img/csharp8-nullable-ref-types/EmbeddedAttrs.png)

### Conclusion

In my opinion this is the most impactful feature of C# 8.0.
It improves code quality and could avoid the famous _NullReferenceException_ in most scenarios.