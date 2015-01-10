---
layout: post
title: Using APIBlueprint for Fun and Profit
published: true
tags:
  - JSONSchema
  - Apiary
  - APIBlueprints
  - Contract First
---

#### Intro
*Contract First Design* was postulated by [Bertrand Meyer](http://en.wikipedia.org/wiki/Bertrand_Meyer) almost 30 years ago.
Yet I have not seen any practical implementation of these idea, when designing a new API for web service.

Afte spending a day with *APIary* and *APIBlueprints* I'd like to note ** *API Blueprints* is nothing more than a template that is used by *APIary* to generate its documentation and mocking services.**

Yet, the nice thing about *APIary* is that they combined together multiple related technologies and mixed in some nice desing.

IMHO, next step for *APIary* should be to add functionality to generate stub code for poluar frameworks like Djanog, Rails, Express.js, Netty, PlayFramework, etc. That would allow to convert my spec into working service.
**This Is Cool**.

#### On Using Markdown
I've used Markdown extensivly with [Pandoc](http://johnmacfarlane.net/pandoc/README.html) to generate LateX paper.
Now, with APIBlueprints, I'm using it to produce a spec for my API.
And to write this post.
Markdown is great for proxy plain words into any particular format, but.
Lack of "standard markdown" is a major pain.
Since every Markdown based "language" tries to add/change some syntaxt and impose some syntactic (e.g.  reserver keywords, indentation) and semantic rules.
I must admit that it's getting better since the introduction of "GitHub Flawored Markdown".

*For those who likes to read from orignal source I referer to [original](http://cecs.wright.edu/~pmateti/Courses/7140/Lectures/OOD/meyer-design-by-contract-1992.pdf) work*
