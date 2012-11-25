---
layout: post
published: true
title: Integrating the Dozer memory profiler in your python script.
tags:
 - profiling
 - memory
 - dozer
 - threads
---

Sometimes it's useful to take a quick look on how your python
script consumes the memory. The [Dozer](http://pypi.python.org/pypi/Dozer) memory profiler is a very handy
in this case. Unfortunately it requires that you run a WSGI app, which is not always the case.

A quick hack would be to start a fake wsgi server in the separate thread:
<script src="https://gist.github.com/4142894.js"> </script>

It turns out that this works quiet well :) But, once again, don't forget to set *daemon=True*

Another useful tool, which doesn't require a WSGI app is the [Heapy](http://pypi.python.org/pypi/guppy/) profiler.
This is a PhD research project, the source code is really strange and looks like the project is left behind.
