---
title: git 简明教程与命令
top: true
cover: false
toc: true
mathjax: true
date: 2020-01-30 14:38:39
password:
summary: Git是目前世界上最先进的分布式版本控制系统，可以极大便利我们的项目管理与文件版本管理。本文记录了常用git命令，并对git的使用原理做一些简明的教程。
tags: 
- git 
categories: 
- 工具方法
typora-root-url: git-简明教程与命令
---
## git 简明教程与命令

### What is git

Git是目前世界上最先进的分布式版本控制系统，可以极大便利我们的项目管理与文件版本管理。

git的架构如下图所示：

<img src="git架构图.png" alt="git架构图" style="zoom:50%;" />

从架构图可以看出，git整体分为4部分。

从左向右看：

1. 本地仓库，也就是图中的workspace，顾名思义，这是我们的工作区，即操作文件的地方，
2. index，这其实是在.git文件夹下的一个子文件夹，内部存储了 `git add `后的文件，被称为stage，可以称之为暂存区，
3. repository，也就是本地的git仓库，一般称为版本库，
4. remote，远程仓库，比如著名的github，或者自己搭建的git服务器等。



### 1. 本地仓库

本地的部分为左边三部分：工作区，暂存区和版本库。（也有人将暂存区和版本库合称为版本库，个人还是倾向于分开）。 

#### 1.1 版本保存

**版本保存指的是将修改后的工作区保存至暂存区，再保存至版本库。**

当我们编辑修改好代码之后，首先要执行 `git add` 命令，这一步将会把我们的修改保存到暂存区。执行完这一步之后，我们修改后的代码就相当于有了一个副本，已经可以做到回退了。

但是，这个副本目前还是处于暂存区，要真正持久化保存需要执行 `git commit -m 'your commit comment'`，这样，修改后的内容就会作为一个新的版本（**官方称为提交对象**）保存到仓库的当前分支。并且**只有加入到暂存区之后才能被保存到版本库。**

由上述可以看出，保存新版本的过程为：修改后的文件通过 `git add ` 提交到暂存区，再通过  `git commit`提交到版本库持久化保存。

如下图所示：（蓝色表示已存为版本库，绿色表示处于暂存区还没有存为版本库，红色表示修改后还没有存入暂存区）

<img src="1.png" alt="初始状态" style="zoom:75%;" />

上图的A、B、C、D、E表示不同的文件状态。

首先，我们在工作区修改了文件，会处于上图的working状态，工作区中的文件是最新的版本，其中A，B，C都已经 `git add `  和 `git commit`，即这些步骤已经存在了版本库中。D执行了 `git add ` 但还没有执行 `git commit`，即只存在了暂存区还没有存入版本库。而E还没有执行 `git add `，即最新的改动尚未被git所跟踪 。

#### 1.2 版本回退

**版本回退指的是从暂存区或者版本库回退先前的版本到工作区。**

版本回退一般使用 `git checkout` 、`git reset` 和`git revert`命令。

##### git checkout 

在`Git`里面，`checkout`用于切换分支或者恢复工作树的文件。checkout命令实际上是用来改变HEAD的指向。不会对版本库中节点做改变。 

使用方式如下：

```bash
git checkout [<tree-ish>] [--] <pathspec>
```

这个命令很灵活，既可以带一个`commit`号，又可以带着一个路径。可以用来恢复分支，恢复单个文件或者恢复整个文件夹。

当你想要丢弃某个文件的改动的时候，一般使用这个命令。

1. ###### **恢复文件**

对于上图的初始状态，（假设工作区文件为test.txt）我们执行 `git checkout --test.txt` 就可以将状态E删除，把文件恢复到最近一次 `git add` 或者 `git commit` 状态，如下图所示：

<img src="2.png" alt="after checkout" style="zoom:75%;" />

执行完上述命令，工作区变成了和暂存区一样的状态。

又或者，你想要让工作区回到指定的C状态或者B状态（穿越回过去），可以使用 `git checkout <B commitId> -- test.txt` 就可以让暂存区和工作区都回到B状态，如下图所示：

<img src="3.png" alt="after checkout B" style="zoom:75%;" />

此时，工作区、暂存区都变成了同样的B状态，并且，这个命令不会影响到其他文件。更很神奇的是，版本库的commit节点不会发生任何变化，也就是说，`git checkout <B commitId> -- test.txt` 命令只是让**test.txt文件**在**工作区和暂存区**回到了B节点的状态，对于版本库不会有变化。

可以用 `git diff` 和 `git diff --cached` 分别比较工作区与暂存区、暂存区与版本库的状态。

不过要注意的是，这个命令会使用版本库覆盖暂存区，所以没有被追踪的E状态和暂存区中的D状态会丢失。所以建议使用这个命令回到过去状态之前，把目前的暂存区提交到版本库。

如果，执行完之后你又后悔了，想回到C的状态，可以使用 `git log` 查看C的commitID（关闭窗口再打开也还在），然后使用 `git checkout <C commitId> -- test.txt` 就可以回到C状态。

2. ###### **恢复提交对象**

当然，`git checkout <B commitId> ` 这种去掉文件名的形式也是可以的，实现的效果为恢复所有的文件到B状态。

使用后会显示一段提示，如下图：

<img src="checkout_status.png" alt='初始状态' style="zoom:50%;" />

这里的 `detached Head` 被翻译为游离指针，接下来的操作都是在这个指针所指向的节点上，如果你在这时对文件做出了更改，那么可能在你想要回到之前的分支时会遭到阻拦。

比如，上图是从master分支checkout过来的，在文件改变之后想要checkout回到master时，可能回不去了，这时，就可以像上述建议一样把这个修改后的节点通过 `git branch -c <new-Branch-Name>` 来生成一个新的分支，之后再合并这个分支到master中。


以上的表现形式看来，`git checkout` 像是平行时空，可以在不同不互相干扰的平行时空之间切换。因为这个命令主要是用来切换分支，具体用法如下。

3. ###### **切换分支**

很简单的命令`git checkout <branchName>` 就可以切换到对应的分支。

在那之前，你可以用`git branch` 命令查看你有哪些分支。

Git 的分支，其实本质上仅仅是指向提交对象的可变指针。

执行 `git branch testing` 之后：

<img src='two-branches.png' alt='新建testing分支' style='zoom:60%'/>

这会在当前所在的提交对象上创建一个指针，需要注意的是，这个命令只是新建分支，并不会切换到这个新建的分支上。

git中有一个名为 `HEAD` 的特殊指针，用来指示当前处于哪个分支，当前我们仍然在master分支上。

<img src='head-to-master.png' alt='' style='zoom:60%'/>

所以上述命令执行完之后，完整的指针如上图。

使用`git checkout testing` 就可以切换到testing分支上了。

<img src='head-to-testing.png' alt='' style='zoom:60%'/>



##### git reset

这个命令是git里面少有的会删除版本库节点的命令，要小心使用。

`git reset` 的命令使用方法如下：

```bash
git reset [ –soft | –mixed | –hard] <commitID>
```

`git reset <commitID>` 的意思就是把HEAD所处的分支指向commitID，但是同时它也会改变某些commit。

`git reset` 命令比较特殊的一点是有三个可选参数：soft, mixed, hard

```
soft就是只动repo
mixed就是动repo还有staging(这个是默认参数)
hard就是动repo还有staging还有working
```

同样以上面的图示举例：

假设现在处于初始状态：

<img src="1.png" style="zoom:75%;" />

1. 执行 `git reset --soft <B commit>` 命令之后，版本库回到了B状态，而暂存区和工作区的状态不变，实际结果如下图：

<img src="4.png" style="zoom:75%;" />

需要注意的是，此时C状态的commit会被删除，也就是一旦回到了过去，未来的节点就会被删除，当然，实际上使用 `git reflog` 仍然是可以找到C的commit，然后借助C的commit Id回到C状态的。

2. 对于初始状态，执行 `git reset --mixed <B commit>` 之后，版本库和暂存区都回到了B状态：

<img src="5.png" style="zoom:75%;" />

注意，虽然这时候你仍然可以回到C状态，但是暂存区中的D状态是会丢失的。

3. 对于初始状态，执行 `git reset --hard <B commit>` 之后，版本库、暂存区、工作区全部都回到了B状态（PS: 版本库中的C状态实际上也是可以找回来的，只是执行reset命令之后不再显示出来。）：

<img src="6.png" style="zoom:75%;" />



这看起来和`git checkout <B commitId>` 效果一样，实际上有很大的不同。举个例子：

![](checkout_reset.png)

左边为初始状态，右上为执行reset命令，右下为执行checkout命令。

checkout命令只是将Head指针指向另外一个分支的分支指针，并不会对commit对象进行操作，而reset是将HEAD连同所对应的分支指向新的commit。

大概区别就是一个是切换到平行时空，一个是使当前时空变成和平行时空一样的状态。

需要特别注意的是，处于远程仓库的分支**严格禁止**使用 `git reset` 命令，因为有些人会以远程仓库的版本作为开发的基础，使用reset命令会破坏这些基础，造成某些意想不到的问题。

##### git revert

现在基本上推荐使用 `git revert` 替换 `git reset`。

那么他们的区别在哪里呢？

<img src="/reset.png" alt="reset" style="zoom:38%;" />

很明显， `reset` 之后回退到之前的状态。该状态之后的节点丢弃了。

<img src="/revert.png" alt="revert" style="zoom:38%;" />

`revert` 命令通过新建一个commit来撤销一次commit所做的修改，是一种安全的方式，并没有修改commit history。



### 远程仓库

远程仓库是指托管在因特网或其他网络中的你的项目的版本库。git的精髓就在于，有一个网站，管理众多远程仓库，他就是git hub。

正如我们在第一张图git架构图中看到的，本地的版本库还可以推送到远程仓库，这样，就可以协作开发一个项目。

如果想查看你已经配置的远程仓库服务器，可以运行 `git remote` 命令。 它会列出你指定的每一个远程服务器的简写。 

#### 添加远程仓库

运行 `git remote add <shortname> <url>` 添加一个新的远程 Git 仓库，同时指定一个方便使用的简写：这样你可以在命令行中使用你所设置的 `shortname` 来代替整个 URL。

比如，现在我们添加一个GitHub中的仓库，并把它命名为origin:

<img src='./gitadd.png' alt='添加远程仓库' style='zoom:45%'/>

原本，本地这个文件夹没有任何远程仓库，在我们使用  `git remote add <shortname> <url>` 命令之后，就添加了一个名为origin的远程仓库。现在，这个远程仓库中的master分支就可以用 `origin/master` 表示。

#### PUSH代码到远程仓库

上面，我们已经关联了一个远程仓库，并且命名为origin。

把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。使用 `-u`参数，会把本地的`master`分支和远程的`master`分支关联起来，在以后的推送或者拉取时就可以简化命令。

之后，就可以通过 ` git push origin master` 推送到远程了。