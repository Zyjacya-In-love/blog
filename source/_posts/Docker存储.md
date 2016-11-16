---
title: Docker存储
toc: true
tags:
  - Container
categories:
  - Docker
date: 2016-11-16 10:52:52
---

在布置caffe in docker的过程中, 发现docker容器的存储空间不够用了, 记录一下怎么给容器扩容

<!--more-->

### **基本信息**

docker默认单个容器可以使用数据空间大小10GB，docker可用数据总空间100GB，元数据可用总空间2GB。

- `docker info`可以查看Data Space Total、Metadata Space Total等信息:

```
Containers: 18
 Running: 6
 Paused: 0
 Stopped: 12
Images: 171
Server Version: 1.12.1
Storage Driver: devicemapper
 Pool Name: docker-253:0-25952953-pool
 Pool Blocksize: 65.54 kB
 Base Device Size: 10.74 GB
 Backing Filesystem: xfs
 Data file: /dev/loop0
 Metadata file: /dev/loop1
 Data Space Used: 63.73 GB
 Data Space Total: 107.4 GB
 Data Space Available: 43.64 GB
 Metadata Space Used: 54.65 MB
 Metadata Space Total: 2.147 GB
 Metadata Space Available: 2.093 GB
 Thin Pool Minimum Free Space: 10.74 GB
Kernel Version: 3.10.0-327.36.2.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 32
Total Memory: 62.7 GiB
Name: GPU-48
```

- `ll /var/lib/docker/devicemapper/devicemapper/ -h`

```
总用量 60G
-rw-------. 1 root root 100G 11月 16 10:48 data
-rw-------. 1 root root 2.0G 11月 16 10:56 metadata
```

- `docker exec caffe_gpu_shang df -hT`

```
Filesystem                                                                                         Type   Size  Used Avail Use% Mounted on
/dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697 xfs     10G  9.8G  233M  98% /
tmpfs                                                                                              tmpfs   32G     0   32G   0% /dev
tmpfs                                                                                              tmpfs   32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/centos_gpu--48-home                                                                    ext4   887G   82G  760G  10% /home
/dev/mapper/centos_gpu--48-root                                                                    ext4   1.4T  128G  1.3T  10% /etc/hosts
shm                                                                                                tmpfs   64M     0   64M   0% /dev/shm
```
可看到存储空间9.8G已占用98%, 当一个容器的数据空间大于10GB后，那么这个容器将不能写入新的数据文件。

### **容器扩容**

默认来说，如果你使用 Device Mapper 的存储插件，所有的镜像和容器是从一个初始 10G 的文件系统中创建的。

由于需要修改 Device Mapper 管理中的一些卷的信息，我们现在用 root 的身份来运行一些命令。所有以＃开头的命令都必须以 root 身份来执行。只要能访问 Docker 的 Socket 服务，你也可以用普通用户的身份来执行其他的命令（以$开头）。

- 让我们看一下 /dev/mapper ，那里应该有一个对应容器文件系统的符号链接，以 docker-X:Y-Z- 开头：

`cd /dev/mapper/`

```
centos_gpu--48-home
centos_gpu--48-root
centos_gpu--48-swap
control
docker-253:0-25952953-3617340e14b404910420956b8b7668551e632e0745e7f73c05809607dc0773c6
docker-253:0-25952953-3c748974ebd27c722dcbdd0f48c62da0dba377b8fd5cbe37a67378f55fcc9817
docker-253:0-25952953-4c12e865da624e71e96e81c4b41db9ff17f53b6d98d4a5a76ec955498fb62b4a
docker-253:0-25952953-68e8e3af52018ad6cb87b2b0f7f477784551e36ba084134b0c23580b2705da73
docker-253:0-25952953-76530f664d713798f5e6c2bde785a3f9659f2649b564b1bbc6733b3249f6cb82
docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697
docker-253:0-25952953-pool

```

- 进入自己的容器使用`df -h`查看:

```
Filesystem                                                                                          Size  Used Avail Use% Mounted on
/dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697   10G  9.8G  262M  98% /
tmpfs                                                                                                32G     0   32G   0% /dev
tmpfs                                                                                                32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/centos_gpu--48-home                                                                     887G   82G  761G  10% /home
/dev/mapper/centos_gpu--48-root                                                                     1.4T  128G  1.3T  10% /etc/hosts
shm                                                                                                  64M     0   64M   0% /dev/shm
```

- 返回到Host中的`/dev/mapper`文件夹执行:

`ls -l /dev/mapper/docker-*-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`

```
lrwxrwxrwx. 1 root root 7 11月 15 21:18 /dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697 -> ../dm-8
```

- 查看当前卷的信息表

`dmsetup table docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`

```
0 20971520 thin 253:3 1966

第二个数字是设备的大小，表示有20971520个` 512－bytes `的扇区. 这个值略高于` 10GB `的大小
```

- 来计算一下一个` 20GB `的卷需要多少扇区

`echo $((20*1024*1024*1024/512))`
41943040

精简快照目标的一个神奇的特点是它不会限制卷的大小。当你创建它的时候，一个精简的卷使用0个块，当你开始往块里面写入的时候，它们会从共用的块池中进行分配。你可以写0个块，或者是10亿个块，这个和精简快照目标没关系。文件系统的大小只和 Device Mapper 表有关系，只是需要装载一个新的表，这个完全和之前的是一样的，但是有更多的扇区。

- 改变第二个数字，要非常小心保持其他的值不变

`echo 0 41943040 thin 253:3 1966 | dmsetup load docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`

`dmsetup resume docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`



- 使用下面的命令激活新表:

`resize2fs /dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`

**Centos7**

`xfs_growfs /dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697`

```
meta-data=/dev/mapper/docker-253:0-25952953-e44d175cbcf2a014478cbe0207f2709f96fa595a259404c8b1780a8552d67697 isize=256    agcount=16, agsize=163824 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=2621184, imaxpct=25
         =                       sunit=16     swidth=16 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=16 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 2621184 to 2621440

```

- 进入容器重新查看容量大小

`df -h`

```
Filesystem                                                                                          Size  Used Avail Use% Mounted on
/dev/mapper/docker-253:0-25952953-68e8e3af52018ad6cb87b2b0f7f477784551e36ba084134b0c23580b2705da73   20G  8.7G   12G  44% /
tmpfs                                                                                                32G     0   32G   0% /dev
tmpfs                                                                                                32G     0   32G   0% /sys/fs/cgroup
/dev/mapper/centos_gpu--48-home                                                                     887G   82G  761G  10% /home
/dev/mapper/centos_gpu--48-root                                                                     1.4T  132G  1.2T  10% /etc/hosts
shm                                                                                                  64M     0   64M   0% /dev/shm
```

大小为20G , 使用率44%

### **容器瘦身**

通过导出导入给容器瘦身

`sudo docker export (CID) | docker import - base`





