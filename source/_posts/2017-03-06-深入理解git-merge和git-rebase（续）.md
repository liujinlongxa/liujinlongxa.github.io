---
title: 深入理解git merge和git rebase（续）
categories: Git
tags:
  - Git
date: 2017-03-06 09:34:43
---

本文接上一篇，主要讲解一下git rebase的用法和使用中注意的事项。

## git rebase

### 基本原理

git rebase，也叫做变基，也是Git种一种合并代码的手段，与git merge不同的是，rebase是直接将一个分支从他们的共同父节点开始后的所有Commit依次合并到另外一个分支上。如下图：

![1](http://7xn88v.com1.z0.glb.clouddn.com/e3bdbc7fff28676ccefdcb16f4d43095.png)

两个分支develop和master，如果通过命令`git merge`将develop分支合并到master分支，Git会将‘C’，‘3’以及两个分支的共同父节点‘B’进行三路合并，合并完成后生成一个新节点，如下图：

![2](http://7xn88v.com1.z0.glb.clouddn.com/ce919bb1921ee5c302baafbb0f45785f.png)

如果`git rebase`将develop分支变基到master分支上，Git会将develop分支上的所有commit(1,2,3)依次合并到master，每一次合并都会生成一个新的提交(如下图1',2',3')，合并完成后，如下图：

![3](http://7xn88v.com1.z0.glb.clouddn.com/0478f567c5748186a200cd4da8b5ddd8.png)

可以明显的看到，使用`git rebase`，分支线依然保持为一条，分支线看起来也没有那么乱，这也是`git rebase`相对于`git merge`的一个优点，但是另一方面，`git rebase`在合并每一个提交并生成一个新的提交时，会改写原来提交的提交时间（不会改写提交人），而且`git rebase`也没有留下合并的痕迹，可追溯性没那么强。

### 为什么那么多冲突

在使用`git rebase`的过程中经常会遇到这种情况，在执行的`git rebase`操作后，遇到了一个冲突，修改冲突后执行`git rebase --continue`，然后又来一个冲突，冲突一个接一个，而且有时候同一个冲突会出现好几次。

其实，这是由rebase的原理造成的，Git在合并每一个Commit时都会判断是否有冲突，如下图：

![4](http://7xn88v.com1.z0.glb.clouddn.com/90ff7dd4f79dddc65656820254dba48b.png)

两个分支develop和master以及他们各自的文件内容，现在要将develop变基到master上，Git会先将develop的第一个commit和master分支的最新提交合并，合并后如下：

![5](http://7xn88v.com1.z0.glb.clouddn.com/31c92a8c54e271fb69053a22c984c6dd.png)

会报一个冲突，解决完冲突后，执行`git rebase --continue`，根据冲突不同的解决方案，可能会与不同的结果，如下图：

![6](http://7xn88v.com1.z0.glb.clouddn.com/74efc09f8e231d078914a36f5223e289.png)

上图中的两种冲突解决方案，在执行`git rebase --continue`后，依然再会报一个冲突，应为在合并develop的第二个提交时，依然有冲突。下面的解决方案则不会造成再次冲突，因为这种解决方案是完全用develop分支覆盖了master分支，如下图：

![7](http://7xn88v.com1.z0.glb.clouddn.com/510fa8570fd012eb1723d6c521eb5cc6.png)

以上就是造成`git rebase`冲突太多的具体原因。

### 有那么点用的rerere

冲突太多怎么办，Git提供了一个辅助工具`git rerere`命令，具体用法可以参考[git rerere](https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-Rerere)。rerere命令能够记住解决一个冲突的方法，这样在下一次看到相同冲突时，Git 可以为你自动地解决它。

那么rerere能够避免`git rebase`带来的冲突吗，答案是否定的。因为rerere在判断两个冲突是否为相同冲突是根据冲突体的两部分是否完全一样来进行的，就上面的例子而言，不管是哪种冲突解决方案，第一次和第二次冲突的冲突体都不完全一样，因此rerere都不会自动帮我们修复。

![8](http://7xn88v.com1.z0.glb.clouddn.com/b80ef2ec61407d7a737a0997da6c960d.png)

而且，rerere命令也会记住错误的冲突解决方案，下次遇到相同的冲突时会直接应用错误的方案。不过你可以使用`git rerere forget <pathspec>`命令来删除Git记住的冲突解决方案。

说了这么多，那rerere命令到底有什么用呢？

`git rerere`命令为我们提供了一种减少冲突的方案：当你要保证一个长期分支会干净地合并，但是又不想要一串中间的合并提交。 将rerere功能打开后偶尔合并，解决冲突，然后返回到合并前。 如果你持续这样做，那么最终的合并会很容易，因为rerere可以为你自动做所有的事情。`git rerere`也可以将冲突的解决方案共享给项目组的其他成员。

所以说rerere命令还有一点用的，不过他并不能彻底解决冲突多的问题，减少冲突还是需要我们在平时使用时规范git的使用方法，使用统一的git工作流。

## 后记

本来打算写一篇的，但是因为太长了，所以分为两篇来写。由于写的比较仓促，文中如果有什么纰漏，欢迎指出。

参考资料：
[Pro Git](https://git-scm.com/book/zh/v2)
[Why do I have to resolve the same conflict over and over?](http://stackoverflow.com/questions/13825079/why-do-i-have-to-resolve-the-same-conflict-over-and-over)
