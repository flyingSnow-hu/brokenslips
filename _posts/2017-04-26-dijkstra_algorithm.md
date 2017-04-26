---
layout: timemachine
title: Dijkstra及寻路算法
tags: others sample
date: 2017-04-26
category: internal-strength
---
# Dijkstra算法
## 基本Dijkstra
```pseudocode
   function Dijkstra(Graph, source, target):

       create vertex set Q

       for each vertex v in Graph:             // 初始化部分
           dist[v] ← INFINITY                  // 初始距离是正无穷
           prev[v] ← UNDEFINED                 // 在最终路径上的前驱节点，初始是空
           add v to Q                          // 通通加入未访问节点列表Q

        dist[source] ← 0                        // 到起点的距离，起点到起点为0

        while Q is not empty:
            u ← vertex in Q with min dist[u]    // 取出Q队列里距离最短的节点u
            if u == target:
                finish                          // 算法结束

            remove u from Q

            for each neighbor v of u:           // u的每一个未访问邻居v
                alt ← dist[u] + length(u, v)
                if alt < dist[v]:               // u的总距离加上u到v的距离，跟v的当前距离做比较，更新v的距离
                    dist[v] ← alt
                    prev[v] ← u                 //如果更新了，u就是v的前驱

        return dist[], prev[]
```

## 以优先队列优化的Dijkstra
主要优化点是从Q里面取出最小值的过程，可以用斐波那契堆
``` pseudocode
  function Dijkstra(Graph, source):
      dist[source] ← 0                                    // 到起点的距离，起点到起点为0

      create vertex set Q

      for each vertex v in Graph:           
          if v ≠ source
              dist[v] ← INFINITY                          // 各种初始化
              prev[v] ← UNDEFINED                         // 各种初始化

          Q.add_with_priority(v, dist[v])                 // Q变成了斐波那契堆


      while Q is not empty:                              // The main loop
          u ← Q.extract_min()                            // Remove and return best vertex
          for each neighbor v of u:                      // only v that is still in Q
              alt ← dist[u] + length(u, v)
              if alt < dist[v]
                  dist[v] ← alt
                  prev[v] ← u
                  Q.decrease_priority(v, alt)

      return dist[], prev[]
```


## A*算法
加了启发函数的 Dijkstra算法，计算距离时多附加一个启发函数的估值，表示到顶点的估算距离  
启发函数一般是欧几里德距离/曼哈顿距离/切比雪夫距离  
Dijkstra算法可以视为估值函数为 f(x)=0 的A*算法  

## 贝尔曼-福特算法（Bellman–Ford algorithm）

Dijkstra算法是深度优先遍历的，BF算法是广度优先遍历，优点是可以处理负权重的边，缺点是复杂度高
