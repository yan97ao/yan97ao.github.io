---
title: python的常用json库对比
date: 2016-06-10 10:29:26
tags:
	- python
	- json
categories: tech
---

openstack nova中典型的一次api查询操作，需要经过api->conductor->mysql->conductor->api再返回的过程，组件之间的RPC操的数据传递，都需要经过json序列化和反序列化
并且API数据返回时，也需要序列化成json之后再通过wsgi返回
随着数据量上升，这期间多次json序列化和反序列化的性能开销逐渐成为很严重的性能问题
openstack本身基于微服务的架构不是很好改动，所以短期来看，更换更好性能json库也许能解决问题

<!-- more -->

测试环境：虚拟机一台 1 x Intel Xeon E312xx (Sandy Bridge), 1G RAM, Centos 6.5(Python 2.6.6)
安装pip
``` bash
wget ttps://bootstrap.pypa.io/get-pip.py
python get-pip.py
```
安装依赖
``` bash
pip install simplejson anyjson ujson
```

安装过程中遇到error: invalid command 'bdist_wheel'这样报错，是因为pip和setuptools版本不兼容导致，需升级setuptools或者降级pip
``` bash
pip install setuptools --upgrade
```
安装过程中遇到./python/py_defines.h:39:20: 错误：Python.h：没有那个文件或目录,需安装python-devel库
``` bash
yum install python-devel
```

测试代码
``` python
import time
from functools import wraps

import json
import simplejson
import anyjson
import ujson

sample_text = {
    'words': "Night gathers, and now my watch begins. It shall not end until my death. I shall take no wife, hold no lands, father no children. I shall wear no crowns and win no glory. I shall live and die at my post. I am the sword in the darkness. I am the watcher on the walls. I am the fire that burns against the cold, the light that brings the dawn, the horn that wakes the sleepers, the shield that guards the realms of men. I pledge my life and honor to the Night's Watch, for this night and all the nights to come.",
    'list': range(100),
    'dict': dict((str(i),100-i) for i in xrange(100)),
    'int': 100,
    'float': 100.123456
}

sample_json = json.dumps(sample_text)

def call_dumps(lib):
    t0 = time.time()
    for i in xrange(10000):
        lib.dumps(sample_text)
    t1 = time.time()
    print "run %s.dump: %s seconds" % (lib.__name__, str(t1-t0))

def call_loads(lib):
    t0 = time.time()
    for i in xrange(10000):
        lib.loads(sample_json)
    t1 = time.time()
    print "run %s.loads: %s seconds" % (lib.__name__, str(t1-t0))

def main():
    for i in [json, simplejson, anyjson, ujson]:
        call_dumps(i)

    print ""

    for i in [json, simplejson, anyjson, ujson]:
        call_loads(i)

if __name__ == "__main__":
    main()
```

测试结果
``` text
run json.dump: 7.7235019207 seconds
run simplejson.dump: 12.3510508537 seconds
run anyjson.dump: 12.1558740139 seconds
run ujson.dump: 0.218187093735 seconds

run json.loads: 38.5133662224 seconds
run simplejson.loads: 21.7097730637 seconds
run anyjson.loads: 20.8557491302 seconds
run ujson.loads: 0.256334781647 seconds
```
这样看来，在python 2.6.6的环境下，anyjson和simplejson相对于内置的json各有优劣，但是ujson有碾压级别的优势

另外在2015年初版本的13寸rmbp上(python 2.7.11)
``` text
run json.dump: 0.372946023941 seconds
run simplejson.dump: 0.537473917007 seconds
run anyjson.dump: 0.539379835129 seconds
run ujson.dump: 0.123563051224 seconds

run json.loads: 0.699669122696 seconds
run simplejson.loads: 0.428148031235 seconds
run anyjson.loads: 0.437167167664 seconds
run ujson.loads: 0.16900396347 seconds
```
ujson依旧有明显优势
