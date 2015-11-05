---
layout: post
title: See the Truth on Android
comments: true
---

Writing tests is painful, it is not the part of our job we all prefer, however it is mandatory. Indeed, it allows us to guarantee behaviors, features and non-regression. Therefore, since it is such a pain but so important, we need great tools to help us. [Truth](https://github.com/google/truth) is one of them.

<!-- more -->

## What is Truth ?

> Truth is an assertion/proposition framework appropriate for testing and driven by some extensibility needs.

Truth can be used in place of [Fest](https://code.google.com/p/fest/) or [AssertJ](http://joel-costigliola.github.io/assertj/). But, it goes beyond and gives you sweet tricks to write more elegant and readable tests.

Once again, best way to explain a framework is always code. Let's imagine we design a `Cart` class with the following API :

{% highlight java %}

public class Cart {

  public void removeItem(Item item) { ... }

  public void addItem(Item item) { ... }

  public float total() { ... }

  public int count() { ... }

  public void clear() { ... }

  public Map<Item, Integer> content() { ... }

}
{% endhighlight %}

The design is obviously very simple since its only purpose is to illustrate this article.

Now let's test it. Since we are in 2015, I assume I don't need to prove why testing framework over JUnit are useful and why you should all be using one.

For all my tests in this article, I will use the following `generateCart` method :

{% highlight java %}
private Cart generateCart() {
  return new Cart(
      new Item(1L, "Item1", 2.0f),
      new Item(2L, "Item2", 3.0f),
      new Item(2L, "Item2", 3.0f)
  );
}
{% endhighlight %}

With Truth, you only need to add the following dependency to your gradle build :

{% highlight groovy %}

    testCompile "com.google.truth:truth:0.27"

{% endhighlight %}

And our tests will look like that :

{% highlight java %}
public class CartTest {

  @Test
  public void should_add_remove_item() {
    Cart cart = generateCart();

    cart.addItem(new Item(3L, "Item3", 10.0f));

    assert_().that(cart.count()).isEqualTo(4);
    assert_().that(cart.total()).isEqualTo(18.0f);

    cart.removeItem(new Item(3L, "Item3", 10.0f));

    assert_().that(cart.count()).isEqualTo(3);
    assert_().that(cart.total()).isEqualTo(8.0f);
  }

  @Test
  public void should_have_right_content() {
    Cart cart = generateCart();

    assert_()
      .that(cart.content())
      .containsKey(new Item(1L, "Item1", 2.0f));
  }

  @Test
  public void should_clear() {
    Cart cart = generateCart();

    cart.clear();

    assert_().that(cart.count()).isEqualTo(0);
    assert_().that(cart.total()).isEqualTo(0f);
  }

  @Test
  public void should_count() {
    Cart cart = generateCart();

    assert_().that(cart.count()).isEqualTo(3);
  }

  @Test
  public void should_compute_total() {
    Cart cart = generateCart();

    assert_().that(cart.total()).isEqualTo(8.0f);
  }
}
{% endhighlight %}

As we can see our tests are easily read from left to right. There are also some very useful helper methods, especially on collections, to ensure our tests are understandable by most.

## Why Truth ?

Now, you could ask yourself : *why choose Truth over Fest or AssertJ* ?

First of all, Fest does not seem to be maintained anymore. It did not even migrate from GoogleCode that will soon close its doors. So it is not a real option.

About AssertJ, it is a great framework that I have been using a lot in all my tests. It works perfectly. However, Truth does the exact same things **AND** more...

### Easily extensible

Thanks to its architecture, Truth is perfect to use when we want to extend the framework to our custom objects. We can therefore design an API that will add readability and meaning to our code. Let's see an example with our `Cart` object. First, we need to write a custom `Subject`. `Subject` is a class that represents all the assertions possible on our object.

{% highlight java %}
public class CartSubject extends Subject<CartSubject, Cart> {

  private static final SubjectFactory<CartSubject, Cart> FACTORY = new SubjectFactory<CartSubject, Cart>() {
    @Override
    public CartSubject getSubject(FailureStrategy fs, Cart target) {
      return new CartSubject(fs, target);
    }
  };

  public static SubjectFactory<CartSubject, Cart> cart() {
    return FACTORY;
  }

  public CartSubject(FailureStrategy failureStrategy, Cart subject) {
    super(failureStrategy, subject);
  }

  public CartSubject isEmpty() {
    return hasCount(0);
  }

  public CartSubject hasCount(int count) {
    if (getSubject().count() != count) {
      fail("hasCount", count);
    }
    return this;
  }

  public CartSubject hasTotal(float total) {
    if (getSubject().total() != total) {
      fail("hasTotal", total);
    }
    return this;
  }

  public CartSubject contains(long id) {
    if (!getSubject().content().containsKey(new Item(id, "id", 0))) {
      fail("contains", id);
    }
    return this;
  }
}
{% endhighlight %}


  Now, our tests will look as follow :

  {% highlight java %}
  public class CartTest {

    @Test
    public void should_add_remove_item() {
      Cart cart = generateCart();

      cart.addItem(new Item(3L, "Item3", 10.0f));

      assert_().about(cart()).that(cart).hasCount(4).hasTotal(18f);

      cart.removeItem(new Item(3L, "Item3", 10.0f));

      assert_().about(cart()).that(cart).hasCount(3).hasTotal(8f);
    }

    @Test
    public void should_have_right_content() {
      Cart cart = generateCart();

      assert_().about(cart()).that(cart).contains(1L);
    }

    @Test
    public void should_clear() {
      Cart cart = generateCart();

      cart.clear();

      assert_().about(cart()).that(cart).isEmpty().hasTotal(0f);
    }

    @Test
    public void should_count() {
      Cart cart = generateCart();

      assert_().about(cart()).that(cart).hasCount(3);
    }

    @Test
    public void should_compute_total() {
      Cart cart = generateCart();

      assert_().about(cart()).that(cart).hasTotal(8f);
    }
  }
  {% endhighlight %}

As we can see, calls are chainable and since we chose method names, it makes a lot of sense and suits best to the context of our `Cart` class.

*What about Android ?* Well, it looks like a perfect case for testing `View`!

For instance, here is a small `Subject` I wrote to test a `TextView`. It's great because our tests will match perfectly Android API.

{% highlight java %}
public class TextViewSubject extends Subject<TextViewSubject, TextView> {
  private static final SubjectFactory<TextViewSubject, TextView> FACTORY = new SubjectFactory<TextViewSubject, TextView>() {
    @Override
    public TextViewSubject getSubject(FailureStrategy fs, TextView target) {
      return new TextViewSubject(fs, target);
    }
  };

  public static SubjectFactory<TextViewSubject, TextView> textView() {
    return FACTORY;
  }

  public TextViewSubject(FailureStrategy failureStrategy, TextView subject) {
    super(failureStrategy, subject);
  }

  public TextViewSubject hasText(String expected) {
    if (!getSubject().getText().equals(expected)) {
      fail("hasText", expected);
    }
    return this;
  }

  public TextViewSubject hasAlpha(float alpha) {
    if (getSubject().getAlpha() != alpha) {
      fail("hasAlpha", alpha);
    }
    return this;
  }

  public TextViewSubject hasTextSize(float textSize) {
    if (getSubject().getTextSize() != textSize) {
      fail("hasTextSize", textSize);
    }
    return this;
  }

  public TextViewSubject hasPaddingTop(float paddingTop) {
    if (getSubject().getPaddingTop() != paddingTop) {
      fail("hasPaddingTop", paddingTop);
    }
    return this;
  }

}
{% endhighlight %}

Then, I use the `TextViewSubject` with a new library called  [screenshot-tests-for-android](http://facebook.github.io/screenshot-tests-for-android/#getting-started) from Facebook that deals great with testing custom views independently. Here is an example how it works associated with Truth:

{% highlight java %}
@RunWith(AndroidJUnit4.class)
public class ItemViewTest {

  @Test
  public void should_check_item_view() {
    LayoutInflater inflater = LayoutInflater.from(InstrumentationRegistry.getTargetContext());

    ItemView view = (ItemView) inflater.inflate(R.layout.item_view, null, false);

    ViewHelpers.setupView(view).setExactWidthDp(300).layout();

    view.bind(new Item(1L, "Tomatoes", 3f));

    TextView nameView = (TextView) view.findViewById(R.id.item_name);

    assert_().about(textView())
        .that(nameView)
        .hasAlpha(1)
        .hasText("Tomatoes")
        .hasPaddingTop(10)
        .hasTextSize(24f);

    Screenshot.snap(view).record();
  }
}
{% endhighlight %}

So as you may think, we could indeed create a Truth-Android the same way Square implemented [AssertJ-android](https://github.com/square/assertj-android). AssertJ-android supports all the views (and more) from the Android framework. It took time to build it and it is a real success. Right now, I just write `ViewSubject` according to my needs in my projects. However, it would be definitely interesting to create a library to regroup and share them.

### Brings failure strategies to testing

Strategies are the other feature that made me try Truth and adopt it. Let's see what it means.

First, Truth differentiates 3 failure strategies :

 1. The first one we all know is **assert**. It exists since testing exists. Principle is simple, if we assert something wrong in our test, we stop and fail. That's what we have been using for all our tests.

 2. The second one is **assume**. It means that when we assume something in our tests, if it fails our tests is just ignored. It could look useless but it's very useful on Android especially when dealing with fragmentation. Imagine you want to test a feature of your app that is only available on Lollipop. Then, when you run your instrumentation tests, you want to not run this test on device pre-lollipop. Thanks to **assume**, the test will be ignored since it is not applicable. Here is an example :

{% highlight java %}
   @Test
   public void should_test_super_new_feature() {
       assume().that(Build.VERSION.SDK_INT)
               .isGreaterThan(Build.VERSION_CODES.LOLLIPOP);

               // Some assertions
               ...
   }
{% endhighlight %}

 **Assume** can be used for various use cases such as : my local database is not configured but I still want to execute my unit tests or I want to run some tests specific to Windows or Linux (file system) for instance.

 <ol start=3>
 3. The last strategy available is **expect**. It means you go through all your assertions even if some fail and you generate a small report at the end. Best use case to my opinion is when you need to assert that a object is correctly filled. You don't want to fix your test, launch again, fix again, launch again. You want all what is wrong in one time. For instance, I always write a test for testing my `Parcelable` implementation. With Truth, it does work perfectly :

 {% highlight java %}
 @RunWith(RobolectricTestRunner.class)
 public class ItemTest {

  private final Expect EXPECT = Expect.create();
  private final ExpectedException thrown = ExpectedException.none();

  @Rule public final TestRule wrapper = new TestRule() {
    @Override
    public Statement apply(Statement base, Description description) {
      Statement expected = EXPECT.apply(base, description);
      return thrown.apply(expected, description);
    }
  };

  @Test
  public void should_restore_from_parcelable() {
    Item item = new Item(1L, "Tomatoes", 3f);

    Parcel parcel = Parcel.obtain();
    film.writeToParcel(parcel, 0);
    parcel.setDataPosition(0);

    Item fromParcel = Item.CREATOR.createFromParcel(parcel);
    EXPECT.that(fromParcel.name).isEqualTo("Tomatoes");
    EXPECT.that(fromParcel.id).isEqualTo(1);
    EXPECT.that(fromParcel.price).isEqualTo(3f);
  }
 }
 {% endhighlight %}

Finally, it is also possible to implement your own strategy and benefit directly from the rest of the framework. An example Truth's developers give is to deal better with exception. I never personally needed it but feel free to try.

## But be careful…

Truth is great but be careful while using it.

1. First, it is still in beta version (0.27 at the time of this article). Therefore it may contain some bugs and API may change. However, I am using it since some time now and it is production ready to my opinion.

2. Secondly, you must be sure you are not abusing the new strategies Truth offers. Indeed, some tests may not run unexpectedly because of **assume** or developers can feel encourage to tests too many things thanks to **expect** instead of having very focus and small tests.

## Conclusion

I hope this article helps you to see how powerful Truth is and that you will give it a try. Obviously, the next great step would be to code a Truth-Android. Anyone interested ?
