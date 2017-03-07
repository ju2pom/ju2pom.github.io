---
---
# A great WPF tool
(Snoop homepage)[https://snoopwpf.codeplex.com/]
Snoop for WPF is a quite old tool (last release update was in 2012) but I can't work on WPF/Xaml without it. I can even say that it's always launched on my computer.
Here is what it looks like
![Snoop](/images/snoop.jpg)

Looks pretty simple isn't it? 

## What is for?
When developing a WPF you can often have to relaunch your application every single minute just to check a new margin/color/whatever !
Thanks to Snoop you can launch your application, "snoop" it, and then you can change almost any property of any element **Time saver**.


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
You can either pick an element in the visual tree or use "CTRL+SHIFT" and move the mouse fly over the control you want to inspect.
