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

### Intro
*Contract First Design* was postulated by [Bertrand Meyer](http://en.wikipedia.org/wiki/Bertrand_Meyer) almost 30 years ago.
Yet I have not seen any practical implementation of these ideas, when designing a new API for web service.

`Until now!`

[Apiary](http://apiary.io/how-it-works) got quite far to delivering something that can be used in practice.
They provide special language -- [APIBlueprint](http://apiblueprint.org/) plus nice documentation reader with embedded mock server and API debugger with simple editor (that can even sync with your *GitHub* account).
After playing a day with *Apiary* and *APIBlueprints* I'd like to note: *API Blueprints* is nothing more than a template that is used by *APiary* to generate its documentation and mocking services.
I found it very clumsy.
Yet, the nice thing about *APIary* is that they combined together multiple related technologies and mixed in some nice design.

### How To
Here I will attempt to describe experience I had, when building sample API: [Xmetrics](http://docs.xmetricsv1.apiary.io/)

First things first.
I want explicit API versioning, unfortunately *Apiary* does not support this.
So I just put `/v1` into every resource I've described in API.

When describing API calls I'd like to think about 4 essential things:

 - Query string parameters
 - Input data structures
 - Output data structures
 - Errors

*Apiary* only has support for structured documentation of query string parameters.
For Input, Output and Errors description I use plain *Markdown* tables right after Action declaration.

Here is how complete section of one API call looks like:

    ### <API-call-name> [POST]
    
    #### Input JSON Description
    | Name | Description | Details |
    | ---- | ----------- | ------- |
    
    #### Output JSON Description
    | Name | Description | Details |
    | ---- | ----------- | ------- |
    
    #### Errors Description
    | Name | Description | Details |
    | ---- | ----------- | ------- |
    
    + Request (application/json)
    
        + Body
        
            ... sample JSON request ...
        
        + Schema
        
            ... JSON-Schema ...
    
    + Response 200 (application/json)

        + Body
        
            ... sample JSON response ...
    
        + Schema
        
            ... JSON-Schema ...

    + Response 500 (application/json)

        + Body
        
            ... sample JSON response ...
    
        + Schema
        
            ... JSON-Schema ...
 
One feature that surprised me and looks very neat -- is support for [JSON Schema](http://json-schema.org/).
It really helps to shape your API in the right way.
The documentation does not clarify how to use or add schema,
[luckily, there us a support](http://support.apiary.io/knowledgebase/articles/147279-json-schema-validation).

IMHO, next step for *APIary* should be to add functionality to generate stub code for popular frameworks like Djanog, Rails, Express.js, Netty, PlayFramework, etc. That would allow to convert my spec into working service.
**This Is Cool**.

### On Using Markdown
I've used Markdown extensively with [Pandoc](http://johnmacfarlane.net/pandoc/README.html) to generate LateX paper.
Now, with APIBlueprints, I'm using it to produce a spec for my API (and, actually, to write this post as well).
Markdown is great for proxy plain words into any particular format, but.
Lack of "standard markdown" is a major pain.
Since every Markdown based "language" tries to add/change some syntax and impose some syntactic (e.g.  reserver keywords, indentation) and semantic rules.
I must admit that it's getting better since the introduction of "GitHub Flavored Markdown".

*For those who likes to read from original source I refer to [original](http://cecs.wright.edu/~pmateti/Courses/7140/Lectures/OOD/meyer-design-by-contract-1992.pdf) work*
