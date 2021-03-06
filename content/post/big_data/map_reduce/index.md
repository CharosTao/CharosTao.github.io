---
title: "Map_reduce"
date: 2022-07-06T17:38:27+08:00
draft: false
categories: big_data
tags : 
    - Map_reduce
lastmod: 2022-07-06T21:23:00+08:00
description: "Mapreduce记录"
---

# MapReduce 

  MapReduce是处理和生成大数据集的编程模型和相关实现。
用户指定一个处理key/value对以生成一组中间key/value对的map函数，
以及一个合并同一中间key关联的所有中间值的reduce函数。
该模型可以表达许多现实世界的任务。
底层运行时系统会进行跨大规模机器集群的并行计算，
处理机器故障，并调度机器间通信以有效利用网络和磁盘.

  MapReduce受到了函数式编程中map和reduce函数的启发，
提供简单而强大的接口，可以实现大规模计算的并行和分布，
使用该接口可以实现高性能的大型商业计算集群。

## 编程模型

MapReduce的输入和输出都是key/value对，
用户需要实现map和reduce函数。

用户使用map函数将输入的key/value对转化为中间key/value对，
MapReduce将所有同一中间key关联的所有value组合在一起并
将其传递给reduce函数。

用户使用reduce函数接受同一中间key的所有value，对这些
value进行计算和处理，通常来说处理的结果只有1个或0个。

```
map (k1, v1)  ->  list (k2, v2)

reduce (k2, list(v2))  ->  list (v2)

# 其中输入和输出的key/value不在同一域上，中间和输出在同一域上
```
### 运行过程

![MapReduce执行过程] (./MapReduce_overview.png "MapReduce overview")

上图为此 MapReduce 框架实现的示意图，下文基于此图对 MapReduce 的执行过程进行描述，
描述的序号与图中的序号相对应：

> 1. MapReduce 库会先把文件切分成 M 个片段（ 每个大小为 16MB~64MB ），存储在 GFS 文件系统 ，接着，它会在集群中启动多个 程序副本 。
> 2. 这些程序副本中，一个为 master ，剩余为 worker ，master 对 worker 进行任务分配，共有 M 个 map 任务以及 R 个 reduce 任务
（ M 同时为文件片段数 ， R 由用户指定），master 会给每个空闲的 worker 分配一个 map 任务或者一个 reduce 任务 。
> 3. 被分配了 map 任务的 worker 会读取相关的输入数据片段，这些数据片段一般位于该 worker 所在的服务器上（ master 调度时会优先使
map 任务执行在存储有相关输入数据的服务器上，通过这种 本地执行 的方式降低服务器间网络通信，节约网络带宽 ）。它会解析出输入数据
中的 键值对 ，并将它们传入用户定义的 Map 函数中，Map 函数所生成的 中间键值对 会被缓存在内存中 。
> 4. 每隔一段时间，被缓存的中间键值对会被写入到本地硬盘，并通过分区函数分到 R 个区域内 。这些被缓存的键值对
在本地硬盘的位置会被传回 master ，master 负责将这些位置转发给执行 reduce 任务的 worker 。
> 5. 所有 map 任务执行结束后，master 才开始分发 reduce 任务 。当某个执行 reduce 任务的 worker 从 master 获取到了这些位置信息，
该 worker 就会通过 RPC 的方式从保存了对应缓存中间数据的 map workers 的本地硬盘中读取数据 （ 输入一个 reduce 任务中的中间数
据会产生自所有 map 任务 ）。当一个 reduce worker 读完所有中间数据后，会 根据中间键进行排序，使得具有相同中间键的数据可以聚
合在一起 。（需要排序是因为中间 key 的数量一般远大于 R ，许多不同 key 会映射到同一个 reduce 任务中 ）如果中间数据的数据量太
大而无法放到内存中，需要使用外部排序 。
> 6. reduce worker 会对排序后的中间数据进行遍历，对于每个唯一的中间键，将该中间键和对应的中间值的集合传入用户提供的 Reduce 函
数中，Reduce 函数生成的输出会被追加到这个 reduce 任务分区的输出文件中 （ 即一个 reduce 任务对应一个输出文件，即 R 个输出文件，
存储在 GFS 文件系统，需要的话可作为另一个 MapReduce 调用的输入 ）。
> 7. 当所有的 map 和 reduce 任务完成后，master 会唤醒用户程序 。此时，用户程序会结束对 MapReduce 的调用 

### master数据结构

master节点维护着一些数据结构，它对每个map和reduce任务存储着状态（运行中，已完成和空闲），对每个非空闲任务的机器的标识。

### 错误容忍

1. worker错误 --> master节点每隔一段时间向worker发送ping维持连接，若连接失败，则master将会把worker标记为失败，并重置该worker的任务，并能重新调度新的任务。
已经完成的map任务会重新执行，因为map输出在worker节点的本地存储着；而reduce的不需要，其输出在全局文件系统中。出现错误worker的reduce任务会转给新的worker，并会被
告知给其它节点。
2. master错误 --> master一般有副本，若只有一个master，则需要客户端重试以执行任务。


### 本地存储

将输入数据（ 由 GFS 系统管理 ）存储在集群中服务器的本地硬盘上，GFS 将每个文件分割为大小为 64MB 的 Block ，
并且对每个 Block 保存多个副本（通常3个副本，分散在不同机器上）。master 调度 map 任务时会考虑输入数据文件
的位置信息，尽量在包含该相关输入数据的拷贝的机器上执行 map 任务 。若任务失败，master 尝试在保存输入数据副本
的邻近机器上执行 map 任务，以此来*节约网络带宽* 。

### 任务粒度

如上所述，我们将 map 阶段细分为 M 个片段，将 reduce 阶段细分为 R 个片段。理想情况下，M 和 R 应该远大于工作机器的数量。
让每个worker执行许多不同的任务可以改善动态负载平衡，并在worker发生故障时加快恢复速度：
它已完成的许多映射任务可以分布在所有其他worker机器上。在我们的实现中，M 和 R 的大小是有实际限制的，
因为 master 必须做出 O(M + R) 的调度决策并在内存中保持 O(M * R) 的状态，如上所述。 （然而，内存使用的常数因素很小：
状态的 O(M * R) 部分由每个 map 任务/reduce 任务对大约一个字节的数据组成。

此外，R 经常受到用户的限制，因为每个 reduce 任务的输出最终都在一个单独的输出文件中。
在实践中，我们倾向于选择 M 以便每个单独的任务大约是 16 MB 到 64 MB 的输入数据（这样上面描述的局部性优化是最有效的），
并且我们使 R 是worker数量的小倍数我们期望使用的机器。我们经常使用 2,000 台工作机器执行 M = 200,000 和 R = 5,000 的 MapReduce 计算

### 备用任务
此模式是为了缓解 straggler (掉队者) 问题 ，即 ：一台机器花费了异常多的时间去完成 最后几个 map 或 reduce 任务，
导致整个计算时间延长的问题 。可能是由于硬盘问题，可能是 CPU 、内存、硬盘和网络带宽的竞争而导致的 。

解决此问题的方法是：当一个 MapReduce 计算 接近完成 时，master 为正在执行中的任务执行 备用任务 ，
当此任务完成时，无论是主任务还是备用任务完成的，都将此任务标记为完成 。这种方法虽然多使用了一些计算资源，
但是有效降低了 MapReduce Job 的执行时间 。

### Combiner 函数
某些情况下，每个 map 任务生成的中间 key 会有明显重复，可使用 Combiner 函数 在 map worker 上将数据进行部分合并，
再传往 reduce worker 。

Combiner 函数 和 Reduce 函数的实现代码一样，区别在于两个函数输出不同，Combiner 函数的输出被写入中间文件，
Reduce 函数的输出被写入最终输出文件 。

这种方法可以提升某些类型的 MapReduce 任务的执行速度（ 如 word count 任务）。

### 临时中间文件
对于有服务器故障而可能导致的 reduce 任务可能读到部分写入的中间文件 的问题 。可以使用 临时中间文件 ，
即 map 任务将运算结果写入临时中间文件，一旦该文件完全生成完毕，以原子的方式对该文件重命名 。

## MapReduce 的优点
适合PB级以上海量数据的离线处理

隐藏了并行化、容错、数据分发以及负载均衡等细节

允许没有分布式或并行系统经验的程序员轻松开发分布式任务程序

伸缩性好，使用更多的服务器可以获得更多的吞吐量

## MapReduce 的限制
不擅长实时计算
无法进行流式计算，因为 MapReduce 的输入数据是静态的
无多阶段管道，对于先后依赖的任务，MapReduce 必须把数据写入硬盘，再由下一个 MapReduce 任务调用这些数据，造成了多余的磁盘 I/O

## 相关问题总结

### MapReduce 如何节约网络带宽
集群中所有服务器既执行 GFS ，也执行 MapReduce 的 worker

master 调度时会优先使 map 任务执行在存储有相关输入数据的服务器上

reduce worker 直接通过 RPC 从 map worker 获取中间数据，而不是通过 GFS ，因此中间数据只需要进行一次网络传输

R 远小于中间 key 的数量，因此中间键值对会被划分到一个拥有很多 key 的文件中，传输更大的文件（ 相对于一个文件拥有更少的 key ）效率更高

### MapReduce 如何获得好的负载均衡

通过备用任务缓解 straggler 问题
使 task 数远多于 worker 数，master 将空闲任务分给已经完成任务的 worker


## 改进

### 分区函数

### 保证顺序

### 输入输出类型

### 跳过不良记录

### 状态信息

### 计数器
