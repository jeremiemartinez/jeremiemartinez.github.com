---
layout: post
title: Help developers with custom lint rules
comments: true
---

Last month, I attended a great BarCamp talk at Droidcon Paris from [Matthew Compton](https://twitter.com/ambergleam). It was about writing your own Lint rules. I was really intrigued and wanted to explore a bit more this great subject. Therefore, I came up with this article that aims to share my thoughts and dive into some concrete examples on how to integrate custom rules into your Android project.

<!-- more -->

## Definition

If you are an Android developer, I have no doubt you already know what [Lint](http://developer.android.com/tools/help/lint.html) is, but here is a quick reminder:

> Lint is a static code analysis tool that checks your Android project source files for potential bugs and optimization improvements

Lint is the one that reminds you when you forgot to call `show()` on your `Toast`. It is also the one that makes sure you add a `contentDescription`on your `ImageView` to support accessibility. It exists tons of examples like these ones. Indeed, Lint can help you on a huge variety of subjects such as: correctness, security, performance, usability, accessibility, internationalization, and so on.

Lint is easy to use since it can be run on any Android project with a simple Gradle task: `./gradlew lint`.
It will generate a report about what it found out and classify the issues by category, priority or severity. This report should always be monitored since it is a great way to guarantee code quality and prevent some bugs in your app.

After this quick introduction, I hope that we can now all agree that Lint is a great help to understand some usages of the Android API Framework.


## Writing your own, What use cases ?

Something that most developers don't know is that you can write your own Lint rules. You may wonder, but why ?
Well, there is a couple of use cases where having custom Lint rules can be very useful:

1. If you are writing a library and you want to help developers to use it correctly, Lint rules are great since you can easily show them that they are forgetting something or doing something wrong.

2. If you have a new developer integrating your team, Lint rules can also be a great way to help him respecting your best practices or your naming conventions.

## Some examples

As you might know, I recently joined [CaptainTrain](https://play.google.com/store/apps/details?id=com.capitainetrain.android) Android team and the following examples are based on two Lint rules, I implemented for our app. I think it shows perfectly how Lint can ensure that developers follow project code practices.

Let's get started.

### Gradle

Custom Lint rules must be implemented in a new module. An example of a `build.gradle` for this module could be:
{% highlight groovy %}
apply plugin: 'java'

targetCompatibility = JavaVersion.VERSION_1_7
sourceCompatibility = JavaVersion.VERSION_1_7

configurations {
    lintChecks
}

dependencies {
    compile 'com.android.tools.lint:lint-api:24.3.1'
    compile 'com.android.tools.lint:lint-checks:24.3.1'

    lintChecks files(jar)
}

jar {
    manifest {
        attributes('Lint-Registry': 'com.capitainetrain.android.lint.CaptainRegistry')
    }
}

defaultTasks 'assemble'

task install(type: Copy) {
    from configurations.lintChecks
    into System.getProperty('user.home') + '/.android/lint/'
}
{% endhighlight %}

As you can see, we need two compile dependencies to implement our custom Lint rules, so make sure you have them. We also need to precise a `Lint-Registry`, we will see later what it is but for now remember it is mandatory. Finally we created a small task to help us installing quickly our new Lint rules.

To compile and deploy this module, you will use the following command: `../gradlew clear build install`.

Now that we configured our module, let's see how we can code our first rule.

### First rule: Attr must always be prefixed

In CaptainTrain project, to avoid clash in attributes we always prefixed them by `ct`. This can be easily forgotten by new developers (like me), therefore I wrote the following rule:

{% highlight java %}
public class AttrPrefixDetector extends ResourceXmlDetector {

 public static final Issue ISSUE = Issue.create("AttrNotPrefixed",
        "You must prefix your custom attr by `ct`",
        "To avoid clashes, we prefixed all our attrs.",
        Category.TYPOGRAPHY,
        5,
        Severity.WARNING,
        new Implementation(AttrPrefixDetector.class,
                               Scope.RESOURCE_FILE_SCOPE));

 @Override
 public boolean appliesTo(@NonNull Context context,
                          @NonNull File file) {
   return LintUtils.isXmlFile(file);
 }

 @Override
 public boolean appliesTo(ResourceFolderType folderType) {
    return ResourceFolderType.VALUES == folderType;
}

 @Override
 public Collection<String> getApplicableElements() {
    return Collections.singletonList(TAG_ATTR);
 }

 @Override
 public Collection<String> getApplicableAttributes() {
    return Collections.singletonList(ATTR_NAME);
 }

 @Override
 public void visitElement(XmlContext context, Element element) {
    final Attr attributeNode = element.getAttributeNode(ATTR_NAME);
    if (attributeNode != null) {
        final String val = attributeNode.getValue();
        if (!val.startsWith("android:") && !val.startsWith("ct")) {
            context.report(ISSUE,
                    attributeNode,
                    context.getLocation(attributeNode),
                    "You must prefix your custom attr by `ct`");
        }
    }
 }
}
{% endhighlight %}

As you can see we extend `ResourceXmlDetector`. A [`Detector`](https://android.googlesource.com/platform/tools/base/+/master/lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Detector.java) is a class that will permit us to find problems and then report an [`Issue`](https://android.googlesource.com/platform/tools/base/+/master/lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Issue.java). First of all, we need to specify what we are looking for:

- The first `appliesTo` method will keep only XML files.
- The second `appliesTo` method will keep only `values` resource folders.
- The `getApplicableElements` method will keep only `attr` XML elements.
- The `getApplicableAttributes` method will keep only `name` XML attributes.

After this filtering, we need to implement the `visitElement` method where our algorithm is very simple. Once we find a `attr` XML tag with a `name` attribute that does not come from Android neither starts with `ct`, we report an `Issue`. The issue is declared as follow at the top of the class:

{% highlight java %}
    public static final Issue ISSUE = Issue.create("AttrNotPrefixed",
            "You must prefix your custom attr by `ct`",
            "To avoid clashes, we prefixed all our attrs.",
            Category.TYPOGRAPHY,
            5,
            Severity.WARNING,
            new Implementation(AttrPrefixDetector.class,
                                Scope.RESOURCE_FILE_SCOPE));
{% endhighlight %}

Each parameter is important and mandatory:

- `AttrNotPrefixed` is the id of our lint rule. It must be unique.
- `You must prefix your custom attr by ct` is a brief description.
- `To avoid clashes, we prefixed all our attrs.` is a more detailed explanation.
- `TYPOGRAPHY` is the Category.
- `5` is the priority. It must be between 1 and 10.
- `WARNING` is the Severity. We chose only `WARNING` because code can still be run safely even with this problem.
- `Implementation` is the bridge between the `Detector` that will find the issue and the `Scope` required to analyze the issue. In our case, we need to be in at the resource file level to analyze this prefix problem.

As you may think, the code required is quite easy and understandable. You only need to be careful to the scope you are using and the values you enter for your `Issue`.

The result in your Lint report will look like this :

<center>![App screenshot]({{ site.baseurl }}public/images/log_rules_result.png)</center>

### Second rule: Log in production is forbidden

In CaptainTrain app, we wrapped all our `Log` calls into a new class. Since we must never log in production, this class aims to disable logging when our `BuildConfig.DEBUG` is false. It also helps to format our logs and some other sweet features. This rule looks like this:

{% highlight java %}
public class LoggerUsageDetector extends Detector
                                 implements Detector.ClassScanner {

    public static final Issue ISSUE = Issue.create("LogUtilsNotUsed",
            "You must use our `LogUtils`",
            "Logging should be avoided in production for security and performance reasons. Therefore, we created a LogUtils that wraps all our calls to Logger and disable them for release flavor.",
            Category.MESSAGES,
            9,
            Severity.ERROR,
            new Implementation(LoggerUsageDetector.class,
                                Scope.CLASS_FILE_SCOPE));

    @Override
    public List<String> getApplicableCallNames() {
        return Arrays.asList("v", "d", "i", "w", "e", "wtf");
    }

    @Override
    public List<String> getApplicableMethodNames() {
        return Arrays.asList("v", "d", "i", "w", "e", "wtf");
    }

    @Override
    public void checkCall(@NonNull ClassContext context,
                          @NonNull ClassNode classNode,
                          @NonNull MethodNode method,
                          @NonNull MethodInsnNode call) {
        String owner = call.owner;
        if (owner.startsWith("android/util/Log")) {
            context.report(ISSUE,
                           method,
                           call,
                           context.getLocation(call),
                           "You must use our `LogUtils`");
        }
    }
}
{% endhighlight %}

As you can see we find the same pattern than before. First, we have two methods `getApplicableCallNames` and `getApplicableMethodNames` that will allow us to specify what we are looking for. Then we find the issue and create it. The only difference is that we are not extending `XmlResourceDetector` anymore but extending simply `Detector`and implementing `ClassScanner` interfaces to handle Java class checks. Well, in fact it did not change that much from the previous rule… Indeed if we look closely to `XmlResourceDetector`, it is just a `Detector` implementing `XmlScanner`. So to sum up for any Lint rule, all we need is to extend `Detector`and implements the right `Scanner` interface.

Finally, we also changed the scope of our `Issue` and chose `CLASS_FILE_SCOPE`. Indeed, to be able to find this issue, we only need to analyse a single Java class file. Sometimes, you will need to analyze several Java class files to raise an issue, so you will need to use `ALL_CLASS_FILES`. You can see that choosing your scope is important, so be careful. All scopes are available [here](https://android.googlesource.com/platform/tools/base/+/master/lint/libs/lint-api/src/main/java/com/android/tools/lint/detector/api/Scope.java)

It might not be clear but several issues can be reported in a same `Detector`. It is even the right way to do it since we can therefore process all of them in a single pass, hence improving performances.

The result in your Lint report for this second rule will look like this :

<center>![App screenshot]({{ site.baseurl }}public/images/attr_rules_result.png)</center>


### Registry

We are missing one last thing: Registering ! Indeed, we need to register our newly created issues into the list of all the lint checks processed:

{% highlight java %}
public final class CaptainRegistry extends IssueRegistry {
    @Override
    public List<Issue> getIssues() {
        return Arrays.asList(LoggerUsageDetector.ISSUE, AttrPrefixDetector.ISSUE);
    }
}
{% endhighlight %}

As we can see, it is also very simple, we simply need to extend `IssueRegistry` and implements the `getIssues` method to return our custom issues. This class must be the same than the one we declared in our `build.gradle` at the beginning.

## Conclusion

Of course, I just showed two very simple examples but I hope it is now clear that Lint can be very powerful. It is only up to you to write rules that will suit your needs.

We only saw two types of `Detector`/`Scanner` but it exists many more such as: `GradleScanner`, `OtherFileScanner`, … Explore them and use the right one.

To start writing your own rules, I encourage, first, to read the system Lint rules to help you understand what can be accomplished and the way of doing it. Their source code is available [here](https://android.googlesource.com/platform/tools/base/+/master/lint/libs/lint-checks/src/main/java/com/android/tools/lint/checks).

Finally, Lint can be a great tool to help you fixing development mistakes. Use it ! ;)

Find below all materials that helped me:

- [https://github.com/bignerdranch/linette](https://github.com/bignerdranch/linette)
- [https://speakerdeck.com/ambergleam/linty-fresh](https://speakerdeck.com/ambergleam/linty-fresh)
- [Source code](https://android.googlesource.com/platform/tools/base/+/master/lint/)
- [http://tools.android.com/tips/lint-custom-rules](http://tools.android.com/tips/lint-custom-rules)
- [https://github.com/googlesamples/android-custom-lint-rules](https://github.com/googlesamples/android-custom-lint-rules)
