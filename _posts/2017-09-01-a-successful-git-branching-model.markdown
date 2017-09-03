---
layout:     post
title:      "一个成功的git分支模型"
subtitle:   "A successful Git branching model"
date:       2017-09-01 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/branching-illustration@2x (1).png"
header-mask: 0.3
catalog:    true
tags:
    - git
    - 翻译
---


> git是团队开发协作中很重要的工具，git也是一个工程师应该掌握的基本技能之一。最近看了一些git相关的文章，发现了这篇文章广为流传的神作，特此翻译一下，也当作是巩固一下git的知识，作者是Vincent Driessen——[A successful Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)<br>

&emsp;&emsp;在这篇文章中，我讲介绍一种开发模型，在过去的一年中，我在多个项目中（无论是工作中的还是私人的）中都使用了它，并且证明这种模型是非常成功的。我早就想写这篇文章了，奈何一直找不到一个时间能过一次写完它，直到现在。我不会讲任何的项目细节，仅仅是关于会讲分支策略和发布管理。<br>
![图片](/img/in-post/git-model/git-model@2x.png)
&emsp;&emsp;这篇文章把git当成是所有源代码的版本管理工具。（顺便提一下，如果你对git感兴趣，我们公司的[GitPrime](https://www.gitprime.com/)提供了很多草鸡棒的软件工程性能的实时数据分析。

## 为什么是git?
&emsp;&emsp;通过一次关于git的优缺点的和集中式源代码控制系统对比的彻底的分析，[看这里](http://git.or.cz/gitwiki/GitSvnComparsion)。关于这个摩擦出了很多火花。作为一个开发人员，目前的我更倾向于使用git。Git真正的改变了开发人员关于合并和分支的认识。在CVS/Subversion时代那个时代，合并和分支一直被认为是可怕的（小心合并冲突，它们会打人的），并且你一段时间只能做一件事情。<br>
&emsp;&emsp;但是使用git,这些操作都不是事儿，并且些操作是你日常工作流程的核心部分之一。例如，在CVS／Subversion[说明](http://svnbook.red-bean.com/)中，分支和合并在最后一章中才提到（针对高级用户）
，但是在git的所以书中基本第三章就讲了分支和合并。<br>
&emsp;&emsp;由于git简单高效和可重复的特性，分支和合并再也不是什么让人害怕的事情了。版本控制工具应该致力于解决分支／合并问题，而不是其他的。<br>
&emsp;&emsp;关于工具已经讲的够多了，让我们开始讲讲开发模型吧。这里我要讲的模型，是指为了能够有序的开发，每一个开发人员应该遵守的一系列步骤。<br>
## 分散而集中
&emsp;&emsp;我们使用的仓库，和分支模型可以很好的配合，是真正的中央仓库。要注意，这个仓库被认为是唯一的中央仓库（由于Git是分布式版本控制系统，在技术层面上说没有什么中央仓库）。我们把这个仓库命名为orign,因为这个名字对所有的git使用者来说都很熟悉。<br>
![图片](/img/in-post/git-model/centr-decentr@2x.png)
&emsp;&emsp;每一个开发者都pulls并且pushs到origin上。除了集中的push-pull这种关系，每一个开发者都有可能从其他人中pull修改，以组成一个新的小团队。例如在一个两人或者多人合作开发一个新的功能时，为了避免在将代码过早的推送到origin上，这种方法是可行的。如上图所示，这里有Alice和Bob,Alice和David，Clair和Daivd三个小组。<br>
&emsp;&emsp;严格的说，这唯一说明的一点是Alice定义了一个git remote，命名为bob，并且关联了bob的仓库，反之亦然。
## 主分支
&emsp;&emsp;在核心方面，这个开发模式深受现有模型的启发。中央仓库有两个永久的主分支：<br>
<ol>
    <li>master</li>
    <li>develop</li>
</ol>
&emsp;&emsp;基于origin的master分支，应该对每一个Git使用者都很熟悉，另一个分支是develop。
![图片](/img/in-post/git-model/main-branches@2x.png)
&emsp;&emsp;我们认为origin/master主分支的HEAD指针永远指向源代码的一个产品状态。<br>
&emsp;&emsp;我们认为develp/master主分支的HEAD指针永远指向下一个版本的最新的一次开发修改提交。有些人把这个称为集成分支。这就是任何的构建完成的地方。<br>
&emsp;&emsp;当develop分支上的代码已经稳定可以发布了，所有的更改都会被合并到master分支上，并且会打上一个标签。具体的工作细节接下来会谈。<br>
&emsp;&emsp;因此，每次更改合并到master分支上时，都会产生一个新的发布版本。我们严格的对待这一过程，所以理论上，在每一master上产生一个新的commit, 我们可以使用git hook 脚本自动的构建和推送我们的项目到生产服务器上。
## 支持分支
&emsp;&emsp;除了主分支master和develop，我们的开发模型还会用到很多支持分支，以支持团队成员之间的并行开发，方便追踪版本，为下一次生产发布做准备，协助快速的修复线上的问题。不同于主分支，这些分支都是临时分支，它们最终会被移除了。<br>
&emsp;&emsp;我们可能会用的不同类型的分支：
<ol>
    <li>Feature branches</li>
    <li>Release branches</li>
    <li>Hotfix branches</li>
</ol>
&emsp;&emsp;每一个分支都有明确的目的，并且必须明确它们时基于哪一个分支的，最后会合并到那些分支。接下来我们会一一介绍它们。<br>
&emsp;&emsp;绝不是说这些分支使用的不同的技术。所有的分支类型都取决于我们是怎么使用它。它们都是普通的git分支。<br>

### Feature分支
分支可能来自：<br>
&emsp;&emsp;develop<br>
分支必须合并到：<br>
&emsp;&emsp;master<br>
分支一般命名为：<br>
&emsp;&emsp;任何的东西，除了master，develop，release-* 或者 hotfix-*<br>
![图片](/img/in-post/git-model/fb@2x.png)
&emsp;&emsp;Feature branches（或者其他的名字，例如topic branches）用来开发一个即将使用的新功能，或者不远将来的一个新版本。在开始开发一个新功能的时候，未来哪个发布版本中会包含这个功能是并不确定的。Feature branches的实质就是在存在于新功能开发的整个生命周期，但最后会被合并到develop(为了将新功能添加到即将发布的版本中)或者被废弃（万一效果不好）。<br>
Feature branches只存在于开发者的仓库中，不会存在于orgin仓库中。
#### 创建一个feature分支
&emsp;&emsp;当你开始开发一个新的功能，基于develop分支创建一个新分支。
```
$ git checkout -b myfeature develop
切换到一个新的，叫做myfeature的分支
```

#### 混合一个已完成新功能到develop分支
&emsp;&emsp;完成新功能也许会被合并到develop分支，以供之后的新版本发布。
```
$ git checkout develop
切换到develop分支
$ git merge --no-ff myfeature
合并
(修改的细节)
$ git branch -d myfeature
删除myfeature
$ git push origin develop
```
&emsp;&emsp;使用--no-ff指令在合并分支的时候产生一个新的commit对象，即使使用fast-forward方式合并分支。这个可以避免丢失已存在的feature brancd的历史信息，并且将所有的commit组合起来。
对比一下：
![图片](/img/in-post/git-model/merge-without-ff@2x.png)
&emsp;&emsp;在之后的例子中，会在git历史记录中看到那些commit信息组成了一个新的功能——你也许得阅读很长的log信息。恢复一个完整的功能（也就是以组commit）,这确实是一个令人头疼的问题，但是使用--no-ff 指令就可以很容易解决这个问题。<br>
&emsp;&emsp;是的，这会产生更多(空)的commit，但是这样做益大于弊。<br>
#### Release分支
分支可能来自：<br>
&emsp;&emsp;develop<br>
分支必须合并到：<br>
&emsp;&emsp;develop 和 master<br>
分支一般命名为：<br>
&emsp;&emsp;release-*
&emsp;&emsp;release 分支为一个新的产品的发布做准备。它们可以用来小心的检查每一个细节。此外，它们可以用来解决少数的bug和为发布准备元数据（版本号，编译数据等）。所有的这些工作都在release分支上进行，这保证了下一次大的版本发布时develop分支是干净的。<br>
&emsp;&emsp;从一个develop（绝大多数）分支创建一个release分支,就反应了这个新的版本的目的。至少这些需要被发布的新功能，在最后都会被合并到develop分支。所有未来发布的功能将会等到下一次创建分支的时候进行开发。<br>
 &emsp;&emsp;可以明确的是，在创建release分支的时候就会给即将发布的版本一个版本号，而不是其他更早的时候。
这个时候，develop分支会反映下一个发布的变化，但是并不清楚下一个版本是0.3或者1.0，直到release 分支开始工作。这个决定是在开始使用release分支的时候决定的，并且由项目的版本号制定规则决定。
#### 创建release分支
&emsp;&emsp;release分支来之于develop分支。例如，把版本1.15当作是目标的线上工作版本，我们将会有一次重大的改版。develop分支已经为下一版本做好准备，并且我们决定下一个版本为1.2版本（而不是1.16或者2.0）。所以我们新建一个分支，并且它的名字能够反映版本号。
```
$ git checkout -b release-1.2 develop
切换到一个新分支release-1.2
$ ./bump-version.sh 1.2
修改成功，版本命名为1.2
$ git commit -a -m "Bumped version number to 1.2"
[release-1.2 74d9424] Bumped version number to 1.2
1 files changed, 1 insertions(+), 1 deletions(-)

```

&emsp;&emsp;在创建并切换到一个新的分支，我们修改版本号。在这里bump-version.sh文件是一个虚构的shell脚本文件，用来表示修改了某些文件以反映产生了新的版本。（当然也可以是一些手动的修改，关键是要有一些文件被修改。）最后新的版本号没提交。<br>
这个新的分支也许会存在一段时间，直到发行版明确推出。在这段时间，也许会在这个分支上修复bug(而不是在develop分支上)。但是在这上面添加重大的新的功能是明确禁止的。它们必须被合并到develop分支，然后等待下一个版本。
#### 完成一个release分支
&emsp;&emsp;当release分支已经准备好了可以真正的发布，接下来需要做一些操作。首先，将release分支合并到master上（因为在master上的每一个commit都是一个版本）。接下来，这个master上的commit必须在它的历史记录中必须是容易识别的。最后，release分支上的修改必须合并到develop分支上，这样未来的版本中也包含bug的修改。<br>
前两个步骤用git的写法：
```
$ git checkout master
Switched to branch 'master'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2
```

&emsp;&emsp;现在这个版本完成了，并且做了标记供将来参考。
```
校订：也许你也会使用-s或者-u指令来标记你自己标签密码
```

&emsp;&emsp;为了保存release分支上的修改，你必须将这些修改合并到develop分支上。
```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Summary of changes)
```
这个步骤可能会导致合并冲突（很有可能会发生，因为我们更改了版本号）。如果有冲突，那就修改它然后提交。<br>
现在我们真正的完成了，并且release分支可以被移除了，因为我们不在需要它。
```
$ git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
```
### Hotfix分支
分支可能来自：<br>
&emsp;&emsp;master<br>
分支必须合并到：<br>
&emsp;&emsp;develop 和 master<br>
分支一般命名为：<br>
&emsp;&emsp;hotfix-*
&emsp;&emsp;Hotfix分支也是为了新版发布而存在的，这点和release分支很像，尽管是意外。它们的出现是为了快速的处理线上的问题。当线上产品产生一个严重的bug，必须马上被修复，就会基于master分支创建一个hotfix分支。<br>
&emsp;&emsp;其核心是当一个人在快速修复产品问题时，其他的人能够继续它的工作。
![图片](/img/in-post/git-model/hotfix-branches@2x.png)
#### 创建一个hotfix分支
Hotfix分支基于master分支创建。例如，1.2版本是目前线上跑着的版本，因为一个严重的bug而发生了一些问题。但是在develop分支上的修改还不稳定。我们会创建一个hotfix分支然后开始修复问题：
```
$ git checkout -b hotfix-1.2.1 master
Switched to a new branch "hotfix-1.2.1"
$ ./bump-version.sh 1.2.1
Files modified successfully, version bumped to 1.2.1.
$ git commit -a -m "Bumped version number to 1.2.1"
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1
1 files changed, 1 insertions(+), 1 deletions(-)
```

&emsp;&emsp;不要忘记在创建分支后修改版本号。<br>
&emsp;&emsp;然后修改bug，再提交。<br>
```
$ git commit -m "Fixed severe production problem"
[hotfix-1.2.1 abbe5d6] Fixed severe production problem
5 files changed, 32 insertions(+), 17 deletions(-)

```
####完成hotfix分支
&emsp;&emsp;当完成修改，bug修改要被合并回mater分支，到那时也必须合并到develop分支，为了确保下一个版本发布时没有这个bug。这个release分支完成时的工作很像。<br>
&emsp;&emsp;首先，更新master反正，标记版本。
```
$ git checkout master
Switched to branch 'master'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2.1
```

```
校正：也许你也会使用-s或者-u指令来标记你自己标签密码
```

&emsp;&emsp;接下来，合并到develop分支。
```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff hotfix-1.2.1
Merge made by recursive.
(Summary of changes)

```

&emsp;&emsp;有一个例外是当前已经存在一个release分支，则这个bug修改应该被合并到release分支，而不是develop分支。将bug修改合并到release分支最终也会导致它被合并到develop分支，当release分支完成的时候。（如果develop分支需要马上修复bug，等不到下一个的release完成，你也可以小心的把hotfix分支合并到develop分支）。<br>
&emsp;&emsp;最后，移除这个临时分支
```
$ git branch -d hotfix-1.2.1
Deleted branch hotfix-1.2.1 (was abbe5d6).
```
## 总结
&emsp;&emsp;虽然这个分支模型并没有什么令人震惊的新东西，但是文章开头的那张大图里的内容，在我们的项目中被证明是极其有用的。<br>
它给出了一个非常优雅精辟的模型，让人们易于理解，并且团队成员之间可以分享对分支和发布的共同理解。<br>
[这里有一个高清的PDF版本](http://nvie.com/files/Git-branching-model.pdf) 。把它打印下来挂墙上吧，这样就可以再任何时间的快速的浏览回顾一下。<br>
更新：给有需要的人：[gitflow-model.src.key](http://github.com/downloads/nvie/gitflow/Git-branching-model-src.key.zip) <br>
如果你想要联系我，我的Twitter账号是[@nvie](http://twitter.com/nvie)



