---
title: 使用HTTPS方法clone下来的Git仓库push时报错
categories: Git
tags:
  - Git
date: 2016-07-30 11:22:55
---

昨天在使用Git时遇到了一个问题，今天记录一下。

一般我在克隆Github的项目时都是使用ssh的方式进行的，这样只要你的ssh-key配置正确，就没有问题。昨天心血来潮试了一下https克隆了一个项目，修改完后，准备push的时候需要了问题，问题如下：


>remote: Permission to liujinlongxa/Spoon-Knife.git denied to xxxx.
fatal: unable to access 'https://github.com/liujinlongxa/Spoon-Knife.git/': The requested URL returned error: 403

解决方案是修改git的配置文件，

`vim .git/config`

将


>[remote "origin"]
        url = https://github.com/liujinlongxa/Spoon-Knife.git


改为


>[remote "origin"]
        url = https://liujinlongxa@github.com/liujinlongxa/Spoon-Knife.git

即前面加上用户名，这样就可以正常的push，或者直接运行命令

```shell
git config --local remote.origin.url https://liujinlongxa@github.com/liujinlongxa/Spoon-Knife.git
```

也可以达到相同的效果。
