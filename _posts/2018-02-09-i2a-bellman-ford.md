---
layout: articles
title: 算法导论·Bellman-Ford 算法
tags: introduction_to_algorithms bellman-ford
date: 2018-02-09
category: introduction_to_algorithms
---
# 算法导论·Bellman-Ford 算法

```python
from pprint import pprint

from datastructure.weighteddigraph import WeightedDigraph
from datastructure.heap import MinHeap

INF = float('inf')

def main():
	g = WeightedDigraph()
	g.add_edge('s','t',6)#这里改成-6就会有负环
	g.add_edge('s','y',7)
	g.add_edge('t','z',-4)
	g.add_edge('t','y',8)
	g.add_edge('t','x',5)
	g.add_edge('y','x',-3)
	g.add_edge('y','z',9)
	g.add_edge('x','t',-2)
	g.add_edge('z','x',7)
	g.add_edge('z','s',2)
	
	vertices = g.get_verticles()
	length_dict = {v:0 if v == 's' else INF for v in vertices}

	edges = g.get_edges()
	for n in range(0,len(vertices)-1):
		for edge in edges:
			if length_dict[edge.from_] + edge.weight < length_dict[edge.to_]:
				length_dict[edge.to_] = length_dict[edge.from_] + edge.weight

	pprint(length_dict)

	for edge in edges:
		if length_dict[edge.from_] + edge.weight < length_dict[edge.to_]:
			print('有负权环！')
			pprint(length_dict)

if __name__ == '__main__':
	main()

```