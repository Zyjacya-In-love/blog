---
title: Caffe Tools
toc: true
tags:
  - Caffe
categories:
  - Caffe
date: 2016-10-23 19:23:06
---

在编译完caffe之后, 在` ./build/tools`下面可以找到caffe本身提供的工具

<!--more-->

### **命令行**

如果要执行caffe程序，都需要加` ./build/tools/ `前缀,

`sudo sh ./build/tools/caffe train --solver=examples/mnist/train_lenet.sh`

**train : 训练或finetune模型（model)**

其中的`<args>`参数有：

- `-solver` : 必选参数, 模型的配置文件

`# ./build/tools/caffe train -solver examples/mnist/lenet_solver.prototxt`

- `-gpu` : 可选参数, 指定用哪一块gpu进行训练, `-gpu all`使用所有的gpu

`# ./build/tools/caffe train -solver examples/mnist/lenet_solver.prototxt -gpu 2`

- `-snapshot` : 可选参数, 指定从快照（snapshot)中恢复训练, 可以在solver配置文件设置快照，保存solverstate。

`# ./build/tools/caffe train -solver examples/mnist/lenet_solver.prototxt -snapshot examples/mnist/lenet_iter_5000.solverstate`

- `-weights` : 可选参数, 加载预先训练好的caffemodel模型来fine-tuning新的模型，不能和-snapshot同时使用

`# ./build/tools/caffe train -solver examples/finetuning_on_flickr_style/solver.prototxt -weights models/bvlc_reference_caffenet/bvlc_reference_caffenet.caffemodel`

- `-iteration` : 可选参数，迭代次数，默认为50。 如果在配置文件文件中没有设定迭代次数，则默认迭代50次。
- `-model` : 可选参数，定义在protocol buffer文件中的模型。也可以在solver配置文件中指定。
- `-sighup_effect` : 可选参数。用来设定当程序发生挂起事件时，执行的操作，可以设置为snapshot, stop或none, 默认为snapshot
- `-sigint_effect` : 可选参数。用来设定当程序发生键盘中止事件时（ctrl+c), 执行的操作，可以设置为snapshot, stop或none, 默认为stop

**test : 测试模型**

`test`参数用在测试阶段，用于最终结果的输出，要模型配置文件中我们可以设定需要输入accuracy还是loss.

`# ./build/tools/caffe test -model examples/mnist/lenet_train_test.prototxt -weights examples/mnist/lenet_iter_10000.caffemodel -gpu 0 -iterations 100`

- 利用训练好了的权重（-weight)，输入到测试模型中(-model)，用编号为0的gpu(-gpu)测试100次(-iteration)

**time : 显示程序执行时间**

time参数用来在屏幕上显示程序运行时间

`# ./build/tools/caffe time -model examples/mnist/lenet_train_test.prototxt -iterations 10`
- 这个例子用来在屏幕上显示lenet模型迭代10次所使用的时间。包括每次迭代的forward和backward所用的时间，也包括每层forward和backward所用的平均时间。

**device_query : 显示gpu信息**

device_query参数用来诊断gpu信息

`# ./build/tools/caffe device_query -gpu all`

```
I1119 06:48:51.387487   966 caffe.cpp:138] Querying GPUs all
I1119 06:48:51.495178   966 common.cpp:177] Device id:                     0
I1119 06:48:51.495232   966 common.cpp:178] Major revision number:         3
I1119 06:48:51.495241   966 common.cpp:179] Minor revision number:         5
I1119 06:48:51.495249   966 common.cpp:180] Name:                          Tesla K40m
I1119 06:48:51.495256   966 common.cpp:181] Total global memory:           11995578368
I1119 06:48:51.495268   966 common.cpp:182] Total shared memory per block: 49152
I1119 06:48:51.495411   966 common.cpp:183] Total registers per block:     65536
I1119 06:48:51.495424   966 common.cpp:184] Warp size:                     32
I1119 06:48:51.495432   966 common.cpp:185] Maximum memory pitch:          2147483647
I1119 06:48:51.495440   966 common.cpp:186] Maximum threads per block:     1024
I1119 06:48:51.495446   966 common.cpp:187] Maximum dimension of block:    1024, 1024, 64
I1119 06:48:51.495470   966 common.cpp:190] Maximum dimension of grid:     2147483647, 65535, 65535
I1119 06:48:51.495477   966 common.cpp:193] Clock rate:                    745000
I1119 06:48:51.495484   966 common.cpp:194] Total constant memory:         65536
I1119 06:48:51.495491   966 common.cpp:195] Texture alignment:             512
I1119 06:48:51.495498   966 common.cpp:196] Concurrent copy and execution: Yes
I1119 06:48:51.495506   966 common.cpp:198] Number of multiprocessors:     15
I1119 06:48:51.495512   966 common.cpp:199] Kernel execution timeout:      No
I1119 06:48:51.917004   966 common.cpp:177] Device id:                     1

```

### **lmdb转换**

在深度学习的实际应用中，我们经常用到的原始数据是图片文件，如jpg,jpeg,png,tif等格式的，而且有可能图片的大小还不一致。而在caffe中经常使用的数据类型是lmdb或leveldb，因此必须从原始图片文件转换成caffe中能够运行的db（leveldb/lmdb)文件, 在caffe中提供了convert_imageset.cpp，存放在根目录下的tools文件夹下。编译之后生成对应的可执行文件放在` buile/tools/` 下面，这个文件的作用就是用于将图片文件转换成caffe框架中能直接使用的db文件。

`convert_imageset [FLAGS] ROOTFOLDER/ LISTFILE DB_NAME`

需要带四个参数：

- `FLAGS`: 图片参数组，

- `ROOTFOLDER/`: 图片存放的绝对路径，从linux系统根目录开始

- `LISTFILE`: 图片文件列表清单，一般为一个txt文件，一行一张图片

- `DB_NAME`: 最终生成的db文件存放目录

**关于FLAGS参数组**

`-gray`: 是否以灰度图的方式打开图片。程序调用opencv库中的imread()函数来打开图片，默认为false

`-shuffle`: 是否随机打乱图片顺序。默认为false

`-backend`:需要转换成的db文件格式，可选为leveldb或lmdb,默认为lmdb

`-resize_width/resize_height`: 改变图片的大小。在运行中，要求所有图片的尺寸一致，因此需要改变图片大小。 程序调用opencv库的resize（）函数来对图片放大缩小，默认为0，不改变

`-check_size`: 检查所有的数据是否有相同的尺寸。默认为false,不检查

`-encoded`: 是否将原图片编码放入最终的数据中，默认为false

`-encode_type`: 与前一个参数对应，将图片编码为哪一个格式：png,jpg ...

我们以caffe自带的图片作为例子, 图片存放在`example/images/`下,

1. **生成图片清单**

`vim examples/images/createImageslist.sh` : 创建shell文件

```shell
# /usr/bin/env sh
DATA=examples/images
echo "Create train.txt..."
rm -rf $DATA/train.txt
find $DATA -name *cat.jpg | cut -d '/' -f3 | sed "s/$/ 1/">>$DATA/train.txt
find $DATA -name *bike.jpg | cut -d '/' -f3 | sed "s/$/ 2/">>$DATA/tmp.txt
cat $DATA/tmp.txt>>$DATA/train.txt
rm -rf $DATA/tmp.txt
echo "Done.."
```

`# bash examples/images/createImageslist.sh` 
Create train.txt...
Done..

生成了train.txt文件可作为`LISTFILE`的参数：

```
cat.jpg 1
fish-bike.jpg 2
```

2. **生成db文件**

`vim examples/images/creat_lmdb.sh` 添加以下内容:

```
#!/usr/bin/en sh
DATA=examples/images
rm -rf $DATA/img_train_lmdb
build/tools/convert_imageset --shuffle \
--resize_height=256 --resize_width=256 \
/root/caffe/examples/images/ $DATA/train.txt  $DATA/img_train_lmdb

```

- `shuffle`打乱图片顺序, 设置参数`resize_height`和`resize_width`将所有图片尺寸都变为256*256

```
# sh examples/images/creat_lmdb.sh 
I1119 07:18:30.111515  1004 convert_imageset.cpp:86] Shuffling data
I1119 07:18:30.674074  1004 convert_imageset.cpp:89] A total of 2 images.
I1119 07:18:30.674923  1004 db_lmdb.cpp:35] Opened lmdb examples/images/img_train_lmdb
I1119 07:18:30.690393  1004 convert_imageset.cpp:153] Processed 2 files.
# ls examples/images/img_train_lmdb/
data.mdb  lock.mdb
```

这样就把原始图片转化为lmdb文件

### **图片均值**

图片减去均值后，再进行训练和测试，会提高速度和精度, 计算所有训练样本的平均值，计算出来后，保存为一个均值文件, 在以后的测试中，就可以直接使用这个均值来相减，而不需要对测试图片重新计算。

1. 计算二进制格式的均值

caffe中使用的均值数据格式是`binaryproto`, 提供了一个计算均值的文件`compute_image_mean.cpp`，放在caffe根目录下的tools文件夹里面。编译后的可执行文件放在` build/tools/` 下面, 使用刚刚生成的`examples/images/img_train_lmdb/`lmdb文件

`./build/tools/compute_image_mean examples/images/img_train_lmdb/ examples/images/meam_shang.binaryproto`

在`examples/images/`下生成meam_shang.binaryproto文件

两个参数：

- `examples/images/img_train_lmdb/` : 表示需要计算均值的数据，格式为lmdb的训练数据

- `examples/images/meam_shang.binaryproto` : 计算出来的结果保存文件

2. 用python计算均值

```
#!/usr/bin/env python
import numpy as np
import sys,caffe

if len(sys.argv)!=3:
    print "Usage: python convert_mean.py mean.binaryproto mean.npy"
    sys.exit()

blob = caffe.proto.caffe_pb2.BlobProto()

bin_mean = open( sys.argv[1] , 'rb' ).read()
blob.ParseFromString(bin_mean)
arr = np.array( caffe.io.blobproto_to_array(blob) )
npy_mean = arr[0]
np.save( sys.argv[2] , npy_mean )
```
将python文件放在`examples/images/`目录下:

`python convert_mean.py meam_shang.binaryproto mean_shang.npy`

- mean.binaryproto 就是经过前面步骤计算出来的二进制均值。

- mean.npy就是我们需要的python格式的均值