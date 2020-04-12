---
title: 二叉树基础（上）：什么样的二叉树适合用数组来存储？
date: 2020-04-11 14:27:05
tags:
- algorithm

categories:
- Data Structure
---

前面我们讲的都是线性表结构，栈、队列等等。今天我们讲一种非线性表结构，树。

### 什么是树？

<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-1.jpg">
<div style="clear: both"></div>

如图，“树”这种数据结构真的很像我们现实生活中的“树”，这里面每个元素我们叫作“节点”；用来连线相邻节点之间的关系，我们叫作“父子关系”。

<!--more-->

比如下面这幅图，A 节点就是 B 节点的父节点，B 节点是 A 节点的子节点。B、C、D 这三个节点的父节点是同一个节点，所以它们之间互称为兄弟节点。我们把没有父节点的节点叫作根节点，也就是图中的节点 E。我们把没有子节点的节点叫作叶子节点或者叶节点，比如图中的 G、H、I、J、K、L 都是叶子节点。

<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-2.jpg">
<div style="clear: both"></div>

除此之外，关于“树”，还有三个比较相似的概念：高度（Height）、深度（Depth）、层（Level）。它们的定义是这样的：
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-3.jpg">
<div style="clear: both"></div>

这三个概念的定义比较容易混淆，描述起来也比较空洞。我举个例子说明一下，你一看应该就能明白。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-4.jpg">
<div style="clear: both"></div>

**记忆诀窍：**
* 在我们的生活中，“高度”这个概念，其实就是从下往上度量，比如我们要度量第 10 层楼的高度、第 13 层楼的高度，起点都是地面。所以，树这种数据结构的高度也是一样，从最底层开始计数，并且计数的起点是 0。
* “深度”这个概念在生活中是从上往下度量的，比如水中鱼的深度，是从水平面开始度量的。所以，树这种数据结构的深度也是类似的，从根结点开始度量，并且计数起点也是 0。
* “层数”跟深度的计算类似，不过，计数起点是 1，也就是说根节点的位于第 1 层。

### 二叉树（Binary Tree）
二叉树，顾名思义，每个节点最多有两个“叉”，也就是两个子节点，分别是左子节点和右子节点。不过，二叉树并不要求每个节点都有两个子节点，有的节点只有左子节点，有的节点只有右子节点。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-5.jpg">
<div style="clear: both"></div>

这个图里面，有两个比较特殊的二叉树，分别是编号 2 和编号 3 这两个：
* 其中，编号 2 的二叉树中，叶子节点全都在最底层，除了叶子节点之外，每个节点都有左右两个子节点，这种二叉树就叫作**满二叉树**；
* 编号 3 的二叉树中，叶子节点都在最底下两层，最后一层的叶子节点都靠左排列，并且除了最后一层，其他层的节点个数都要达到最大，这种二叉树叫作**完全二叉树**；

满二叉树很好理解，也很好识别，但是完全二叉树，有的人可能就分不清了。我画了几个完全二叉树和非完全二叉树的例子，你可以对比着看看。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-6.jpg">
<div style="clear: both"></div>

### 如何表示（或者存储）一棵二叉树？

想要存储一棵二叉树，我们有两种方法，一种是基于指针或者引用的**二叉链式存储法**，一种是**基于数组的顺序存储法**。
* 我们先来看比较简单、直观的**链式存储法**。从图中你应该可以很清楚地看到，每个节点有三个字段，其中一个存储数据，另外两个是指向左右子节点的指针。我们只要拎住根节点，就可以通过左右子节点的指针，把整棵树都串起来。这种存储方式我们比较常用。大部分二叉树代码都是通过这种结构来实现的。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-7.jpg">
<div style="clear: both"></div>

* 我们再来看，**基于数组的顺序存储法**。我们把根节点存储在下标 i = 1 的位置，那左子节点存储在下标 2 * i = 2 的位置，右子节点存储在 2 * i + 1 = 3 的位置。以此类推，B 节点的左子节点存储在 2 * i = 2 * 2 = 4 的位置，右子节点存储在 2 * i + 1 = 2 * 2 + 1 = 5 的位置。
总结如下：如果节点 X 存储在数组中下标为 i 的位置，下标为 2 * i 的位置存储的就是左子节点，下标为 2 * i + 1 的位置存储的就是右子节点。反过来，下标为 i/2 的位置存储就是它的父节点。通过这种方式，我们只要知道根节点存储的位置（一般情况下，为了方便计算子节点，根节点会存储在下标为 1 的位置），这样就可以通过下标计算，把整棵树都串起来。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-8.jpg">
<div style="clear: both"></div>
不过，我刚刚举的例子是一棵完全二叉树，所以仅仅“浪费”了一个下标为 0 的存储位置。如果是非完全二叉树，其实会浪费比较多的数组存储空间，如下图所示。
<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-9.jpg">
<div style="clear: both"></div>
所以，如果某棵二叉树是一棵完全二叉树，那用数组存储无疑是最节省内存的一种方式。因为数组的存储方式并不需要像链式存储法那样，要存储额外的左右子节点的指针。这也是为什么完全二叉树会单独拎出来的原因，也是为什么完全二叉树要求最后一层的子节点都靠左的原因。

### 二叉树的遍历
经典的方法有三种，前序遍历、中序遍历和后序遍历。其中，前、中、后序，表示的是节点与它的左右子树节点遍历打印的先后顺序。
* **前序遍历**是指，对于树中的任意节点来说，先打印这个节点，然后再打印它的左子树，最后打印它的右子树；
* **中序遍历**是指，对于树中的任意节点来说，先打印它的左子树，然后再打印它本身，最后打印它的右子树；
* **后序遍历**是指，对于树中的任意节点来说，先打印它的左子树，然后再打印它的右子树，最后打印这个节点本身；

<img width="500px" height="auto" style="float: left;" src="./binary-tree1/binary-tree1-10.jpg">
<div style="clear: both"></div>

**实际上，二叉树的前、中、后序遍历就是一个递归的过程**。比如，前序遍历，其实就是先打印根节点，然后再递归地打印左子树，最后递归地打印右子树。

遍历递推公式代码实现：
```
void preOrder(Node* root) {
  if (root == null) return;
  print root // 此处为伪代码，表示打印root节点
  preOrder(root->left);
  preOrder(root->right);
}

void inOrder(Node* root) {
  if (root == null) return;
  inOrder(root->left);
  print root // 此处为伪代码，表示打印root节点
  inOrder(root->right);
}

void postOrder(Node* root) {
  if (root == null) return;
  postOrder(root->left);
  postOrder(root->right);
  print root // 此处为伪代码，表示打印root节点
}
```

补充：
* 从我前面画的前、中、后序遍历的顺序图，可以看出来，每个节点最多会被访问两次，所以遍历操作的时间复杂度，跟节点的个数 n 成正比，**也就是说二叉树遍历的时间复杂度是 O(n)**；
* **按层遍历二叉树，利用队列实现**；