---
layout: post
title: "Platform intrinsics in .NET Core 3.0"
date: 2019-09-20
---

<p class="intro">
    <span class="dropcap">Q</span>uick look at the new platform intrinsics feature.
</p>

<br/>

### .NET Conf 2019 Countdown series

I'm excited to be part of the .NET Conf with this *every day* mini-post series until the 23th September.

* [IL-Linker in .NET Core 3.0]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %})
* [The Bundler in .NET Core 3.0]({% post_url 2019-09-17-netconf-netcore3-bundler-single-file %})
* [Crossgen as build step with .NET Core 3.0]({% post_url 2019-09-18-netconf-netcore3-crossgen %})
* [The IAsyncDisposable interface in .NET Core 3.0]({% post_url 2019-09-19-netconf-netcore3-IAsyncDisposable %})
* Platform intrinsics in .NET Core 3.0
* [.NET Conf 2019 is right ahead]({% post_url 2019-09-22-netconf-netcore3-my-watch-list %})

It's definitely worth attending a [.NET Conf 2019 local event](https://www.dotnetconf.net/local-events) to get together with other .NET friends.
Join me on the 30th september at [Community .NET Conf 2019 Event](https://www.meetup.com/de-DE/Basel-NET-User-Group/events/264124718/).

## Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/) and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0) to try this out.

## What are platform dependent intrinsics

In the previous post about [Crossgen as build step with .NET Core 3.0]({% post_url 2019-09-18-netconf-netcore3-crossgen %}), I explained some basic concepts of how .NET internally works. For instance that your source code is compiled into CIL and therefor is CPU/platform independent.
This is also one of the key benefits that you get from .NET. But sometimes it does makes sense to be CPU/platform specific.
Maybe you do have an application that you runs only on a Intel CPU - why shouldn't you use specific feature of it?

Exactly that are *platform dependent intrinsics* - a specific feature or semantic that is not universally available on all platforms and isn't easily recognizable by the JIT.
For instance the *AES instruction set* for applications performing encryption and decryption using [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) is supported by some Intel and AMD processors.
The purpose of the instruction set is to improve speed and minimize the attack surface of a [Side-channel Attacks](https://en.wikipedia.org/wiki/Side-channel_attack).

### How are they exposed in .NET

The high level .NET API's are located in the `System.Runtime.Intrinsics` namespace - 
you could find a set of already implemented intrinsics in the [CoreCLR repostiory](https://github.com/dotnet/coreclr/tree/master/src/System.Private.CoreLib/shared/System/Runtime/Intrinsics).
Furthermore they are separated by platform x64, x86, Arm and Arm64 and share when applicable the implementation (e.g. x86 and x64).
What would happen when calling an hardware intrinsic method on a platform that doesn't support it? <br/> 
A `PlatformNotSupportedException` would be thrown. But you don't have to catch exceptions, there is a easy way to check it:

{% highlight c# %}
static void Main(string[] args)
{
    Console.WriteLine("AES instruction set is supported: " + System.Runtime.Intrinsics.X86.Aes.IsSupported);
}
{% endhighlight %}

That returns `true` on my machine - aren't you curious if it's supported on your machine as well?
It's worth mentioning that the intrinsics are mainly implemented as part of the RyuJIT and recognized by the `IntrinsicAttribute`.

## Conclusion

I'm seriously not a expert in that field of specialized CPU instructions. But knowing that something like this exists and is new to the .NET world, could become important in the future.
Sure, Hardware Intrinsics definitely aren’t for everyone, but they can be used to boost perf in some computationally heavy workloads (ML.NET or graphical rendering for games).
Even if you never use it directly - it will benefit performance of low-level implementations in libraries that you maybe use already today!