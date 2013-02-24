---
layout: post
title: "F#, Windows Phone 7 & Visual Studio 2012"
date: 2013-02-24
comments: true
categories: [F#, Windows Phone]
---
F# 2.0 supported Silverlight 4, Silverlight 5, and Windows Phone 7.1, as several versions of FSharp.Core.dll were
shipped as part of the standalone F# redistribution. There were even some nice [Visual Studio 2010 templates](http://blogs.msdn.com/b/dsyme/archive/2010/08/20/f-windows-phone-7-silverlight-templates-now-on-visual-studio-gallery.aspx) made by [Daniel Mohl](http://bloggemdano.blogspot.co.uk/).

In F# 3.0 there is no standalone redistributable package anymore, and Visual Studio 2012 only ships with two versions of FSharp.Core.dll: the full .NET 4.5 version, and a portable class library version targeting Profile47 (which includes .NET 4.5, Silverlight 5.0, and .NET for Windows Store apps).

I tried to create versions for Windows Phone 7 and for Profile88 (which additionally includes .NET 4.0, Silverlight 4.0, and Windows Phone 7.1) from the [source code](https://github.com/fsharp/fsharp), and I even got them to [compile](https://github.com/fsharp/fsharp/pull/102), but unfortunately I still get a bunch of Invalid Program exceptions at runtime when running on the device on both versions.

So I tried another approach - to reuse the FSharp.Core.dll version from F# 2.0 - and it seems to work just fine. Type providers and other new F# 3.0 features that depend on the library won't work, but at least we can use Visual Studio 2012 and some of the features like the triple quotes.

To make this work we must force the F# compiler to compile for Silverlight 4, which is no longer supported, so we need to modify the file `C:\Program Files (x86)\Microsoft SDKs\F#\3.0\Framework\v4.0\Microsoft.FSharp.targets` and comment out the following lines:

{% codeblock lang:xml %}
<Error
    Condition="'$(SilverlightVersion)' != '' and '$(SilverlightVersion)' != 'v5.0'"
    Text="In this version of Visual Studio, F# for Silverlight can only target Silverlight v5.0. Use a prior version of Visual Studio to target previous versions of Silverlight with F#."
/>
{% endcodeblock %}

Then if you take Daniel Mohl's template, change the manifest to state it supports Visual Studio 2012, and upgrade the .csproj and .fsproj files, it all works. I've uploaded the updated template to the [Visual Studio Gallery](http://visualstudiogallery.msdn.microsoft.com/241d3a0a-a0a7-42f5-badf-bbbed30514c8).
