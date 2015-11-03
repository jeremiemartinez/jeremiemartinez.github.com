---
layout: post
title: No more excuses, Android testing is possible
comments: true
---

*This article is a translation of an article I wrote in French and published on [Xebia blog](http://blog.xebia.fr/2015/04/17/plus-dexcuses-les-tests-en-android-cest-possible/)*

In the Android community , access to samples , tutorials , articles on the latest widget or library is commonplace. However , it is very difficult to get good articles or documentation on the development of an architecture for testing an Android application.

We often hear :
> Testing Android is a real headache !

Or
> That is just impossible !

The purpose of this article is to show you that in 2015, it is possible to easily and effectively test an Android application. Be careful, you will have no more excuses…

<!-- more -->

## History and misconceptions


When we look at the documentation in the Android SDK for testing, there is the `ActivityInstrumentationTestCase2` class. It helps writing tests running directly on an Android device.

> With the name of this class, hard to be reassured. A unit test Activity (UI)? On a device? Instrumentation? 2?

This situation gave too many misconceptions about writing Android tests:

**It necessarily requires a device connected for testing**: Well… no! A device is not always necessary but it remains interesting to run some tests on it directly in order to guarantee graphic behaviors. The main problem with running on a device is that they are not unit tests anymore. Their execution becomes very slow.

From these problems, Robolectric was born. This library aims to mock (shadowing) some of the Android framework to allow tests to run directly on a JVM by loading various classes required by Android Framework into the classloader. Second misconception: **Robolectric it's complicated, difficult to implement and there are shadows in all directions.** True, Robolectric took some time to stabilize their API, but since the release of version 2 (and 3.0 recently), clarity of API and simplicity of implementation has improved considerably.

**You cannot run the tests in the IDE**, which involves efficiency problems when writing of tests and debugging them. With the arrival of AndroidStudio and Gradle 1.1.0 plugin, unit tests are now supported directly in the IDE. Compilation, partial execution, debugging, everything is there.


## Unit testing vs functional testing


To understand this article, it is important to differentiate between functional testing and unit testing. Unit testing is a way to test independently a small piece of code containing the technical logic. In the contrary, functional testing will allow you to test user experience.

The real question is: **when should I do unit tests or functional tests ?**

It is important that your UT are not dependent on the application lifecycle. Ideally, they should be as detached as possible to the Android framework. They should be short, quick and bring a real profit. However, be careful to not test Android implementation, it has no interest at all. Robolectric should therefore be used sparingly. Testing Android services (e.g `TelephonyManager`, `ConnectivityManager`, …) is useless, but testing their use by mocking them makes a lot more sense.

Also your FT will allow you to integrate with the Android framework and thus fully test your user experience, check that the `Activities` are linked correctly and display the right information to the user. Therefore, it seems important to define with your entire team critical user experience paths that represent your application.


## Adopt an architecture adapted for testing


To make a testable application, we need to adopt a clean and modular architecture to facilitate testing of each party. The rest of this article will be based on an MVP architecture (Model-View-Presenter).

- **Model**: This is the data that will be displayed on screen. It can also be the supplier or the data source.
- **View**: It is the user interface that will display the data. It will also take care of achieving the right actions according to user interactions, such as a click for instance.
- **Presenter**: This is the middle man, the organizer. Depending on interactions received from the **View**, it will retrieve and display data from the **Model**.

One may wonder why MVP architecture is a real asset in writing Android tests. Well, it actually allows to manage the responsibilities defining by the roles. Therefore, by following these roles, we obtain a modular application where each class can be tested easily.

To better illustrate this architecture, nothing better than code : Installed Apps. The application is available on [Github](https://github.com/jeremiemartinez/installed_apps). The purpose of this Android application is very simple since it only displays a list of applications installed on your Android device. It also contains a field for filtering by name.

<center>![App screenshot]({{ site.baseurl }}public/images/testing_screenshot_app.png)</center>

The project contains only 4 classes:

- `App`: This class is the **Model**. It represents an installed application. It will be easily testable by unit testing.
- `AppsProvider`: This class provides the data, i.e the list of installed applications. It is difficult to test completely because it uses heavily the `PackageManager` from Android SDK. Therefore his behavior will be checked by functional testing.
- `AppsActivity`: This is our `Activity`, it represents our **View**. Its purpose is to display the data provided. It is obviously strongly coupled to the Android lifecycle and will also be checked by functional tests.
- `AppsPresenter`: This is our **Presenter**. Its purpose is to respond to user interactions (e.g filter) transmitted by `AppsActivity` and to react accordingly. This is an important class to test because it contains all the logic of the application. With the MVP architecture, it becomes easily testable by unit testing since it contains no direct link to the Android framework.


## So that's great but how do we write these tests?


Now to write our tests. The first step is to set up our build.gradle. You will find under this [link](https://gist.github.com/jeremiemartinez/e8cf07d11f5db8910587) a bootstrap build containing all the necessary dependencies and configurations.

### Unit tests
Let's start by writing our unit tests. First, you need to add to your `build.gradle`:

{% highlight groovy %}

    // UNIT TESTING
    testCompile(
            'junit:junit:4.11',
            'com.android.support:support-annotations:22.0.0',
            'com.squareup.assertj:assertj-android:1.0.0',
            'org.mockito:mockito-core:1.9.5',
            'org.assertj:assertj-core:1.7.0'
    )
    testCompile('org.robolectric:robolectric:2.4') {
        exclude group: 'commons-logging'
        exclude group: 'org.apache.httpcomponents'
    }
{% endhighlight %}

As we can see, there are dependencies on libraries used for classic testing: JUnit, Mockito, AssertJ and also to Robolectric.

We saw previously that only two classes seemed testable by unit testing: `App` and `AppsPresenter`. So we find them in the unit tests project(`src/test`) as `AppTest` and `AppsPresenterTest`. Note that `AppTest` uses a Robolectric runner via `@RunWith(RobolectricTestRunner.class)` annotation. Indeed, Robolectric is mandatory because we want to check our implementation of `Parcelable` in addition to other methods. This test is a very good example of an unit test where Robolectric brings has a real benefit. Indeed, we do not test the Android implementation, but rather our implementation of `Parcelable`, i.e adding and retrieving data in the `Parcel`.

{% highlight java %}
        @Test public void should_restore_from_parcelable() {
        // Given
        App app = new App("MyTitle");

        // When
        Parcel parcel = Parcel.obtain();
        app.writeToParcel(parcel, 0);
        parcel.setDataPosition(0);

        // Then
        App fromParcel = App.CREATOR.createFromParcel(parcel);
        assertThat(fromParcel.title).isEqualTo("MyTitle");
    }
{% endhighlight %}

The `AppsPresenterTest` class is also very important. Indeed, it tests all the logic of our application. It uses `RunWith(MockitoJUnitRunner.class)` annotation in order to use Mockito. We mock our `AppsActivity` and our `AppsProvider`. Mockito allows us to validate the behavior of our methods including the fact that some calls are made on `AppsActivity`. It also allows us to control the data processed in mocking calls to `AppsProvider`.

As we can see, these tests are very simple and their purpose is only to illustrate this article. However, we realize that our entire logic is well tested by unit testing, thus limiting the number of bugs on the technical parts thanks to UT and on user experience thanks to FT.

### Functional tests
Again, let's start with the `build.gradle`. You need to add the following dependencies and configurations:

{% highlight groovy %}
// INTEGRATION TESTING
configurations {
 androidTestCompile.exclude module: 'support-v4'
 androidTestCompile.exclude module: 'support-v7'
 androidTestCompile.exclude module: 'android'
}
androidTestCompile('com.android.support.test.espresso:espresso-core:2.0',
 'com.android.support.test.espresso:espresso-idling-resource:2.0',
 'com.android.support.test:testing-support-lib:0.1',
 'com.squareup.spoon:spoon-client:1.1.8')
androidTestCompile('com.android.support.test.espresso:espresso-contrib:2.0') {
 exclude group: 'com.android.support', module: 'support-annotations'
}
{% endhighlight %}


We can see that we use version 2 of Espresso. Espresso allows us to execute our tests directly on a device. Indeed, it can launch our application and then generate user actions. It also validates certain parts of the display of our views. It is therefore the perfect companion for our functional tests. To complete our build configuration, we must add to our `build.gradle` in order to force Gradle to change the runner used by the Espresso one :

{% highlight groovy %}
defaultConfig {
 applicationId "fr.xebia.jmartinez.installed"
 minSdkVersion 15
 targetSdkVersion 22
 versionCode 1
 versionName "1.0"
 testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
}
{% endhighlight %}

We also use Spoon that is a tool developed by Square, which allows to take screenshots during the tests and generate a visual report of all tests executed by our application.

Note that we could have used other frameworks as Robotium for instance. We chose to use Espresso because it is the solution adviced by Google and its API seems more usable and extensible.

Now, let's write these tests. Our app is simple, a single test is sufficient to validate its set of features. This test is contained in the class `AppsActivityTest` (contained in `src/androidTest`):

{% highlight java %}
@Test public void testFilterInstalledApp() {
        onView(withId(R.id.filter)).check(matches(isDisplayed()));
        takeScreenshot("Display_All_Installed_Apps");
        onView(withId(R.id.filter)).perform(typeText("Installed"));
        onView(withText(R.string.app_name)).check(matches(isDisplayed()));
        takeScreenshot("Display_Filtered_Installed_Apps");
    }
{% endhighlight %}

As we can see, writing a function test is simple and very intuitive thanks to the Espresso API. First we check that the filter field is displayed. We are there fore that our `AppsActivity` is launched and we take a screenshot with Spoon. We then type the characters "Installed" in the text field and we check that the name of our application is present in the list. We finally take a new screenshot. Our critical user experience path has actually been tested and all our classes also : `AppsProvider` and `AppsActivity`.

Finally, you will also find, in this project, a package internal with three very useful classes:

- `JUnitTestCase`: This is the superclass of our class tests. It provides several utility methods for handling `Activity` lifecycle in Espresso. It also includes the use the two following JUnit 4 Rules.
- `ActivityRule`: This is a Rule JUnit 4 and was developed by [Jake Wharton](https://twitter.com/JakeWharton). It simply allows you to launch an `Activity` each time our tests start in a easier and cleaner way.
- `SpoonRule`: This is also a Rule JUnit 4. Its purpose is to enable Spoon to take screenshots in our tests.



## And in the IDE, it works too?


Well yes! Finally ! With the release of Gradle plugin and Android Studio 1.1.0, tests are now supported directly in the IDE. While this feature is still in "experimental" mode, it is completely usable and works very well. The screenshots below can prove it:

<center>![App screenshot]({{ site.baseurl }}public/images/testing_mode_screenshot.png)</center>

As we can see, simply select the "Artifact Test": **Unit Tests** or **Android Instrumentation Tests**. This choice allows to compile and executable the right type of test. Note that both are not available simultaneously for now.


## To go further …


There are many tools to help you improve your tests in Android. Indeed, in order to monitor and have metrics on your test coverage, it is interesting to set up a tool like [Jacoco](http://eclemma.org/jacoco/) for instance. If you want your user experience tests to be writter by your QA or non-technical people, a tool like [Cucumber](https://cucumber.io/) becomes interesting. Finally, a library as [Gwen](https://github.com/shazam/gwen), implemented by Shazam will allow you to give coherence and a clarity to your tests.

These are just some examples of available libraries in order to continuously improve your tests. So why not think about it, evaluate them and put them in place if this seems relevant?


## Conclusion


We saw that it is now possible to easily set up testing by adopting a modular architecture. These tests can permit you to reach honorable coverage on Android and with all the comforts of an IDE. So there is no more reasons or excuses to not implement tests in your applications.
