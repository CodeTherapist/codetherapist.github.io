---
layout: post
title:  "The Bundler in .NET Core 3.0"
date:   2019-09-17
---

<p class="intro">
    <span class="dropcap">D</span>id you ever had the dream of a portable single file app?
</p>

<br/>

### .NET Conf 2019 Countdown series

I'm excited to be part of the .NET Conf with this *every day* mini-post series until the 23th September.

* [IL-Linker in .NET Core 3.0]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %})
* The Bundler in .NET Core 3.0
* 2019-09-18
* 2019-09-19
* 2019-09-20
* 2019-09-23

It's definitely worth attending a [.NET Conf 2019 local event](https://www.dotnetconf.net/local-events) to get together with other .NET friends.
Join me on the 30th september at [Community .NET Conf 2019 Event](https://www.meetup.com/de-DE/Basel-NET-User-Group/events/264124718/).

## Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try out this feature.

## The Bundler

Maybe you never heard about this Bundler. It is part of the [.NET Core setup repository](https://github.com/dotnet/core-setup/tree/master/src/managed/Microsoft.NET.HostModel/Bundle). Consequently, it ships as part of the .NET Core 3.0 SDK.

Let's try that out with a bare minimum project setup:

{% highlight cmd %}
    dotnet new console
{% endhighlight %}

Now, you need to modify the .csproj to opt-in this feature:

{% highlight xml %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
    <PublishSingleFile>true</PublishSingleFile>
  </PropertyGroup>
</Project>
{% endhighlight %}

The important element is `<PublishSingleFile>true</PublishSingleFile>` that invokes the bundler during the publish process.
To do so, execute the following command in the root directory of your console app:

{% highlight cmd %}
    dotnet publish -c Release
{% endhighlight %}

Did you ever had the dream of a portable single file .NET console app, that doesn't require any pre-installed framework?
**This Bundler and the SCD (Self-contained deployment) model enables exactly this *portable single file app* experience!**

The Bundler packs your application and the whole .NET Core framework into a single runnable file (e.g. exe on windows).
At the first look, it looks similar what the [IL-Merge tool](https://github.com/dotnet/ILMerge) does.
But the bundler is different than IL-Merge - it keeps all the assemblies (preserves the assembly identities) as is, instead of merging the IL code into one single assembly. With preserving your assemblies, it is still debuggable and stacktraces from exceptions matches the source. 

Pay attention, it isn't cross-platform portable. There are native platform dependencies linked during the *publish* process. 
So, you couldn't run that console app exe on any other platform than windows.
Thanks to the tooling, you could create an equivalent output of the app with ease for another platform (see [RID Catalog](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog)):

**Windows**

{% highlight cmd %}
    dotnet publish -r win10-x64 -c Release
{% endhighlight %}

**Linux**s

{% highlight cmd %}
    dotnet publish -r linux-x64 -c Release
{% endhighlight %}

**macOS**

{% highlight cmd %}
    dotnet publish -r osx-x64 -c Release
{% endhighlight %}

## IL-Linker + Bundler = <3

When you use the IL-Linker with this Bundler together, you get a much smaller single file.
You need only add `<PublishTrimmed>true</PublishTrimmed>` to your .csproj.

**Windows**

* Without IL-Linker: **65,9 MB (69.101.810 bytes)**
* With IL-Linker: **25,3 MB (26.550.389 bytes)**

**Linux**

* Without IL-Linker: **74,2 MB (77.873.394 bytes)**
* With IL-Linker: **34,0 MB (35.709.620 bytes)**

**macOS**

* Without IL-Linker: **69,9 MB (73.372.861 bytes)**
* With IL-Linker: **29,6 MB (31.110.783 bytes)**

*Note: The size is reduced by the IL-Linker not by the Bundler.* <br/>
See my other post [IL-Linker in .NET Core 3.0]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %}).

# Conclusion

Combining both tools (IL-Linker & Bundler) is a smart combo and adds a new opportunity how your .NET app could be distributed.
Without the burden of dependency management (like dll hell) or complex installation procedures - everything is included.

I'm excited about how that could change the publishing experience and distribution of .NET apps.
That's it for today - thanks for reading :)
