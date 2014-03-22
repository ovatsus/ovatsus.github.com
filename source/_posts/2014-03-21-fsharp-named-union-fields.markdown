---
layout: post
title: "F# named union fields"
date: 2014-03-21 22:37
comments: true
categories: [F#]
---

One of the new features in F# 3.1, and one that I really like, is the possibility to name fields of discriminated unions. From the [F# 3.1 announcement blog post](http://blogs.msdn.com/b/fsharpteam/archive/2013/06/27/announcing-a-pre-release-of-f-3-1-and-the-visual-f-tools-in-visual-studio-2013.aspx):

>Discriminated union types are a powerful feature of F# which make programming with shaped data simple, accurate and robust. They also greatly reduce the number of classes needed to represent data in many common circumstances.

>In F# 3.1, it is now possible to name union fields within each case of a union type. This is important for large-scale software engineering, particularly when union cases have a large number of fields or multiple fields with the same type.

> ![](http://blogs.msdn.com/cfs-file.ashx/__key/communityserver-blogs-components-weblogfiles/00-00-01-39-71-metablogapi/2021.31blog_5F00_1_5F00_0EEA8EAA.png)

What wasn't originally mentioned on that post, I just found out today, is that pattern match by name is not limited to a single element of the case: we can match on multiple elements by separating them by `;`.

When we pattern match by position, the several elements are separated by `,`, and as that didn't work with union field names, I assumed it wasn't possible to do at all. But it turns out it is possible if you use `;` instead. Here's an example taken from F# Data:

{% codeblock lang:fsharp %}
type InferedType =
  | Primitive of typ:System.Type * unit:option<System.Type> * optional:bool
  | ...

// positional pattern match:
match x with
| Primitive(typ, _, false) -> ...
| ...

// pattern match by name 
match x with
| Primitive(typ = typ; optional = false) -> ...    
| ...
{% endcodeblock %}
