---
layout: post
title: Talk on Android network stack
comments: false
---

I recently gave a talk to my *backend* colleagues about the network stack we use on Android. This stack uses [okio](http://square.github.io/okio), [OkHttp](http://square.github.io/okhttp) and [Retrofit](http://square.github.io/retrofit). Slides are in French and available on [Speaker Deck](https://speakerdeck.com/jeremiemartinez/la-stack-reseau-android-disponible-egalement-pour-vos-backs). Code from live coding is available on [Github](https://github.com/jeremiemartinez/okhttp_stack_talk). The idea between this talk was to show how much powerful OkHttp/Retrofit are and how stupid it would be to not use it when backend server needs to consume other backend APIs. We, on Android, are obviously experts on this subject and therefore have great tools at your disposal.

<!-- more -->

<script async class="speakerdeck-embed" data-id="1be98ff29645405f83b457b6bcdf8ccf" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>
