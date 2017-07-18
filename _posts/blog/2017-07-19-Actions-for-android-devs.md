---
title: "Google Assistant Development for Android Devs (Part 1)"
author: Patrick Jackson
categories: Development
tags: [Google Assistant, Actions on Google, conversational interfaces, Android, Kotlin]
---


# Google Assistant Development for Android Devs (Part 1)

![](https://storage.googleapis.com/banjotms.appspot.com/assistant_surfaces.svg)

There is a lot of buzz and excitment around the [Google Assistant Actions](https://developers.google.com/actions/), and with good reason:  this is an entirely new platform for building experiences for a number of devices.  Building a conversational Action with Google Assistant means it will be available on Android, iOS, Google Home, and others as they become available.


For the uninitiated, Actions on Google are third-party apps that can be built using Google's APIs.  Actions extend what the Google Assistant can do by letting user talk to them.  They can be accessed with trigger words such as 'Ok Google, talk to Ticketmaster', or 'Ok Google, ask ticketmaster find Yankee's games this weekend'. (Ticketmaster Action coming soon!)

<img src="https://storage.googleapis.com/banjotms.appspot.com/Screenshot_20170718-103901.png" width="300"> 

At Ticketmaster Mobile Studio, we decided to use the languages and dev environment we are most familiar with - Kotlin/Java, IntelliJ, & Gradle.  This allows us to use the language and tools we are comfortable with, and to reuse some code with Android apps.  Unfortunately, the documentation and examples are only in nodejsdo not provide any Java/Kotlin.

Looking through the json specs and creating response objects to match is pretty tedious, so we have decided to open source an [unofficial Kotlin/Java Actions-on-google SDK](https://github.com/TicketmasterMobileStudio/actions-on-google-kotlin).  This is an early release and will we be adding functionality, with a goal of matching the official SDK.  Currently, all the conversation components are supported.  Using the Kotlin SDK you can get up and running in no time.  And since Kotlin has great interop with Java, you can use it from Java as well.

 
Over the next few weeks, I'll be posting about our experience building a Action from the perspective of an Android developer, as well as some updates on the [Actions on Google Kotlin SDK.](https://github.com/TicketmasterMobileStudio/actions-on-google-kotlin)
