---
layout: post
title: "Importing Messages from Windows Phone to Android"
date: 2015-12-20 01:05
comments: true
categories: [F#, Type Providers, Windows Phone, Android]
---

Searching around the web I couldn't find any way to to transfer messages from Windows Phone to Android that actually worked.

I could export all the data from Windows Phone using the [contacts+message backup app](https://www.microsoft.com/en-gb/store/apps/contacts-message-backup/9nblgggz57gm), but there was no Android app that understood that .msg format.

However, I found the [SMS Backup & Restore app](https://play.google.com/store/apps/details?id=com.riteshsahu.SMSBackupRestore&hl=en) that seemed to work well for exporting/importing messages between Android devices.

So with a bit of trial and error and the help of the [XML Type Provider](http://fsharp.github.io/FSharp.Data/library/XmlProvider.html) I managed to create an F# script to convert from one format to another. 

The trickier part was trying to figure out what DateTime format was being used in both cases (thanks [Lincoln](https://twitter.com/LincolnAtkinson/status/678314345508433921)), all the rest was pretty straightforward. The .msg format has a lot less fields than the .xml format used by the Android app, but it ended up still working ok with those missing fields. 

It converts both SMS and MMS, though it only retains the text and photos from the MMS's, vCards, embedded location maps and other items are skipped. but it should be fairly easy to add that support if needed.

Here is the code:

<script src="https://gist.github.com/ovatsus/2dd82172dcd8308d559c.js"></script>