---
title: 基本概念
tags:
  - git
categories: git
copyright: true
---

最开始linux的作者都是手动把社区的人贡献的代码合并的内核源码中，然后代码越来越多，版本、冲突不好解决就把源码放在bitkeeper，bitkeeper是商用的，但是给了作者免费的授权，过了一段时间社区有个人想破解这个gitkeeper被发现了，然后就收回了免费使用权，于是linux作者就自己写了一个版本控制系统，Git。

Git是把仓库的版本快照都放在本地，和SVN不一样，如果仓库服务器卡了，SVN是提不了代码的，Git可以生成新的版本快照存在本地，然后在推到服务器，如果仓库服务坏了，代码丢了，SVN就废了，Git就不一样了，他把快照放在本地，服务器用不了还可以把本地的快照推到新服务器就好了。

### 创建项目

执行init就可以把一个项目交给Git管理，会创建一个.git的隐藏目录，需要提交代码的时候先add在commit就好了，不过这样commit会把所有改变都提交上去，有些文件比如intellij的.idea文件我们不想提上去就要创建一个.gitignore文件，把不想提的代码写进去，commit之后只是在本地的Git更新了版本。

windows在git init之前需要先执行git config --global core.autocrlf false，Mac不需要。

存放代码的地方叫做工作区，.git文件中分为暂存区和仓库，当执行add命令的时候会把工作区的代码放到暂存区，commit的时候会把缓存区的代码变更放到仓库中。

### 术语

tracked：提交到暂存区或者仓库中的文件；

untracked：从来没有被git追踪过版本的，比如新建的文件，或者在.gitignore文件中排除的文件；

三种状态：modified：修改未提交；staged：在暂存区；commited：提交到本地仓库；

### 工作流

#### 集中式工作流：

需求的更新频率很低，开发人两个人一下的小工程，不需要分支，直接在master上开发。

#### 功能分支工作流：

适合少人数开发或者公司内部系统，没有什么版本概念，每个人都从一个主分支上拉到自己的featrue分支进行开发，开发完了在合并到主分支上，合并之后有bug就要从分支新建一个bugfix分支进行修改在合并到主分支，合并之后就可以把bug分支删除了；

#### GitFlow工作流：

他适合项目的版本稳定迭代那种，稳定并且缓慢，如果业务发展很快，迭代很快，还可能设计多个版本并行开发，那么GitFlow并不合适。首先刚创建安完git项目会有一个muster分支，muster是用来发布上线的，然后还有一个develop分支，develop也不是用来开发的，它是用来把成员开发的代码合并起来；还要再有一个release分支，这个分支是用来测试的；还要有一个bugfix分支，用来修改线上的bug。流程差不多是这样：

1.  首先创建git项目，这时候只有muster分支；
2.  以muster分支为基础创建develop分支；
3.  以develop分支为基础创建一堆featrue/XX分支，对应每个开发人；
4.  开发人在自己的featrue分支上开发；
5.  开发完事儿了提交代码然后提交merge request，让leader进行code review然后合并到develop分支，删除这个开发人的分支；
6.  全部合并到develop分支之后，以develop分支为基础创建release分支；
7.  在release分支上QA，然后改bug；
8.  最后把release分支合并到develop分支和master分支，删除release分支然后把master打一个tag，在上线；
9.  上线之后发现bug需要新建bugfix分支，修改在合并到master和develop分支，然后在master分支上打tag，在上线；

#### 升级版GitFlow：

在快速迭代并且多版本并行开发的项目中，使用工作流式非常麻烦，如果都以develop分支为基准，那么不同版本的代码还有很多bug就开始合并，想想都让人头大，解决的办法可以是develop分支把不同版本区分开来，然后在master和release之间多出来一个staging分支，用来合并不同版本的代码打tag上线，因为在QA测试之后代码基本稳定，这时候合并不会有后续频繁的改bug在合并导致的代码冲突问题。

### Git服务器

企业自己的git服务器免费的就是gitlab

### rebease（变基）

主要是解决分支太多，比如集中式工作流，两个人频繁的解决冲突，这样提交历史看着就很乱

### 高阶

1.  git生成的40位hash值是为了判断提交之后是否有被修改过，使用前7位的短hash值就可以定位到那次commit，使用git show 【7位短hash值】可以查看那次提交的具体，和reset类似也可以使用git show HEAD^查看head指向的commit明细，git show HEAD~5等等。
2.  查看有多少个分支没有合并到主分支：`git log --abbrev-commit --oneline master..featrue/xiaohong`这个就是查看featrue/xiaohong分支有多少提交没有合并到master分支。把master和featrue/xiaohong换个位置就是master中有多少提交是featrue/xiaohong没有的。
3.  对比不同的版本差异：分两种情况：
    *   代码没提交，使用git diff HEAD；
    *   代码已提交，git diff HEAD^ HEAD；

4.  把缓存区的代码按功能分次提交：`git add -i`进入交互模式，选择部分文件从缓存区中还原到unchecked状态，就是选择revert，然后选择文件，然后先把缓存区中的文件提交，在把剩下的文件add --all……；
5.  暂存代码：比如说你正在开发，突然说线上的代码有bug，这时候你需要创建bugfix分支去修改bug，但是切换分支要提交代码，提交开发一半的代码就很low，这个时候可以用暂存功能。执行`git stash`，就会把代码暂存，然后你修改的代码会恢复到上次提交状态，修改的代码不见了，这个时候就可以去修复bug了，改完了之后切换回这个分支执行`git stash apply`，把之前暂存的代码恢复然后继续开发。
6.  对commit的备注和代码调整（本地）：git commit --amend，对上次的commit修改备注，然后:wq退出就修改了备注；如果想在提交之后补充代码，直接改代码，然后git add --all .然后在执行git commit --amend，就可以；
7.  撤回：git reset --hard HEAD，工作区和暂存区的代码撤回到上一个版本，但是他不能删除新增的类，要手动删；git reset HEAD，只撤销暂存区上次的add；git reset --hard HEAD^工作区、暂存区、仓库都回到上一个版本。
8.  撤销合并分支：主分支刚合并准备上线，但是通知先不上，就要把合并的分支还原，使用git revert -m 1 HEAD就可以，如果以后又要合并master上线还要在执行一下git revert HEAD才可以。
9.  定位bug实在那次提交的：git可以使用二分查找定位bug代码是那次提交的，但是要配合自己写的测试代码，用的时候在百度；
10.  共享子项目代码：subtree；
11.  项目迁移：项目迁移的时候会把之前的commit抹掉，因为不想让信任看到，而且跨团队的代码是比较敏感的，最好让迁移的代码看起来像全新的。
12.  新版本提前上线：比如同时开发1.1版本和1.2版本，1.2版本的一些功能需要提前上线，就可以把1.2版本需要提前上线的功能放到1.1版本上去，使用git cherry-pick featrue/v1.2 [1.2版本的commit hash值]；
13.  权限控制：master分支是要被保护起来的，不允许普通开发人员在master上push和merge，gitlab或者码云上都可以设置用户权限；