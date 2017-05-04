---
layout: post
title: Snoop for WPF
date:   2017-03-2 03:00:08
categories: WPF
comments: true
description: WPF, Xaml, Snoop, Visual Studio
---
# A great WPF tool
[Snoop for WPF](https://snoopwpf.codeplex.com/) is a quite old tool (last release update was in 2012) but even in 2017 I still can't work on WPF/Xaml without it. I always have it launched on my computer.
Here is what it looks like

> Looks pretty simple isn't it? 
![Snoop](/images/snoop.jpg)

> Here is the meat !
![Snoop2](/images/snoop2.jpg)

## What is for?
When developing a WPF you can often have to relaunch your application every single minute just to check a new margin/color/whatever !
Thanks to Snoop you launch your application once, "snoop" it, and then you can change almost any property of any element **Time saver**.

## Pros
- I like it's minimal size, thanks to that I can keep it alaws opened
- As it's a standalone tool you don't have to enable/disable anything in Visual Studio
- Your application UI stays very clean even when you are inspecting it
- You can use snoop's property grid on top of your window without having Visual Studio poping on top !

## Cons
- The main window UX is not really user friendly
- The workspace in not remembered between two sessions
- Some issues when dealing with multiple windows in the same application

# How to use
To "snoop" your WPF application you have to drag'n drop the right most target icon anywhere in the WPF window.
Then you've got a big window with your WPF visual tree on one side and a kind of property grid on the other side.
You can either pick an element in the visual tree or use "CTRL+SHIFT" and move the mouse over the control you want to inspect.

But you can also:
- Inspect datacontext
- Change datacontext property (which can update bindings)
- Obeserve raised events
- Call methods (I never tried)

# Tips
1. WPF controls expose quite a lot of properties which can quickly make the property grid overcroweded ! So Snoop allows you to reduce that number with predefined categories (Layout, Grid, Colors ...). You can even create custom categories but I have never been able to make custom categories persist.
2. You can also use the search field and clear it using *esc* key
3. One final tip I discovered this year in 2017: you can use <code>|</code> (pipe) in the search field. Here is my favorite: *<code>usel|snaps</code>* which will let you tweak *UseLayoutRounding* and *SnapsToDevicePixels*

# Alternatives
1. Visual Studio 2015 introduced a new tool that has exactly the same goal (Debug->Windows->Live Visual Tree & Live Property Explorer)
2. Xaml Spy (http://xamlspy.com/)


