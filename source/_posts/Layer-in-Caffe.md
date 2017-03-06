---
title: Layer in Caffe
toc: true
tags:
  - Layers
  - 工厂模式
categories:
  - Caffe源码
date: 2017-01-24 11:38:05
---

Caffe中的数据的存储交换以及操作都是以blob的形式进行的，layer是模型和计算的基础，net整和并连接layer, 其中Layer层是整个框架的基本计算模块.

<!--more-->

所有的Pooling，Convolve，apply nonlinearities等操作都在这里实现。在Layer中`input data`用`bottom`表示, `output data`用`top`表示。每一层定义了三种操作`setup（Layer初始化）`, `forward（正向传导，根据input计算output）`, `backward（反向传导计算，根据output计算input的梯度）`

### **重要成员函数**

`前向传播`和`后向传播`是caffe中layer层的核心计算单元, Caffe中所有的Layer都要用这两种方法传递数据, Forward和Backward有CPU和GPU（部分有）两种实现, 在`layer.hpp`中,  

```c++
/** @brief Using the CPU device, compute the layer output. */
virtual void Forward_cpu(const vector<Blob<Dtype>*>& bottom,
  const vector<Blob<Dtype>*>& top) = 0;
  // 前向传播虚函数  
 
/**
* @brief Using the GPU device, compute the layer output.
*        Fall back to Forward_cpu() if unavailable.
*/
virtual void Forward_gpu(const vector<Blob<Dtype>*>& bottom,
  const vector<Blob<Dtype>*>& top) {
// LOG(WARNING) << "Using CPU code as backup.";
return Forward_cpu(bottom, top);
}

/**
* @brief Using the CPU device, compute the gradients for any parameters and
*        for the bottom blobs if propagate_down is true.
*/
virtual void Backward_cpu(const vector<Blob<Dtype>*>& top,
  const vector<bool>& propagate_down,
  const vector<Blob<Dtype>*>& bottom) = 0;
  // 后向传播虚函数
  
/**
* @brief Using the GPU device, compute the gradients for any parameters and
*        for the bottom blobs if propagate_down is true.
*        Fall back to Backward_cpu() if unavailable.
*/
virtual void Backward_gpu(const vector<Blob<Dtype>*>& top,
  const vector<bool>& propagate_down,
  const vector<Blob<Dtype>*>& bottom) {
// LOG(WARNING) << "Using CPU code as backup.";
Backward_cpu(top, propagate_down, bottom);
}
```
- Layer类派生出来的层类通过这实现两个虚函数(注释处)，产生了各式各样功能的层类。Forward是从根据bottom计算top的过程，Backward则相反（根据top计算bottom）。

为什么用了一个包含Blob的容器（vector），对于大多数Layer来说输入和输出都各连接只有一个Layer，然而对于某些Layer存在一对多的情况，比如LossLayer和某些连接层。在网路结构定义文件（*.proto）中每一层的参数bottom和top数目就决定了vector中元素数目。

### **重要成员变量**

```c++
LayerParameter layer_param_;           // 这个是protobuf文件中存储的layer参数
vector<share_ptr<Blob<Dtype>>> blobs_; // 这个存储的是layer的参数，在程序中用的
vector<bool> param_propagate_down_;    // 这个bool表示是否计算各个blob参数的diff，即传播误差
vector<Dtype> loss_;
```
- `blobs_`是Layer学习到的参数

- 每一层有一个loss值，只不多大多数Layer都是0，只有LossLayer才可能产生非0的loss。计算loss是会把所有层的`loss_`相加。

### **Layer派生类**

![](/img/Layer-in-Caffe/layer.png)

1. **NeuronLayer类** 

  定义于neuron_layers.hpp中，其派生类主要是元素级别的运算（比如Dropout运算，激活函数ReLu，Sigmoid等），运算均为同址计算（in-place computation，返回值覆盖原值而占用新的内存）。

2. **LossLayer类** 

  定义于loss_layers.hpp中，其派生类会产生loss，只有这些层能够产生loss。

3. **数据层** 

  定义于data_layer.hpp中，作为网络的最底层，主要实现数据格式的转换。

4. **特征表达层**

  定义于vision_layers.hpp，实现特征表达功能，更具体地说包含卷积操作，Pooling操作，他们基本都会产生新的内存占用（Pooling相对较小）。

5. **网络连接层和激活函数**

  定义于common_layers.hpp，Caffe提供了单个层与多个层的连接，并在这个头文件中声明。这里还包括了常用的全连接层InnerProductLayer类。


### **工厂方法模式**

工厂方法模式包含如下角色：

- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

![](/img/Layer-in-Caffe/FactoryMethod.jpg)

```c++
///////////////////////////////////////////////////////////
//  ConcreteFactory.cpp
//  Implementation of the Class ConcreteFactory
///////////////////////////////////////////////////////////

#include "ConcreteFactory.h"
#include "ConcreteProduct.h"

Product* ConcreteFactory::factoryMethod(){

	return  new ConcreteProduct();
}
```

```c++
#include "Factory.h"
#include "ConcreteFactory.h"
#include "Product.h"
#include <iostream>
using namespace std;

int main(int argc, char *argv[])
{
	Factory * fc = new ConcreteFactory();
	Product * prod = fc->factoryMethod();
	prod->use();
	
	delete fc;
	delete prod;
	
	return 0;
}
```

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。这个核心类仅仅负责给出具体工厂必须实现的接口，而不负责哪一个产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品。

> [工厂方法模式(Factory Method Pattern)](http://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/factory_method.html)

### **创建Layer**

在caffe创建layer的时候即使用到的工厂方法模式, 下面我们先来看两个很重要的宏定义. 

在宏定义中, `#`是把参数字符串化，`##`是连接两个参数成为一个整体

第一个宏定义` REGISTER_LAYER_CLASS`实际上是为每一个layer创建一个creator函数:

```c++
#define REGISTER_LAYER_CLASS(type)                                             \
  template <typename Dtype>                                                    \
  shared_ptr<Layer<Dtype> > Creator_##type##Layer(const LayerParameter& param) \
  {                                                                            \
    return shared_ptr<Layer<Dtype> >(new type##Layer<Dtype>(param));           \
  }                                                                            \
  REGISTER_LAYER_CREATOR(type, Creator_##type##Layer)
```
REGISTER_LAYER_CREATOR负责将创建层的函数放入LayerRegistry

```c++
#define REGISTER_LAYER_CREATOR(type, creator)                                  \
  static LayerRegisterer<float> g_creator_f_##type(#type, creator<float>);     \
  static LayerRegisterer<double> g_creator_d_##type(#type, creator<double>)    \
```
以EuclideanLossLayer为例, 在该类的最后, 调用 REGISTER_LAYER_CLASS(EuclideanLoss);来注册这一个类. 通过上面的两个宏定义, 实际上是"创建"了下面的函数.
```c++
template <typename Dtype>   
// create一个EuclideanLossLayer对象, 并返回对象指针                                                 
shared_ptr<Layer<Dtype> > Creator_EuclideanLossLayer(const LayerParameter& param) 
{                                                                            
  return shared_ptr<Layer<Dtype> >(new EuclideanLossLayer<Dtype>(param));           
}                                                                                       \
static LayerRegisterer<float> g_creator_f_EuclideanLoss("EuclideanLoss", Creator_EuclideanLossLayer<float>);     
static LayerRegisterer<double> g_creator_d_EuclideanLoss("EuclideanLoss", Creator_EuclideanLossLayer<double>);
```
**LayerRegistry**

```c++
template <typename Dtype>
class LayerRegistry {
 public:
  typedef shared_ptr<Layer<Dtype> > (*Creator)(const LayerParameter&);// 定义类型 Creator为函数句柄
  typedef std::map<string, Creator> CreatorRegistry; // 定义一个 <LayerName, CreatorHandler>的Creator列表

  static CreatorRegistry& Registry() {
    static CreatorRegistry* g_registry_ = new CreatorRegistry();
    return *g_registry_;
  }
```

**LayerRegisterer**

构造函数只做一件事: 在LayerRegistry的registry list中, 添加一个layer的creator

```c++
template <typename Dtype>
class LayerRegisterer {
 public:
  LayerRegisterer(const string& type,
                  shared_ptr<Layer<Dtype> > (*creator)(const LayerParameter&)) {
    // LOG(INFO) << "Registering layer type: " << type;
    LayerRegistry<Dtype>::AddCreator(type, creator);
  }
};

// Adds a creator.向Registry列表中添加一组<layername, creatorhandlr>
  static void AddCreator(const string& type, Creator creator) {
    CreatorRegistry& registry = Registry();
    CHECK_EQ(registry.count(type), 0)
        << "Layer type " << type << " already registered.";
    registry[type] = creator;
  }
  
```

**总结 : 创建一个新layer后, 先写一个静态函数创建并返回该函数的对象 (Creator), 然后创建对应的LayerRegisterer对象, 该对象在构造时会调用 LayerRegistry 中的 AddCreator, 将该layer 注册到 registy中去.**

> [参考博客: Caffe源码解析之Layer](http://imbinwang.github.io/blog/inside-caffe-code-layer)
