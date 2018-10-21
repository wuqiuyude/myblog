---
layout:     post
title:      "深入React技术栈读书笔记（2）——diff算法"
subtitle:   "react diff algorithmic"
date:       2018-10-21 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.jpg"
header-mask: 0.3
catalog:    true
tags:
    - React
---


>本文主要《深入React技术栈》这本书的读书笔记系列之二，主要是讲React的diff算法<br>

&emsp;&emsp;React的diff算法和Virtual Dom的结合，使得React可以高效的渲染页面。diff算法无疑是React中最优秀的一部分之一。

## 传统的diff算法
传统diff算法时间复杂度是O(n)
##React diff
React diff算法的时间复杂度是O(n)，为了降低时间复杂度的，React制订了很多策略，来降低时间复杂度，这个过程也称之为调和
diff算法有如下三个策略：
1、DOM节点跨层级的移动操作发生频率很低，是次要矛盾；

2、拥有相同类的两个组件将会生成相似的树形结构，拥有不同类的两个组件将会生成不同的树形结构，这里也是抓前者放后者的思想；

3、对于同一层级的一组子节点，通过唯一id进行区分，即没事就warn的key。
基于以上三个策略，React对tree diff、component diff 和element diff算法进行了优化。
### tree diff
对树进行分层比较，两棵树只会对同一层次的节点进行比较。既然DOM节点跨层级的操作少到可以忽略不计，React只会对相同层级的DOM节点进行比较，就是同一个父节点下的所有子节点，当发现节点已经不存在是，则该节点及其子节点就会被完全删除掉，不会进行下一步的比较。这样只需要对树进行一遍遍历，就可以完成整个DOM树的比较。
![图片](http://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)
但是如果出现了DOM节点跨层级的移动操作呢？
![图片](http://calendar.perfplanet.com/wp-content/uploads/2013/12/vjeux/1.png)
React会直接删除A节点然后重新创建A节点。此时的diff执行过程是：create A -> create B -> create C -> delete A。
出现跨层级移动时，并不会出现想象中的移动操作，而是以A为根节点的证个树被重新创建，这是一种影响React性能的操作，因此官方建议不要进行DOM节点跨层级的操作。（开发的时候应该保持稳定的DOM结构会有助于性能的提升，例如，可以通过CSS隐藏或显示节点，而不是真正的移除或者添加DOM节点）（v-if或者v-show）
### component diff
React是基于组件构建的应用，对于组件间的比较所采取的策略也是非常简洁、高效的。
1、如果是同一类型的组件，按照原策略继续比较virtual DOM树即可.
2、如果不是，则将该组件判断为dirty component，从而替换整个组件下的所有子节点。
3、对于同一类型的组件，有可能其virtual Dom没有任何变化，如果能够确切知道这点，那么就可以节省大量的diff运算时间。因此，React允许用户通过shouldComponentUpdate()来判断该组件是否需要进行diff算法分析
当组件D变为组件G时，即使这两个组件结构相似，一旦React判断D和G是不同的类型的组件，就不会比较两者的结构，而是直接删除组件D，重新创建组件G及其子节点。虽然当两个组件是不同类型但结构相似时，diff会影响性能，但正如React官方博客所言：不同类型的组件很少存在相似DOM树的情况，因此这种极端因素很难在实际开发中造成重大的影响。
### element diff
当节点处于同一层的时候，diff提供了三种节点操作
1、INSERT_MARKUP（插入）：新的组件类型不在旧集合中，即全新的节点，需要对新节点执行插入操作
2、MOVE_EXISTING：旧集合中有新组件类型，且element是可更新的类型，就只需要做移动操作，可以复用以前的DOM节点
3、REMOVE_NODE: 旧组件类型，在新集合里也有，但对应的element不同则不能直接复用和更新，需要执行删除操作，或者就组件不在新集合中，也需要执行删除操作。
React提出优化策略：允许开发者对同一层级的同组子节点，添加唯一key进行区分。
lastIndex: 访问过的节点在旧集合中的最右的位置（即最大的位置），初始化的值为0；
_mountedIndex表示节点的位置。
比较当前节点在旧集合中的位置和lastIndex的位置 if(child._mountUndex < lastIndex_),否则不执行操作。这是一种顺序优化的手段。
B在旧集合中的位置B._mountIndex = 1,此时lastIndex 1





