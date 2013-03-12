---
layout: post
title: Install Couchdb-Lucene in CircleCI
---

{{ page.title }}
================

We at [Ecom](http://www.ecom.fi) are using the great [CircleCI](https://circleci.com) service for running our integration test suite.

Recently they added support for Apache Couchdb as a service for the test runs.

We have some searches however that are backed by [Couchdb-Lucene](https://github.com/rnewson/couchdb-lucene) so we needed a way to integrate test those also. What I found out, is that installing and running couchdb-lucene with user privileges is actually possible.

Here's a few lines of bash to fetch, install and run couchdb-lucene in CircleCI:

<script src="https://gist.github.com/nnarhinen/5141201.js"></script>

Just add that bash script to your circle.yml under checkout -> post