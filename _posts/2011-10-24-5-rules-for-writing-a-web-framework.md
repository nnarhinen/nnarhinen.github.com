---
layout: post
title: 5 rules for writing a web framework
---

{{ page.title }}
================

<a href="#rule5">tl;dr; - click here</a>

## #1: Learn from others
Make yourself familiar with existing products. To really learn the problem domain, you have to use other frameworks to learn their pitfalls and strengths. Study the code bases of leading projects (if open source) to learn about their structure. Search through their issue trackers and mailing lists for potential problems users are experiencing that their architecture prevents overcoming.

### Fork an existing project
Why do the ground work from scratch? Take an (open source) project and build on top of that. Ditch stuff you don't need, add stuff you need and think you do better. Web frameworks are normally complex systems and writing every line of code by yourself almost certainly makes sure the project never gets even the initial release.

## #2: Document from day one
A web framework without documentation is a web framework no one (except you) is going to adopt. You need tutorials, example projects, and hard core per-feature documentation. Implement best practices for your framework and document them.

Writing documentation makes you also look outside of the box. When writing documentation to others, you discover better their needs. Writing a web framework for yourself without documentation almost certainly makes sure it has features that only add value for yourself and contains blocking issues for others.

You could even try <a href="http://en.wikipedia.org/wiki/Behavior_Driven_Development">Behavior driven development</a>. You get some feature documentation while you add tests for your project. It also makes you discover when you are breaking **backwards compatibility**, one of the things you really will have to consider while writing a web framework for others. Breaking BC means your users won't update even though you would later discover security issues. And because they can't update, you have to backport security fixes to old releases and believe me: *that's hell*.

## #3: Utilize the web
There are standards for the web, *use them*.

For me, the web equals HTTP. And I (almost) start to cry every time I see HTTP abused. Supporting only GET and POST (or even worse, only POST) or mixing the semantics of the verbs is unforgivable. Not only do you break everything that makes the web so awesome, but you also guide others (your framework users) to break it more. Make yourself familiar with specifications: HTTP status codes, standard request and response headers etc, and learn their semantics. 

The web is a great platform to build on top of and there are plenty of users that already know how the web works (or should work, at least..). Don't let them down by not using the full power and standards of it.

(A great starting point is to learn ~everything about <a href="http://en.wikipedia.org/wiki/Representational_state_transfer">REST</a> and <a href="http://en.wikipedia.org/wiki/HATEOAS">HATEOAS</a>)

## #4: Avoid magic
As a web dev I always avoid frameworks that adverts them self like: "Make a blog in 5 minutes". If I wanted a blog in five minutes, I would open an account on blogspot or tumblr..

Web frameworks are meant to do the heavy lifting for the web dev. To unleash the power of the web dev to actually add value. This however doesn't mean the heavy lifting should involve magic. Having magic built-in your framework usually (or always) results in a product that is suitable to do just the magic stuff it was meant for, not generalizing for wider use.

A good example of this are form helper libraries from various frameworks. They seem great at first glance; "Wow, I just write five lines of php and get a form with 20 inputs!!". Nothing wrong with this, as long as it suits your use case. Binding a model to a form to handle all validation and stuff can be handy and desired. However, in my experience, in every project there comes this situation when the helper gets in the way. And usually it cannot be just left out. And then you end up using hours to workaround the limitations of the magic library that was initially supposed to help you..

Concentrate on things that really add value for the web developer; implement good structure for projects, help them utilize the web, add some nice helpers here and there but don't make them tie their business logic in to the internals of your framework.

<a name="rule5"></a>
## #5: Don't write one
**This is the most important rule.** (And the one I've been following for years after I first discovered it..)

The world (or at least the web) is full of web frameworks, some of them are even good. Writing a web framework is hard and you really have to know what you are doing. If your motivation is *"Existing frameworks are too complicated to use"* you definitely aren't sharp enough to write one yourself.

That's all for this time, if you disagree with me or want me to clarify things, please don't hesitate to contact me.