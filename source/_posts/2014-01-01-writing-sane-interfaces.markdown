---
layout: post
title: "Writing Sane Interfaces"
date: 2014-01-01 11:15:27 -0500
comments: true
categories: 
---

Halfway through 2012, I started my first full time software development gig. Spending day in and day out extending and maintaining the same codebase has taught me a number of things that freelance projects never did, and I'd like to share a few lessons I've learned. These are all things I *knew* intellectually, but ultimately had to find out for myself through frustration to incorporate into my every day practice.

When tasked with a new feature that is core to an application, my first impulse is to talk and think through the functionality and edge cases. What does it need to do? What's the most reasonable, non-arcane, and maintainable architecture I can come up with? By this point, I'm eager to sit down and write some code. I invariably regret it when I do. The code that results will fit the feature spec in that it *does what it's supposed to*, but can be an unpleasant experience to leverage and maintain.

<!--more-->

{% blockquote %}
The question I forget to ask myself most often is: "what should the interface look like?"
{% endblockquote %}

Devising a sane interface is a crucial part of the planning and decision making process. It can also substantially increase the satisfaction and joy I get from implementing it. The more I can treat a feature like a library, referencing the source code little or never, the better. Below I outline some frustrations I've inherited or created and hopefully provide some useful thoughts in how to avoid them.

### Unintuitive Roles

I'm certain that I'm not alone in feeling like the hardest parts of software development and design are naming/organizing code into logical pieces. From a name, the next person under the hood or I should be able to intuit what an object or function is supposed to do. 

I have three main questions when looking at code: what does this do, why this needs to happen/why I might use it, and how it accomplishes what it does.

1. What (does this do)? I expect classes with descriptive method names to tell me this. In functional programming, providing a [context prefix](http://stackoverflow.com/questions/421965/anyone-else-find-naming-classes-and-methods-one-of-the-most-difficult-part-in-pr#answer-422061) can serve a similar purpose to group related functions. I have not succeeded in writing a sane interface if a name is unclear or (worse) misleading.
1. Why (does it do it)? I look to the documentation to provide some context as to why I might use a chunk of code.
1. How (does it do it)? I want the code itself tells me this. If I've written something subtle/complicated or leveraged a library that has unexpected usage or poor naming conventions, then I document it inline.

Most of what I write these days is object oriented. Before I start writing code, I come up with a first draft of what the acting classes are in a new feature. I try not to give them names until I'm satisfied with their roles. Giving something a name prematurely often leads me to design the rest of the system to support one member's role. Proper encapsulation and established functional roles makes finding the code I need to extend or methods I need to call much more intuitive. When applications get large, the importance becomes even more pronounced.

### Multiple entry points for the same functionality

Over the course of maintaining code, I try to stay mindful of my power to worsen a solid interface. If I add to or change a method, then maybe its name should change. If I need to do something slightly different than an existing method, I need to have a pretty damn good reason to tack on a new one that has more or less the same guts. I once had the privilege of being on a team that inherited a large PHP application with two User models and a sum total of six methods to add a new one, none of which appeared to do the same thing. Ultimately the most frustrating parts of this (beyond offending my sensibilities with 50+ line methods) was that the models' method names and documentation provided zero clues as to how they were different. I found out after much heartache that the main difference was that only one of the methods created a working user.

Applications that solve real world problems often require code that can handle more than exactly one use case. This rarely (if ever) should entail duplication for each scenario. As an example: I recently implemented a queuing system. The system needed to be able to enqueue people waiting for a call center agent, various agent to agent transfers, and jobs for background processing. Web clients needed to be able to subscribe to queues to receive stats updates, so the queue identifier had to be predictable and consistent. One catch was that some queues needed to be routed to customers, while others were used on an application wide basis.

My first draft provided customer as an optional parameter to Queue construction. With the additional constraint that queues support internationalization/locales, I opted to create a QueueIdentifier class whose responsibility was naming queues based on an arbitrary set of designations. This extracted any `if customer then ... else ...` logic from the Queue. In my experience, seeking a generalized solution from the beginning will keep the application logic simpler when new constraints are introduced.

In short, seeing multiple entry points to do strikingly similar (if not identical) things raises red flags for me. It's unintuitive, difficult to maintain, and in my eyes is characteristic of a difficult interface.