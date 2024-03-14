---
layout: article
title: DualKeyDictionary
tags: C# dualkey dictionary
date: 2021-09-06
category: c#
---
# DualKeyDictionary

## 双键字典，我们的目标是  

```C#
public class DualKeyDictionary<TK1, TK2, TV> dict;
```

```C#
dict.Set(k1, k2, $"{k1}.{k2}");
dict.Get(k1, k2);

foreach (var pair in dict){
    // pair.key1, pair.key2, pair.value
} 

foreach(var pair in dict.GetCollectionByKey1(k1)){

} 

foreach(var pair in dict.GetCollectionByKey2(k2)){

}
```

## 单键字典的实现



(图缺)

## 扩展到双键字典

每个 entry 应该处于两条链上，所以设两个指针，分别指向两条链的下一个节点。(图缺)

```C#  
dict.Set("Apple", "Argentina", "AA");
dict.Set("Apple", "Brasil", "AB");
dict.Set("Apple", "China", "AC");

dict.Set("Banana", "Argentina", "BA");
dict.Set("Banana", "Brasil", "BB");
dict.Set("Banana", "China", "BC");

dict.Set("Coconut", "Argentina", "CA");
dict.Set("Coconut", "Brasil", "CB");
dict.Set("Coconut", "China", "CC");
```   

* 加进一个丹麦火龙果(图缺)  


## 删除和空洞

每次删除的时候，entries 里就会留下一个空洞。MS 的做法是以一个 freeList 链表储存起来，我们这里也一样，不过空洞链表只需要用到 next1 指针。
增加新值的时候，优先考虑使用 freeList 里的位置，否则使用当前 count + 1。如果 entries 满了就要扩容。

## 扩容

Entries 不够用时候向上扩容：扩容到最近的质数

```C#
public static readonly int[] primes = {
    3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
    1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
    17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
    187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
    1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369};
```
** 向下扩容：** 不存在的


## 其他

### IEnumerable
GetEnumerator()
IEnumerator

```C#
    IEnumerator.Current
    public void Dispose()
    public bool MoveNext()
    public void Reset()
```

### 性能

字符串 Hash 的性能  
质数的性能  
频繁扩容造成的性能损失和扩容过度无法回收的内存  