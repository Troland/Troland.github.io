---
title: 常用 Git 命令及工作流
date: 2017-12-09 22:15:24
tags:
- Git
categories:
- Tech
---

工作过程中使用 Git 作为版本控制和协作的工具经常会遇到各种问题，自己整理了自己常用的命令及一些常见的工作流。

## 工作区、暂存区、历史区

工作区：新添加的文件，任何不在暂存区的文件，不在版本记录中的文件。
暂存区：当把工作区的文件用`git add`之后文件便会在暂存区中。
版本库：当使用`git commit`之后文件就会记入版本库中。

## 创建仓库

`git init`，用 `git init --bare /path/to/repo.git`创建一个裸的中央仓库。
## 日志

`git log --oneline`简略地显示提交信息为一行即只显示`SHA1`值和少量的提交信息。

`git log -p`显示提交的详细信息。

`git log -p`显示最为详细的提交视图。

`git log --graph --oneline --decorate`详细信息的缩略图。

`git log --oneline master..origin/master`查看两个分支或者提交之间的差别。`master`和`origin/master`可以是提交的 ID 或者是分支。


## 检出提交

`git checkout commitID file`切换文件到指定的提交。

**检出提交 不会损害现在的分支，因为这个已经处于游离的状态。HEAD 一般指向 master 或是其他的本地分支。当检出提交的时候就不再指向一个分支。而是指向一个提交，是一个分离的 HEAD 的状态**。

当切换到那个提交的时候，那个 commit 上更改文件并不会影响到这个提交，当你在这个状态下添加文件并且提交的时候会发现那个处于游离状态的 commitID 会变化为 commitID2。再次切换回之前的分支的时候，那个 commitID 上的内容并没有被影响，请看如下代码：

```
git checkout 75e7a
touch a.css
git add a.css
git commit -am "commit messaege" // 这个时候commid会变
git checkout test
git log -p 75e7a
```

`git commit -am`这里只对已经进版本的文件有用，新添加的文件得先`git add`。

`git checkout HEAD file`选择文件在暂存区中的修改回归到其最近的提交。

## 不可撤消及危险的命令

`git rebase`，`git commit --amend`，`git reset`，都不要用在仓库的公共的分支、提交上面。**只适宜操作自己本地的提交即在自己本地的未发布到分支上的提交。**

`git rebase <base>`即将本分支的提交变更到目标提交，base 可以是分支、提交等，例如:

```
git checkout new-feature
git rebase master
git checkout master
git merge new-feature
```

这样产生一个快速向前的线性提交，而不用再多一次合并提交。

`git reset --hard`和`git reset`的区别是当在暂存区里面有文件的时候，`--hard`会把处于暂存区的文件清除掉，而未加`--hard`的时候只是把暂存区的文件退回到工作区域，文件并没有消失。


比如下面的操作：

```
touch a.css
touch b.js
git add b.js

git status // 查看工作状态

git reset --hard // 重设这个时候处于暂存区的文件 b.js 将不会在工作区域中存在！

```

`git reset --hard`通常和`git clean -f`一起使用，因为前一个命令只是是将暂存区的文件删除，但是在工作区的文件并没有删除。
## 回滚

当提交发生了错误的时候可以使用`git revert`回滚到指定的提交。

## 变基和合并

根据[Gitbook](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)的描述:

> 如果是本地的更改使用`rebase`如果是和他人共享的提交出去分支就不要使用`merge`。


## 拉取及合并

`git fetch <remote> [branch]`当未加上分支名即拉取远程仓库所有的分支否则是拉取指定分支。

`git pull`是`git fetch`和`git merge`的快捷方式。

`git pull remote branch:localbarnch`若不写 localbarnch 表示默认和当前分支合并。
`git merge --no-ff <branch>`将指定分支并入当前分支，但总生成合并提交，用来记录仓库中发生的所有合并。

当你不想提交暂存区的本地的修改的时候可以先用`git stash`然后当合并完成再用`git stash pop`取出之前在暂存区的本地修改。

## 提交

`git push <remote> --all`本地所有分支推送到远程仓库不包含 tags。

`git push <remote> --tags`本地所有标签推送到远程仓库。

`git push -u origin branch`将分支推送到远程中央仓库(origin)即在远程也创建一个分支并和本地的分支关联在一起。`-u`标记将它添加为远程跟踪的分支。设置完可以直接使用`git push`提交更新了。

`git branch -u origin/test`分支会和上游分支 test 同步后就可以直接使用`git pull`拉取代码。

**只推送到那些以 --bare 方式初始化的仓库。**

*为防止覆盖中央仓库的历史，会拒绝你会导致非快速向前合并的推送请求。如果远程提交历史和本地提交历史分叉，就需要先拉取远程分支再进行提交。慎重使用 --force 选项*

## 提交引用 

`git show commitID`显示提交 id 的信息。
## 分支

`git branch -m <branch>`重命名当前分支

`git checkout -b <new-branch> <existing-branch>`以`<existing-branch>`作为基准，若无则是当前分支。

`git branch -r`查看远程的分支。

若是想要抓取远程的分支到本地首先得把远程的分支先 fetch 到本地，如下：

```
git fetch remote remotebranch
git co -b new-branch remote/remotebranch
```
`git push origin --delete [分支名]`和`git push 远程名 :[分支名]`删除远程分支。

## Reset、Checkout、Revert


## Pull Request

Pull Request 是协作者向目标仓库发起合并请求的方式，在 Github 上假设用户 tristan Fork 了 isolde 的项目，这个时候 tristan 有了项目的副本，然后在本地`git clone user/a.git`到本地，然后创建功能分支`git co -b new-feature`在 new-feature 分支上作了修改然后提交到自己的远程仓库，然后在 Github 上面点击 Pull Request 按钮指明自己的源库和源分支和目标仓库(即 isolde 的仓库)和目标分支，然后写上标题就可以了。Github 会发邮件给 isolde。

当 isolde 接到合并请求进行审阅，然后提建议，tristan 可以进行修改，提交之后会自动通知到 Pull Request中，当 isolde 觉得可以了就会合并请求并关闭 Pull Request。

## 工作流

### 中心化工作流

中心化非常简单即只有一个 master 分支，所有的成员都只是在这一分支下工作。

### 功能分支的工作流

每个人在开始新功能的时候创建功能分支，完成了提交到远程服务器，发布一个 Pull Request，审核通过，再合并到本地的 master 分支然后再推送到远程中央仓库，最后关闭 Pull Request。

### GitFlow 工作流

一共两个分支：master 和 dev。dev 用于合并功能分支，最后的 master 会为提交打标签。功能分支不合并进 master。

其实还不止这些，当开发分支新功能收集完毕了可以 fork 开发分支为发布分支，在这个发布分支进行其它 bug 修复，文档等。完成了再合并进 master 并打版本号。随后合并进 dev 分支。 

![](/images/git-workflow.png)

开发者要在本有一个 master 和 dev 分支。

当功能开发完成使用如下命令：

```
git pull origin develop
git checkout develop
git merge some-feature
git push
git branch -d some-feature
```

`git checkout -b release-0.1 develop`在这个分支上面进行测试文档整理。
一旦发布好了就会合并进`master`和`dev`分支，并删除`release-0.1`发布分支。

```
git checkout master
git merge release-0.1
git push
git checkout develop
git merge release-0.1
git push
git branch -d release-0.1
```

最后要打标签

```
git tag -a 0.1 -m "Initial public release" master
git push --tags
```

紧急修复是唯一可以直接从 master 创建的分支。修复完成合并进 master 和 dev 分支。在 master 上打上更新版本。

当发现紧急 bug

```
git checkout -b issue-#001 master
git checkout master
git merge issue-#001
git push
```

最后开发分支也要合并

```
git checkout develop
git merge issue-#001
git push
git branch -d issue-#001
```

### Fork 工作流

每个开发都都有两个仓库，一个本地的，一个是服务器的。**开发都是可以推送到自己的服务器仓库。只有项目管理者可以推送到官方仓库。**

大型项目和开源的项目大抵选择这种模式。

大概过程是项目创建创建裸仓库`git init --bare path/to/repo.git`，开发者进行 fork，最后开发者在自己的私有仓库进行更改，好了向官方提交 Pull Reqeust，然后项目维护都审核，通过则合并到 master 中。

[github](https://github.com) 即是这种模式。

## 钩子

钩子分为本地钩子和服务端钩子。
钩子即在特定事件发生的时候触发的脚本，在`.git/hooks`下面有默认的示例钩子。

### 常用的本地钩子

- pre-commit 提前之前做的事可以用来做`lint`代码风格，测试等检查。
- prepare-commit-msg 在文本编辑器生成提交信息后调用。用来方便修改压缩的或者合并的提交的提交信息。
- commit-msg 用户输入完提交信息之后用来警告开发者开发没有符合规范。
- post-commit 提交之后的操作。用来在提交之后发送邮件之类的。
- post-checkout 当切换分支或者提交的时候激活。比如 python 在切换分支之后有可能生成 .pyc 文件。这里可以在切换之后删除这些多余的文件。
- pre-rebase 在 git rebase 之前调取，可以用来警告开发者。

一般来说对于团队来说比如定义代码规范可以在**pre-receive**中进行设置，在本地钩子也可以但是 **.git/hooks** 是不会记入版本历史的所以可以采取符号链接的形式。

**post-receive** 是在 push 推送成功之后可以发送邮件和持续集成系统 jenkins 等。

## 日志引用

当执行了`git reflog`用来查看引用日志。记录了仓库中的所有更改，而不管有没有提交。比如当使用了`git reset` 去除了功能分支：

```
ad8621a HEAD@{0}: reset: moving to HEAD~3
298eb9f HEAD@{1}: commit: 一些提交信息
bbe9012 HEAD@{2}: commit: 继续开发
9cb79fa HEAD@{3}: commit: 开始新特性开发
```

若要回到 reset 之前的状态可以使用`git checkout HEAD@{1}`。这样会处于分支分离的状态，可以从那里创建新的分支。