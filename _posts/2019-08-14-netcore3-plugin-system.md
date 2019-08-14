---
layout: post
title:  "A plugin system with .NET Core"
date:   2019-08-14
---

<p class="intro">
    <span class="dropcap">I</span>mplementing a (mini) plugin system with .NET Core 3.0
</p>

<br/>

### Prerequisites

You need [VS 2019](https://visualstudio.microsoft.com/vs/) and [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0) (currently in preview 8 while posting this).

### Getting started

In this post I show how you could implement a plugin system that can unload the plugins dynamically.
I also provide some background information behind the techniques and classes involved.
Unlike the [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0), the [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) let's you unload the plugin types and the owned assemblies - sounds promising, right?

#### The PluginFinder

Usually before we load an assembly in our application, we should probe it for plugins that our application supports.

#### The PluginHost

The plugin host acts as a registry of the known plugins.

#### The Plugin

Every plugin needs at least a name to be identified and properly hosted by the plugin host.

<p class="alert alert-danger">
    <b>Be aware that the following implementation is an example and not bullet proof production ready.</b>
</p>

### Implementing the PluginFinder class

The plugin finder is responsible for loading and scanning an assembly for plugins.
This means we need to store the information about which assemblies have plugins and unload the assembly after scanning.

{% highlight c# %}
public class PluginFinder<TPlugin> where TPlugin : IPlugin
{
    public PluginFinder() { }

    public IReadOnlyCollection<string> FindAssemliesWithPlugins(string path)
    {
        var assemblies = Directory.GetFiles(path, "*.dll", new EnumerationOptions() { RecurseSubdirectories = true });
        return FindPluginsInAssemblies(assemblies);
    }

    private IReadOnlyCollection<string> FindPluginsInAssemblies(string[] assemblyPaths)
    {
        var assemblyPluginInfos = new List<string>();
        var pluginFinderAssemblyContext = new PluginAssemblyLoadingContext(name: "PluginFinderAssemblyContext");
        foreach (var assemblyPath in assemblyPaths)
        {
            var assembly = pluginFinderAssemblyContext.LoadFromAssemblyPath(assemblyPath);
            if (GetPluginTypes(assembly).Any())
            {
                assemblyPluginInfos.Add(assembly.Location);
            }
        }
        pluginFinderAssemblyContext.Unload();
        return assemblyPluginInfos;
    }

    public static IReadOnlyCollection<Type> GetPluginTypes(Assembly assembly)
    {
        return  assembly.GetTypes()
                        .Where(type =>
                        !type.IsAbstract &&
                        typeof(TPlugin).IsAssignableFrom(type))
                        .ToArray();
    }
}
{% endhighlight %}


### Implementing the PluginHost class

The plugin host stores all plugin instances by name and allows unloading them.
We load the assembly into the __pluginAssemblyLoadingContext_.
After that, the _Activator_ creates a new instance of our plugin types and adds it to the dictionary.

{% highlight c# %}
public class PluginHost<TPlugin> where TPlugin : IPlugin
{
    private Dictionary<string, TPlugin> _plugins = new Dictionary<string, TPlugin>();
    private readonly PluginAssemblyLoadingContext _pluginAssemblyLoadingContext;

    public PluginHost()
    {
        _pluginAssemblyLoadingContext = new PluginAssemblyLoadingContext("PluginAssemblyContext");
    }

    public TPlugin GetPlugin(string pluginName)
    {
        return _plugins[pluginName];
    }

    public IReadOnlyCollection<TPlugin> GetPlugins()
    {
        return _plugins.Values;
    }
               
    public void LoadPlugins(IReadOnlyCollection<string> assembliesWithPlugins)
    {
        foreach (var assemblyPath in assembliesWithPlugins)
        {
            var assembly = _pluginAssemblyLoadingContext.LoadFromAssemblyPath(assemblyPath);
            var validPluginTypes = PluginFinder<TPlugin>.GetPluginTypes(assembly);
            foreach (var pluginType in validPluginTypes)
            {
                var plutinInstance = (TPlugin)Activator.CreateInstance(pluginType);
                RegisterPlugin(plutinInstance);
            }
        }
    }

    public void Unload()
    {
        _plugins.Clear();
        _pluginAssemblyLoadingContext.Unload();
    }
}
{% endhighlight %}

### Implementing the plugins in another assembly

The plugin interface defined by the application is simple.

{% highlight c# %}
public interface IPlugin
{
    string Name { get; }
}
{% endhighlight %}

If we leave it that way, our plugin can not do anything yet.
That's boring, right? Lets add another interface to be suitable for math operations.

{% highlight c# %}
public interface IMathOperationPlugin : IPlugin
{
    decimal Calculate(decimal operand1, decimal operand2);   
}
{% endhighlight %}

Don't be surprised by the chosen operations - they are well-known.

{% highlight c# %}
public class AdditionOperation : PluginBase, IMathOperationPlugin
{
    public override string Name => nameof(AdditionOperation);

    public decimal Calculate(decimal operand1, decimal operand2)
    {
        return operand1 + operand2;
    }
}

public class DivideOperation : PluginBase, IMathOperationPlugin
{
    public override string Name => nameof(DivideOperation);

    public decimal Calculate(decimal operand1, decimal operand2)
    {
        return operand1 / operand2;
    }
}

public class MultiplyOperation : PluginBase, IMathOperationPlugin
{
    public override string Name => nameof(MultiplyOperation);

    public decimal Calculate(decimal operand1, decimal operand2)
    {
        return operand1 * operand2;
    }
}

public class SubstractOperation : PluginBase, IMathOperationPlugin
{
    public override string Name => nameof(SubstractOperation);

    public decimal Calculate(decimal operand1, decimal operand2)
    {
        return operand1 - operand2;
    }
}
{% endhighlight %}

### Putting all together

Let's get seriously about our code and do some math!

{% highlight c# %}
class Program
{
    static void Main(string[] args)
    {
        DoCalculation();
        GC.Collect();
        GC.WaitForPendingFinalizers();

        Console.ReadKey();

    }

    [MethodImpl(MethodImplOptions.NoInlining)]
    private static void DoCalculation()
    {
        // Create a plugin finder and scan the sub directory "Plugins" for assemblies with plugin defined.
        var pluginFinder = new PluginFinder<IMathOperationPlugin>();
        var assemblyPaths = pluginFinder.FindAssemliesWithPlugins(Path.Combine(Directory.GetCurrentDirectory(), "Plugins"));

        GC.Collect();
        GC.WaitForPendingFinalizers();

        // Create a plugin host and load the plugin assemblies.
        var pluginHost = new PluginHost<IMathOperationPlugin>();
        pluginHost.LoadPlugins(assemblyPaths);
        
        // Use the plugins and print the result of each calculation.
        decimal value1 = 10;
        decimal value2 = 5;
        foreach (var operation in pluginHost.GetPlugins())
        {
            var result = operation.Calculate(value1, value2);
            Console.WriteLine($"Calculation with {operation.Name}: {result}");
        }
        pluginHost.Unload();
    }
}
{% endhighlight %}

The _[MethodImpl(MethodImplOptions.NoInlining)]_ attribute is required to ensure the method is not inlined by the runtime - otherwise everything would live until the end of the application and would prevent the unloading of our assemblies.

Maybe you wonder about the calls of _GC.Collect()_ and _GC.WaitForPendingFinalizers()_.
Those calls are added to demonstrate immediately the effect of _AssemblyLoadContext.Unload()_ method.
By design _AssemblyLoadContext.Unload()_ only triggers the unloading process and the actual unloading will happen when the garbage collection runs - this behavior can be observed during debugging. When for whatever reason a type is referenced by long lived object on the heap (e.g. a static field), the assembly can never be unloaded!

Let's debug it and see what's happening with our module list.
Before we load any plugin assembly, our module list contains everything that is actually used by the console app.

<a href="/assets/img/dotnet30-plugin/dotnet-30-before-find-plugin.png" target="_blank"> 
    ![dotnet-30-before-find-plugin](/assets/img/dotnet30-plugin/dotnet-30-before-find-plugin.png)
</a>

Just after the scan, the list is growing and our plugin assembly is added to the list.

<a href="/assets/img/dotnet30-plugin/dotnet-30-after-find-plugin.png" target="_blank"> 
    ![dotnet-30-after-find-plugin](/assets/img/dotnet30-plugin/dotnet-30-after-find-plugin.png)
</a>

Even though we have already called _AssemblyLoadContext.Unload()_ (inside _pluginFinder.FindAssemliesWithPlugins_), the assembly stays in the module list.
Right after a full GC, the plugin assembly is removed.

<a href="/assets/img/dotnet30-plugin/dotnet-30-after-find-plugin-collected.png" target="_blank"> 
    ![dotnet-30-after-find-plugin-collected](/assets/img/dotnet30-plugin/dotnet-30-after-find-plugin-collected.png)
</a>

The plugin host will load the assembly again and execute all calculations.

<a href="/assets/img/dotnet30-plugin/dotnet-30-after-calculation.png" target="_blank"> 
    ![dotnet-30-after-calculation](/assets/img/dotnet30-plugin/dotnet-30-after-calculation.png)
</a>

Triggering GC will remove our plugin assembly again.

<a href="/assets/img/dotnet30-plugin/dotnet-30-after-calculation-plugin-collected.png" target="_blank"> 
    ![dotnet-30-after-calculation-plugin-collected](/assets/img/dotnet30-plugin/dotnet-30-after-calculation-plugin-collected.png)
</a>

### The AssemblyLoadContext

Basically the [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) is the successor of the [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0) and provides identical functionality - except the security boundary (isolation).
The smallest security boundary is the process and therefore you would need to use inter-process communication to properly isolate data and code execution.

The [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0) is obsolete and you should prefer [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) especially for new work and .NET Core.
Under .NET Core the [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0) is already limited. It does not provide isolation, unloading, or security boundaries.

Every .NET App has at least one (not collectible) [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) named "Default" where all the assemblies are loaded by the .NET runtime.

#### Type != Type

When you deal with multiple [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) instances you could run in the following exception:

 ![dotnet-30-type!=type](/assets/img/dotnet30-plugin/dotnet-30-type!=type.png)

This happens because you can load different versions of the same assembly side by side into the same process.
The direct referenced assembly has a different version than the side loaded library.

#### Migrate from AppDomain to AssemblyLoadContext

Maybe you still using the [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0) in an application.
Now, the following code shows how to replace [AppDomain](https://docs.microsoft.com/en-us/dotnet/api/system.appdomain?view=netcore-3.0) methods by the appropriate equivalent method of [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0):

{% highlight c# %}
    // Create new "context" for loading assemblies:
    var appDomain = AppDomain.CreateDomain("MyAppDomain");
    var assemblyLoadContext = new MyAssemblyLoadContext(name: "MyAssemblyLoadContext", isCollectible: true);

    // Get all assemblies:
    var assembliesFromAppDomain = AppDomain.CurrentDomain.GetAssemblies();
    var assembliesFromAssemblyLoadContext = AssemblyLoadContext.Default.Assemblies;

    // Load an assembly:
    AppDomain.CurrentDomain.Load(AssemblyName.GetAssemblyName("path"));
    AssemblyLoadContext.Default.LoadFromAssemblyName(AssemblyName.GetAssemblyName("path"));

    // Load an assembly from path or byte array:
    AppDomain.CurrentDomain.Load(File.ReadAllBytes("path"));
    AssemblyLoadContext.Default.LoadFromStream(File.OpenRead("path"));
    // or
    AssemblyLoadContext.Default.LoadFromAssemblyPath("path");
{% endhighlight %}

### Conclusion

I'm excited about the new capability of the [AssemblyLoadContext](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.loader.assemblyloadcontext?view=netcore-3.0) class and how it is implemented. It extends the possibilities regarding the architecture and functionality of an application.
Hopefully you like my post and you could take something useful away from it. Let me know what you think :)