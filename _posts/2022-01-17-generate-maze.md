---
layout: article
title: 生成迷宫
tags: maze
date: 2022-01-17
category: tips
---
# 生成迷宫

基本的深度优先搜索算法：方格迷宫
(图1)

假如给定这样一个迷宫，起点是左下角，终点是右上角，容易想到使用深度优先搜索寻找路线。
同样的道理，对于一个未初始化的迷宫，也可以使用深度优先搜索生成路线。整个过程是等用于把图展开成树的过程。

这里假设有一个链表 path，类型是 point，初始状态只包括起点一个元素。
Step 方法负责进行一小步深度搜索。每次从链表的尾部取出一个节点，以随机方向访问它的邻居，如果这个节点有未搜索过的邻居，则把邻居标记为已搜索、压入链表并返回。如果所有邻居都已经搜索过，则把尾节点从链表中弹出。
一直循环 Step 方法直到 path 为空，即生成了一个迷宫。

```C#
    public bool Step()
    {
        if (path.Count == 0) return true;

        var crnt = path.Last.Value;
        ShuffleDirections();
        foreach (var direction in directions)
        {
            var next = crnt + direction;
            var gridState = getGridState(next.x, next.y);
            if(gridState == GridState.Unchecked)
            {
                setGridState(next.x, next.y, GridState.Way);
                BreakWall(crnt.x, crnt.y, direction);
                BreakWall(next.x, next.y, -direction);
                path.AddLast(next);
                return false;
            } 
        }

        // 这个点的周围没有路了         
        path.RemoveLast();

        return false;
    }
```

以 15×15 的迷宫为例，生成的迷宫如下：
(图2)

看起来很完美的迷宫。
称为完美迷宫，即没有回路，也没有孤立区域的迷宫。用图论来解释，就是可以用生成树表示的迷宫，迷宫中两点有且仅有一条路径。
这个算法不受尺寸限制，可以设计成任何比例。
从游戏的角度考虑，完美迷宫的任意两点之间有且只有一条路径，所以我们可以先生成迷宫，再在合适的位置放置起点和终点，甚至可以放置任意的房间和打卡点供玩家探索。

但试走一次就知道，这个迷宫有一个很明显的问题：以人类的视角不够复杂。
体现在两个方面：一是没有足够长的岔路，所有的岔路都很快走到尽头，作为人类一瞥而知这是条死路。而且主路线上有很长一段没有岔路。这个问题是深度优先的搜索方式造成的。
二是没有回环，也大大降低了迷宫的乐趣。需要注意的是回环不能包括主路线上的点，否则就形成了两条正确路线，会降低迷宫难度。

所以还要使用一些手段增加迷宫的对人类难度。

## 增加规模

这是最简单的手段，不过听起来不是很完美。下面是一个未完成的100×100的迷宫：（图缺）

这样的迷宫看起来让人眼花，但是实际测试下来，仍然没法避免之前说的两个缺点，乐趣没有和规模同步提升。

## 增加岔路

深度优先搜索的方式倾向于优先搜索主路线，如果要增加岔路，自然让人联想到广度优先搜索。对于上面的代码，只要把16行的插入从链表尾部改到链表头部，就变成了广度优先搜索。
不过单纯的广度优先搜索结果也不完美，比深度优先更糟糕：
(图3)

但是既然这两种代码如此相像，我们可以结合这两种方式，在每一个节点随机地决定采用广度搜索还是深度搜索。甚至可以尝试在不同的阶段给予不同的概率策略。
(图4)
这样看起来就好多了。

## Prim 算法

既然深度优先搜索从链表尾部取节点，广度优先搜索从链表头部取节点，而我们实现了每一步随机从尾部或者头部取节点。进一步容易想到，如果是从链表的所有节点里随机取下一个节点呢。
基于以上的算法，从链表中间随机取下一个节点的算法就是Prim算法。

## Kruskal 算法

构造普通的二维迷宫可以视作权重相等的最小生成树 Minimum spanning tree 问题，prim 算法和 kruskal 算法都是最小生成树的算法。由于我们的迷宫内权重处处相等，所以对两种算法都做了简化，以随机取代了排序，这样比之于标准算法节省了大量时间和复杂度。

## 递归分割

## 启发函数
控制岔路和转角的概率

（参考：随机选取链表元素）

## 动态分配起点和终点

## 增加回环

方法四：增加Z轴
方法五：改变形状：六边形、圆环形、任意路点

## 量化迷宫的难度


----
参考
https://blog.csdn.net/imred/article/details/105329806?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_default&spm=1001.2101.3001.4242.1&utm_relevant_index=3