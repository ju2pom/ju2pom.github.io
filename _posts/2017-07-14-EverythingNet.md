---
layout: post
title: Everything Search meets .NET
date:   2017-07-14 03:00:08
categories: .NET
comments: true
description: C#, Search, .NET, Library
---
# What

I've been using [Everything](https://www.voidtools.com/) for a long time and I use it every single day. If you don't know this tool yet, in short it's like the 'Windows Search' but it just works and it's incredibly fast. Don't hesitate to read the [Faq](https://www.voidtools.com/faq/)

When I discovered a library was a available I tought it would be great (and useful) to create a .NET library that wraps the native C library.
So I have created a .NET library that exposes most of Everything capabilities with a fluent API. You'll find the GitHub repo [here](https://github.com/ju2pom/EverythingNet).

# Why

I think there are a lot of potential in using this library in modern applications (WPF or UWP). Any application that deals with specific file format  could find such files in a fraction of a second.

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

Don't loose one more second to search for files and go to [Everything](https://www.voidtools.com/) and install the tool.

Check the [github](https://github.com/ju2pom/EverythingNet) repo for more sample code.

# Disclaimer

I've have no link with **Everything/Voidtools** I'm just a fan and long time user. But I am the author of EverythingNet library.
