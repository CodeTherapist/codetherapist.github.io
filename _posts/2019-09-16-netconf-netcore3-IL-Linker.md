---
layout: post
title:  "IL-Linker in .NET Core 3.0"
date:   2019-09-16
---

<p class="intro">
    <span class="dropcap">T</span>rim your .NET Core 3.0 app with the IL-Linker.
</p>

<br/>

### .NET Conf 2019 Countdown series

I'm excited to be part of the .NET Conf with this *every day* mini-post series until the 23th September.

* IL-Linker in .NET Core 3.0
* [The Bundler in .NET Core 3.0]({% post_url 2019-09-17-netconf-netcore3-bundler-single-file %})
* [Crossgen as build step with .NET Core 3.0]({% post_url 2019-09-18-netconf-netcore3-crossgen %})
* [The IAsyncDisposable interface in .NET Core 3.0]({% post_url 2019-09-19-netconf-netcore3-IAsyncDisposable %})
* [Platform intrinsics in .NET Core 3.0]({% post_url 2019-09-20-netconf-netcore3-platform-intrinsics %})
* [.NET Conf 2019 is right ahead]({% post_url 2019-09-22-netconf-netcore3-my-watch-list %})

It's definitely worth attending a [.NET Conf 2019 local event](https://www.dotnetconf.net/local-events) to get together with other .NET friends.
Join me on the 30th september at [Community .NET Conf 2019 Event](https://www.meetup.com/de-DE/Basel-NET-User-Group/events/264124718/).

## Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try out this feature.

## How to use the IL-Linker

The .NET Core 3.0 SDK ships with an additional linker (originally from mono) that can be opt-in.
The IL-Linker scans statically the IL instructions of your application to detect which code is actually required, and trims unused framework libraries. This can reduce the deployment size of your application, depending on the subset of framework assemblies you use.
It is primarily a deployment feature rather than useful for development scenarios.

To test this, you could use the console application template:

{% highlight cmd %}
    dotnet new console
{% endhighlight %}

Then, enable this *trimming* behavior with the following changes in the *csproj* file:

{% highlight xml %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
    <PublishTrimmed>true</PublishTrimmed>
  </PropertyGroup>
</Project>
{% endhighlight %}

You will need the `<RuntimeIdentifier>` element or specify the specific RID while publishing because this is only supported for self-contained applications (SCD):

{% highlight cmd %}
    dotnet publish -r win10-x64 -c Release
{% endhighlight %}

Let's compare the size of the application publish directory:

**Trimmed: 25,3 MB (26.548.001 bytes)**

**Not trimmed: 65,8 MB (69.091.753 bytes)**

Around ~30-40 MB smaller - not that much but still enough to be worth doing it.

### Framework-Dependent Deployment (FDD)

As the name *Framework-Dependent* already implies, this deployment model relies on the existence of a (shared) framework installed on the target environment.
The publish directory of your application contains the application itself and everything needed (non-framework dependencies).

### Self-Contained Deployment (SCD)

This deployment model is a lot larger in output size as the FDD is, but for a good reason.
The publish directory contains everything needed to run on the target environment - the framework and your application.
Therefor it's also isolated from any installed framework on the target environment.
The SCD possibility is in my humble opinion one of the strongest feature of .NET Core and something that the "old" .NET Framework had it's issues (it is not supported).

![framework-deployment-models](/assets/img/netconf-netcore3-IL-Linker/framework-deploy-models.png)

## Caveats

The IL-Linker is a static analyzer and does not consider *Reflection* API's.
That means for instance: when you invoke a method via reflection or searching types and load them dynamically - **that wouldn't work and your app will fail at runtime!**

How could you address this issue? There are two options:

**Option 1**

Referencing a type from that assembly somewhere:

{% highlight c# %}
    var t =  System.Type.GetType("MyAssembly.MyClass, MyAssembly");
    // or direct reference
    typeof(MyAssembly.MyClass);
{% endhighlight %}

**Option 2**

Add a "reference" in the .csproj (this is not a real assembly reference):

{% highlight xml %}
<ItemGroup>
    <TrimmerRootAssembly Include="MyAssembly.MyClass" />
</ItemGroup>
{% endhighlight %}

## Conclusion

I like this feature, even if it has some caveats as mentioned above.
It's cool that this tech comes originally from mono and is reused.
Let me know what you think about it.
