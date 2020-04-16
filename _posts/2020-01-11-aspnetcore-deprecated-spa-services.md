---
layout: post
title: "The deprecated ASP.NET Core SpaServices"
date: 2020-01-11
---

<p class="intro">
    <span class="dropcap">A</span>SP.NET Core (Vue) SPA: How to solve the deprecated "Microsoft.AspNetCore.SpaServices".
</p>
<br/>

### Intro

Almost half year ago, there was an [announcement](https://github.com/dotnet/aspnetcore/issues/12890){:target="_blank"} that the two nuget packages [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/){:target="_blank"} and [Microsoft.AspNetCore.NodeServices](https://www.nuget.org/packages/Microsoft.AspNetCore.NodeServices/){:target="_blank"} are getting deprecated. Thus they will be removed and no longer supported starting with .NET 5. At the time of writing this post, ASP.NET Core 3.1 is marking the methods of those packages as deprecated, but still functional and supported.

Needless to say, that many developer are not really happy with the deprecation decision because they rely on one or both nuget packages.
I wrote as well a [comment to this discussion](https://github.com/dotnet/aspnetcore/issues/12890#issuecomment-518545514){:target="_blank"} to encourage that there should be at least a migration guide.
For everyone that doesn't know the packages, they are usually used for the following scenarios (but not limited to):

* ASP.NET Core SPA (React, Angular, Vue or others)
    * Server-side prerendering
    * HMR (Hot Module Reload)
* Executing javascript within a node process out of a .NET process

With this post, I try to help other developers to migrate, be aware and solve the issue early as possible!

### My ASP.NET Core Vue SPA

I have a side project, an ASP.NET Core Vue SPA application that does using those nuget packages for HMR (BTW. this works similiar for other SPA frameworks). The `Startup.cs` does look like this (ommited unrelevant parts):

{% highlight c# %}
    public void Configure(IApplicationBuilder app, IHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            // Here we add the webpack middleware with the deprecated method:
            #pragma warning disable CS0618 // Type or member is obsolete
            app.UseWebpackDevMiddleware(new WebpackDevMiddlewareOptions { HotModuleReplacement = true, });
            #pragma warning restore CS0618 // Type or member is obsolete
        }
        else
        {
            app.UseExceptionHandler("/Home/Error");
        }

        // ...

        app.UseEndpoints(endpoints =>
        {
            // ...
            endpoints.MapFallbackToController("Index", "Home");
        });
    }
{% endhighlight %}

This `app.UseWebpackDevMiddleware()` adds a middleware to the pipeline. This middleware hosts an instance of a Webpack compiler in memory so that it serve up-to-date Webpack-built resources. Incoming requests that match Webpack-built files will be handled by returning the Webpack compiler output directly and since the Webpack compiler instance is retained in memory, incremental compilation is vastly faster that re-running the compiler from scratch.

This is a tremendous productivity boost while working on the front-end part of the application, because you don't have to restart the whole ASP.NET Core application process. I personally loved this and many other developers too. So, let's see how I removed the deprected package(s) and remain same productivity!

### The Solution

The proposed solution does only work when you have a mini-web-server as most front-end frameworks provide today (e.g. Angular CLI, create-react-app, vue-cli-service). For my ASP.NET Core Vue SPA application, I used the [VueCliMiddleware](https://www.nuget.org/packages/VueCliMiddleware/){:target="_blank"} nuget package that does the heavy lifting. The `Startup.cs` has been changed to the following: 

{% highlight c# %}
public void Configure(IApplicationBuilder app, IHostEnvironment env)
{
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller}/{action=Index}/{id?}");

        if (env.IsDevelopment())
        {
            // This forwards everything to the "vue-cli-service":
            endpoints.MapToVueCliProxy(
                "{*path}",
                new SpaOptions { SourcePath = "ClientApp" },
                npmScript: "serve",
                regex: "Compiled successfully");
        }

        // ...
    });

    app.UseSpa(spa =>
    {
        spa.Options.SourcePath = "ClientApp";
    });
}
{% endhighlight %}

The solution works fundamentally different from the previous solution.
Incoming requests that match Webpack-built files will be "proxied" through the ASP.NET Core process to the "behind" running mini-web-server (here the vue-cli-service) - this works the same for other SPA frameworks like React and Angular.

In this first picture, you can see the ASP.NET Core app runs on localhost:5000 (red box).
Immediately after that, a "vue-cli-service" process is started and builds the vue app (blue box).

![dotnet-process](/assets/img/aspnetcore-deprecated-spa-services/dotnet-process.png)

After the "vue-cli-service" is finished building, it listens on localhost:8080 to serve the vue app.

![vue-cli-service](/assets/img/aspnetcore-deprecated-spa-services/vue-cli-service.png)

**Note:** When your application runs in production, there is no additional web server required - everything is served by a single web server like IIS or Kestrel.

### Conclusion

In this post I described how you can remove the deprecated dependency and how you can replace it with something that is similar. 
I essentially explained this with Vue because there is only officially support for Angular and React out of the box.

When you have to migrate your ASP.NET Core 2.x SPA project: I recommend to have a look at the starter project template, to see how you could change the `Startup.cs`:

* For Angular, have a look at the (official) [ASP.NET Core SPA Angular Template](https://github.com/dotnet/aspnetcore/tree/master/src/ProjectTemplates/Web.Spa.ProjectTemplates/content/Angular-CSharp){:target="_blank"}.
* For React, have a look at the (official) [ASP.NET Core SPA React Template](https://github.com/dotnet/aspnetcore/tree/master/src/ProjectTemplates/Web.Spa.ProjectTemplates/content/React-CSharp){:target="_blank"}.
* For Vue, have a look at the (unofficial) [ASP.NET Core Vue SPA template](https://github.com/SoftwareAteliers/asp-net-core-vue-starter){:target="_blank"} - that has the same solution as I used here.
