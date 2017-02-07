---
layout: post
title: "Firebase vs permissions"
comments: true
---

Two weeks ago for our last Android version, we release a new feature with Firebase : [Personal App Indexing](https://firebase.google.com/docs/app-indexing/android/personal-content). It was the first time we release a feature based on Firebase in our [Trainline application](https://play.google.com/store/apps/details?id=com.capitainetrain.android). Indeed, we were still using Play Services because we did not have a good reason to migrate yet. Since Personal App Indexing was only available with Firebase, no choice!

**Spoiler**: we had problems with permissions.

<!-- more -->

After a beta phase and a roll out phase, we started to see users complaining about permissions. It was very strange since we did not change anything. We even check any change related to permissions with an unit test to avoid surprises on release day.

After a few investigation, users were seeing a notification and then a dialog:


![Firebase Bug]({{ site.baseurl }}public/images/firebase_permissions_bug.png){: .center-image }


We were once again very surprised because we had always been careful about making the app work without the Play Services.

Playing with other apps, it appears we were not alone: Trello, Maps, Keep or Evernote had the same problem. All of them also implemented Personal App Indexing.

We then figured out this notification was displayed because these users were denying some permissions to Play Services:


![Play Services Permissions]({{ site.baseurl }}public/images/permissions_play_services.png){: .center-image }


**First conclusion**: Play Services don't seem to work if they don't have all their permissions granted. Even if the feature you are using does not rely on them. Crazy.

How can we know that all permissions were granted to propose the feature only in that case?

In fact, this information is available with the Play Services. When you build a new `GoogleApiClient` and connect to it, if a permission is missing we get a `SERVICE_MISSING_PERMISSION` in the `ConnectionResult`:

{% highlight java %}
final ConnectionResult connectionResult = new GoogleApiClient.Builder(this).
                addApi(AppIndex.API).
                build().
                blockingConnect(3, TimeUnit.SECONDS);
        if (connectionResult.getErrorCode() == ConnectionResult.SERVICE_MISSING_PERMISSION) {
            return;
        }
{% endhighlight %}

**Second conclusion**: Firebase API is simpler , they handle Play Services connection for you but their implementation does not check for missing permission.

Therefore, our only way to solve the problem was to find out how can we retrieve this information with Firebase?

**Third conclusion**: You can't.

We solve the problem by using the Play Services to check for the `SERVICE_MISSING_PERMISSION` and if everything was alright, then we use the Firebase Personal App Indexing feature. Otherwise, we don't.

This story ends up with us releasing a hotfix and 10 lines of code I am not proud of. Hopefully this article could help others.

**Final conclusion**: be very careful when integrating third party libraries.

Firebase and Play Services version: 10.0.1
