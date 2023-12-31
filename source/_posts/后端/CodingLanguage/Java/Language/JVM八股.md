---
title: JVM八股
tags:
  - Java
  - 基础知识
  - JVM
categories:
  - 后端
  - CodingLanguage
  - Java
  - Language
date: 2023-11-15 15:04:43
---
## ReadMe
## JVM
### 基础概念（定义、使用）
### 类加载
### 内存区域

### 内存分配
1. 对象优先在Eden分配，当Eden区没有足够空间进行分配时，JVM发起依次Minor GC
2. 大量连续内存空间对象（字符串、数组）直接进入老年代，避免放入新生代影响新生代的GC频率和成本。其由JVM决定，看堆内存大小以及对象大小
3. 长期存活的对象进入老年代，每个对象有对象年龄计数器，经历过MinorGC则增加，超过一定阈值进入。
4. 
### 垃圾回收（GarbageCollection）
#### 定义
##### 判断回收条件
1. 引用技术法，对象中由引用计数器统计引用数量，为0时则不再被使用。但无法回收互相引用的对象。
2. 可达性方法，从GC roots的根对象作为起始节点【一些常驻的异常对象，即JVM内部的对象，所有Synchronized同步锁的持有对象】根据引用关系，向下搜索，搜索的过程就是**引用链**，在链上是可达的。不在链上的需要被回收。
##### 回收策略
1. 分代回收，根据对象存活周期划分内存（代），不同代采用GC
	1. 新生代区域【Minor GC / Young GC】8：1：1的Eden，survivor。新创建的对象保存到Eden，多次经历GC后则从survivor到old。其中两个survivor中**必须有一个是空的**。
	2. 老年代区域【Full GC / Major GC(具体看是OldGC 还是Full GC)】老年代内存占比大，回收时间场，需要把存活对象重新整理成连续的空间，成本高。
	3. Mixed GC
2. 方法区回收，回收废弃的常量和类型，只有满足以下三个条件才回收。
	1. 所有实例被回收
	2. 加载该类的ClassLoader被回收
	3. Class对象无法通过任何途径访问，包括反射
##### 回收算法
1. **标记-清除**，根据引用链标记存活的对象，回收未被标记的对象。效率高但存在空间碎片化问题【无连续的空间会导致提前触发Full GC】
2. **标记-复制**，把内存分为两块大小相同的空间（1：1）或（8：1：1），复制存活对象。适用于新生代回收率极高的场景，不适合老年代。**分配担保机制**，超过10%的对象存活且超过survivior空间大小，忽略对象年龄直接进入老年代。
3. **标记-整理**，在标记-清除算法基础上，添加整理操作，**对象移动排序整理**。实际中，可以通过设定阈值决定整理，其余则是清除。
#### GC算法举例
##### G1
定义：GarbageFisrt 垃圾回收器，在Stop tow World时间的优化，其设计理念是**实现一个停顿时间可控的低延迟垃圾收集器**。
运行过程：
1. 初始标记（StopTowWorld），只标记GC roots能直接关联的对象，保证后续阶段用户程序并发运行时，新对象分配在正确的位置。
2. 并发标记（Not StopTowWorld），从GC roots顺着引用链遍历堆，找出存活对象，耗时较长，但是可以和用户线程并发执行。
3. 最终标记（StopTowWorld），处理并发标记阶段，用户线程继续运行产生的引用变动。
4. 筛选回收（StopTowWorld），根据以上三个阶段标记完成的数据，**计算出各个区域的回收价值和成本，再根据用户期望的停顿时间来决定回收多个Region，保证停顿时间可控。**
优缺点：


##### CMS
定义：Concurrent Mark Sweep并发停顿的收集器，使用于要求**响应速度较高**，停顿时间忍受度低的场景，**负责收集老年代区域，采用标记-清除**算法。
运行过程：
1. 初始标记（StopTowWorld），只标记GC roots能直接关联的对象
2. 并发标记（CMS Concurrnet mark），不需要StopTowWorld,从直接关联的对象开始遍历引用链，**用户线程可以和GC线程并发**。
3. 重新标记（CMS remark），需要StopTowWorld，修正用户线程继续运行导致变动的一部分对象，该阶段可以多线程并行标记。
4. 并发清理（Concurrent sweep）, 遍历整个老年代内存空间，清理掉可回收的对象。清理完成后，重置CMS收集器的数据结构，等待下一次垃圾回收。
优缺点：
- 用两次断案赞的StopTowWorld代替整段长时间的StopTowWorld
- 存在大量内存碎片，GC进行时会降低吞吐量，产生浮动垃圾（用户线程的继续执行会不断产生新的垃圾）
