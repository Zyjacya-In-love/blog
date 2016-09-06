---
title: Linux 搭建FTP服务器
date: 2016-07-06 10:43:47
tags:
- Linux
- FTP
---

在`Centos6.5`上搭建ftp服务器

<!--more-->

准备工作: `chkconfig --list`查看是否安装vsftpd服务

1. 安装`VSFTP`

    `sudo yum install vsftpd`
    
    - 创建用户`sudo adduser wxbftp` , 设置密码`sudo passwd wxbftp`

2. 配置`VSFTP`

    `sudo vi /etc/vsftpd/vsftpd.conf`
    
    ```bash
    anonymous_enable=NO  #关闭匿名访问
    
    local_enable=YES #允许本地用户登录
    
    write_enable=YES #允许ftp写操作
    
    local_umask=022
    dirmessage_enable=YES
    xferlog_enable=YES
    connect_from_port_20=YES
    xferlog_std_format=YES
    
    ascii_upload_enable=YES 
    ascii_download_enable=YES
    
    chroot_list_enable=YES
    chroot_list_file=/etc/vsftpd/chroot_list #在对应路径下创建chroot_list文件并把用户名"wxbftp"添加到里面
    
    listen=YES
    
    pam_service_name=vsftpd
    chroot_local_user=YES # 限制用户仅能访问自己的主目录  
    local_root=/data/test #设置用户的主目录(不设置时，默认为用户的家目录`/home/userftp`)
    # 在user_list文件中添加用户名"wxbftp"
    userlist_enable=YES
    userlist_deny=NO
    userlist_file=/etc/vsftpd/user_list
    # end file
    ```
3. 重启服务 `sudo service vsftpd restart`

4. 设置开机自启动 `sudo chkconfig vsftpd on`

### ftp服务管理

1. 启动ftp服务：service vsftpd start 
2. 查看ftp服务状态：service vsftpd status 
3. 重启ftp服务：service vsftpd restart 
4. 关闭ftp服务：service vsftpd stop 

### root登录

出于安全方面的考虑, `root`是不能登录`ftp`服务的, 通过下面的更改则可以root登录

1. 删除或注释/etc/vsftpd.ftpusers中的root
2. 删除或注释/etc/vsftpd.user_list中的root
3. 更改/etc/vsftpd/vsftp.conf中的userlist_enabel=NO
4. `sudo service vsftpd restart`
5. `sudo chkconfig vsftpd on`