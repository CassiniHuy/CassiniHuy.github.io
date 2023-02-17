---
title: nvidia-tensorflow 1.15.5的signal模块的一个内存泄漏bug
date: 2023-01-09 01:16:00
categories: [随笔]
tags: [tensorflow 1.15, nvidia, 内存泄露, RTX 4090]
---

最近实验室新进了一台有RTX 4090的工作站，于是在这台工作站上跑起了以前tensorflow 1时代写的代码，用的是nvidia ngc提供的docker镜像运行。

但是，发现session执行时间一次比一次长，同时发现内存也在缓慢增长，reset_default_graph也不能阻止这个过程；并且这个问题只在新服务器上出现，旧的RTX 3090显卡的服务器并没有这个问题。

于是开始了三四天的排查。

提出issue在：

https://github.com/NVIDIA/tensorflow/issues/76

结论：tf.signal.rfft, tf.signal.dct等方法有bug，通过降低nvida-tensorflow的版本到1.15.4解决。

下面是排查思路和过程。

<!--more-->


## 问题的发现

我最初只是发现，代码运行时，每轮的progress bar一次比一次运行时间长，并且在第二轮会直接出现segmentation fault (core dumped)错误；当时并没有发现内存泄漏的问题。

## 问问搜索引擎呢？

网上搜了一圈，搜索出来的结果全都在说是因为在循环中新建了图结点，但是经过排除并不是这个问题。。。

但是获得一个信息，也就是session执行时间越来越长，是内存泄漏的问题。

## 是显卡的问题吗？

假设镜像没有问题，那么是新显卡的问题吗？因为刚换了新服务器，我最初怀疑是Nvidia对显卡的支持问题，也就是显卡的问题，因为在RTX 3090上代码运行并没有问题。

首先，我跑过PyTorch的代码，没有任何问题。

但是经过测试，发现其他代码运行良好。

## 是镜像的问题吗？

在RTX 3090服务器上用的是20.12版本的镜像，这个镜像不支持新的RTX 4090显卡，于是在新服务器上用的是22.12版本的镜像。那么是因为新镜像的吗？

由于20.12版本的镜像不支持RTX 4090，只能在旧服务器上测试22.12版本的镜像，但是由于实验室网络问题，导致这个测试无法进行。。。

于是作罢，反正新服务器上的问题是一定要解决的，于是这个问题的答案貌似也没那么重要了。

## 到底是哪儿耗时在一直增加？

我想要知道是tensorflow的哪个操作的运行时间越来越长，通过网上搜索，发现了timeline工具。

例如如下代码，通过tensorflow自带的timeline工具记录每个op的执行时间：

```python
import tensorflow as tf
from tensorflow.python.client import timeline

run_options = tf.RunOptions(trace_level=tf.RunOptions.FULL_TRACE)
run_metadata = tf.RunMetadata()

...
# session.run的时候加上options和run_metadata参数
sess.run(output, options=run_options, run_metadata=run_metadata)
...

# 注意只能记录一次session.run的timeline数据
# 因此想要比较不同迭代次数的执行时间，每次迭代都记录一个timeline的json文件
for i in range(max_iter):
    ...
    tl = timeline.Timeline(run_metadata.step_stats)
    ctf = tl.generate_chrome_trace_format()
    with open(f'timeline_{i}.json', 'w') as f:
        f.write(ctf)
    ...
...
```

然后打开edge://tracing/，载入timeline的json日志文件，通过比较不同迭代时候的执行时间，来找到是哪儿个操作非常耗时。

通过timeline找到session执行时每个op的耗时，发现rfft一次比一次耗时：**第10次迭代和第120次迭代相比，session.run中rfft执行时间从0.2ms变成3.0ms**

其他op都没有这个问题，那么为什么rfft会随着迭代一次比一次耗时？并且内存一直泄漏？

## 到底是哪儿内存泄漏？

通过objgraph、pympler(这个工具比较直观)等等通过检查python对象引用及其内存占用的工具，没有发现任何异样，也就是都显示没有内存泄漏。

但是，通过psutil.virtual_memory发现内存确实是一直增加的。

可见内存泄漏是发生在比python解释器更底层的地方。

## 终于发现问题：真的是nvidia-tensorflow的bug吗？！

为了测试是不是反复调用rfft会导致内存泄漏，写了如下代码进行测试：

```python
import tensorflow as tf
import time     # 用来检查session的执行时间
import psutil   # 用来检查内存占用

graph = tf.Graph()
with graph.as_default():
    frames = tf.random.uniform((144, 400))
    spec = tf.signal.rfft(frames, [512, ])

with tf.Session(graph=graph) as session:
    session.run(tf.global_variables_initializer())
    session.graph.finalize()
    for i in range(10000):
        stime = time.time()

        session.run(spec)       # rfft computation

        etime = time.time()
        elapsed = etime - stime
        if i % 500 == 0:
            print(f'{i:05} Time: {elapsed}')
            used_mem = psutil.virtual_memory().used                     # memory usage
            print(f"{i:05} Memory used: {used_mem / 1024 / 1024} Mb\n") # time cost
```

输出：
  
```
00000 Time: 0.04090452194213867
00000 Memory used: 1823.8671875 Mb

00500 Time: 0.0012400150299072266
00500 Memory used: 1833.21875 Mb

01000 Time: 0.0013303756713867188
01000 Memory used: 1845.03125 Mb

01500 Time: 0.001249551773071289
01500 Memory used: 1856.59765625 Mb

02000 Time: 0.001224517822265625
02000 Memory used: 1868.40234375 Mb

02500 Time: 0.0013511180877685547
02500 Memory used: 1880.21484375 Mb

03000 Time: 0.0013630390167236328
03000 Memory used: 1892.02734375 Mb

03500 Time: 0.0014331340789794922
03500 Memory used: 1904.0859375 Mb

...
```


- 发现如果一个**loop里如果调用tf.rfft会导致内存一直增加，并且执行时间变慢**！！！

还发现一个有趣的事实，如果上面代码的*frames*变量是其他类型，则可能不会导致这个问题：

```PYTHON
...
graph = tf.Graph()
with graph.as_default():
    frames = tf.zeros((144, 400)) # 或者ones
    # 或者 frames = tf.convert_to_tensor(numpy.random.random(144, 400))
    # 或者 frames = numpy.random.random(144, 400)
    spec = tf.signal.rfft(frames, [512, ])
...
```
输出：
```
00000 Time: 0.1534738540649414
00000 Memory used: 1996.67578125 Mb

00500 Time: 0.0013377666473388672
00500 Memory used: 1996.953125 Mb

01000 Time: 0.001354217529296875
01000 Memory used: 1996.953125 Mb

01500 Time: 0.0013136863708496094
01500 Memory used: 1996.953125 Mb

02000 Time: 0.0010759830474853516
02000 Memory used: 1996.6796875 Mb

02500 Time: 0.0010845661163330078
02500 Memory used: 1996.6796875 Mb

03000 Time: 0.0009725093841552734
03000 Memory used: 1996.6796875 Mb

03500 Time: 0.0011267662048339844
03500 Memory used: 1996.6796875 Mb

04000 Time: 0.0012097358703613281
04000 Memory used: 1996.6484375 Mb

04500 Time: 0.0013315677642822266
04500 Memory used: 1996.6484375 Mb

05000 Time: 0.0012805461883544922
05000 Memory used: 1996.63671875 Mb
...
```


输入如果是numpy类型、或者tensorflow的const Tensor、tf.ones这类函数创建的Tensor，不会导致这个问题！！！

但是经过测试，如果输入类型是Variable和tf.random模块得到的对象会有这个问题。

- 另外，除了rfft，其他时频变换方法如tf.fft, tf.ifft, tf.dct也有这个问题！！！

## 到底怎么办呢？

由于任务需要，输入只能是Variable类型的输入，因此一时间感觉不知道怎么办。

目前确定了不是自己代码的问题，由于不知道bug出现在哪儿一层，于是假设试着**降tensorflow的版本到1.15.4**，结果问题就解决了。

就这样吧。

## 总结

nvidia-tensorflow 1.15.5 的tf.signal模块的一些频域变换函数有bug。

提交issue：https://github.com/NVIDIA/tensorflow/issues/76

