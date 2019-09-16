---
layout: articles
title: 算法导论·动态规划·编辑距离
tags: introduction_to_algorithms dynamic_programming
date: 2018-01-27
category: introduction_to_algorithms
---
# 算法导论·动态规划·编辑距离
原书习题 15-5 第一部分
```python
#encoding:utf-8

from collections import namedtuple

x = 'algorisssssthmssss'
y = 'altroidddthm'
z = [''] * len(y)
costs = namedtuple('Costs',['copy','replace','delete','insert','twiddle','kill'])(0,1,2,2,1,0)

m = len(x)
n = len(y)

i = j = 0
total_costs = 0

def copy():
	global x,z,i,j,costs,total_costs
	z[j] = x[i]
	i += 1
	j += 1
	total_costs += costs.copy

def undo_copy():
	global i,j,z,costs,total_costs
	j -= 1
	i -= 1
	z[j] = ''
	total_costs -= costs.copy

def replace():
	global x,z,i,j,costs,total_costs
	z[j] = y[j]
	i += 1
	j += 1
	total_costs += costs.replace

def undo_replace():
	global i,j,z,costs,total_costs
	j -= 1
	i -= 1
	z[j] = ''
	total_costs -= costs.replace

def delete():
	global x,z,i,j,costs,total_costs	
	i += 1
	total_costs += costs.delete

def undo_delete():
	global i,costs,total_costs
	i -= 1
	total_costs -= costs.delete

def insert():
	global x,z,i,j,costs,total_costs
	z[j] = y[j]
	j += 1
	total_costs += costs.insert

def undo_insert():
	global j,z,costs,total_costs
	j -= 1
	z[j] = ''
	total_costs -= costs.insert
 
def twiddle():
	global x,z,i,j,costs,total_costs
	z[j] = x[i+1]
	z[j+1] = x[i]
	i += 2
	j += 2
	total_costs += costs.twiddle

def undo_twiddle():
	global i,j,z,costs,total_costs
	j -= 2
	i -= 2
	z[j+1] = ''
	z[j] = ''
	total_costs -= costs.twiddle

def kill():
	global x,z,i,j,costs,total_costs,m
	i = m + 1
	total_costs += costs.kill

def undo_kill(i_):
	global i,costs,total_costs
	i = i_
	total_costs -= costs.kill

def finish():
	global x,y,z,i,j,costs,total_costs
	if ''.join(z) == y:
		print ('OK!')
	else:
		print('ERROR:\n\ty:{0}\n\tz:{1}'.format(','.join(y),','.join(z)))

def memoization():
	global x,y,z,i,j,m,n,costs,total_costs

	memo = {}

	def get_minimal_cost():
		# save values
		# try each operation
			# do operation
			# save minimal value
			# undo
		global x,y,z,i,j,m,n,costs,total_costs
		if (i,j) in memo: return memo[(i,j)]

		# i_,j_,total_costs_ = i,j,total_costs
		min_cost = float('inf')
		min_operations = []

		if j<n and i<m and y[j]==x[i]:
			copy()
			sub_cost,sub_op = get_minimal_cost()
			cost = costs.copy + sub_cost
			if min_cost > cost:
				min_cost = cost
				min_operation = ['copy'] + sub_op
			undo_copy()

		if j<n and i<m:
			replace()
			sub_cost,sub_op = get_minimal_cost()
			cost = costs.replace + sub_cost
			if min_cost > cost:
				min_cost = cost
				min_operation = ['replace'] + sub_op
			undo_replace()

		if i<m:
			delete()
			sub_cost,sub_op = get_minimal_cost()
			cost = costs.delete + sub_cost
			if min_cost > cost:
				min_cost = cost
				min_operation = ['delete'] + sub_op
			undo_delete()

		if j<n:
			insert()
			sub_cost,sub_op = get_minimal_cost()
			cost = costs.insert + sub_cost
			if min_cost > cost:
				min_cost = cost
				min_operation = ['insert'] + sub_op
			undo_insert()

		if i < m-1 and j < n - 1 and x[i] == y[j+1] and y[j] == x[i+1]:
			twiddle()
			sub_cost,sub_op = get_minimal_cost()
			cost = costs.twiddle + sub_cost
			if min_cost > cost:
				min_cost = cost
				min_operation = ['twiddle'] + sub_op
			undo_twiddle()

		if i>=m and j>=n:
			i_ = i
			kill()
			cost = costs.kill
			if min_cost > cost:
				min_cost = cost
				min_operation = ['kill']
			undo_kill(i_)

		# min_operation()
		memo[(i,j)] = (min_cost,min_operation)
		return min_cost,min_operation

	min_cost,operations = get_minimal_cost()
	print(min_cost)
	for op in operations:
		eval('{0}()'.format(op))
		print(op,'->',''.join(z))
	finish()

def dynamic_prog():	
	global x,y,z,m,n,costs

	memo = {}#(i,j)->(cost,op,next_node)
	memo[(m,n)] = (0,'finish',None)

	EMPTY = (0,'',None)
	MAX = (float('inf'),'',None)
	
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

			if i_ >= n:
				c = costs.kill
				if c<min_cost[0]:
					min_cost = (c,'kill',(m,n))

			memo[i_,j_] = min_cost

	#output the result
	p = get_memo(0,0)
	while p[2] != None:
		eval('{}()'.format(p[1]))
		print(p,''.join(z))
		p = get_memo(*p[2])

	finish()

if __name__ == '__main__':
	# memoization()
	dynamic_prog()


```