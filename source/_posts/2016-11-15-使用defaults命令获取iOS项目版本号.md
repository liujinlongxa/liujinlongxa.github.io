---
title: 使用defaults命令获取iOS项目版本号
categories: iOS Tips
tags:
  - 其他
date: 2016-11-15 09:51:44
---

defaults命令可以用来读取和修改plist文件，因此可以用它来读取iOS项目里的Info.plist文件，具体用法如下：

```shell
defaults read <plist文件的绝对路径> <Key>
```

例如，读取项目的版本号，则名为如下：

```shell
defaults read ~/Project/Info.plist CFBundleShortVersionString
```

这里要注意，**路径一定要是绝对路径，不能是相对路径**。

有了这个命令，我们就可以很方便的在脚本中获取版本号，编写一些更复杂的应用。

其实，获取版本号还可以使用xcode自带的[agvtool](https://developer.apple.com/legacy/library/documentation/Darwin/Reference/ManPages/man1/agvtool.1.html)，但是如果项目中有多个.xcodeproj文件，agvtool就不能正确获取版本号了，暂时还没找到解决方案。

### 参考资料

[Automating Version and Build Numbers Using agvtool](https://developer.apple.com/library/content/qa/qa1827/_index.html)
[Include Build Number in ipa info command](https://github.com/nomad/shenzhen/issues/160)
