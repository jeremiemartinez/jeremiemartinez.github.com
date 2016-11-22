---
layout: post
title: "Story of my metro pass"
comments: true
---

I have a fun story to tell from last night that makes so much sense in Android development that I thought I should share it.

<!-- more -->

Last night, I went home following my little routine and my usual metro line. When I went down to my stop, I was stopped by a person from the metro company ([RATP](https://en.wikipedia.org/wiki/RATP_Group)) asking for my metro pass. Actually, she was not interested in controlling my metro pass but rather by its version. Indeed, RATP will stop supporting an old version of the card and its users will be forced to upgrade in the coming months. Six million users will have to verify that they have the right version. Every single one of them: old people, young people, busy people, forgetful people, every one.

To solve this problem, RATP decided to randomly stop people during the coming months and make them aware that an update was about to come and encourage them to go to desk and to queue in order to get the new version of their metro pass. What a mess! If only they could update cards directly… Oh wait…

This is exactly what happens with an Android application. The Android app is the metro pass and the server is the card machine. Nothing will force your users to upgrade your app to avoid the stupid bug you wish never existed. And let's be clear: forcing users to upgrade or uninstall/reinstall is an experience as awful as forcing a user to queue at the counter on a busy Monday morning.

I am often asked at conferences why we are so careful with our releases. Even our backend developers often asked why we pay so much attention to API, to payloads and so on… The answer is always the same: once the application is released, we don't own it anymore. We will have to maintain it as long as necessary.

A bad and quickly designed API or a stupid bug are not excuses your user can hear. Every release has to be maintained, so be careful.

I love this quote from Joshua Block:

> APIs, like diamonds, are forever

Well, your old app versions, like diamonds, are forever.
