---
layout: post
title: TDD is hard
---

{{ page.title }}
================

Today I saw this tweet in my Twitter timeline:

<blockquote class="twitter-tweet"><p>TDD is not hard.Good software is hard.TDD is easy tool that helps write good software.</p>&mdash; Rob Conaway (@robconaway) <a href="https://twitter.com/robconaway/status/145524776461541376" data-datetime="2011-12-10T15:26:25+00:00">December 10, 2011</a></blockquote>
<script src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

I have tried to experiment with TDD a lot and usually failed miserably. Almost always the reason has been scoping. I don't know what to mock and what to integrate.

My experiments with BDD (or functional TDD with Gherkin or whatever you want to call it) has however been much more successfull. For me it feels a lot more intuitive to write tests ahead for much more concrete behavior or feature, than to know which methods my class is going to have and what they do.

My programming divides to two main parts:

    a) create a resource and save it to database, fetch it and modify it (sometimes even delete)
    b) show the resource in UI and add some visual properties and behaviors to it

How should/could I apply TDD to this? Seems there is no point of writing tests ahead for simple setters and getters. And for the save method or such it feels like mocking a database (or http-) connection isn't feasible, as then it just would be "when save() is called, an attemp to persist should happen", not that it would check if it actually goes to the datastore.. 

For the UI part writing (functional) tests with some automation tool is always an option, sometimes even a good one. However, for me doing UI coding is more intuitive doing it "visually" than to write some unit tests for ui behavior. Also to this I have found Gherkin and "BDD"-tools be more helpful than "traditional" TDD constraints. For me writing "Given I have 5 products in my database, When I open product listing, Then I should see 5 products And the first one should be 'Product 1'" is a intuitive thing to do. However, I don't always do the actual testing with any automation tool or such. I just write the tests against my ViewModels (I'm using MVVM as the approach..)

Every tutorial on TDD and every TDD workshop I have attended has had this naive Minesweeper, Calculator or Poker game example as the "problem domain". *I have never had to write a calculator, poker gamer or minesweeper in the real world*.. After each of these tutorials/workshops them I have been enthuastic to try TDD at my own projects. Then I always hit the same barrier: where to start.. Those simple examples are for already solved problem domains. Everyone knows how a poker game works, so the tests are simple to come up with. Hovewer if you need to implement something like "Product listing for an web-app", I know only (or hopefully at least..) that I need to have a product resource with some properties, that I need to have a way to query the database and that I need to show it somehow to the end-user. Then we are again in the situation a) from above..

Now the big question is, am I not doing good software if I find doing TDD so hard? Leave a comment, ping me on twitter or drop an email..