---
layout: post
title: "Internationalizing an Android app"
comments: true
---

At Trainline Europe, we sell train tickets for several countries in several languages. One could think translating an app is easy, but one would be wrong.

Indeed, translating an app is not only about translating words, it is integrating a whole new language, culture and market into the same product. If you did not think about it from the beginning of your app, you will end up in a lot of troubles.

I will walk you through some good practices we have at Trainline Europe based on our experience and the lessons we learned, sometimes the hard way.

<!-- more -->

This article is a non-exhaustive repository of rules we learned while writing code for a multi-language app.

### 1. Follow formatting rules

>How do you write a date? a hour? a duration? An amount? How do you separate decimals? What’s the first day of the week?

These are several questions where languages differ. By respecting these simple rules, your users will feel home and “local” in your app. Indeed, being able to parse without any trouble a date or a price is a mandatory first step. What's the point of having a beautiful layout if your users have trouble parsing simple information? None.

SEARCH RESULT DETAIL SCRENSHOT IN FR AND EN

### 2. Follow typographic rules

It is exactly the same problem that the previous rule. Indeed, if you don't respect the typographic rules of each language, your user will have trouble to parse simple data in your layout.

For instance if you don't respect non-breakable spaces in French before punctuation, you could have this kind of behavior:

NON BREAKABLE SPACES SCREENSHOT

These non-breakable spaces are also important for your brand. You don't want your brand to be cut in half.

### 3. Never hardcode a string

This may be silly, but if you were supporting only one language, you might never have used the `strings.xml` files. You need to extract your strings and rely on the Android mechanism for handling language changes.

### 4. Avoid concatenating two strings

Not following this rule can lead to strange problems. Let's see an example:
I want to translate something like “Cancel inward of Jeremie” where `inward` and `Jeremie` are variables.

- 1st solution: `Cancel` + `part` + `of` + `passenger`
- 2nd solution: `Cancel %{part} of %{passenger}`

The problem with the first solution is that in German the structure of the sentence is different and it would not work: `%{part} von %{passenger} stornieren`

Another problem can be found with an example as trivial as: `Pay $30`. While in most of languagues, `Pay` + `formattedPrice` will work, it does not in German: `30€ Bezahlen`. Therefore, the only solution you have is: `Pay %{formattedPrice}` that would be `%{formattedPrice} Bezahlen` in German.

### 5. Don't assume punctuation, genders or plurals

For instance, a sentence can start with a punctuation sign in Spanish: `¡Hola!`

Gender in Spanish can end with an `a` or an `o`, but in French we will add an `e` at the end. Then, an implementation that would just concatenate an `e` would disgracefully fail in other languages.

Finally, while in English, plurals are supported with `one` and `other`: ticket or tickets. But in Russian:

ADD RUSSIAN TRANSLATION

Once again a proof that languages are full of surprises.

### 6. Use meaningful variable name over position

Translating is difficult but it is even more difficult when you don't understand what you are translating. In order to help your translators, you should always prefer naming variable with a meaningful name rather than just the position. Exactly like your method signatures, naming the parameters is very important to understand what the method does.

At Trainline, we adopted the following format: `%{variable}`. We also implemented our own parser to be able to replace variable with their content dynamically. The implementation can be found here: LINK?

Let's see an example:
 -  `The %{type} card ending in %{lastDigit} and expires on %{date}.`
- `The $1 card ending in $2 and expires on $3.`

I think the example speak for itself. Indeed, readability is king.

### 7. Avoid reusing a string

Indeed, it could be a mistake. My advice is to make a new one with the same content instead. We recently encountered this problem in Trainline Europe website:

- English: Cancel (an action) / Cancel (a ticket)
Spanish
- Spanish: Cancelar (una acción) / Anular (un billete)

We were using a common `cancel` translation since it was translated the same in English, but once we added Spanish it was not working anymore.

### 8. Context is key

If you are able to choose a tool (and you should) where you can add screenshots, comments and explanation about each variable, your translators will thank you. More they have context, more precise their translations can be.

### 9. Avoid using locales as variable of other locales

Chaining and embedded locales can lead to errors from translators. As an advice, you should avoid this situation and try to create full locales readable at once. Once again, help your translators with as much context as you can.

### 10. Avoid reflexion and dynamically building keys

Every time you use reflexion to dynamically generate keys, you lose the only way you had to find which locales are used and where. Moreover, if you use a shrinker that will remove unused strings from your final APK, these locales will be removed. Of course, Google provides a way to let him know thanks to the `keep.xml` files. In Trainline Europe app, we use this technique for keys sent by the server for carriers or discount cards.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="
@string/data_card_description_*,
@string/data_card_long_*,
@string/data_card_short_*,
@string/data_carrier_*"
    tools:shrinkMode="strict" />
{% endhighlight %}

### 11. Think RTL

Last rule but not least, Right-To-Left (RTL) languages like Arabic can be very tricky. Indeed, it does not only impact strings but layouts too. Fortunately, Android provides a way since API 17 with attributes like `toStartOf`, `marginStart`, and so on. If you did not think about it at beginning, you may have to update most of your layout files.

### Conclusion

Android is already very smart. Indeed since N, you can have a list of speaking languages and order them by preferences. Unfortunately, I am still missing region choices. Indeed in my case my phone is in English and some apps use this information to guess the preferred currency whereas I would still like to buy in euros. I need English from France as barbaric it may sound.

But back on the topic, to get the most of the Android system, you should embrace it and be ready to translate your app at any moment.

Finally, I hope I proved with this article that you cannot assume anything about languages. You will always be surprised. By respecting simple rules you ensure your app future and growth in multiple markets.

PS: Thanks [Vincent](https://twitter.com/vincevlo) who did a huge part of the work with me in compiling the rules of this article.
