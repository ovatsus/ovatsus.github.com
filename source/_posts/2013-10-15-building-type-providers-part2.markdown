---
layout: post
title: "Building Type Providers - Part 2"
date: 2013-10-15 22:00
comments: true
categories: [F#, Type Providers]
---






Type Providers are the biggest new feature of F# 3.0. They allow us to leverage the power of the static type system in areas that have typically been the territory of dynamic languages and stringly typing. A picture is worth a thousand words, so here's 12 pictures that show type providers in action:

> [All your types are belong to us](http://blogs.msdn.com/b/dsyme/archive/2013/01/30/twelve-type-providers-in-pictures.aspx)

There are already several good samples and articles about using type providers, but there still isn't a lot about creating type providers. Having worked recently on [FSharp.Data](http://fsharp.github.com/FSharp.Data/) I decided to share some tips about building type providers.

<!-- More -->

## Getting Started

The basic steps to creating a type provider are:

* Get the `ProvidedTypes.fs` and `ProvidedTypes.fsi` files from the [F# 3.0 Sample Pack](http://fsharp3sample.codeplex.com/) and add it to your project.

* Follow the instructions in [Tutorial: Creating a Type Provider](http://msdn.microsoft.com/en-gb/library/hh361034.aspx) to create instances of `ProvidedTypeDefinition`, `ProvidedProperty`, `ProvidedMethod`, etc. in order to to define the API you want.

After you've read that tutorial, let's carry on...

## Hiding the underlying representation ##

In the simplest of cases, there's no underlying representation, so you just set the base class of your provided types to `obj`. You'll create some expressions by wrapping code in `<@@ ... @@>` to create quotations, and assign them to the `InvokeCode` and `GetterCode` properties of your provided methods and properties, completely ignoring the `this` argument that is passed to them. After compilation, there will be no mention whatsoever of your provided types, they will have been all erased, and the only thing left is that code you defined inside the quotations. Effectively, the type providers are only giving you syntactic sugar and IntelliSense over things you could already accomplish without using them.

However, in most cases, there will be some underlying data representation, and you will probably want to create some types to hold that data. A big part of the logic will be in these types, and you'll specify them as the base classes of your provided types. In this case, the F# compiler will copy and embed those base types in the assembly that is consuming the type provider. This means that in addition to the syntactic sugar, the assembly that's referencing the type providers will also have some more additional code.

As expected, all the members of the base types will appear on the IntelliSense of the provided types. In some situations, that's a good thing, and exactly what you wanted. On other cases (and I would guess most of the time), you intended those members to be just internal code to be used inside the implementation of the provided methods and properties, and you don't want them to show up, as after all Type Providers are mostly about providing a nice API to its users. You can see this effect on the current version of the [World Bank type provider on TryF#](http://www.tryfsharp.org/Learn/data-science#world-bank-type-provider). If you press the dot after `data`, you'll see that in addition to `Countries` and `Regions`, the IntelliSense pop-up will also include `_GetCountries`, `_GetRegions` and other members that aren't intended for direct use. The ideal would be for them not to show up. How can we do that?

* If you came from an OO background, the first thing you'd think would be to make those members protected, but there is no `protected` modifier in F#. Even if there was, you're not really creating a type and inheriting from the base type, it's all erased.
* You can't also make those members `internal`, because your runtime representation types are always considered to belong to the type provider assembly, and not to the assembly that is using the type provider, even when the F# compiler copies those types over.
* In C#, you can decorate methods and properties with `[System.ComponentModel.Browsable(false)]` to have them not show up on IntelliSense, but unfortunately that doesn't work in F#.
* You can create an interface with those members, and make your runtime representation types implement that interface. That way, they won't show up on IntelliSense because F# always does explicit interface implementation. This won't work for static members though.
* The constructors of the erased to types don't show up on the IntelliSense, so you can replace some static members like `Parse` and `Load` with constructors instead.
* If you define a `ProvidedMethod` with the same parameter types of one of the methods of the runtime representation type, just with a different return type, that method will effectively be hidden, and you won't need to do anything to hide it.
* `ProvidedTypeDefinition` has a `HideObjectMethods` property that allows you to hide the `GetHashCode()`, `GetType()`, `ToString()` and `Equals` methods from IntelliSense, but if the base type overrides any of those methods, they will show up anyway. I still haven't found a workaround for this.


The [FSharp.Data](http://fsharp.github.com/FSharp.Data/) version of the World Bank type provider (and this also applies to the CSV, XML, JSON and Freebase type providers) uses interfaces and constructors to hide the unwanted members, so you don't see any `_Something` method.

## Splitting into two assemblies ##

When the type provider starts to get bigger, you might want to separate the runtime and design-time components into two separate assemblies. You also need to do this when you want to support multiple .NET framework profiles (like Silverlight or Portable Class Libraries), where you can have a single design-time assembly, but a different runtime assembly for each profile.

In this scenario, the design-time assembly can't just reference the runtime assembly, because there are multiple versions of them. And even if you're not supporting multiple profiles, the design-time assemblies of type providers are loaded inside the F# compiler process, which makes any assembly dependency for the design-time component to not work correctly. To work around this, usually there's no dependency between the design-time assembly and the runtime assembly, and instead we include all the files of the runtime assembly directly in the design-time assembly. This may increase the size of the design-time component, but it's not a big deal because that dll doesn't need to be included in the final application.

But with this solution, comes a big problem: your quotations will stop working. Why? Because you're referencing the types of the wrong assembly. To overcome this, the `TypeProviderConfig` that is passed to your type provider has a `RuntimeAssembly` property which allows you to load the runtime component using `Assembly.LoadFrom`, get the types you need using `Assembly.GetType`, and construct your expressions manually from that. Most of the samples in the [F# 3.0 Sample Pack](http://fsharp3sample.codeplex.com/) use this. Great, problem solved, right?

Back in January, I started adding support for portable class libraries in [FSharp.Data](http://fsharp.github.com/FSharp.Data/) because I wanted to use it on some Windows Phone and Windows 8 apps. After changing a couple of places from quotations to manually creating expression and using the reflection API, I though to myself: 

> "No! The code was nice and clean with type safe quotations, and now I'm turning it to this mess with error prone reflection calls using a stringly typed api. It's a huge step back. There has to be a better way."

And eventually after a couple of failed attempts and a lot of discussion on the github issue, I finally arrived at a solution: process the quotations to fix the type references to the right assembly. This is done by `AssemblyReplacer`, which you can find [here](https://github.com/fsharp/FSharp.Data/blob/master/src/Providers/AssemblyReplacer.fs). It's pretty much independent of the rest of [FSharp.Data](http://fsharp.github.com/FSharp.Data/), so you can just copy it over to your type provider project if you need it.

There are places where even without splitting the type provider into a design-time and runtime assembly, it isn't possible to use quotations, and you still need to keep creating expressions manually. That only happens when you use generics parameterized by a type generated by the type provider. For those, cases [Tomas](http://tomasp.net/) has created a nice dynamic operator (?) implementation in `QuotationBuilder` which is available [here](https://github.com/fsharp/FSharp.Data/blob/master/src/Providers/QuotationBuilder.fs).

## Coming Next ##

* Debugging type providers
* Supporting multiple .NET framework profiles
* Common errors found during type provider development
