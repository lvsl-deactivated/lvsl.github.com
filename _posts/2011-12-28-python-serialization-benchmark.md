---
layout: post
published: true
title: Benchmark of serialization libraries in python
tags:
 - serialization
 - json
 - pickle
 - msgpack
 - yaml
---

This is my little survey of serialization libraries
available in python.

Serialization libraries used in benchmark:
 * [json](http://docs.python.org/library/json.html)
 * [cjson](http://pypi.python.org/pypi/python-cjson) 
 * [ujson](http://pypi.python.org/pypi/ujson/)
 * [yaml](http://pyyaml.org/) 
 * [cPickle](http://docs.python.org/library/pickle.html)
 * [msgpack](http://msgpack.org/)

Let's start with the code of the benchmark:

{% highlight python %}
#
# Simple serialization banchmark
#

import csv
import sys
import time

import cPickle as pickle
import msgpack
import yaml
import json
import cjson
import ujson

dumpers = (
    ('yaml', lambda x: yaml.dump(x, Dumper=yaml.CDumper)),
    ('cPickle', pickle.dumps),
    ('json', json.dumps),
    ('cjson', cjson.encode),
    ('ujson', ujson.dumps),
    ('msgpack', msgpack.dumps),
)

table = {}
for power in range(1000, 10 ** 5, 1000):
    data = [{'integer': 1, 'string': 'test', 'float': 42.0}] * power
    for dumper_name, dump in dumpers:
        start = time.time()
        res = dump(data)
        end = time.time()
        row = [power, len(res), "%.6f" % (end - start)]
        if dumper_name in table:
            table[dumper_name].append(row)
        else:
            table[dumper_name] = [row]

# display results as CSV, so you can plot it
# in your favourite chart tool
csv_writer = csv.writer(sys.stdout)
csv_writer.writerow(('format', 'power', 'size(bytes)', 'time(sec.)'))
for k,v in table.items():
    for row in v:
        csv_writer.writerow([k] + row)
{% endhighlight %}

Below is the diagram of serialization times.

<img src="/static/serialization-in-python-benchmark.png"/>
