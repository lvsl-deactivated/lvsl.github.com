---
layout: post
title: Introducing "rot" - an ultimate stdout/stderr redirector and rotator written in pure python
published: true
tags:
  - python
  - epoll
  - asyncio
  - subprocess
  - linux
  - rotation
---

**Code tested with Ubuntu 11.10**

Let's start with an example.
{% highlight python %}
p = subprocess.Popen(["spam_program", "-a", "-b", "-c", "100500"],
                     stdout=open('spam_out.log', 'wb'))
{% endhighlight %}

Consider you have some `spam_program` which will emit 20GB to stdout/stderr...
This program is one of many(hundreds or even thousands) your test framework will start.

How can we handle this?
The first thought is to `> /dev/null`,  which is nice!
Except that, the author of `spam_program` may want to grep(sick!) through this massive log file.

The other option might be to set `logrotate` each time the program is started, to rotate this huge file.
Disadvantage of this approach is that you have to provide config file to `logrotate` and
have permissions to restart the daemon, and yes it's a daemon...


Or you can simply run a program using rot :)
Rot is a zero-install and configure tool written in python.

You can find it here:
[http://github.com/lvsl/rot][repo]

Here is example usage:
{% highlight sh %}
$rot --stdout-limit 100M \
     --stdout-count 4    \
     --stdout-file spam.stdout.log \
 -- spammy_program -a -b -c 100500
{% endhighlight %}

[repo]: http://github.com/lvsl/rot
