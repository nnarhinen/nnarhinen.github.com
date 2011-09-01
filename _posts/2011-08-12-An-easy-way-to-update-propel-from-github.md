---
layout: post
title: An easy way to update propel from github
---

{{ page.title }}
================

We were using svn:externals to keep <a href="http://www.propelorm.org">Propel ORM</a> (and other libs) up-to-date in our projects. (You should always have your libraries project-local).

Recently <a href="http://propel.posterous.com/propel-wont-die">Propel ORM switched to GitHub</a>.

I wanted to keep my development process lean and highly automated, that's why I wrote this bash-script to update Propel to the desired tag.

Hope it will be useful to others also (use at own risk..)

<script src="https://gist.github.com/1141974.js?file=update_propel.bash"></script>