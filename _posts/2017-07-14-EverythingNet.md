---
layout: post
title: Everything Search meets .NET
date:   2017-07-14 03:00:08
categories: .NET
comments: true
description: C#, Search, .NET, Library
---

# What
I've been using [Everything](https://www.voidtools.com/) for a long time and I use it every single day. Everything is like 'Windows Search' but works better way faster ! Don't hesitate to read the [Faq](https://www.voidtools.com/faq/)

When I discovered a native C library was also available I tought it could be useful to create a .NET library that wraps C library.
So I decided to create a .NET library that exposes most of Everything capabilities in a fluent API (I love fluent APIs).

You can find my GitHub repository [here](https://github.com/ju2pom/EverythingNet).

# Why
I think there are a lot of potential in using this library in modern applications (WPF or UWP).

* Applications can gather all its associated files (base on extension for exemple)
* Disk optimisation applications (easily find duplicates, big files, old files ...)
* ...

# Code
```csharp
Everything everything = new Everything();
var searchResult = everything.Search()
 .Name("temp")
 .Or
 .Name("tmp");
```

```csharp
Everything everything = new Everything();
var searchResult = everything.Search()
 .Name()
 .Extension("xaml");
```

```csharp
Everything everything = new Everything();
var searchResult = everything.Search()
 .File()
 .Picture();
```

# Conclusion
Don't loose one more second and go to [Everything](https://www.voidtools.com/) and install !

Check my [github](https://github.com/ju2pom/EverythingNet) repo for more sample code and try it out in your own application.

# Disclaimer
I've have no link with **Everything/Voidtools** I'm just a fan and long time user.
