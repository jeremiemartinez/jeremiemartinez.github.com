---
layout: post
title: "DevOps on Android: From one Git push to production"
comments: true
---

DevOps is a well known movement whose main objective is to automate software delivery. Indeed, DevOps aims at continuous testing, code quality, feature development and easier maintenance releases. Therefore, one of DevOps final goal is for developers to execute fast, reliable and automated release, ideally without any human involved during the process. It is called continuous delivery. I wrote this article to demonstrate that we can now achieve such goal on Android too and to share my thoughts and feedback about it.

<!-- more -->

## Continuous Integration as a starting point

To get continuous delivery, having a strong continuous integration is mandatory. It has been around Android environment for some time now, but to be clear let's see a reminder.

First of all, every Android application should have a continuous integration. Yes it is said. Indeed, it provides several benefits that cannot be ignored when building an app nowadays. In my opinion, its best advantages are:

* Build automation: no more : “but it builds on my machine“. It must build everywhere.
* Fail early: building after each push guarantees we break as early as possible.
* Testing each build: make sure our tests are always run.
* Constant packaging: prevent human mistakes when packaging our binary.
* Faster release: since we can be confident in each build, release becomes easier.
* Increase confidence: finally, we trust our code, our process and we reduce bad surprises.

### A classic continuous integration process

First, we need an integration server like Jenkins or Travis. The following jobs are my standard configuration:

* A job that will be launched by watching if a push on our source repository (Git, SVN, …) has been done. It watches dev branch and will compile our code, run unit tests and package our debug APK.
* A job that runs once the first job succeed. It will run integration tests (with Espresso or Robotium) to ensure user experience by recreating scenarios and checking graphical content. You can use connected devices (difficult when your CI server is not easily reachable), Genymotion or the brand new built-in emulators that comes with Android Studio 2.0 (check it out!).
* A job that will run measurement on our code (for instance with Sonarqube) in order to monitor code quality. I run this job every midnight for instance.
* Finally, a job that will run once we push our master/release branch. It will compile our code and generate the release APK.

Et voila! As you can see, it is pretty simple and ensure all advantages I enumerated before.

### Testing is the key

I wrote an article about [testing on Android](http://jeremie-martinez.com/2015/04/17/tests-android/). Testing is really important because it is our only tool to prove automatically that our app is working as it should. There are a lot of tools to help us writing great tests, so choose wisely.

Also, be pragmatic about the libraries you choose to integrate in your app. Indeed, you know it will be easier to test your app when your library have a good test coverage. They have thought about how to test it properly and drives their development by tests (IMO, [OkHttp](https://github.com/square/okhttp) and [Retrofit](https://github.com/square/retrofit) are great examples). It is likely you will be able to test it while using it.

Finally, libraries like [Dagger](https://github.com/google/dagger) can help you increase testability. Indeed, it will oblige you to follow the Single Responsibility Principle and separate correctly your code, hence easier testing.

Once you have a strong continuous integration, let's see how to level up.

## Continuous delivery: Level++

For instance at Captain Train, we release every [6 weeks](http://cyrilmottier.com/2014/12/09/a-story-of-software-development-methodologies/) and we are very careful about it. Currently:

* We have a beta phase.
* We support 4 locales.
* We support 3 types of device (phone, 7 and 9 inches tablets).
* We always check our Play Store listings.
* We create release notes.
* We use the rollout feature.
* We upload our 72 screenshots (6 screenshots * 4 locales * 3 types)
* We have a wear companion.

It takes time. A lot. Lately, we decided that it was time to try to automate this process. Even if our main goal was to reduce the time required to release. It would be great if we could also prevent human mistakes and be consistent release to release. It also gives us a great responsibility since developers control the whole release process. Indeed, marketing/communication teams would have to see with us how they can integrate their changes in our release.

But let's be honest… On Android, developers does not control everything, Google does. However, it exposes a HTTP API to enable developer to interact easily with the Play Store console. They also provide clients for various languages such as Java (of course!), Python, Ruby, etc…

On this article, I will focus on the Java client since this is probably the easiest access for Android developer.

### Code your own publisher

Let's see how we can code our own custom Play Store publisher. It's a two step process: first, we will configure our console to enable our client to operate, then we will discover the API. The most difficult thing as often with Google is to configure... Documentation can be found [here](https://developers.google.com/android-publisher/#publishing). Be careful, it is a bit outdated.

#### Configuration

First of all, if it is not already done, you must create a new project in the Google [console](https://console.developers.google.com/apis). Then we need to enable `Google Play Android Developer API`.

<center>![Enable API in console]({{ site.baseurl }}public/images/publisher_api_console.png)</center>

Then, we must create a credential of type `Service account key`:

<center>![Create service account key]({{ site.baseurl }}public/images/publisher_credential_console.png)</center>

Then, fill up the tiny form and download your credential as a JSON file. You need to save three values : `private_key_id`, `private_key` and `client_email`. Save the value `private_key` in its own file `secret.pem`.

Once its done, let's go to the second console! \o/

Connect to your Play Store [console](https://play.google.com/apps/publish). You must go to `Settings` > `API access`:

<center>![Enable API access]({{ site.baseurl }}public/images/publisher_playstore_console.png)</center>

Then you must simply link your project. Finally in `Service accounts`, grant rights to the email you had under `client_email` in the JSON file.

Aaaaand it's done. You are all setup!

#### API discovery

Let's now dive into the API through the Java client. We create a separate Java project to our publisher and we add the following dependency (available on [maven central](http://search.maven.org/#artifactdetails%7Ccom.google.apis%7Cgoogle-api-services-androidpublisher%7Cv2-rev20-1.21.0%7Cjar)):

{% highlight groovy %}
compile 'com.google.apis:google-api-services-androidpublisher:
         v2-rev20-1.21.0'
{% endhighlight %}

The next step is to create a new `AndroidPublisher`. First, we instantiate a `GoogleCredential` by giving a transport client, a JSON factory, the private key ID that corresponds to the `private_key_id`, an account ID that corresponds to the `client_email`, the `ANDROIDPUBLISHER` scope and finally the key file containing your private key and corresponding to `private_key`. Yes, all of this.

Then, we can create an `AndroidPublisher` instance with our application package.

{% highlight java %}
http = GoogleNetHttpTransport.newTrustedTransport();
json = JacksonFactory.getDefaultInstance();

Set<String> scopes =
      Collections.singleton(AndroidPublisherScopes.ANDROIDPUBLISHER);

GoogleCredential credential = new GoogleCredential.Builder().
                setTransport(http).
                setJsonFactory(json).
                setServiceAccountPrivateKeyId(KEY_ID).
                setServiceAccountId(SERVICE_ACCOUNT_EMAIL).
                setServiceAccountScopes(scopes).
                setServiceAccountPrivateKeyFromPemFile(secretFile).
                build();

publisher = new AndroidPublisher.Builder(http, json, credential).
                setApplicationName(PACKAGE).
                build();
{% endhighlight %}

The `AndroidPublisher` is the main entry to the Google API. It has a `edits` method that will allow us to change the data we want from the console.

To start a new edition, you must start with an `insert` request and store its `id` that you will use in every next call.

{% highlight java %}
AndroidPublisher.Edits edits = publisher.edits();

AppEdit edit = edits.insert(PACKAGE, null).execute();
String id = edit.getId();
{% endhighlight %}

Now, we can start to change our console data. For instance, if you want to change your listings:
{% highlight java %}
Listings listings = edits.listings();

Listing listing = new Listing().
                        setFullDescription(description).
                        setShortDescription(shortDescription).
                        setTitle(title);

listings.update(PACKAGE, id, "en_US", listing).execute();
{% endhighlight %}

You can also upload screenshots:
{% highlight java %}
Images images = edits.images();

FileContent content = new FileContent(PNG_MIME_TYPE, file);

images.upload(PACKAGE, id, "en_US", "phone5", content).execute();
{% endhighlight %}

Last example, let's upload an APK:
{% highlight java %}
// APK upload
Apks apks = edits.apks();
FileContent apkContent = new FileContent(APK_MIME_TYPE, apkFile);
Apk apk = apks.upload(PACKAGE, id, apkContent).execute();
int version = apk.getVersionCode();

// Assign APK to Track
Tracks tracks = edits.tracks();
List<Integer> versions = Collections.singletonList(version)
Track track = new Track().setVersionCodes(versions);
tracks.update(PACKAGE, id, "production", track).execute();

// Update APK listing
Apklistings apklistings = edits.apklistings();
ApkListing whatsnew = new ApkListing().setRecentChanges(changes);
apklistings.update(PACKAGE, id, version, "en_US", whatsnew).execute();
{% endhighlight %}

I encourage you to explore the [API](https://developers.google.com/android-publisher/api-ref/). It is very powerful and pretty straightforward.

Last step, you must commit your edition. Indeed, so far Google recorded every change you requested but they will only be saved once you committed the edition. I also advice you to validate your change before attempting to commit.

{% highlight java %}
edits.validate(PACKAGE, id).execute();
edits.commit(PACKAGE, id).execute();
{% endhighlight %}

As you can see, the `id` you retrieve at the beginning of our code can be considered as a transaction `id`. You start your transaction by calling `insert`, `update`/`upload` your changes, `validate` and finally `commit`. Pretty simple.

## Conclusion

It is now possible to follow the DevOps movement and have a strong continuous delivery even for Android App. At Captain Train, we chose to write our own publisher tool to ensure that we control every step and byte of this important step. We execute it as a script once our release job succeeded. However, it also exists a Jenkins [plugin](https://github.com/jenkinsci/google-play-android-publisher-plugin) or a Gradle [plugin](https://github.com/Triple-T/gradle-play-publisher) that can handle that for you. Just be sure to understand what is going on under the hood. You are dealing with production after all. It must be used with care.

Such tools and processes can allows you to be able to release a new version in production by simply pushing the master branch. It is easy, fast, reliable and time saving.

It is obvious that what work in one project, will not work for every team/company/app. It depends a lot on your team and their relation to communication/marketing team. However, I would say to conclude that continuous delivery should be a goal for every developers team and every step that brings you closer to that point is a win.
