---
layout: article
title: 算法导论·Dijkstra算法
tags: introduction_to_algorithms dijkstra
date: 2018-02-06
category: introduction_to_algorithms
---
# 算法导论·Dijkstra算法

```python
# encoding:utf-8

from pprint import pprint

from datastructure.weighteddigraph import WeightedDigraph
from datastructure.heap import MinHeap

def main():
	g = WeightedDigraph()
	g.add_edge('s','t',10)
	g.add_edge('s','y',5)
	g.add_edge('y','t',3)
	g.add_edge('t','y',2)
	g.add_edge('t','x',1)
	g.add_edge('y','x',9)
	g.add_edge('y','z',2)
	g.add_edge('z','x',6)
	g.add_edge('x','z',4)
	g.add_edge('z','s',7)

	heap = MinHeap()
	distances = {}
	for v in g.get_verticles():
		heap.push([0 if v == 's' else float('inf'),v,[]])

	while not heap.is_empty():		
		top_distance,top_verticle,path = distances[top_verticle] = heap.pop()
		out_edges = g.get_edges_from(top_verticle)
		for edge in out_edges:
			dvs = [d_v for d_v in heap.get_all() if d_v[1] == edge.to_]
			for d_v in dvs:
				if top_distance + edge.weight < d_v[0]:
					d_v[0] = top_distance + edge.weight
					d_v[2] = edge
					# pprint('**************{}'.format(heap.get_index(d_v)))
					# pprint(heap.get_all())
					heap.swim(heap.get_index(d_v))
					# pprint(heap.get_all())

	pprint(distances)


if __name__ == '__main__':
	main()

```