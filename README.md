# Tutorial: Build Modern Cross-Platform Apps with .NET

[1. Introduction](README.md)  
[2. Avalonia UI](2_Avalonia.md)  
[3. Dependency Injection](3_DependencyInjection.md)  
[4. MVVM Design](4_MVVM.md)  
[5. Dialogs and Tools](5_DialogsTools.md)  
[6. Unit Testing](6_UnitTesting.md)  
[7. Reactive](7_Reactive.md)  
[8. Deployment](8_Deployment.md)  
[9. Multiple Environments](9_MultipleEnvironments.md)

## 1. Introduction

by Etienne Charland

I started programming in .NET back in the days of .NET 1.0 beta 1. Transitioning from Visual Basic was a big deal, and there was a lot of excitement around .NET! Things have gone a very long way since.

Design patterns have evolved over time. Tools have evolved. I used to program WinForms apps with N-Tier design; which is considered rubbish spaghetti code in today's standards.

The reason I'm writing this tutorial is that although .NET has evolved a lot and is very popular for web development, it is mostly dead when it comes to desktop development, and there's a long learning curve to get the right tools. I've spent a lot of time to upgrade my skills, and I'm hoping that it will help others sharpen their blades!

### Advantages of .NET

.NET is really a fantastic programming platform. Some people prefer simple scripting languages that deliver quick results, but I absolutely hate scripting languages. I want to build something clean and solid that will be easy to expand in the future. **Over time, the easy road gets hard, and the long road gets easy.** It may be harder for some simple tasks, but it's easier for complex tasks. I recommend C# because most of the code you'll find online is in C#. If you want fancier, some people really like F#.

Some advantages of .NET/C#
- Highly performant*
- Cross-platform (Windows, Linux, MacOS, iOS, Android)
- Can embed the framework to have no dependencies
- Great community support
- Intuitive and compact language syntax
- Same language for desktop or web applications

*Note about performance. You can argue that C++ is a lot more performant, and you are absolutely right. Do your low-level processing that requires performance in C++. However, the performance of an overall application more depends on its overall design and structure, and badly-structured code can become performance pits. Nowadays, performance more depends on efficient multi-threading, and any kind of multi-threading in C++ is an absolute mess. Multi-threading in .NET is a breeze (with await/async keywords). .NET allows building high-level applications with a structure that make it very efficient, calling individual components as needed and native C++ libraries when needed. Performance of .NET is a lot better than Python or JAVA.

### The problem with .NET for Desktop

Microsoft released WinForms back in 2002. Then, they released WPF in 2006 which allowed the use of MVVM design pattern to fully separate the logic from the UI. So far so good; except that MONO never considered WPF important enough to support it, and thus WPF has never worked on Linux or MacOS, leaving MONO with archaic WinForms the only cross-platform solution.  In 2015, they released UWP to develop Windows Store apps. You can search for UWP jobs across the USA and you'll find about 1, as a testimony of its success. It comes with tons of limitations and again, works only on Windows. There is UNO that builds on top of UWP that makes it cross-platform, but you get all the limitations of UWP. Microsoft then released WinUI 1, 2 and 3, which have a long list of issues. They are working on MAUI to allow running apps on MacOS, but even there, it will not support Linux. I don't know whether you've learnt all of these platforms -- I personally haven't.

What I'm trying to say is that the desktop toolkit situation is a complete mess, and that's why most programmers have moved to other alternatives.

But WAIT! There is one more option: [Avalonia UI](https://avaloniaui.net/). A rewrite of WPF with all the modernizations of UWP and WinUI that works on all platforms: Windows, Linux, MacOS, and soon iOS, Android and even Blazor. It is mature enough for commercial use.

**Although the tools presented here are very powerful, not many people are using this combination of tools because of the sharp learning curve, and the help online is limited. I'm hoping that this tutorial will help grow the community.**

### My Setup

I got tired of Windows taking ownership of our computer and dumped it ahead of Windows 11's release. I'm using Linux; if you're a programmer, Arch-based is a good option to be cutting-edge. I choose [Garuda Linux](https://garudalinux.org/) and love it (but do NOT install arch-based for your wife and grand-ma!). I got Windows in a VM and rarely use it at all. This forced me to modernize all my apps.

For programming IDE, Visual Studio works only in Windows, and Visual Studio Code is very limited. I'm using [JetBrains Rider](https://www.jetbrains.com/rider/) which works on Windows, Linux and MacOS. They have free license for students and for open source developpers (as long as you have an active project history). [See their offers here.](https://www.jetbrains.com/rider/buy/#discounts) Even on Windows, you'll get a great productivity boost by switching to JetBrains Rider -- try it and you won't look back. JetBrains Rider has great support for Avalonia.

Basically -- .NET works best if you dump Windows, WPF and Visual Studio.

There are many topics to cover to build modern applications, and I'll cover a good selection of tools and designs. You could read a whole book on each topic, and I'll summarize the essentials with my own experience, with links to learn more.

I will use the [432hz Batch Converter](https://sourceforge.net/projects/converter432hz/) several times for sample codes. It can be downloaded for Windows, Linux or MacOS.

You can post your comments in the [Discussions section](https://github.com/mysteryx93/Modern.Net-Tutorial/discussions).

[> Next: Avalonia UI](2_Avalonia.md)
