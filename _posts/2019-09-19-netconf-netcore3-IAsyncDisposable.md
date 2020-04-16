---
layout: post
title:  "The IAsyncDisposable interface in .NET Core 3.0"
date:   2019-09-19
---

<p class="intro">
    <span class="dropcap">I</span>mplementing IAsyncDisposable on disposable objects.
</p>

<br/>

### .NET Conf 2019 Countdown series

I'm excited to be part of the .NET Conf with this *every day* mini-post series until the 23th September.

* [IL-Linker in .NET Core 3.0]({% post_url 2019-09-16-netconf-netcore3-IL-Linker %})
* [The Bundler in .NET Core 3.0]({% post_url 2019-09-17-netconf-netcore3-bundler-single-file %})
* [Crossgen as build step with .NET Core 3.0]({% post_url 2019-09-18-netconf-netcore3-crossgen %})
* The IAsyncDisposable interface in .NET Core 3.0
* [Platform intrinsics in .NET Core 3.0]({% post_url 2019-09-20-netconf-netcore3-platform-intrinsics %})
* [.NET Conf 2019 is right ahead]({% post_url 2019-09-22-netconf-netcore3-my-watch-list %})

It's definitely worth attending a [.NET Conf 2019 local event](https://www.dotnetconf.net/local-events) to get together with other .NET friends.
Join me on the 30th september at [Community .NET Conf 2019 Event](https://www.meetup.com/de-DE/Basel-NET-User-Group/events/264124718/).

## Prerequisites & Setup

You will need [Visual Studio 2019](https://visualstudio.microsoft.com/vs/preview/){:target="_blank"} and [.NET Core 3.0 SDK](https://dotnet.microsoft.com/download/dotnet-core/3.0){:target="_blank"} to try this out.

## The IAsyncDisposable interface

Every .NET developer will sooner or later use the `Dispose()` method (indirectly with a `using` statement) and implement the `IDisposable`.
The pattern for disposing an object, has been available as a concept with .NET for a long time, going back to the .NET Framework 1.1.

Since then, a lot has evolved - new things like `async/await` were added and multi-threaded systems are no longer indispensable and more important nowadays with the cloud. Thus there is a need to do the disposal also asynchronously.

Sure, you could do the following to dispose an object asynchronously in a `async/await` context (not recommended):
 
{% highlight c# %}
    await Task.Run(() => disposableObj.Dispose());
{% endhighlight %}

Wrongly usage of `Task.Run()` could cause thread pool starvation - thread pool threads, are a globally shared resource.
Beside that, you couldn't use `disposableObj` in a `using` statement.
The new interface ([IAsyncDisposable](https://docs.microsoft.com/en-us/dotnet/api/system.iasyncdisposable){:target="_blank"}) helps.
The `DisposeAsync()` fulfills exactly the same purpose as the `Dispose()` method of [IDisposable](https://docs.microsoft.com/en-us/dotnet/api/system.idisposable?view=netcore-3.0){:target="_blank"} and should follow the smae implementation rules:

* DisposeAsync/Dispose could be called multiple times, subsequent calls must be ignored
* DisposeAsync/Dispose shouldn't throw exception
* DisposeAsync/Dispose must be implemented when it holds another disposable object or/and unmanaged resources

So, the `using statement` would be like this:

{% highlight c# %}
await using (var disposableObject = new DisposableObject())
{
    //...
}
{% endhighlight %}

Similar to the non await using statement, it expands to (internally):

{% highlight c# %}
    DisposableObject disposableObject = null;
    try
    {
        disposableObject = new DisposableObject();
        //...
    }
    finally
    {
        if (disposableObject != null)
            await disposableObject.DisposeAsync();
    }
{% endhighlight %}

Well, let's see how you could implement `IAsyncDisposable` on a class that does already implementing `IDisposable` in your own library:

{% highlight c# %}

public class DisposableObject : IAsyncDisposable, IDisposable
{
    private bool disposed = false;

    public DisposableObject()
    {
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (disposed)
            return;

        if (disposing)
        {
            // Free any other managed objects here.
        }

        disposed = true;
    }
    
    public virtual ValueTask DisposeAsync()
    {
        try
        {
            Dispose();
            return default;
        }
        catch (Exception exception)
        {
            return new ValueTask(Task.FromException(exception));
        }
    }
}

{% endhighlight %}

Usually, you are good to go by calling the existing dispose method, but this isn't always possible - it depends on the structure/purpose of your class. You maybe wonder, why `DisposeAsync` is returning a ValueTask and not a `Task` - It's because of performance.
`ValueTask` is a struct that doesn't add pressure to the GC by allocating on the heap. This could be important when many objects getting disposed in a tight loop. The `try/catch` is only required when the class isn't sealed or/and you aren't sure that under no circumstances an exception is thrown. 
Further, there is no `CancellationToken` support for `DisposeAsync` because there is no well-known scenario to cancel cleanup process and leave the object in an inconsistent state.

`IAsyncDisposable` is not inherited from  `IDisposable` by intension, allowing developers to choose between implementing one or both. Depending on your class usage/purpose, it's more appropriate to offer only `DisposeAsync` when it is likely used in an asynchronously fashion.
Another benefit of having `IAsyncDisposable` is to perform a resource-intensive dispose operation without blocking the main thread of a GUI application for a long time.

Last but not least: this exists in the .NET Standard 2.1 and later - implemented by the .NET Core 3.0.
As you maybe already know, the "Windows" .NET Framework (e.g. 4.7.2, 4.8 and so on) will never implement .NET Standard 2.1 and thus no 
`IAsyncDisposable` interface.

## Conclusion

Overall, I think the `IAsyncDisposable` interface and the `await` keyword for the `using` statement is a meaningful extension.
It rounds off the `async/await` programming model that is meanwhile used a lot (`File.ReadAllTextAsync(...)`, `httpClient.PostAsync(...)`, `DbContext.SaveChangesAsync(...)`, just to name few).
Feel free to leave a comment - I'm always happy to get feedback of any kind.