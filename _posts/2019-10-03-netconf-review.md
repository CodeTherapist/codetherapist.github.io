---
layout: post
title: ".NET Conf 2019 review"
date: 2019-10-03
---

<p class="intro">
    <span class="dropcap">T</span>he .NET Conf 2019 is over - here my review of it!
</p>
<br/>

### .NET Conf 2019

Overall it was great! Huge thanks to everyone that contributed to that event.
Especially to the community speakers that took the effort to prepare and present a topic.
The organization and moderation served by Microsoft was good as well.

There is a lot of great content that can be watched on demand - unfortunately not all content is ready to be watched.
Most community talks (Day 3) is still not available while writing this post.

### Highlights

Be aware that the following content is my subjective impression: what I watched and liked.
It is possible that it doesn't match with your own thoughts - which is totally okay.
Feel free to share with me your thoughts, maybe I missed an interesting aspect.

#### .NET Core 3.0 & C# 8

Of course, the whole release of .NET Core 3.0 is the biggest thing they announced - even if everyone knew that already.
Second, the C# 8.0 language update that goes along side with .NET Core 3, is also worth watching.

So, download [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0){:target="_blank"} and update your apps or libraries.

#### Secure your NuGet package eco-system

Firstly, I thought that this is a boring talk about private nuget feeds etc.
But it turned out to be useful, for single developers up to huge enterprise developer teams!

In a nutshell, the nuget client lets you decided which: package sources, package types, package author you trust.
Quickly recap how that could look like:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <config>
        <!-- The value "require" will enable the validation. -->
        <add key="signatureValidationMode" value="require" />
    </config>
    <trustedSigners>
        <author name="microsoft">
            <certificate fingerprint="3F9001EA83C560D712C24CF213C3D312CB3BFF51EE89435D3430BD06B5D0EECE" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
        </author>
        <repository name="nuget.org" serviceIndex="https://api.nuget.org/v3/index.json">
            <certificate fingerprint="0E5F38F57DC1BCC806D8494F4F90FBCEDD988B46760709CBEEC6F4219AA6157D" hashAlgorithm="SHA256" allowUntrustedRoot="false" />
            <owners>microsoft;aspnet;nuget</owners>
        </repository>
    </trustedSigners>
</configuration>
{% endhighlight %}

#### Tips and Tricks for .NET Debugging in Visual Studio

This talk had two thing I did not knew: _Data Breakpoints_ and the _function Return Value_.

The Data Breakpoints are well-known for developers with a C++ background, now they are available in .NET:

![vs2019-data-breakpoint](/assets/img/netconf-review/vs2019-data-breakpoint.png)

As the name already said, the breakpoint is hit when data (in this case the property) is changed.
That is extremely useful when you have no clue "who" and "when" a specific property changes.
It also shows the previous value and the "incoming" value:

![vs2019-data-breakpoint-hit](/assets/img/netconf-review/vs2019-data-breakpoint-hit.png)

Now, the _function Return Value_ is best explained with the following code snippet:

{% highlight c# %}
public static string GetGreeting(string country) 
{
    var result = $"Hello World from {country}";
    return result;
}
{% endhighlight %}

Did you ever catch yourself writing similar code like above, just to see what the computed return value of your function is?
This is exactly where the _function Return Value_ feature kicks in: by simply provide a "shadow local variable" called $ReturnValue.
You can see this value in the locals window:

![vs2019-return-value](/assets/img/netconf-review/vs2019-func-return-value-locals.png)

Or you can access it the watch window:

![vs2019-return-value](/assets/img/netconf-review/vs2019-func-return-value.png)

#### Increase your .NET Productivity with Visual Studio 2019

Visual Studio is still the IDE that I use most of the time.
So, I'm always crave for productivity tips, to get my job done easier.

Something that I missed when I stopped using resharper, was fixing namespace accordingly to the directory structure in my projects.
But this is now available as a code-fix:

![vs2019-fix-namespace](/assets/img/netconf-review/vs2019-fix-ns.png)

Next, a quickfix that is also useful: creating a parameter out of a local variable:

![vs2019-fix-parameter](/assets/img/netconf-review/vs2019-fix-param.gif)

I like to see VS 2019 is catching up with missing refactorings and simplify our daily tasks.

Finally, I hope you like my "mini" review - feel free to share with me your thoughts.






