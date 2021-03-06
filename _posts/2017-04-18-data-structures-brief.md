---
layout: article
title: 数据结构与算法简要提纲
tags: data-structure
date: 2017-04-18
category: internal-strength
---
# 数据结构与算法简要提纲

## 数据结构
### 线性表
 * 链表：单链表、双链表、循环链表
  * 时间复杂度：
	  * 索引：O(n)
	  * 查找：O(n)
	  * 插入：O(1)
	  * 删除：O(1)

  * 栈：push、pop、LIFO
    * 时间复杂度：
      * 索引：O(n)
      * 查找：O(n)
      * 插入：O(1)
      * 删除：O(1)

  * 队列：enqueue、dequeue、FIFO
    * 时间复杂度：
      * 索引：O(n)
      * 查找：O(n)
      * 插入：O(1)
      * 删除：O(1)
  * 跳表：带跳跃索引的链表

### 树
 * 无向、联通、无环图
 * 二叉树：最多两个子节点
  * 满二叉树：有0或2个子节点
  * 完美二叉树：所有叶子节点的深度一样的满二叉树
  * 完全二叉树：完美二叉树减去最后一层靠右边的部分连续节点
  * 二叉查找树BST：左子节点<本节点<右子节点，时间复杂度O(log(n))
 * 字典树：按路径顺序存储字符串
 * 二进制索引树BIT：树状数组，快速区间求和，时间复杂度O(log(n))
 * 线段树：允许查找一个节点在若干条线段中出现的次数，时间复杂度O(log(n))
 * 堆：最大堆、最小堆，时间复杂度O(log(n))
 * 斐波那契堆
 * B树，红黑树，AVL树，Splay Tree, Treep

### 哈希表
 * 任意长度的数据映射到固定长度，可能有冲突
 * 解决冲突：链地址-每个哈希对应一个元素列表、开放地址-每个哈希对应一个元素，哈希值不固定

### 图
 * 边和顶点的集合
 * 无向图、有向图
 * 深搜：前序中序后续遍历
 * 广搜：按层级遍历
 * 拓扑排序 Topological sorting  
   在图论中，由一个有向无环图的顶点组成的序列，当且仅当满足下列条件时，称为该图的一个拓扑排序:
   * 每个顶点出现且只出现一次；
   * 若A在序列中排在B的前面，则在图中不存在从B到A的路径。
   也可以定义为：拓扑排序是对有向无环图的顶点的一种排序，它使得如果存在一条从顶点A到顶点B的路径，那么在排序中B出现在A的后面
   * 深搜和广搜都是树的拓扑排序
 * Dijkstra算法：在有向图中查找单源最短路径的算法，时间复杂度：O(V^2)
 * A*算法：带启发函数的Dijkstra算法
 * Bellman-Ford 算法：一种在带权图中查找单一源点到其他节点最短路径的算法。虽然时间复杂度大于 Dijkstra 算法，但它可以处理包含了负值边的图。
 * Floyd-Warshall 算法：是一种在无环带权图中寻找任意节点间最短路径的算法。时间复杂度：O(V^3)
 * 最小生成树算法：一种在无向带权图中查找 **最小生成树** 的贪心算法。换言之，能在一个图中找到连接所有节点的边的最小子集。时间复杂度：O(V^2)
 * Kruskal 算法：也是一个计算最小生成树的贪心算法，但在 Kruskal 算法中，图不一定是连通的。时间复杂度：O(E*logV)

## 算法
### 排序算法
 * 快速排序
 * 归并排序
 * 桶排序
 * 基数排序
