---
title: 深入理解git merge和git rebase
categories: Git
tags:
  - Git
date: 2017-03-01 15:08:21
---

`git merge`和`git rebase`是我们平时在使用Git过程中用到比较多的两个命令。本文将主要介绍这个两个命令的基本用法以及使用中应该注意的事项。

## git merge

### 不同的合并方式

顾名思义，`git merge`是用来将一个分支的代码合并到另一个分支。如下图，将develop分支上的代码合并到master分支上。

![1](http://oldblog.shicishuzhai.com/dafea7a6a7e2ce161b7b941ad4474186.png)

上图中有一个`Fast-forward`字样，这是Git的一种合并方式。

![2](http://oldblog.shicishuzhai.com/31c51a26b1165b47797425e5d34a04a4.png)

如上图，在master分支的B点时牵出一个develop分支，develop分支又有了3个新的提交1，2，3，而master分支此时没有新的提交，这是如果合并的话，develop分支不用动，master只用把分支的头指针指向develop的最新的一个提交3即可。合并后的结果如下图：

![3](http://oldblog.shicishuzhai.com/44379a35e3a385da59d1d996efd017bb.png)

这个合并方式看似把两个分支合并了，实际上并没有真正进行合并操作，也没有留下合并的操作。默认`git merge`是采用`Fast-forward`方式进行合并的，如果不想采用这种方式，可以在命令后面加入`--no-ff`选项，如下：

![4](http://oldblog.shicishuzhai.com/161174b53c54146aa465eacf41140787.png)

不使用`Fast-forward`的方式进行合并，Git会为合并生成一个新的提交，合并后的结果如下图：

![5](http://oldblog.shicishuzhai.com/091081cb40b5eedbc5dc4ab2c8e4e13e.png)

Commit C即是Git自动生成的合并提交。一般情况下我们在合并代码时都会加上`--no-ff`，这样可以更清晰的看见合并操作。

### 一个不大不小的坑

如下图：

![6](http://oldblog.shicishuzhai.com/e92aed55a7593922b525c2723894b202.png)

 在牵出develop分支后，master分支又有了新的提交C，这时候如果把develop分支合并到master，就不能简单的通过移动master分支头指针来进行了，这时候默认执行的是**recursive**(递归)的策略进行合并的，这种策略下，Git会对两个分支的头结点（C与3）以及他们的共同父节点（B）进行三路合并，这种情况下就可能出现代码冲突。

 而且Git在合并两个分支时，并不会根据Commit的提交时间来判断哪个分支的代码更新，这就可能造成一个隐藏的问题：

 想象一下这种场景，一个团队内分为两个小组分别开发A和B两个功能版本，A版本的开发人员修改了某个文件（假定是file1），B版本的的开发人员发现他也要对file1做同样的修改才能继续开发，于是他对file1做了同样的修改（或者使用cherry-pick把A版本的相应的commit拉过来）。过了一段时间，A版本的开发人员发现他以前对file1的修改有问题，于是又把file1改了回去，而这时候B版本的开发人员并没有做同样的修改。这样，将来A版本和B版本合并到主干分支后，B版本的代码就会覆盖A的修改，于是错误的代码又被合并到了主干分支上。下面就是这个过程的示意图：

![7](http://oldblog.shicishuzhai.com/51b92d9f3934ffa53a1f2c9770a9895f.png)

如图，序号1，2，3表示的是Commit的提交顺序，FeatureA分支在commit 2上把文件内容改为了`bbb111`, FeatureB分支把文件内容改为了`bbb222`，在commit 6上FeatureA有吧内容改回了`AAA`，而B分支没有做同样的修改，当把FeatureA合并到主干时，由于FeatureA相对于共通父节点Commit 1来说没有变化，因此就会使用FeatureB的内容作为最终的内容，而且这种情况下不会报任何冲突。这就极有可能造成错误的代码又被合并到了主干分支上，而且这种问题极难被察觉，就好像莫名其妙的发现代码丢失了一样。

要避免这种情况的发生，只能从流程上来规范git的操作。如果一个修改要应用到多个分支上，应该单独为这个修改建立一个临时分支，修改完成后，每一个需要用到这个修改的分支都合并这个临时分支。将来如果这个临时分支又有了新的提交，依然是每个分支都要合并。这样就可以避免出现上面的情况。

未完待续。
