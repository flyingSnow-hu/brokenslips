---
layout: article
title: AI博弈树基础笔记
tags: others ai
date: 2017-04-11
category: ai
---
# AI博弈树基础笔记  

## 基础问题

 * 比特棋盘：基础中的基础，可以分类表示各种棋盘信息，比如所有棋子、所有红方的棋子、某个位置的象可到达的所有位置以及象眼位置  
 * 走法产生：跟特定棋类知识有关，可以用比特棋盘预存成表，产生顺序可以调整以配合剪枝  
 * 博弈树：与或树？极大极小值搜索，负极大值搜索  
 * 启发式搜索：固定深度，带静态估值函数（启发函数）  
 * 深度优先：内存方面最优，搜索速度不差  

## 估值函数  
  * 棋子本身静态价值：车比马好，帅比车重要
  * 棋子的动态控制力：过河兵和非过河兵
  * 和其他棋子的关系：将军的马和被蹩腿的马，威胁和保护
  * 特定棋形：连环马，围棋定势
  * 棋子价值速查表 :[棋子,位置]->估值

## 搜索算法
  * Alpha-Beta 剪枝：基础中的基础  
  ```pseudocode 
  01 function negamax(node, depth, α, β, color)
  02     if depth = 0 or node is a terminal node
  03         return color * the heuristic value of node

  04     childNodes := GenerateMoves(node)
  05     childNodes := OrderMoves(childNodes)
  06     bestValue := −∞
  07     foreach child in childNodes
  08         v := −negamax(child, depth − 1, −β, −α, −color)
  09         bestValue := max( bestValue, v )
  10         α := max( α, v )
  11         if α ≥ β
  12             break
  13     return bestValue
  ```  

  * Fail-soft alpha-beta: 加窗的AB剪枝，一开始就限制ab的范围，如果搜索失败，视返回值情况决定是向下扩展还是向上扩展
  * 渴望搜索: 加什么样的窗需要研究一下
  * 极小窗口搜索（PVS算法/NegaScout算法）：先探测第一个分支，返回最佳值v，第二个分支开始以(v,v+1)开始加窗
  * 迭代深化：  
       - 以时间而非深度去限制搜索层次  
       - 上一级遍历的最佳走法，本层级优先搜索（可以按节点值做个排序？），因为AB剪枝对节点顺序敏感  
  * 历史启发：保存一个走法->得分的字典，每次选出的最佳走法加分(+2^depth)，产生走法节点时按照历史得分排序
  * SSS/DUAL 算法：广搜，等效于AB+置换表，然而不如其灵活
  * MTD 算法+TT：多次空窗AB探测，逼近上界和下界
  * 静止期搜索：动态调整搜索深度，对于变化剧烈的局面多搜索几步：将军、吃子、异常（比如和兄弟节点的差距过大）、威胁、巨变（和父节点的差距过大）

## 局面Hash
  * Zobrist 哈希表：对整个局面做哈希
  * 置换表（TT）：把搜索过的局面存起来，以便  
   1. 以后检索到同样局面直接用
   1. 规则里检查/筛掉重复局面

## 其他手段
  * 开局库：基于输入棋谱
  * 残局库：基于所有残局自动演算
  * 循环探测：同时生成最佳和次佳方案
  * 轻视因子：「我以为你看不到这步棋呢」
  * 技术角度：减少函数开销，查表代替计算，交叉递归减少运算，减少垃圾回收

## 总结：比较好的方案
  1. NegaScout + TT + 历史启发
  1. AB + 历史启发
  1. MTD
  1. AB + TT
