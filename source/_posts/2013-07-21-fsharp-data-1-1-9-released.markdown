---
layout: post
title: "FSharp.Data 1.1.9 released"
date: 2013-07-21 22:03
comments: true
categories: [F#, Type Providers]
---

We've just released version 1.1.9 of [FSharp.Data](https://github.com/fsharp/FSharp.Data), and I realized we've not been announcing new releases since version 1.1.1 when Thomas [first announced](http://tomasp.net/blog/fsharp-data.aspx) FSharp.Data, so I though I'd rectify that. In addition to bug fixes, here what's has been added to FSharp.Data since then:

* The behaviour of CsvProvider can be fine tuned by using the HasHeaders, Schema, IgnoreErrors, SafeMode, and PreferOptionals parameters. See the CsvProvider documentation [here](http://fsharp.github.io/FSharp.Data/library/CsvProvider.html).
* The CSV Parser and Reader can now be used standalone. See the CsvFile documentation [here](http://fsharp.github.io/FSharp.Data/library/CsvFile.html).
* Guid types are now supported in CsvProvider, JsonProvider, and XmlProvider.
* Support to disable caching for CSV files too big to fit in memory.
* Support for cookies, binary files and client certificates in FSharp.Net.Http.
* Save, Filter, Take, TakeWhile, Skip, SkipWhile, and Truncate operations for CsvProvider and CsvFile.
* Support for Windows Phone 7.

There's still a lot of room for improvement, and we love pull requests, so please contribute, even if it's only fixing a type in the docs. A getting started guide is available [here](http://fsharp.github.io/FSharp.Data/contributing.html).