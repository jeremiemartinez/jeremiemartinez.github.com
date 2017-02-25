---
layout: post
title: "Review yourself!"
comments: true
---

This is not an Android focus article like I am used to on this blog. However, it describes a situation that can happen a lot to a mobile developer: a single developer team.

In this blog post, I will explain why and how I continued to do code reviews even if I was alone in my team. Hopefully, it might give ideas to others.

<!-- more -->

## Context

Recently I have been left alone as the only Android developer of Trainline Europe. After 18 months of team work, being alone was not easy. I had to adapt the methodologies, the practices and the habits we followed as a team. The most important practice that I thought I would miss and did not want to lose was **code review**.

## Why is code review important?

I assume that in 2017 the benefits of code review have been well-known for most of readers:

- Sharing knowledge
- Avoid mistakes
- Make sure conventions are respected
- Increase readability of the code
- Keep history of discussions

While sharing knowledge is very difficult when you are alone, all the other advantages keep their values.

## What strategy I adopted when reviewing myself?

First rule I followed was to do Merge/Pull Requests as I had a team:
- Do small and meaningful commits. Mandatory for history and being future proof.
- Describe the work done in each MR description (with screenshots if applicable). It will be kept in the merge commit.
- Cut huge features in several small PR. Small progress are more readable.

The second rule was to never review on the day of the PR. I tried to leave several days between creating the PR and reviewing it, sometimes even a week or so. Indeed, it helps to change my mind and reviewing my work as it was someone else. I also tried to review each MR completely in one pass.

![MR]({{ site.baseurl }}public/images/mr.png){: .center-image }

The third rule was to consider it as someone else work. Set the same standard to your code than you would have with someone else. Indeed because of time, stress or laziness, we tend to be more conciliant with our code. We should not.

When you review the code of someone else, you leave comments, same rule here (#4). Every change needs to be commented. Even if it is small one, you will forget what you had to do later. Don't trust your memory.

![MR]({{ site.baseurl }}public/images/comments.png){: .center-image }

As long as every comment has not been resolved/closed, you cannot merge the PR. If it needs discussion or explanation why this comment is wrong or doesn't apply, then you need to answer it first before merging. With this rule (#5), every decision is documented.

#6: Don't hesitate to ask help to other developers outside your team. While they don't know Android, they will be able to help you with classic mistakes or computer science problems.

Finally, last rule (#7): some PR are more structurant than others. For these PR, I always try to explain to someone else my architecture or my modelisation. Most of the time, explaining helps a lot to clarify your mind and the naive question you could get, have a lot of value because they are Android agnostic.

## Conclusion & feedback

This article can look overkill and time consuming for a single developer team but I truly believe it is worth the hassles. It proved to be very useful for me. Of course, it does not replace a review for another Android developer but it is still valuable.

Hopefully, this feedback/retrospective could help you with the quality of your code and not feeling _that_ alone in your team.
