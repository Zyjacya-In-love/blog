---
title: Caffe on Mac
toc: true
tags:
  - Caffe
  - Mac
categories:
  - Caffe
date: 2018-10-22 11:31:53
---

在Mac平台下编译caffe并编译caffe的Python接口, 添加到环境变量中.

<!--more-->

### **Python环境**

在Mac下, Python的环境有很多, 系统自带的Python环境一般不要动, 有些依赖可能因为系统Python而Crash, 可以使用`brew install`来安装一个Python环境, 这个路径和系统路径不同, 我的本机路径是`/usr/local/Cellar/python@2/2.7.15_1/bin/python`, 可以用`which python`来查看当前Python环境.

- `brew install python` : 安装Python

### **brew 指定版本**

如果之前使用`brew install protobuf`安装过`protobuf`,需要先执行`brew unlink protobuf`来'解绑'protobuf

- 查找可用的protobuf版本` brew search protobuf`
- 安装你需要的版本, 比如` brew install protobuf@3.1`
- `brew link protobuf@3.1`, 如果报错执行`brew link --overwrite --force protobuf@3.1`
- `protobuf -v`验证安装的protobuf版本

### **Caffe环境**

在Mac下我们是用Cmake来构建Caffe的, 保证cmake安装正确:

- `brew install cmake` : 安装Cmake

**安装Caffe依赖:**

- `brew install -vd snappy leveldb gflags glog szip lmdb`
- `brew install hdf5 opencv`
- `brew install --build-from-source --with-python -vd protobuf`
- `brew install --build-from-source -vd boost boost-python`

在system找不到`libboost_python.dylib`, 进入`/usr/local/Cellar/boost/1.67.0_1/lib`文件夹建立软链接:

`ln -sv ../../../boost-python/1.67.0/lib/libboost_python27-mt.dylib libboost_python-mt.dylib`

```bash
lrwxr-xr-x  1 simshang  admin    54B Oct 20 21:20 libboost_numpy-mt.a -> ../../../boost-python/1.67.0/lib/libboost_numpy27-mt.a
lrwxr-xr-x  1 simshang  admin    58B Oct 20 21:20 libboost_numpy-mt.dylib -> ../../../boost-python/1.67.0/lib/libboost_numpy27-mt.dylib
lrwxr-xr-x  1 simshang  admin    51B Oct 20 21:19 libboost_numpy.a -> ../../../boost-python/1.67.0/lib/libboost_numpy27.a
lrwxr-xr-x  1 simshang  admin    55B Oct 20 21:19 libboost_numpy.dylib -> ../../../boost-python/1.67.0/lib/libboost_numpy27.dylib
lrwxr-xr-x  1 simshang  admin    55B Oct 20 21:19 libboost_python-mt.a -> ../../../boost-python/1.67.0/lib/libboost_python27-mt.a
lrwxr-xr-x  1 simshang  admin    59B Oct 20 21:18 libboost_python-mt.dylib -> ../../../boost-python/1.67.0/lib/libboost_python27-mt.dylib
lrwxr-xr-x  1 simshang  admin    52B Oct 20 21:18 libboost_python.a -> ../../../boost-python/1.67.0/lib/libboost_python27.a
lrwxr-xr-x  1 simshang  admin    56B Oct 20 21:18 libboost_python.dylib -> ../../../boost-python/1.67.0/lib/libboost_python27.dylib
```

### **Caffe编译**

在编译Caffe的时候需要设置一下配置:

```bash
cmake 
-DCMAKE_PREFIX_PATH=/Users/simshang/GitProjects/caffe/build/install 
-D python_version=2 
-DBOOST_ROOT=/usr/local/Cellar/boost/1.67.0_1/ 
-DBoost_DEBUG=1 
-DPYTHON_LIBRARY=/usr/local/Cellar/python@2/2.7.15_1/Frameworks/Python.framework/Versions/Current/lib/libpython2.7.dylib
..
```

1. mkdir build
2. copy build_caffe.sh into build dir and run it
3. make all -j8
4. make runtest -j8
5. make pytest -j8
6. make install

> 将pycaffe添加到环境变量中, `vim bash_profile`, 然后`import caffe`

export PYTHONPATH=/Users/simshang/GitProjects/caffe-on-mac/python/:$PYTHONPATH