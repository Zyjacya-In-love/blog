---
title: Caffe.proto
toc: true
tags:
  - .proto
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

### **.proto**

`caffe.proto`位于…\src\caffe\proto目录下，在这个文件夹下还有一个.pb.cc和一个.pb.h文件，这两个文件都是由caffe.proto编译而来的。 

> [caffe.proto](https://github.com/BVLC/caffe/blob/master/src/caffe/proto/caffe.proto)的内容
           
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


1. `.proto `文件定义我们程序中需要处理的结构化数据，在` protobuf `的术语中，结构化数据被称为` Message`

2. `.proto `文件的文件名命名规则为` packageName.MessageName.proto`, 比如caffe.proto

3. `repeated`表示必选的数据, `optional`表示可选的数据, 后面紧跟数据类型

4. `protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/caffe.proto`编译.proto文件, 命令将生成两个文件：`caffe.pb.h`定义了 C++ 类的头文件和`caffe.pb.cc`C++ 类的实现文件, 在生成的头文件中，定义了一个C++类caffe，后面的 Writer 和 Reader 将使用这个类来对消息进行操作。诸如对消息的成员进行赋值，将消息序列化等等都有相应的方法。

  > 查看 [caffe.pb.h](https://github.com/Simshang/caffefile/blob/master/caffe.pb.h)内容
  > 查看 [caffe.pb.cc](https://github.com/Simshang/caffefile/blob/master/caffe.pb.cc)

5. 编写Writer将把一个结构化数据写入磁盘，以便其他人来读取, Writer 需要处理的结构化数据由 .proto 文件描述，经过上一节中的编译过程后，该数据化结构对应了一个C++的类，并定义在 caffe.pb.h 中, Writer 需要 include 该头文件，然后便可以使用这个类, 在 Writer 代码中，将要存入磁盘的结构化数据由一个 caffe 类的对象表示，它提供了一系列的` get/set `函数用来修改和读取结构化数据中的数据成员，或者叫` field`, 当我们需要将该结构化数据保存到磁盘上时，`caffe类 `已经提供相应的方法来把一个复杂的数据变成一个字节序列，我们可以将这个字节序列写入磁盘。
 
### **caffe.pb.cc**

其实caffe.pb.cc里面的东西都是从caffe.proto编译而来的，无非就是一些关于这些数据结构（类）的标准化操作，比如
 
 ```c++
   void CopyFrom();
   void MergeFrom();
   void CopyFrom();
   void MergeFrom();
   void Clear();
   bool IsInitialized() const;
   int ByteSize() const;
   bool MergePartialFromCodedStream();
   void SerializeWithCachedSizes() const;
   void SerializeWithCachedSizesToArray() const;
   int GetCachedSize()
   void SharedCtor();
   void SharedDtor();
   void SetCachedSize() const;
 ```
 
 **BlobProto**
 ```
 message BlobProto {//blob的属性以及blob中的数据(data\diff)
   optional int32 num = 1 [default = 0];
   optional int32 channels = 2 [default = 0];
   optional int32 height = 3 [default = 0];
   optional int32 width = 4 [default = 0];
   repeated float data = 5 [packed = true];
   repeated float diff = 6 [packed = true];
 }
 ```

**Datum**
 ```
   message Datum {
   optional int32 channels = 1;
   optional int32 height = 2;
   optional int32 width = 3;
   optional bytes data = 4;//真实的图像数据，以字节存储(bytes)
   optional int32 label = 5;
   repeated float float_data = 6;//datum也能存float类型的数据(float)
 }
 ```
 
 **LayerParameter**
 ```
 message LayerParameter {
   repeated string bottom = 2; //输入的blob的名字(string)
   repeated string top = 3; //输出的blob的名字(string)
   optional string name = 4; //层的名字
   enum LayerType { //层的枚举（enum，和c++中的enum一样）
     NONE = 0;
     ACCURACY = 1;
     BNLL = 2;
     CONCAT = 3;
     CONVOLUTION = 4;
     DATA = 5;
     DROPOUT = 6;
     EUCLIDEAN_LOSS = 7;
     ELTWISE_PRODUCT = 25;
     FLATTEN = 8;
     HDF5_DATA = 9;
     HDF5_OUTPUT = 10;
     HINGE_LOSS = 28;
     IM2COL = 11;
     IMAGE_DATA = 12;
     INFOGAIN_LOSS = 13;
     INNER_PRODUCT = 14;
     LRN = 15;
     MEMORY_DATA = 29;
     MULTINOMIAL_LOGISTIC_LOSS = 16;
     POOLING = 17;
     POWER = 26;
     RELU = 18;
     SIGMOID = 19;
     SIGMOID_CROSS_ENTROPY_LOSS = 27;
     SOFTMAX = 20;
     SOFTMAX_LOSS = 21;
     SPLIT = 22;
     TANH = 23;
     WINDOW_DATA = 24;
   }
   optional LayerType type = 5; // 层的类型
   repeated BlobProto blobs = 6; //blobs的数值参数
   repeated float blobs_lr = 7; //学习速率(repeated)，如果你想那个设置一个blob的学习速率，你需要设置所有blob的学习速率。
   repeated float weight_decay = 8; //权值衰减(repeated)
 
   // 相对于某一特定层的参数(optional)
   optional ConcatParameter concat_param = 9;
   optional ConvolutionParameter convolution_param = 10;
   optional DataParameter data_param = 11;
   optional DropoutParameter dropout_param = 12;
   optional HDF5DataParameter hdf5_data_param = 13;
   optional HDF5OutputParameter hdf5_output_param = 14;
   optional ImageDataParameter image_data_param = 15;
   optional InfogainLossParameter infogain_loss_param = 16;
   optional InnerProductParameter inner_product_param = 17;
   optional LRNParameter lrn_param = 18;
   optional MemoryDataParameter memory_data_param = 22;
   optional PoolingParameter pooling_param = 19;
   optional PowerParameter power_param = 21;
   optional WindowDataParameter window_data_param = 20;
   optional V0LayerParameter layer = 1;
 }
 ```
 **NetParameter**
 ```
 message NetParameter {
   optional string name = 1;//网络的名字
   repeated LayerParameter layers = 2; //repeated类似于数组
   repeated string input = 3;//输入层blob的名字
   repeated int32 input_dim = 4;//输入层blob的维度，应该等于(4*#input)
   optional bool force_backward = 5 [default = false];//网络是否进行反向传播。如果设置为否，则由网络的结构和学习速率来决定是否进行反向传播。
 }
 ```
 **SolverParameter**
 ```
 message SolverParameter {
   optional string train_net = 1; // 训练网络的proto file
   optional string test_net = 2; // 测试网络的proto file
   optional int32 test_iter = 3 [default = 0]; // 每次测试时的迭代次数
   optional int32 test_interval = 4 [default = 0]; // 两次测试的间隔迭代次数
   optional bool test_compute_loss = 19 [default = false];
   optional float base_lr = 5; // 基本学习率
   optional int32 display = 6; // 两次显示的间隔迭代次数
   optional int32 max_iter = 7; // 最大迭代次数
   optional string lr_policy = 8; // 学习速率衰减方式
   optional float gamma = 9; // 关于梯度下降的一个参数
   optional float power = 10; // 计算学习率的一个参数
   optional float momentum = 11; // 动量
   optional float weight_decay = 12; // 权值衰减
   optional int32 stepsize = 13; // 学习速率的衰减步长
   optional int32 snapshot = 14 [default = 0]; // snapshot的间隔
   optional string snapshot_prefix = 15; // snapshot的前缀
   optional bool snapshot_diff = 16 [default = false]; // 是否对于 diff 进行 snapshot
   enum SolverMode {
     CPU = 0;
     GPU = 1;
   }
   optional SolverMode solver_mode = 17 [default = GPU]; // solver的模式，默认为GPU
   optional int32 device_id = 18 [default = 0]; // GPU的ID
   optional int64 random_seed = 20 [default = -1]; // 随机数种子
 }
 ```

**DataParameter**

```
message DataParameter {
  enum DB {
    LEVELDB = 0;
    LMDB = 1;
  }
  // Specify the data source.
  optional string source = 1;
  // Specify the batch size.
  optional uint32 batch_size = 4;
  // The rand_skip variable is for the data layer to skip a few data points
  // to avoid all asynchronous sgd clients to start at the same point. The skip
  // point would be set as rand_skip * rand(0,1). Note that rand_skip should not
  // be larger than the number of keys in the database.
  // DEPRECATED. Each solver accesses a different subset of the database.
  optional uint32 rand_skip = 7 [default = 0];
  optional DB backend = 8 [default = LEVELDB];
  // DEPRECATED. See TransformationParameter. For data pre-processing, we can do
  // simple scaling and subtracting the data mean, if provided. Note that the
  // mean subtraction is always carried out before scaling.
  optional float scale = 2 [default = 1];
  optional string mean_file = 3;
  // DEPRECATED. See TransformationParameter. Specify if we would like to randomly
  // crop an image.
  optional uint32 crop_size = 5 [default = 0];
  // DEPRECATED. See TransformationParameter. Specify if we want to randomly mirror
  // data.
  optional bool mirror = 6 [default = false];
  // Force the encoded image to have 3 color channels
  optional bool force_encoded_color = 9 [default = false];
  // Prefetch queue (Number of batches to prefetch to host memory, increase if
  // data access bandwidth varies).
  optional uint32 prefetch = 10 [default = 4];
}
```

**LayerParameter**

```
// LayerParameter next available layer-specific ID: 147 (last added: recurrent_param)
message LayerParameter {
  optional string name = 1; // the layer name
  optional string type = 2; // the layer type
  repeated string bottom = 3; // the name of each bottom blob
  repeated string top = 4; // the name of each top blob

  // The train / test phase for computation.
  optional Phase phase = 10;

  // The amount of weight to assign each top blob in the objective.
  // Each layer assigns a default value, usually of either 0 or 1,
  // to each top blob.
  repeated float loss_weight = 5;

  // Specifies training parameters (multipliers on global learning constants,
  // and the name and other settings used for weight sharing).
  repeated ParamSpec param = 6;

  // The blobs containing the numeric parameters of the layer.
  repeated BlobProto blobs = 7;

  // Specifies whether to backpropagate to each bottom. If unspecified,
  // Caffe will automatically infer whether each input needs backpropagation
  // to compute parameter gradients. If set to true for some inputs,
  // backpropagation to those inputs is forced; if set false for some inputs,
  // backpropagation to those inputs is skipped.
  //
  // The size must be either 0 or equal to the number of bottoms.
  repeated bool propagate_down = 11;

  // Rules controlling whether and when a layer is included in the network,
  // based on the current NetState.  You may specify a non-zero number of rules
  // to include OR exclude, but not both.  If no include or exclude rules are
  // specified, the layer is always included.  If the current NetState meets
  // ANY (i.e., one or more) of the specified rules, the layer is
  // included/excluded.
  repeated NetStateRule include = 8;
  repeated NetStateRule exclude = 9;

  // Parameters for data pre-processing.
  optional TransformationParameter transform_param = 100;

  // Parameters shared by loss layers.
  optional LossParameter loss_param = 101;

  // Layer type-specific parameters.
  //
  // Note: certain layers may have more than one computational engine
  // for their implementation. These layers include an Engine type and
  // engine parameter for selecting the implementation.
  // The default for the engine is set by the ENGINE switch at compile-time.
  optional AccuracyParameter accuracy_param = 102;
  optional ArgMaxParameter argmax_param = 103;
  optional BatchNormParameter batch_norm_param = 139;
  optional BiasParameter bias_param = 141;
  optional ConcatParameter concat_param = 104;
  optional ContrastiveLossParameter contrastive_loss_param = 105;
  optional ConvolutionParameter convolution_param = 106;
  optional CropParameter crop_param = 144;
  optional DataParameter data_param = 107;
  optional DropoutParameter dropout_param = 108;
  optional DummyDataParameter dummy_data_param = 109;
  optional EltwiseParameter eltwise_param = 110;
  optional ELUParameter elu_param = 140;
  optional EmbedParameter embed_param = 137;
  optional ExpParameter exp_param = 111;
  optional FlattenParameter flatten_param = 135;
  optional HDF5DataParameter hdf5_data_param = 112;
  optional HDF5OutputParameter hdf5_output_param = 113;
  optional HingeLossParameter hinge_loss_param = 114;
  optional ImageDataParameter image_data_param = 115;
  optional InfogainLossParameter infogain_loss_param = 116;
  optional InnerProductParameter inner_product_param = 117;
  optional InputParameter input_param = 143;
  optional LogParameter log_param = 134;
  optional LRNParameter lrn_param = 118;
  optional MemoryDataParameter memory_data_param = 119;
  optional MVNParameter mvn_param = 120;
  optional ParameterParameter parameter_param = 145;
  optional PoolingParameter pooling_param = 121;
  optional PowerParameter power_param = 122;
  optional PReLUParameter prelu_param = 131;
  optional PythonParameter python_param = 130;
  optional RecurrentParameter recurrent_param = 146;
  optional ReductionParameter reduction_param = 136;
  optional ReLUParameter relu_param = 123;
  optional ReshapeParameter reshape_param = 133;
  optional ScaleParameter scale_param = 142;
  optional SigmoidParameter sigmoid_param = 124;
  optional SoftmaxParameter softmax_param = 125;
  optional SPPParameter spp_param = 132;
  optional SliceParameter slice_param = 126;
  optional TanHParameter tanh_param = 127;
  optional ThresholdParameter threshold_param = 128;
  optional TileParameter tile_param = 138;
  optional WindowDataParameter window_data_param = 129;
}
```

**SoftmaxParameter**

```
// Message that stores parameters used by SoftmaxLayer, SoftmaxWithLossLayer
message SoftmaxParameter {
  enum Engine {
    DEFAULT = 0;
    CAFFE = 1;
    CUDNN = 2;
  }
  optional Engine engine = 1 [default = DEFAULT];

  // The axis along which to perform the softmax -- may be negative to index
  // from the end (e.g., -1 for the last axis).
  // Any other axes will be evaluated as independent softmaxes.
  optional int32 axis = 2 [default = 1];
}
```