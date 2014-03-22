---
layout: post
title: "Maintaining backwards compatibility when changing APIs in F#"
date: 2014-03-22 18:41
comments: true
categories: [F#]
---

### Classes

By now, mostly everyone already knows how to change the internal representation of a class without breaking compatibility. In C#:

{% codeblock lang:csharp %}
// Before
class Rectangle {

    public Point TopLeft { get; private set; }
    public Point BottomRight { get; private set; }

    public Rectangle(Point topLeft, Point bottomRight) {
        TopLeft = topLeft;
        BottomRight = bottomRight;
    }
}

// After
class Rectangle {

    public Point TopLeft { get; private set; }
    public Point Size { get; private set; }

    public Rectangle(Point topLeft, Size size) {
        TopLeft = topLeft;
        Size = size;
    }

    // for backwards compatibility:
    public Rectangle(Point topLeft, Point bottomRight) {
        TopLeft = topLeft;
        Size = new Size(bottomRight.X - topLeft.X, bottomRight.Y - topLeft.Y);  
    }

    // for backwards compatibility:
    public Point BottomRight {
        get { return new Point(TopLeft.X + Size.Width, TopLeft.Y + Size.Height); }  
    }
}
{% endcodeblock %}

and in F#:

{% codeblock lang:fsharp %}
// Before
type Rectangle(topLeft:Point, bottomRight:Point) =

    member __.TopLeft = topLeft
    member __.BottomRight = bottomRight

// After
type Rectangle(topLeft:Point, size:Size) =

    member __.TopLeft = topLeft
    member __.Size = size

    // for backwards compatibility:
    new (topLeft:Point, bottomRight:Point) = 
        Rectangle(topLeft, Point(bottomRight.X - topLeft.X, bottomRight.Y - topLeft.Y))

    // for backwards compatibility:
    member __.BottomRight =
        Point(topLeft.X + size.Width, topLeft.Y + size.Height)
{% endcodeblock %}

### Records

When using record types, things are a little bit more tricky, though:

{% codeblock lang:fsharp %}
// Before
type Rectangle = 
    { TopLeft : Point 
      BottomRight : Point }

// After
type Rectangle = 
    { TopLeft : Point 
      Size : Size }
    member x.BottomRight =
        Point(x.TopLeft.X + x.Size.Width, x.TopLeft.Y + x.Size.Height)
{% endcodeblock %}

This change will keep working at the usages of Rectangle, but not at creation, as there isn't a way (at least that I know of) to make something like `{ TopLeft = Point(2.5, 3.0); BottomRight = Point(4.0, 7.5) }` keep working. This can be avoided if we originally had hidden the fact that Rectangle was representation as a record, like this:

{% codeblock lang:fsharp %}
type Rectangle = 
    private { TL : Point 
              BR : Point }
    member x.TopLeft = x.TL
    member x.BottomRight = x.BR
    static member Create(topLeft, bottomRight) = 
        { TL = topLeft; BR = bottomRight }
{% endcodeblock %}

But for me this seems to defeat some of the advantages of using a record in the first place. You still get structural equality and pretty printing for free, which classes don't have, but you lose the nice pattern matching and type inference capabilities that usually come with records.

### Single case discriminated unions

The situation for discriminated unions is better, though. For each changed case, you can use one active pattern and a function to maintain backward compatibility:

{% codeblock lang:fsharp %}
// Before
type Rectangle = 
    | Rectange of topLeft:Point * bottomRight:Point

// After
type Rectangle = 
    | Rectangle2 of topLeft:Point * size:Size

// for backwards compatibility:
let (|Rectangle|) x =
    match x with
    | Rectangle2(topLeft, size) -> topLeft, Point(topLeft.X + size.Width, topLeft.Y + size.Height)

// for backwards compatibility:
let Rectangle(topLeft:Point, bottomRight:Point) = 
    Rectangle2(topLeft, Size(bottomRight.X - topLeft.X, bottomRight.Y - topLeft.Y))
{% endcodeblock %}

The downside is that the new discriminated union case has to have a new name, otherwise the function for backward compatibility will shadow the case constructor. This might not necessarily be a bad thing, though.

We could also make the discriminated union representation private, like we did for the record:

{% codeblock lang:fsharp %}
type Rectangle = 
    private | Rectangle of topLeft:Point * bottomRight:Point
    member x.TopLeft = 
        match x with
        | Rectangle(topLeft, _) -> topLeft
    member x.BottomRight = 
        match x with
        | Rectangle(_, bottomRight) -> bottomRight
    static member Create(topLeft, bottomRight) = 
        Rectangle(topLeft, bottomRight)
{% endcodeblock %}

When we're deciding between using a record or a single case discriminated union, these considerations might help decide.

### Multiple case discriminated unions

When we have discriminated unions with multiple cases, the same technique also applies, but now we need to use partial active patterns instead (`(|Pattern|_|)` returning an option instead of `(|Pattern|)` returning the type directly). Here's an example:

{% codeblock lang:fsharp %}
// Before
[<RequireQualifiedAccess>]
type JsonValue =
    | String of string
    | Number of decimal
    | Float of float 
    | Object of Map<string, JsonValue>
    | Array of JsonValue[]
    | Boolean of bool
    | Null

// After
[<RequireQualifiedAccess>]
type JsonValue =
    | String of string
    | Number of decimal
    | Float of float 
    | Record of properties:(string*JsonValue)[]
    | Array of elements:JsonValue[]
    | Boolean of bool
    | Null

[<CompilationRepresentation(CompilationRepresentationFlags.ModuleSuffix)>]
module JsonValue =

    [<Obsolete("Please use JsonValue.Record instead")>]
    let (|Object|_|) x =
        match x with 
        | JsonValue.Record properties -> Map.ofArray properties |> Some
        | _ -> None

    [<Obsolete("Please use JsonValue.Record instead")>]
    let Object = Map.toArray >> JsonValue.Record 
{% endcodeblock %}

In this example, because `[<RequireQualifiedAccess>]` was being used, we have to include the active pattern and the compatibility function in a module, and use `[<CompilationRepresentation(CompilationRepresentationFlags.ModuleSuffix)>]` to be able to maintain the exact same syntax. We also marked the backward compatibility constructs with the `Obsolete` attribute because we don't want new users to be using the old discriminated union case. This attribute will make sure the old case doesn't show up in IntelliSense, and will also issue a warning to old users.

There's one extra caveat, though. Consider the following code:

{% codeblock lang:fsharp %}
let f x =
    match x with
    | String s -> ...
    | Number n -> ...
    | Float f -> ... 
    | Object properties -> ...
    | Array a -> ...
    | Boolean b -> ...
    | Null -> ...
{% endcodeblock %}

Even though we're handling all the cases, we get a warning stating **this pattern-matching is not exhaustive**. This is because the compiler doesn't know what `Object` will match on: it's a custom defined partial active pattern, so the implementation could match on multiple cases or even none at all.

### Final thoughts

In all of the above we were only concerned with source compatibility. Binary compatibility has other concerns that would require further exploration.
