---
layout: post
title: "FSharp.Data 2.0.0 released"
date: 2014-03-10 02:00
comments: true
categories: [F#, Type Providers]
---

Version [2.0.0](http://www.nuget.org/packages/FSharp.Data/2.0.0) of [FSharp.Data](http://fsharp.github.io/FSharp.Data/) was just released. You can get the full version history on the [release notes](https://github.com/fsharp/FSharp.Data/blob/master/RELEASE_NOTES.md), but here are the highlights:

* A new type inference algorithm that will generate better and more robust API's in the presence of missing and heterogeneous types. This means that you'll get much less errors when using JsonProvider and XmlProvider on real world messy files. The error messages are now also more descriptive when things go wrong.
* Much improved Freebase support for individuals.
* The functionality of the Http class was greatly extended. It now supports uploading and downloading binary files, and compression across all supported platforms, among many other tweaks.
* Improved the overall performance of the type provider itself when running inside the editor, and also of the generated code.
* The documentation was much improved, there's now a Japanese translation of it, and comprehensive reference documentation.
* Mono is now fully supported, as well as F# 3.1, and the new Portable Class Library profile 7.
* Source link support. This means that you can step trough the source code while debugging if you enable that feature in Visual Studio.

We also took the opportunity of major version bump to do some house cleaning, so things that we weren't previously very happy with were renamed, dropped or, shuffled around. Unfortunately this means breaking changes, but we think the overall outcome is positive. In this process we dropped support for:

* Silverlight and Windows Phone separate packages.
* Apiary provider (which has now moved to a [separate project](https://github.com/fsprojects/ApiaryProvider)).

Thanks to everyone which contributed directly to this release, [Tomas Petricek](http://github.com/tpetricek), [Don Syme](http://github.com/dsyme), [Steffen Forkmann](http://github.com/forki), [Enrico Sada](http://github.com/enricosada), [Cameron Taggart](http://github.com/ctaggart), [Remko Boschker](http://github.com/remkoboschker), [Vasily Kirichenko](http://github.com/vasily-kirichenko), [Diego Frata](http://github.com/diegofrata), [yukitos](http://github.com/yukitos), [Denis Ok](http://github.com/OkayX6), and [James Holwell](http://github.com/jamesholwell), and also to everyone who also contributed indirectly through yak-shaving needed in [FSharp.Formatting](http://tpetricek.github.io/FSharp.Formatting), [FSharp.Compiler.Services](http://fsharp.github.io/FSharp.Compiler.Service), and [FAKE](http://fsharp.github.io/FAKE), including [Dave Thomas](http://github.com/7sharp9), [Xiang Zhang](http://github.com/soloman817), [Anh-Dung Phan](http://github.com/dungpa), [fbmnds](http://github.com/fbmnds), and [Max Malook](http://github.com/mexx). And also thanks to everyone who contributed by just placing issues on github, but I couldn't find a way to generate a list of all of you.

There's still a lot of room for improvement, and we love pull requests, so please contribute, even if it's only fixing a typo in the docs. A getting started guide is available [here](http://fsharp.github.io/FSharp.Data/contributing.html).

For the next versions, we want to add:

* Html type provider, currently being worked on by [Colin Bull](http://github.com/colinbull), [Mathias Brandewinder](http://github.com/mathias-brandewinder) and me.
* Write API for Json, currently being worked on by [Ross McKinlay](http://github.com/pezipink).
* Authentication support, currently being worked on by [Veikko Eeva](http://github.com/veikkoeeva).
* XSD support, which I hope one of you reading this will pick up :).

