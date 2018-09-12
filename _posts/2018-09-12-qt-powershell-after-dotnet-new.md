---
layout: post
title:  "Quick Tip: How to run powershell script after 'dotnet new'"
date:   2018-09-12
---

<p class="intro">
    <span class="dropcap">T</span>his quick tip is about how to run a powershell script after the template is created.
</p>

I like the new <a href="https://github.com/dotnet/templating/" target="_blank"> templating engine</a>. 
It has already a lot of features and they work well.

It is possible to run any external command after the template is created with the 
<a href="https://github.com/dotnet/templating/wiki/Post-Action-Registry#run-script" target="_blank">built-in action</a>.
The key difference is to set the `powershell` as executable and pass the script with the parameter `-File`.

{% highlight json %}
"postActions": [{
         {
          "description ": "Runs the ... script.",
          "manualInstructions": [{ 
            "text": "Run 'filename.ps1'" 
            }],
          "actionId": "3A7C4B45-1F5D-4A30-959A-51B88E82B5D2",
          "args": {
            "executable": "powershell",
            "args": "-File filename.ps1"
          },
          "continueOnError": true
        }
    }]
{% endhighlight %}