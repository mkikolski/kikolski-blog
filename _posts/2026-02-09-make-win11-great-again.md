---
layout: post
title: "How to enjoy using Windows 11"
subtitle: "Or just make it usable"
date: 2026-02-09 02:00:00 +0200
tags: [OS, windows, config, microslop]
excerpt: "That's a post I didn't expect I'll be writing after a year of using Linux with tiling WMs..."
---

## The beginning of the end

That's a post I didn't expect I'll be writing after a year of using Linux with tiling WMs and finally [settling down for one](). Then, around the end of 2025 it finally came time for me to build a new PC - strong enough to handle my AI work, newer games and game development. After using Fedora with Hyprland on my laptop, I couldn't imagine going back to Windows 11 and wasn't quite sure that Fedora was the system I've been looking for (oh boy, how wrong I was). That's when I've decided to give Omarchy a go - it was described as Arch Linux with the training wheels and as a quite busy person, I decided it was going to be my system (not to mention, my friends endorsed it quite heavily). Just for sake of gaming and some tools for work, I've decided to install Windows 11 as the second system. I've used Omarchy for maybe a week and then I felt it for the first time - it was definitely not the thing I was looking for. The configuration didn't allow me chaotic ricing and the whole experience felt far less exciting than my Fedora based laptop. Maybe it was due to lack of time, maybe due to lack of patience but I decided to leave Omarchy alone and instead of replacing it with some other distribution, I've booted into Windows 11 and it was bad. Starting with search bar including web search, preinstalled OneDrive and Cortana, some shitty Copilot features, the overall feel was just as bad, as I remembered it. But then, I started digging...

## Making it work

Well, the last sentence of the previous paragraph might be only partially true - I made sure that some of the features wouldn't bother me as early as when creating the bootable drive for installing the Windows. I used Rufus and I've been happily surprised to see this dialog:

![Windows Experience Dialog](/assets/images/windows-experience.png)

It allowed me to disable annoying stuff like Bitlocker, data collection questions and forcefully created an offline account, but that's all of the lifting that Rufus has done for me.

### Getting rid of annoying shit

Some weeks before building the PC, I've seen some Instagram reel that showcased a tool called [Win11Debloat by raphire](https://github.com/raphire/win11debloat), which was my first step in the fight against Microslop. The process of launching it was quite easy, because it was just one PowerShell command:

```posh
& ([scriptblock]::Create((irm "https://debloat.raphi.re/")))
```

> [!NOTE]
> Well now, when it came to running something in the PowerShell, I remembered that I've also installed PowerShell 7 somewhere around that time and have been using it as a default shell.

What can we do with our debloating script. I think this image from the repo speaks more than a thousand words:

![Win11Debloat Menu](/assets/images/win11debloat.png)

I don't remember the exact settings I've used but all of them are well described and you'll definitely manage to configure it in the way that suits you. This tool alone cleaned up a lot of annoying stuff, but underneath it was still (debloated) Windows and I desperately missed the tiling window manager feel and a proper run menu executed with a shortcut.

### Quick terminal improvements

Before getting to overhauling the whole UX with a new window manager, I decided to take some time to customize the terminal, which is quite important for my work. On Fedora, I'm using zsh with `oh-my-zsh` and after a quick search, I found [`oh-my-posh`](https://ohmyposh.dev/) which is kind of a drop-in replacement for my use case. The installation was really simple and used `winget` package manager. I've decided to go with cobalt2 style - it's not perfect, but I didn't hate the way it looked. I finished up by installing [Catppuccin theme for PowerShell](https://github.com/catppuccin/powershell) and setting the window to be a bit transparent. As ~~an usually unbearable person~~ a daily Linux user I've also installed `fastfetch` via `winget install fastfetch` and thats where my terminal configuration was done:

![Terminal customization](/assets/images/terminal-customized.png)

> [!NOTE]
> Now I see that the order of customization might have been a bit different, but I'll cover all the stuff I've done so the end result shouldn't differ that much.

### Task bar

Before dealing with the window manager, I tried to customize the default Windows taskbar to my liking but it couldn't make that work. When researching the tiling window manager options, I noticed that people were using [YASB](https://github.com/amnweb/yasb) which looked great and had all the stuff I like in Waybar. Installation was also available via `winget` so that's how I proceeded. Styling took me a while but using existing configs (here, I'm using [Cosmic bar](https://github.com/amnweb/yasb-themes)) alongside some trial-and-error tweaks allowed me to get what I wanted.

![Taskbar](/assets/images/yasb-config.png)

### The real deal

Fortunately, the people tend to like tiling window managers and my first google search resulted in finding [Komorebi](https://github.com/LGUG2Z/komorebi) which was exactly what I was looking for. Installation was really simple and also used `winget`:

```posh
winget install LGUG2Z.komorebi
winget install LGUG2Z.whkd
```

For proper shortcut handling, the installation required `whkd` which is a hotkey daemon that allows hotkeys operation with no weird workarounds. Besides tweaking some of the padding values in `~/komorebi.json`, I used the default settings for workspace management and it's all up to you. Besides that, you can add special configurations for certain applications in `~/applications.json`, especially useful when gaming or just using fullscreen applications. The only drawback of tiling WM on Windows is the fact that there's no way to use Windows key as the special key and I resorted to using **alt** which messed up my muscle memory a bit when switching from my Hyprland config.

![Komorebi](/assets/images/komorebi-look.png)

Last, really important step was setting komorebi and yasb to launch automatically. This can be achieved by running:

```posh
komorebic enable-autostart
yasbc enable-autostart
```

These commands put shortcuts for `komorebic start --whkd` and `yasb` in `shell:startup` but why bother if we've got simple commands available.

### Additional customization

To make everything look even better (or just the way I liked it more), I used Windhawk - a marketplace for Windows modifications and installed following packages:

- Acrylic Effect Radius Changer
- Taskbar tray icon spacing and grid
- Windows 11 File Explorer Styler
- Windows 11 Taskbar Styler

Styling is again, totally up to you and easily custimizable so I won't bother sharing the settings, as most of them are set to default. Actually, at first I had one more package installed - Windows 11 Start Menu Styler, but I removed it as I wanted to get rid of Start menu.

### Application launcher

As a replacement, I found **PowerToys Run** which, at it's own is quite good - executed by `Alt` + `Space` shortcut, allows you to run a lot of applications and even perform basic caluclations or execute commands. At this point in time, on my Linux machine I introduced `fzf` into my workflow. For those who don't know, `fzf` is a command-line tool that allows for blazing fast fuzzy searching all the files on the machine. And that's when I came across `Everything` - a Windows app that does virtually the same but with a GUI. It's cool to use by itself, but it's even cooler that the results are accessible from PowerToys Run menu. 

First of all, I installed [PowerToys](https://learn.microsoft.com/pl-pl/windows/powertoys/install?tabs=winget%2Cextract-094) (once again using `winget`). At this point, it's worth mentioning that Run is only one of the PowerToys features and I really didn't have time to explore the full capabilities of this package. The next, logical step was getting [Everything](https://www.voidtools.com/support/everything/installing_everything/), which requires us to download the installer. Make sure that during the install you allow the app to run on startup and install its service. Finally we get to install [EverythingPowerToys](https://github.com/lin-ycv/EverythingPowerToys/wiki) which bridges PowerToys Run with Everything service. Now we can launch regular apps:

![PowerToys Run](/assets/images/powertoys1.png)

As well as search files with Everything:

![EverythingPowerToys](/assets/images/powertoys2.png)

## Closing remarks

I'm aware that this guide is not perfect and doesn't use the whole power of Windows customization. Fortunately there are loads of guides of achieving specific effects, as this one was meant to show you that you can actually make Windows 11 experience bearable. Make sure to let me know what you think or if there are any modifications you use that I haven't mentioned.
