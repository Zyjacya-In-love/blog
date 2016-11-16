---
title: caffe.proto
toc: true
tags:
  - ProtoBuffer
categories:
  - Caffe
date: 2016-10-11 11:34:34
---
讲解caffe的网络定义文件, 开启caffe的源码阅读系列

<!--more-->

### **Protocol Buffers**

[Google Protocol Buffer 的使用和原理](http://www.ibm.com/developerworks/cn/linux/l-cn-gpb/)

`Protocol Buffers `是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。目前提供了` C++、Java、Python `三种语言的 API。

**Protobuf的优点**

- Protobuf 有如 XML，不过它更小、更快、也更简单。你可以定义自己的数据结构，然后使用代码生成器生成的代码来读写这个数据结构。你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。它有一个非常棒的特性，即“向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。这样您的程序就可以不必担心因为消息结构的改变而造成的大规模的代码重构或者迁移的问题。因为添加新的消息中的 field 并不会引起已经发布的程序的任何改变。

- Protobuf 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。

- 使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

**Protobuf的不足**

- Protbuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。

- XML 已经成为多种行业标准的编写工具，Protobuf 只是 Google 公司内部使用的工具，在通用性上还差很多。

- 由于文本并不适合用来描述数据结构，所以 Protobuf 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 .proto 定义，否则你没法直接读出 Protobuf 的任何内容

### **文件解析**

`caffe.proto`位于…\src\caffe\proto目录下，在这个文件夹下还有一个.pb.cc和一个.pb.h文件，这两个文件都是由caffe.proto编译而来的。 
           
在caffe.proto中定义了很多结构化数据，包括：

- BlobProto
- Datum
- FillerParameter
- NetParameter
- SolverParameter
- SolverState
- LayerParameter
- ConcatParameter
- ConvolutionParameter
- DataParameter
- DropoutParameter
- HDF5DataParameter
- HDF5OutputParameter
- ImageDataParameter
- InfogainLossParameter
- InnerProductParameter
- LRNParameter
- MemoryDataParameter
- PoolingParameter
- PowerParameter
- WindowDataParameter
- V0LayerParameter

