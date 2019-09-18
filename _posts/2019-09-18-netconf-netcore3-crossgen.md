---
layout: post
title:  "Crossgen as build step with .NET Core 3.0"
date:   2019-09-18
---

<p class="intro">
    <span class="dropcap">F</span>rom NGEN to Crossgen. 
    Use Crossgen to produce <i>Ready-to-Run Images</i> with <i>dotnet publish</i>.
</p>

<br/>

### .NET Conf 2019 Countdown series

I'm excited to be part of the .NET Conf with this *every day* mini-post series until the 23th September.

* [IL-Linker in .NET Core 3.0]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %})
* [The Bundler in .NET Core 3.0]({% post_url 2019-09-17-netconf-netcore3-bundler-single-file %})
* Crossgen as build step with .NET Core 3.0
* 2019-09-19
* 2019-09-20
* 2019-09-23

It's definitely worth attending a [.NET Conf 2019 local event](https://www.dotnetconf.net/local-events) to get together with other .NET friends.
Join me on the 30th september at [Community .NET Conf 2019 Event](https://www.meetup.com/de-DE/Basel-NET-User-Group/events/264124718/).

## Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try out this feature.

## From NGEN to Crossgen

As you maybe know, your source code is compiled (by [Roslyn](https://github.com/dotnet/roslyn/)) into IL-Code (CIL, Common Intermediate Language): a set of instructions (kind of low level language) that is platform- and processor independent. Unlike native code, before your .NET application can run, it requires a JIT (Just-in-Time) compiler that translates the IL-Code into native code (that is platform- and processor specific). 

This JIT compilation (done by RyuJIT) into native code does take advantage of optimized or specialized instructions of the underlying platform/CPU and can also re-optimize methods on the fly (e.g. Tiered Compilation) - that's great isn't it?

But this process of JIT compilation isn't free and is noticeable especially on the startup - as always software development is a trade-off.
How could we mitigate that? A solution introduced nearly at the beginning of .NET era: the tool [NGEN](https://docs.microsoft.com/en-us/dotnet/framework/tools/ngen-exe-native-image-generator) (.NET Framework 2.0 and onwards).

This NGEN compiled the .NET assembly into a special native image with native code. This concept is called AOT (ahead of time) compilation, *kind of opposite* of JIT. Unfortunately NGEN had by design the drawback, you couldn't create the native images up-front as part of your application build and ship it to the client...

## The successor: Crossgen

Let's try out **Crossgen** that ships as part of .NET Core 3.0 with a console app:

{% highlight cmd %}
    dotnet new console
{% endhighlight %}

It is simple to opt-in this feature - add the element `<PublishReadyToRun>true</PublishReadyToRun>` to your .csproj:

{% highlight xml %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
{% endhighlight %}

After that, publish your console app:

{% highlight cmd %}
    dotnet publish -c Release
{% endhighlight %}

Enabling log verbosity on MSBuild reveals the execution of specific targets to create the *Ready-to-Run Images* with Crossgen.exe:

![msbuild-rtr-targets](/assets/img/netconf-netcore3-crossgen/msbuild-rtr-targets.png)

When you look at the publish output directory, there is no notable difference, except the console app dll size:

* Not Ready-To-Run: **4,00 KB (4.096 bytes)**
* Ready-To-Run: **5,50 KB (5.632 bytes)**

The Crossgen processed assembly is slightly larger and has in the PE file header the flag `CorFlags.ILLibrary=0x00000004` in contrast to the other assembly `CorFlags.ILOnly=0x00000001`.

Crossgen uses internally RyuJIT to compile the native code and includes it into your .NET assembly - but preserves the IL-Code. The IL-Code is kept for some scenarios and could also be used as fall-back when the native code doesn't match the underlying platform. The native code reduces workload for the RyuJIT during startup-time of your application, thus it's faster loaded.

Further, you could exclude assemblies from beeing processed by Crossgen within .csproj:

{% highlight xml %}
<ItemGroup>
  <PublishReadyToRunExclude Include="MyAssembly.dll" />
</ItemGroup>
{% endhighlight %}

Maybe you guest that already. You could combine Crossgen with the [IL-Linker]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %}) and [The Bundler]({% post_url 2019-09-17-netconf-netcore3-bundler-single-file %}):

{% highlight xml %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <RuntimeIdentifier>win10-x64</RuntimeIdentifier>
    <PublishSingleFile>true</PublishSingleFile>
    <PublishTrimmed>true</PublishTrimmed>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
{% endhighlight %}

The result would be a self contained, single file application with pre-compiled native code.

## Limitations & .NET Native

* *ReadyToRun* is supported on .NET 3.0 and later. You can't use it with earlier versions (e.g. .NET Core 2.0 or .NET Framework).
* *ReadyToRun* works only with SCD (Self-Contained Deployment) apps (FDD support is planned).
* *ReadyToRun* does not support cross-targeting: For instance linux-x64 images **must** be compiled on that environment.

[.NET Native](https://docs.microsoft.com/en-us/dotnet/framework/net-native/) is a completely separated tool-chain that is only usable for *UWP (Universal Windows Platform) Apps*. .NET Native doesn't require any JIT and has a optimized and minimalistic runtime with some restrictions (e.g. Relfection API).

## Conclusion

I'm super excited about some new capability that we get with .NET Core 3.0 SDK.
This AOT option is worth trying especially for larger applications.
Will you try out this for your application? Let me know, what you think.


