---
layout: post
title:  "A home for consoles"
date:   2019-08-15
---

<p class="intro">
    <span class="dropcap">I</span>ncrease your development productivity with the Windows Terminal.
</p>

<br/>

## The Windows Terminal

Did you ever lost focus because you had multiple console applications around your desktop and you didn't know which console is what?

 ![terminal-hell](/assets/img/home-for-consoles/terminal-hell.png)

**Looks familiar?** Then, Windows Terminal will save you and could boost your productivity.
The Windows Terminal is basically a tabbed console host for multiple (different) console applications:

 ![windows-terminal-window](/assets/img/home-for-consoles/windows-terminal-window.png)

Spawning a new tab is similar as you used to from modern browsers by hitting "CTRL+SHIFT+T" or a specifc console with "CTRL+SHIFT+4" (means use profile 4).
So, you never need to remember induvidual console apps - just start Windows Terminal with all of them ahead your fingertips.
Furthermore you can also group tabs by running multiple Windows Terminals side by side.

But the real killer features is that you can extend your terminal real quickly with any console app you have installed and customize the appearance.
You need to edit your Windows Ternminal settings file (opens with your favorite editor for JSON):

![windows-terminal-setting](/assets/img/home-for-consoles/windows-terminal-setting.png)

As alternative, the _profiles.json_ is located in:

%userprofile%\AppData\Local\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\RoamingState\

Now the JSON structure consists out of three main properties **globals**, **profiles** and **schemes**.

To add a new console application, you need to define a new profile inside the **profiles** array.
For example, I added additionally to the defaults _Powershell Core 6_, _Git Bash_ and the _VS 2019 Dev Tools_:

{% highlight json %}
{
    "globals": {
       // omitted
    },
    "profiles": [
        {
            "acrylicOpacity": 0.5,
            "background": "#012456",
            "closeOnExit": true,
            "colorScheme": "Campbell",
            "commandline": "C:\\Program Files\\PowerShell\\6\\pwsh.exe",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{4dff6f50-b4c5-4c32-9f4b-7273d81bc58f}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{61c54bbd-c2c6-5271-96e7-009a87ff44bf}.png",
            "name": "PowerShell Core 6",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%USERPROFILE%",
            "useAcrylic": false
        },
        {
            "acrylicOpacity": 0.5,
            "background": "#012456",
            "closeOnExit": true,
            "colorScheme": "Campbell",
            "commandline": "powershell.exe",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{61c54bbd-c2c6-5271-96e7-009a87ff44bf}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{61c54bbd-c2c6-5271-96e7-009a87ff44bf}.png",
            "name": "Windows PowerShell",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%USERPROFILE%",
            "useAcrylic": false
        },
        {
            "acrylicOpacity": 0.75,
            "closeOnExit": true,
            "colorScheme": "Campbell",
            "commandline": "cmd.exe",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{0caa0dad-35be-5f56-a8ff-afceeeaa6101}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
            "name": "cmd",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%USERPROFILE%",
            "useAcrylic": true
        },
        {
            "acrylicOpacity": 0.85000002384185791,
            "closeOnExit": false,
            "colorScheme": "Solarized Dark",
            "commandline": "Azure",
            "connectionType": "{d9fcfdfa-a479-412c-83b7-c5640e61cd62}",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{b453ae62-4e3d-5e58-b989-0a998ec441b8}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{b453ae62-4e3d-5e58-b989-0a998ec441b8}.png",
            "name": "Azure Cloud Shell",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%USERPROFILE%",
            "useAcrylic": true
        },
        {
            "acrylicOpacity": 0.75,
            "closeOnExit": true,
            "colorScheme": "Campbell",
            "commandline": "\"%PROGRAMFILES%\\git\\usr\\bin\\bash.exe\" -i -l",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{2941fd02-24bc-43ae-a4bc-4f171382a4e5}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
            "name": "Git Bash",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%USERPROFILE%",
            "useAcrylic": true
        },
        {
            "acrylicOpacity": 0.75,
            "closeOnExit": false,
            "colorScheme": "One Half Dark",
            "commandline": "%comspec% /k \"%ProgramFiles(x86)%\\Microsoft Visual Studio\\2019\\Enterprise\\Common7\\Tools\\VsDevCmd.bat\"",
            "cursorColor": "#FFFFFF",
            "cursorShape": "bar",
            "fontFace": "Consolas",
            "fontSize": 10,
            "guid": "{0492434b-d7dc-43c6-b03b-33dd3b9c0453}",
            "historySize": 9001,
            "icon": "ms-appx:///ProfileIcons/{0caa0dad-35be-5f56-a8ff-afceeeaa6101}.png",
            "name": "VS 2019 Dev",
            "padding": "0, 0, 0, 0",
            "snapOnInput": true,
            "startingDirectory": "%ProgramFiles(x86)%\\Microsoft Visual Studio\\2019\\Enterprise\\Common7\\Tools\\VsDevCmd.bat,",
            "useAcrylic": true
        }
    ],
    "schemes" : 
    [
       // omitted
    ]
}
{% endhighlight %}

Beside that, you can define custom color schemes or customize the key bindings for commands like _copy_ or _duplicateTab_.

### Where can I get it?

Well, [Windows Terminal is open source](https://github.com/microsoft/terminal) - you can build it right away from source...

Okay, just kidding. You will need at least Windows 10 version 18362.0 or higher and it's available through the [Microsoft Store](https://www.microsoft.com/store/productId/9N0DX20HK701).
Be aware that, Windows Terminal is still in preview (version 0.3.2171.0) as I post this.

### Conclusion

I'm sold - Although it is only in preview it works for me and I think it would for you as well!
It really helps to reduce the flood of open console windows and it unfolds new possibility how you integrate your favorite console applications within your workflow.

Hopefully, I could give you a good tip and you didn't knew already.<br/>
Let me know what you think.