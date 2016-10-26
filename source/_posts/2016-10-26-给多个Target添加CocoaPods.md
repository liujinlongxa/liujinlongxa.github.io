---
title: 给多个Target添加CocoaPods
categories: iOS开发
tags:
  - CocoaPods
  - Target
date: 2016-10-26 22:25:04
---

### 基本用法

一般我们使用CocoaPods进行依赖管理时，如果项目里只有一个Target，则Podfile文件的格式应该是：

```ruby
target 'ZipApp' do  # ZipApp为Target名称
  pod 'SSZipArchive'
end
```

如果项目中有多个Target，可以定义多个这样的block，如下：

```ruby
target 'ZipApp' do  # ZipApp为Target名称
  pod 'SSZipArchive'
end

target 'ZipAppTest' do  # ZipAppTest为另一个Target名称
  pod 'SSZipArchive'
end
```

如果多个Target的依赖的库都是一样的，没有必要每一个Target都把所有库都写一遍，可以定义一个函数，写法如下：

```ruby
def common_pods
    pod 'SSZipArchive'
    pod 'AFNetworking'
    pod 'JSPatch'
end

target 'ZipApp' do  # ZipApp即使Target名称
  common_pods
end

target 'ZipAppTest' do
    common_pods
end
```

### 其他的用法

如果多个Target有嵌套关系，即一个Target是另一个Target的父Target，则可以写成以下形式：

```ruby
target 'ZipApp' do
  pod 'SSZipArchive'

  target 'ZipAppTests' do
    inherit! :search_paths # 这句话表示子Target继承了父Target包含的pod，即ZipAppTests这个Target里包含了SSZipArchive和Nimble两个pod
    pod 'Nimble'
  end
end
```

如果一个Workspace里包含了多个Project，并且每一个Project都有不同的Target，则可以使用以下形式定义多个Project的Target：

```ruby
target 'Target1' do
  project 'Project1'
  pod 'AFNetworking'
end

target 'Target2' do
  project 'Project2'
  pod 'JSPatch'
end
```
