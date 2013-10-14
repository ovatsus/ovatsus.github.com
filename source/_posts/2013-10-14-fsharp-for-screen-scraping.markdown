---
layout: post
title: "F# for Screen Scraping"
date: 2013-10-14 23:50
comments: true
categories: [F#, Screen Scraping, HtmlAgilityPack]
---

One of the many things F# is great for is screen scraping. Here's why:

* Downloading multiple pages asynchronously and in parallel is trivial with F#'s async support 
* Navigating the HTML DOM is a great fit for higher order data processing combined with partial application
* F# Interactive really shines in iterative processes like this, where you try something out, see it didn't work quite well, and keep adjusting until you get it right. Doing a full compile-run cycle on each iteration instead of simply evaluating in the REPL would make this task take much longer 

[Html Agility Pack](http://htmlagilitypack.codeplex.com/) is the obvious candidate to use for screen scraping in .NET, but like other LINQ-like libraries that rely heavily on extension methods, its API isn't ideal for use in F#. A simple wrapper will take care of that problem: 

{% codeblock lang:fsharp An F# wrapper for HtmlAgilityPack - HtmlAgilityPack.FSharp.fs %} 
module HtmlAgilityPack.FSharp

open HtmlAgilityPack

type HtmlNode with 
    
    member x.FollowingSibling name = 
        let sibling = x.NextSibling
        if sibling = null then
            null
        elif sibling.Name = name then
            sibling
        else 
            sibling.FollowingSibling name
    
    member x.FollowingSiblings name = seq {
        let sibling = x.NextSibling
        if sibling <> null then
            if sibling.Name = name then
                yield sibling
            yield! sibling.FollowingSiblings name
    }

    member x.PrecedingSibling name = 
        let sibling = x.PreviousSibling
        if sibling = null then
            null
        elif sibling.Name = name then
            sibling
        else 
            sibling.PrecedingSibling name
    
    member x.PrecedingSiblings name = seq {
        let sibling = x.PreviousSibling
        if sibling <> null then
            if sibling.Name = name then
                yield sibling
            yield! sibling.PrecedingSiblings name
    }

let parent (node : HtmlNode) = 
    node.ParentNode

let element name (node : HtmlNode) = 
    node.Element name

let elements name (node : HtmlNode) = 
    node.Elements name

let descendants name (node : HtmlNode) = 
    node.Descendants name

let descendantsAndSelf name (node : HtmlNode) = 
    node.DescendantsAndSelf name

let ancestors name (node : HtmlNode) = 
    node.Ancestors name

let ancestorsAndSelf name (node : HtmlNode) = 
    node.AncestorsAndSelf name

let followingSibling name (node : HtmlNode) = 
    node.FollowingSibling name

let followingSiblings name (node : HtmlNode) = 
    node.FollowingSiblings name

let precedingSibling name (node : HtmlNode) = 
    node.PrecedingSibling name

let precedingSiblings name (node : HtmlNode) = 
    node.PrecedingSiblings name

let inline innerText (node : HtmlNode) = 
    node.InnerText

let inline attr name (node : HtmlNode) = 
    node.GetAttributeValue(name, "")

let inline (?) (node : HtmlNode) name = 
    attr name node

let inline hasAttr name value node = 
    attr name node = value

let inline hasId value node = 
    hasAttr "id" value node

let inline hasClass value node = 
    hasAttr "class" value node

let inline hasText value (node : HtmlNode) = 
    node.InnerText = value

let createDoc html =
    let doc = new HtmlDocument()
    doc.LoadHtml html
    doc.DocumentNode
{% endcodeblock %}

When we leave the "target" of an operation as the last parameter of a function, we can leverage partial application to remove the need to create anonymous lambdas when using higher order functions such as the ones needed for DOM traversing.

In this case, by taking the `this` parameter of the C# methods and putting it as the last parameter of our wrapper functions `elements`, `descendants`, `innerText`, etc., our code is now much easier to read and to understand than if we used the Html Agility Pack methods directly.

As an example, here is a small script that I used when developing one of my Windows Phone apps to get the list if all the rail stations in Ireland together with their coordinates:
 
{% codeblock lang:fsharp IrelandRailStations.fsx %} 
#r "System.Net"
#r "../lib/portable/FSharp.Data.dll"
#r "../packages/HtmlAgilityPack-PCL.1.4.6/lib/HtmlAgilityPack-PCL.dll"

#load "HtmlAgilityPack.FSharp.fs"

// get a page that lists the stations that start with firstLetter
let getStationListPage firstLetter = 
    "http://www.irishrail.ie/cat_stations_list.jsp?letter=" + (string firstLetter)
    |> Http.AsyncRequestString

// get all the links to stations inside the <ul class="results">
let getStations stationListPage =
    stationListPage
    |> createDoc
    |> descendants "ul"
    |> Seq.filter (hasClass "results")
    |> Seq.head
    |> descendants "a"
    |> Seq.map (attr "href")
    |> Seq.toArray

// get the page for a station
let getStationPage station =
    "http://www.irishrail.ie/" + station
    |> Http.AsyncRequestString

// get the latitude and longitude of a station from the google maps link in the station page
let getCoordinates stationPage = 
    let googleMapsLink = 
        stationPage
        |> createDoc
        |> descendants "div"
        |> Seq.filter (hasId "map-ordnance")
        |> Seq.head
        |> followingSibling "ul"
        |> descendants "a"
        |> Seq.head
        |> attr "href"
    let split (c:char) (s:string) = s.Split c
    let [| "ll" ; coords |] =        
        Uri(googleMapsLink).Query
        |> split '&'
        |> Seq.map (split '=')
        |> Seq.filter (Seq.head >> (=) "ll")
        |> Seq.head
    let [| lat; long |] = coords |> split ',' |> Array.map float
    lat, long

let stationsAndCoords =
    let stations = 
        ['A'..'Z'] 
        |> Seq.map getStationListPage
        |> Async.Parallel
        |> Async.RunSynchronously
        |> Array.collect getStations
    let lat, long = 
        stations
        |> Seq.map getStationPage
        |> Async.Parallel
        |> Async.RunSynchronously
        |> Array.map getCoordinates
        |> Array.unzip
    let stations = 
        stations
        |> Array.map Uri.UnescapeDataString
    Array.zip3 stations lat long
{% endcodeblock %}

Output:

{% codeblock lang:fsharp %}
val stationsAndCoords : (string * float * float) [] =
  [|("Adamstown", 53.335285, -6.4523325); ("Ardrahan", 53.157177, -8.814831);
    ("Arklow", 52.793163, -6.159939); ("Ashtown", 53.37552, -6.3313503);
    ("Athenry", 53.30153, -8.748547); ("Athlone", 53.427322, -7.9368286);
    ("Athy", 52.991997, -6.976196); ("Attymon", 53.321205, -8.606078);
	...|]
{% endcodeblock %}

You can also see that the same pattern was used for making the `String.Split` function play well with partial application.

Another neat feature of F# for scripting (I wouldn't recommend incorporating it in production code), is the ability to de-structure arrays in one liners, as done in `let [| "ll" ; coords |] =` and `let [| lat; long |] =`. The compiler will emit a warning saying that the match is not exhaustive, telling us that this might backfire if there are less than two elements in the array, but for the purpose of a one shot script to download some data it's fine.

And to give a second example, here's a snippet from my Learn On The Go app that processes the html of the lecture videos page page of a Coursera course:

{% codeblock lang:fsharp %}
let lectureSections = 
    let trimAndUnescape (text:string) = text.Replace("&nbsp;", "").Trim().Replace("&amp;", "&").Replace("&quot;", "\"").Replace("apos;", "'").Replace("&lt;", "<").Replace("&gt;", ">")
    let endsWith suffix (text:string) = text.EndsWith suffix
    createDoc lecturesHtmlStr
    |> descendants "h3"
    |> Seq.map (fun h3 ->
        let title = h3 |> innerText |> trimAndUnescape
        let completed = h3 |> parent |> hasClass "course-item-list-header contracted"
        let ul = 
            h3 
            |> parent
            |> followingSibling "ul"
        ul, title, completed)
    |> Seq.filter (fun (ul, _, _) -> ul <> null)
    |> Seq.map (fun (ul, title, completed) -> 
        let lectures =
            ul
            |> elements "li"
            |> Seq.map (element "a")
            |> Seq.map (fun a ->
                let id = a |> attr "data-lecture-id" |> int
                let title = innerText a |> trimAndUnescape
                let quizAttemptedSpan = a |> elements "span" |> Seq.tryFind (hasClass "label label-success")
                let title, quizAttempted =
                    match quizAttemptedSpan with
                    | Some span ->
                        title.Replace(trimAndUnescape span.InnerText, "").Trim(), true
                    | None -> title, false
                let videoUrl = a |> attr "data-modal-iframe" 
                let lectureNotesUrl = 
                    let urls = a |> followingSibling "div" 
                                 |> elements "a" 
                                 |> Seq.map (attr "href") 
                    match Seq.tryFind (endsWith ".pdf") urls with
                    | Some url -> url
                    | _ -> ""
                let viewed = a |> parent |> hasClass "viewed"
                { Id = id
                  Title = title
                  VideoUrl = videoUrl
                  LectureNotesUrl = lectureNotesUrl
                  Viewed = viewed 
                  QuizAttempted = quizAttempted })
            |> Seq.toArray
        { Title = title
          Completed = completed
          Lectures = lectures })
    |> Seq.toArray 
{% endcodeblock %}
