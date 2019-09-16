---
layout: articles
title: 算法导论·动态规划·DNA距离
tags: introduction_to_algorithms dynamic_programming
date: 2018-01-27
category: introduction_to_algorithms
---
# 算法导论·动态规划·DNA距离
原书习题 15-5 第二部分
```python
#encoding:utf-8

from collections import namedtuple

x = 'GATCGGCAT'
y = 'CAATGTGAATC'
x_ = [''] * (len(x)+len(y))
y_ = [''] * (len(x)+len(y))
costs = namedtuple('Costs',['copy','replace','delete','insert','twiddle','kill'])(-1,float('inf'),2,2,float('inf'),float('inf'))

m = len(x)
n = len(y)

i = j = 0 # pointers to src strs x and y
i_ = j_ = 0 # pointer to dist strs x_ and y_

def copy():
	global x,y,x_,y_,i,j,i_,j_,costs
	x_[i_] = y_[j_] = x[i]
	i += 1
	j += 1
	i_ += 1
	j_ += 1

def replace():
	raise Exception('No replace')

def delete():
	global x,y,x_,y_,i,j,i_,j_,costs
	x_[i_]=x[i]
	y_[j_] = ' '
	i += 1
	i_ += 1
	j_ += 1
	# j += 1

def insert():
	global x,y,x_,y_,i,j,i_,j_,costs
	x_[i_] = ' '
	y_[j_] = y[j] 
	# i += 1
	j += 1
	i_ += 1
	j_ += 1
 
def twiddle():
	raise Exception('No twiddle')

def kill():	
	raise Exception('No kill')

def finish():
	global x,y,x_,y_
	# print (x,y)
	print (''.join(x_))
	print (''.join(y_))

def dynamic_prog():	
	global x,y,m,n,costs,i,j,x_,y_

	EMPTY = (0,'',None)
	MAX = (float('inf'),'',None)
	FINISH = (0,'finish',None)

	memo = {}#(i,j)->(cost,op,next_node)
	memo[(m,n)] = FINISH
	
	def get_memo(i_,j_):
		if i_>m or j_>n:
			return MAX
		return memo[(i_,j_)]

	# while i>=0 and j >= 0:		
	for i_ in range(m,-1,-1):
		for j_ in range(n,-1,-1):
			print(i_,j_)
			if i_ == m and j_ == n:
				continue

			min_cost = MAX
			if i_ < m and j_ < n and x[i_] == y[j_]: 
				c = costs.copy + get_memo(i_+1,j_+1)[0]
				if c<min_cost[0]:
					min_cost = (c,'copy',(i_+1,j_+1))

			if i_ < m and j_ < n:
				c = costs.replace + get_memo(i_+1,j_+1)[0]
				if c<min_cost[0]:
					min_cost = (c,'replace',(i_+1,j_+1))

			if j_ < n:
				c = costs.insert + get_memo(i_,j_+1)[0]
				if c<min_cost[0]:
					min_cost = (c,'insert',(i_,j_+1))

			if i_ < m:
				c = costs.delete + get_memo(i_+1,j_)[0]
				if c<min_cost[0]:
					min_cost = (c,'delete',(i_+1,j_))

			if i_ < m - 1 and j_ < n - 1 and x[i_] == y[j_+1] and x[i_+1] == y[j_]:
				c = costs.twiddle + get_memo(i_+2,j_+2)[0]
				if c<min_cost[0]:
					min_cost = (c,'twiddle',(i_+2,j_+2))

			if i_ == n:
				c = costs.kill
				if c<min_cost[0]:
					min_cost = (c,'kill',(m,n))

			memo[i_,j_] = min_cost

	#output the result
	p = get_memo(0,0)
	while p[2] != None:
		eval('{}()'.format(p[1]))
		print(p,'\n\tx_:[{}],\n\ty_:[{}]'.format(''.join(x_),''.join(y_)))
		p = get_memo(*p[2]) 

	finish()

if __name__ == '__main__':
	# memoization()
	dynamic_prog()


```