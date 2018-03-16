---
layout: timemachine
title: Python 各种读文件方法
tags: python readlines
date: 2018-03-116
category: python
---
# Python 各种读文件方法

**需求**：一个很多行的文本文件，我们要逐行处理  
**方法**：四种，见代码  
```python  

@profile('read')
def test_read():
	count = 0
	with open('content.txt') as f:
		contents = f.read()
		for line in contents.splitlines():
			count += len(line)
	print count


@profile('readlines')
def test_readlines():
	count = 0
	with open('content.txt') as f:
		for line in f.readlines():
			count += len(line[:-1])
	print count


@profile('readline')
def test_readline():
	count = 0
	with open('content.txt') as f:
		while True:
			line = f.readline()
			if not line: break
			count += len(line[:-1])
	print count


@profile('for-loop')
def test_for_loop():
	count = 0
	for line in open('content.txt'):
		count += len(line[:-1])
	print count

```

顺便附上 profile 方法：  
```python  
def profile(title=None):
	def decorate(func):
		@wraps(func)
		def wrapper(*args,**kwargs):
			name = title if title is not None else '%s.%s'%(func.__module__,func.__name__)

			print('\033[32m %s 开始 \033[0m'%name)
			start = time.clock()
			r = func(*args,**kwargs)
			end = time.clock()
			print('\033[35m %s 完成:%ss \033[0m'%(name,int(100*(end - start)+0.5)/100.0))

			return r
		return wrapper
	return decorate
```

**数据**：读一个 26k，390w行的文件

|方法|时间|
|---|---|
| read     | 0.41s |  
| readlines| 0.53s |  
| readline | 0.94s |  
| for-loop | 0.39s |  


**结论**: for循环迭代器方法又快又好，最慢的是readline，readlines反倒比 read 还慢一点
