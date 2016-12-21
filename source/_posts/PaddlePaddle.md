---
title: PaddlePaddle
toc: true
tags:
  - Paddle
categories:
  - 深度学习
date: 2016-12-20 19:26:51
---
在2016百度世界大会上开源的分布式深度学习框架[Paddle](https://github.com/baidu/Paddle), 这篇文章是PaddlePaddle的官方文档里关于图像处理方面的学习记录。
<!--more-->

[Paddle开发文档](http://www.paddlepaddle.org/doc_cn/demo/quick_start/index.html)

看了极客头条的 [基于Spark的异构分布式深度学习平台](http://geek.csdn.net/news/detail/58867)文章特别期待Spark on Paddle,  [相关视频](https://spark-summit.org/2016/events/scalable-deep-learning-platform-on-spark-in-baidu/)

### **Docker install**

毫无疑问用docker安装最为方便, 由于我的云主机支持`AVX`, 所以就安装`cpu-demo-latest`版的Paddle

1. 首先从 [hub.docker](https://hub.docker.com/r/paddledev/paddle/)上将镜像pull到本地

   `sudo docker pull paddledev/paddle:cpu-demo-latest`

2. 创建Paddle容器

   `docker run -i -t -d -P --name paddle -v /home/shang:/home/ paddle:shangyan`

[源码浏览](http://162.243.141.242/paddle_html/codebrowser/codebrowser/paddle/)

3. 在容器中安装anaconda, 然后更新镜像

   `docker commit -m"add anaconda" 8fec2af6ccf8`

   - `8fec2af6ccf8`是容器id

> 如果是GPU版本的docker, 使用下面的把bash脚本启动容器:

```bash
export CUDA_SO="$(\ls /usr/lib64/libcuda* | xargs -I{} echo '-v {}:{}') $(\ls /usr/lib64/libnvidia* | xargs -I{} echo '-v {}:{}')"
export DEVICES=$(\ls /dev/nvidia* | xargs -I{} echo '--device {}:{}')
sudo docker run ${CUDA_SO} ${DEVICES} --privileged=true -i -t -d -P --name paddle_gpu_shangyan -v /home/shang:/home/ paddle:shangyan
```
- 将我的`/home/shang`文件夹映射到容器里的`/home`文件夹, 方便传送数据

### **图像分类**

> 该图像分类的任务以`CIFAR-10`为数据集, [Image Classification Tutorial-Paddle](http://www.paddlepaddle.org/doc/demo/image_classification/image_classification.html)

![](/img/PaddlePaddle/cifar.png)

**分类任务:**

![](/img/PaddlePaddle/image_classification.png)



1. 进入`paddle/demo/image_classification/data`, 运行`./download_cifar.sh`下载数据, 然后运行`./preprocess.sh`对数据进行预处理, 成功后:

  `root@7b0f9a66f43c:~/paddle/demo/image_classification/data/cifar-out/test# ls`
  `airplane  automobile  bird  cat  deer  dog  frog  horse  ship  truck`

  - `preprocess.sh` 调用了` ./demo/image_classification/preprocess.py` 去处理下载的原始图片

2. `vgg_16_cifar.py`是模型配置文件, 在训练模型之前要定义, 然后运行`./train.sh`训练模型, 输出一些信息:

  ```
  I1221 09:20:47.534273  1044 Trainer.cpp:170] trainer mode: Normal
  I1221 09:20:47.839715  1044 PyDataProvider2.cpp:257] loading dataprovider image_provider::processData
  [INFO 2016-12-21 09:20:47,855 image_provider.py:51] Image size: 32
  [INFO 2016-12-21 09:20:47,855 image_provider.py:52] Meta path: data/cifar-out/batches/batches.meta
  [INFO 2016-12-21 09:20:47,855 image_provider.py:58] DataProvider Initialization finished
  I1221 09:20:47.855934  1044 PyDataProvider2.cpp:257] loading dataprovider image_provider::processData
  [INFO 2016-12-21 09:20:47,856 image_provider.py:51] Image size: 32
  [INFO 2016-12-21 09:20:47,856 image_provider.py:52] Meta path: data/cifar-out/batches/batches.meta
  [INFO 2016-12-21 09:20:47,856 image_provider.py:58] DataProvider Initialization finished
  I1221 09:20:47.856549  1044 GradientMachine.cpp:134] Initing parameters..
  I1221 09:20:48.485172  1044 GradientMachine.cpp:141] Init parameters done.
  .........
  I1221 09:23:26.772271  1044 TrainerInternal.cpp:165]  Batch=100 samples=12800 AvgCost=2.38281 CurrentCost=2.38281 Eval: classification_error_evaluator=0.8325  CurrentEval: classification_error_evaluator=0.8325
  ```

  - 如果没有GPU, 将shell中的`use_gpu=0`

4. `./predict.sh`预测: 训练好的参数保存在`./cifar_vgg_model/`

  ```bash
  model=cifar_vgg_model/pass-00299/
  image=data/cifar-out/test/airplane/seaplane_s_000978.png
  use_gpu=1
  python prediction.py $model $image $use_gpu
  ```

  - the model of the `300-th` pass is stored at `./cifar_vgg_model/pass-00299`

### **ResNet**

ResNet就不多介绍了, 请看[ResNet介绍-Paddle](http://www.paddlepaddle.org/doc/demo/imagenet_model/resnet_model.html)

1. 运行`get_model.sh`获得ResNet的paddle模型文件, 如果下载不了移步 [百度云](http://pan.baidu.com/s/1o78C9su), 然后根据shell文件中的命令解压, 完成后的文件目录如下:

  `root@7b0f9a66f43c:~/paddle/demo/model_zoo/resnet/model# ls`
  `mean_meta_224`  `resnet_101`  `resnet_152`  `resnet_50`

  - `resnet_50:` model of 50 layers.
  - `resnet_101:` model of 101 layers.
  - `resnet_152:` model of 152 layers.
  - `mean_meta_224:` mean file with 3 x 224 x 224 size in BGR order. You also can use three mean values: 103.939, 116.779, 123.68.

2. 可视化`ResNet_50`网络结构, 运行`./net_diagram.sh`, 生成`ResNet_50.dot`文件

  - 使用`dot -T<type> -o<outfile> <infile.dot>`命令将dot文件转为png, 如下:

  `dot -T png -o ResNet_50.png ResNet_50.dot`

  生成50层的ResNet网络结构如下:

  ![](/img/PaddlePaddle/ResNet_50.png)

3. 用ResNet抽取特征, 运行`./extract_fea_py.sh`, 最终输出:`[INFO 2016-12-21 08:29:45,764 classify.py:219] Done: make image feature batch`, 生成`features`文件夹:

  `root@7b0f9a66f43c:~/paddle/demo/model_zoo/resnet/features# ls`
  `batch_0`
  `python load_feature.py ./features`
  ```
  {
  'cat.jpg': {'res5_3_branch2c_conv': array([-0.11962075, -0.10988633, -0.12072118, ..., -0.01200525,
        0.02182791, -0.00304621], dtype=float32), 'res5_3_branch2c_bn': array([-1.33475351, -1.20347679, -1.34959376, ..., -1.50335991,
       -1.01844168, -1.37495327], dtype=float32)},
  'dog.jpg': {'res5_3_branch2c_conv': array([-0.11691939, -0.11028718, -0.08920972, ...,  0.00291708,
        0.0172889 , -0.08754676], dtype=float32), 'res5_3_branch2c_bn': array([-1.29832339, -1.20888269, -0.92463595, ..., -1.28948379,
       -1.08349776, -2.58606839], dtype=float32)}
  }
  ```

4. 对测试图片进行预测分类, `./predict.sh`, 最终输出如下:

  ```
  [INFO 2016-12-21 08:37:47,909 layers.py:1985] output size for avgpool is 1*1
  I1221 08:37:47.910166   928 Util.cpp:155] commandline:  --use_gpu=1
  I1221 08:37:50.655699   928 Util.cpp:130] Calling runInitFunctions
  I1221 08:37:50.656028   928 Util.cpp:143] Call runInitFunctions done.
  I1221 08:37:52.103798   928 GradientMachine.cpp:123] Loading parameters from model/resnet_50
  [INFO 2016-12-21 08:37:55,102 classify.py:179] Label of example/dog.jpg is: 156
  [INFO 2016-12-21 08:37:57,137 classify.py:179] Label of example/cat.jpg is: 282
  ```

  - predict.sh调用了`classify.py`

> 具体的命令可以vim这些shell文件查看

**打个广告,一起来:**

  ![](/img/PaddlePaddle/ad.png)


