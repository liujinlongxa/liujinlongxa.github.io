---
title: 使用ownCloud作为OmniFocus的云同步服务器
categories: 其他
tags:
  - 其他
  - WebDAV
  - ownCloud
date: 2016-08-24 22:47:22
---

平时一直使用OmniFocus作为自己的时间管理工具，但是由于我使用的是破解版的（正版太贵，买不起啊），同步就变成了一件很麻烦的事。OmniFocus提供了两种同步方式，如下图：

![1](http://7xn88v.com1.z0.glb.clouddn.com/901f5c4cb502f3fc557e3c3cfa79ad2f.png)

一种是使用Omni官方的服务器同步，这种方式需要注册Omni官方账号，应该只有正版能用吧，我没有尝试；另一种方式是使用WebDAV进行同步，这里着重介绍第二种方式。

本文主要分为以下几个部分：

1. WebDAV简介
2. ownCloud简介以及搭建方法
3. 使用ownCloud作为OmniFocus的同步服务器
4. ownCloud的一些其他应用

## WebDAV简介

[WebDAV](http://baike.baidu.com/link?url=pXeqnwgtPiYSKdYGX9YjBH_XeAJsH-aRkQqEYPsbkQwf9hlbnF9RSuAHvCMLboSuENx55T8J9Pd27E6ybo7Aba)是一种基于 HTTP 1.1协议的通讯协议，可以实现文件的同步。通俗来讲，我们可以利用WebDAV来实现类似百度云一样的网盘功能。

[这里](http://blog.sina.com.cn/s/blog_4834fe810101fw17.html)有几个支持WebDAV协议的网盘，但是都是国外的，国内据我了解目前只有[坚果云](http://www.jianguoyun.com/s/campaign/cpclanding/main?sch=bdcpc_nutstore_off&bdtj=1)一家支持WebDAV。至于国内的各大网盘为什么都不支持WebDAV，可以看[这里](http://www.zhihu.com/question/21511143)。

最初我也本来打算使用国外的网盘来实现WebDAV，但是担心速度不理想，而且国外网盘的免费空间都很小，所以就放弃了。至于坚果云，我也没有去尝试。

进一步Google，于是就找到了ownCloud。

## ownCloud简介以及搭建方法

[ownCloud](https://owncloud.org/)是一个来自KDE社区开发的免费软件，提供私人的Web服务。当前主要功能包括文件管理（内建文件分享）、音乐、日历、联系人等等，可在PC和服务器上运行。简单来说就是一个基于Php的自建网盘。

下面就跟着我使用ownCloud一步一步搭建自己的私人网盘。

### 准备工作

因为要把ownCloud搭建在服务器上，因此你需要有一个VPS，内存至少128M，越大越好，我这里使用的是[搬运工VPS](https://bandwagonhost.com/clientarea.php?action=products)，如何购买VPS可以看我的[这篇博客](http://liujinlongxa.com/2016/06/11/%E6%89%93%E9%80%A0%E8%87%AA%E5%B7%B1%E7%9A%84%E7%BF%BB%E5%A2%99VPS:%E6%90%AC%E8%BF%90%E5%B7%A5VPS%E8%B4%AD%E4%B9%B0%E4%BD%BF%E7%94%A8%E6%B5%81%E7%A8%8B%E5%85%A8%E8%AE%B0%E5%BD%95/)。

### 1. 安装PHP和MySQL环境

由于ownCloud使用php实现的，所以需要在你服务器上安装PHP和MySQL环境，这个网上有很多教程，也可以参考ownCloud给出的[官方教程](https://doc.owncloud.org/server/9.1/admin_manual/installation/php_54_installation.html)。

### 2. 下载服务器源码

在任一文件夹下输入命令：

```shell
wget https://download.owncloud.org/community/owncloud-9.1.0.tar.bz2
tar xvf owncloud-9.1.0.tar.bz2
```

然后改变owncloud这个文件的权限，命令如下：

```shell
chmod 777 owncloud
```

### 3. 添加管理员账号

使用浏览器访问地址http://serveraddress/owncloud/ (serveraddress即你服务器的ip地址)就可以看到如下页面：

![2](http://7xn88v.com1.z0.glb.clouddn.com/5de5eb45600b86bae8e2c0cfdaadb319.png)

在这里自定义一个管理员的账号和密码，然后点击FinishSetup，这样一个ownCloud就搭建完成了。

## 使用ownCloud作为OmniFocus的同步服务器

ownCloud搭建完成后，就可以把它作为Omni的同步服务器了，具体方法如下：

打开OmniFocus的同步设置页面，选择WebDAV方式同步，然后在Address中填入如下地址：

> http://<serveraddress>/owncloud/remote.php/dav/files/<adminname>

<serveraddress>和<adminname>分别替换为你的服务器地址和刚才填的管理员用户名，如下图：

![3](http://7xn88v.com1.z0.glb.clouddn.com/264b1ec833e6078643e9c2f40b2bf7aa.png)

然后点击“Sync Now”进行同步，会提示输入密码，输入刚才添加的管理员密码，然后就可以同步成功了。

使用浏览器访问http://serveraddress/owncloud/ ，就可以看到刚刚同步上去的文件了，如下图：

![4](http://7xn88v.com1.z0.glb.clouddn.com/0da7c4ef259b48e8f608540c27091cbd.png)

## ownCloud的一些其他应用

ownCloud还提供了多平台的客户端，包括iOS，Android，Mac，Windows等，你可以下载一个Mac客户端，然后设置一个同步文件夹，凡是放在这个文件夹里的文件，都会被自动同步到服务器上。如下图：

![5](http://7xn88v.com1.z0.glb.clouddn.com/d7b6c828536bab1e8fa5a69a12332ebf.png)

我把Quiver的笔记数据也同步到了服务器上。

## 结语

其实ownCloud的功能还远不止这些，还有很多功能还待以后慢慢探索。

类似ownCloud的还有一个叫[seafile](https://www.seafile.com/home/)的服务，两者的区别和优劣可以参考[这里](http://www.zhihu.com/question/23929945)。

## 参考资料

[ownCloud官方文档](https://doc.owncloud.org/server/9.1/admin_manual/contents.html)
[ownCloud关于WebDAV使用的文档](https://doc.owncloud.org/server/9.1/user_manual/files/access_webdav.html)
[使用OwnCloud创建私有云](http://jjliu.blog.ustc.edu.cn/198/)
