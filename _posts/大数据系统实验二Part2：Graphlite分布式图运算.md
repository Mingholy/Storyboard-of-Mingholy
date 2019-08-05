---
title: 大数据系统实验二Part2：Graphlite分布式图运算
date: 2016-05-08 23:00
cover: '7.jpg'
category: "Bigdata"
tags:
    - C++
    - Graph Compute
---
### GraphLite与图运算系统

GraphLite是陈世敏老师和学长写的一个开源的分布式图运算系统.

该系统是一个同步图运算系统,我们需要实现的是`class Vertex`的一个子类.这其中包括:  
1. 进行图计算的`Compute()`函数;
2. 系统提供的函数,可以在`Compute()`中调用:
    如:`getValue()`,`mutableValue()`,`getOutEdgeIterator()`,`sendMessageTo()`,`sendMessageToAllNeighbors()`,`voteToHalt()`等.

同步图运算系统的架构是master-worker模式,每一个worker对应一个graph partition.也就是图的一部分.

<!--more-->

计算的过程分为多个"**超步(superstep)**",在一个超步内,对每一个顶点**并行地**调用`Compute()`函数.在超步中,顶点计算执行算法的过程通常是:
1. 接收上一个超步发出的in-neighbor的消息,即接收由上一个超步中各顶点发送给邻居顶点的信息;
2. 计算当前顶点的value;
3. 形成信息,并发送给当前的邻居顶点.

这个过程贯彻了两个特性:

1\. BSP模型(Bulk Synchronous Processing):

  * 全部计算分成多个超步
  * 超步之间进行全局同步
  * 超步内部全部并行
    * 对多个运算单元进行运算
    * 超步内部的算法都是无依赖地分布式运行
  * 相邻超步之间存在依赖:上一个超步的输出结果作为下一个超步的输入.

2\. 基于顶点的编程模型:  

  * 每个顶点有自己的Value
  * 以顶点为中心的运算
  * 如上文所说,程序员实现一个`Compute()`函数,在每个超步内对每个顶点调用一次该函数
  * `Compute()`函数负责接收,计算并发送消息.

所有的顶点具有两种状态,Active和Inactive.它们之间的转换靠两个机制:

1. vote to halt: active -> inactive
2. message received: inactive -> active

如果一个顶点接收到信息,那么它将从Inactive状态重新回到Active状态.当所有的顶点都处于Inactive的时候,图运算将自动停止.

### 实验问题:求一个图的K-Core

这里解决的问题是求一个图的K-Core.

K-core是一个图的子图,使得该子图的所有顶点的度都不小于K.

输入前两行是顶点数和边数,之后每一行为一个由空格隔开的三元整数组,分别代表一条边的起点和终点,以及该边的权重(本题用不上).
{% codeblock lang:cpp %}
     num_vertex_in_this_partition
     num_edge_in_this_partition
     src_vertex_id dest_vertex_id
     src_vertex_id dest_vertex_id
     ...
{% endcodeblock %}
输出只需要列出K-Core子图中存在的顶点id即可.

对于每个顶点,需要维护两个值:`is_deleted`和`current_degree`,它们分别记录着当前顶点的活跃状态,以及当前顶点的度数.当度数小于K时,需要将该顶点逻辑上从图中删除,即将`is_deleted`置真.删除顶点后发送消息给它的相邻顶点,并在相邻顶点的`Compute()`函数中对自己的值进行相应的更新.

需要指出的是,这里担任各worker间全局同步工作的是`Aggregator`,算法输出正确的结果主要在于如何使用这个`Aggregator`.

我的想法是:
>* 对于一个顶点,`Compute()`开始的时候需要初始化顶点的度数,这一操作有GraphLite本身提供的方法来实现.
* 之后马上要检查该顶点是否已被删除,如果`is_deleted`显示它已被删除,那么它的`Compute()`将不执行任何操作,尤其是不得发送任何信息.它将保持对其他顶点的静默状态.
* 接着收取上一个超步(如果存在的话)接收到的所有信息,并对信息加和得到一个负数-n(后面会解释),该负数的绝对值n表明,在上一个超步结束之后,有n个顶点不符要求而被删除.
* 获取该顶点当前度数d,并计算d-n得到新的度数.
* 比较新的度数与K.如果一个顶点经过计算后发现自己要被删除了,首先它要置`is_deleted`为`true`标识它已被删除的状态.
* 接着它要给它的所有邻居顶点发送一条信息,该信息是一个整数"-1",表示要对度-1.
* 最后对本地的`Aggregator`值加上message的值,如果是0就+0,如果是-1就+(-1).
* 全局`Aggregator`的计算就是将本地的`Aggregator`值求和.如果得到的值不为0,则表明这个超步中还有顶点被删除,图没有达到稳定状态,继续进行下一个超步;如果值为0,那么说明这个超步中没有顶点被删除,则以后也不会有顶点被删除,图已经达到稳定状态,可以结束.
* 如果全局`Aggregator`为0,则进行的下一个超步所需要做的仅仅是对每个顶点调用`voteToHalt()`,使其变为不活跃状态.
* 而还在图中的顶点的`is_deleted`并没有改变,于是这时输出`is_deleted`为假的顶点即可.

其次就是对各输入输出函数的类型进行一致化,由于顶点数目较大,需要使用`int64_t`.

代码附下:
{% include_code source.cc %}
