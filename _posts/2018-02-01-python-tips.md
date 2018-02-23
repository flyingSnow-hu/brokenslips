---
layout: timemachine
title: python 鸡零狗碎
tags: python tips
date: 2018-02-01
category: python
---
# python 鸡零狗碎

 * os.path.join() 可以连接结尾是正反斜线的字符串，不需要特意去掉
 ```python
 print os.path.join('aa\\','bb\\')# 'aa\bb\'
 ```

 * os.path.join() 是按照路径解释字符串，而非普通字符串
  ```python
  print os.path.join('aa/','/bb')# '/bb'
  ```
  这里'/bb'实际上是个根目录的路径，所以把'aa/'挤兑没了

 * 即使是r字符串结尾也不能是反斜线（转义符）
  ```python
  print r'some_path\' # error
  ```
   ```python
   print r'some_path\\' # right
   ```

 * python 中模拟 enum
  ```python
  def __error_enum(**enums):
    return type('__ERROR_CODE', (), enums)
 
  ERROR_CODE = __error_enum(
    FILE_ERROR=1100,
    UNKNOWN_PLATFORM=2100,
    UNKNOWN_CHANNEL=2200, 
    UNKNOWN_LANGUAGE=2300,
  )
  ```
