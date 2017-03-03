---
layout: post
title: "Review yourself!"
comments: true
---

This is not an Android focus article like I am used to on this blog. However, it describes a situation that can happen a lot to a mobile developer: a single developer team.

In this blog post, I will explain why and how I continued to do code reviews.

<!-- more -->

## Context

In this context, I had to adapt the methodologies, the practices and the habits we followed as a team. The most important practice I wanted to keep was **code review**.

## Why is code review important?

I assume that in 2017 the benefits of code review have been well-known for most of readers:

- sharing knowledge;
- avoid mistakes;
- make sure conventions are respected;
- increase readability of the code;
- keep history of discussions.

While sharing knowledge is very difficult when you are alone, all the other advantages keep their values.

I only see advantages by continuing this practice. In the short run, code reviews can help you avoid mistakes. In the long term, it will help new joiners to understand reasons behind certain pieces of code.

## What strategy I adopted when reviewing myself?

#### Do MR/PR as usual

- Do small and meaningful commits. Mandatory for history and being future proof.
- Describe the work done in each MR description (with screenshots if applicable). It will be kept in the merge commit.
- Cut huge features in several small PR. Small progress are more readable.

#### Never code and review the same day

I tried to leave several days between creating the PR and reviewing it, sometimes even a week or so. Indeed, it helps to change my mind and reviewing my work as it was someone else when I come back to it.

![MR]({{ site.baseurl }}public/images/mr.png){: .center-image }


#### Review as it was someone else's code

Because of time, stress or laziness, we tend to be more compliant with our code than we should. By doing the review several days later, it is easier to be objective.

#### Comments, comments, comments…
Exactly like you would do with someone else's code, I tried to comment PR as much as possible. It is great to keep reminder about everything required before merging. Don't trust your memory.

![MR]({{ site.baseurl }}public/images/comments.png){: .center-image }

#### Close everything

As long as every comment has not been resolved/closed, you cannot merge the PR. It makes sure every remark has been taken into account. It also can help to document some decision you may have.

#### Get all the help you can

Don't hesitate to ask other team developers for review. While they don't know Android, they will be able to help you with classic mistakes or computer science problems.

#### Explain to clarify

Finally, some PR, like architecture or object oriented design, are more structuring than others. For these PR, a good tip is to explain it to someone else. It helps clarifying your own mind and also since it is Android agnostic, other developers can help.

## Conclusion & feedback

These practices can look overkill and time consuming for a single developer team but I truly believe it is worth the hassles and it proved to be very useful for me.
