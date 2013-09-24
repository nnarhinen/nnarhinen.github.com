---
layout: post
title: Ring session middleware and compojure
moot: compojuresession
---

{{ page.title }}
================

I wanted to add session support to my web-application written in Clojure+Ring+Compojure.

Searching from google I found [information how to use sessions using ring.middleware.session](https://github.com/ring-clojure/ring/wiki/Sessions). However the examples would not work as is when  your are using [Compojure](https://github.com/weavejester/compojure).

The key to success was to [actually find out how compojure routes work](http://stackoverflow.com/questions/3488353/whats-the-big-idea-behind-compojure-routes)

The complete code is presented below - main points:

 * on line 13 we use the compojure destructure format to extract session information from request to a value
 * on line 14 we read the key :count from the session map, if not found we will give it a default value of 0
 * on line 15 we increment the count with one and add to the session map
 * on line 18 we add the modified session to be sent with the response

 <script src="https://gist.github.com/nnarhinen/6691550.js"></script>
