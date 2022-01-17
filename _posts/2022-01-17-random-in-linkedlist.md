---
layout: article
title: 随机选取链表元素
tags: algorithms random linkedlist
date: 2022-01-17
category: internal-strength
---
#随机选取链表元素

**问题1：** 有一个长度未知但有限的链表K={k1, k2, k3...}，现在需要从链表中选取一个元素，使得 K 中每一个元素被选中的概率都相等。  
**解决1：** 从前到后遍历链表。设当前元素为 kn，则从[0,n-1] 中随机一个整数 r，如果 r==0 则令 kn 为选中值。等链表遍历过一次之后，将选中值返回。**时间复杂度为O(n)。**  

```python
ret = k.first
crnt_node = k.first
count = 1
while crnt_node.next != None:
    # 这里把整数变成了等价的连续区间概率
    if rand() < 1 / count:
        ret = crnt_node
    crnt_node = crnt_node.next
    count++
```

> 💡考虑第一个节点被选中的概率 p1：  
> 当 n = 1 时，概率是 1；  
> 当 n = 2 时，概率是 1/2；  
> 当 n = 3 时，概率是在 n=2 选中的基础上乘以 2/3，因此是 1/2 × 2/3；  
> 继续推算 n=4 时概率是 1/2 × 2/3 × 3/4；  
> ...  
> 假设链表长度为L，n = L 时概率是 p1 = 1/2 × 2/3 × 3/4 × ... × (L-1)/L = 1/L  
> 考虑第 k 个节点被选中的概率 pk，从 n=k 开始：  
> 当 n = k 时，概率是 1/k；  
> 当 n = k+1 时，概率是 1/k × (k/k+1)；  
> 当 n = k+2 时，概率是 1/k × k/(k+1) × (k+1)/(k+2)；  
> ...  
> 假设链表长度为L，n = L 时概率是 pk = 1/k × k/(k+1) × (k+1)/(k+2) × ... × (L-1)/L = 1/L  

----
**问题2：**有一个长度未知但有限的链表K={k1, k2, k3...}，现在需要从链表中选取一个元素，使得 K 中每一个元素被选中的概率权重为W={w1, w2, w3...}。**时间复杂度依然是O(n)。**  
**解决2：**从前到后遍历链表。设当前元素为 kn，则从[0,1) 中随机一个浮点值 r，如果 r≥(Σn-1)/(Σn) 则令 kn 为选中值。等链表遍历过一次之后，将选中值返回。这里 Σn = w1+...+wn  

```python
ret = k.first
crnt_node = k.first
summed_weight = k.first.weight
while crnt_node.next != None:
    old_weight = summed_weight
    summed_weight = summed_weight + crnt_node.weight
    if rand() >= old_weight / summed_weight:
        ret = crnt_node
    crnt_node = crnt_node.next
``` 

> 💡考虑第一个节点被选中的概率 p1：  
> 当 n = 1 时，概率是 1；  
> 当 n = 2 时，概率是 w1/(w1+w2)；  
> 当 n = 3 时，概率是在 n=2 选中的基础上乘以 (w1+w2)/(w1+w2+w3)，因此是 w1/(w1+w2) × (w1+w2)/(w1+w2+w3)=w1/(w1+w2+w3)；  
> ...  
> 假设链表长度为L，n = L 时概率是 p1 = w1/(w1+w2) × (w1+w2)/(w1+w2+w3) × ... × Σ(L-1)/ΣL = w1/ΣL  
>   
> 同理可得 pk = wk/ΣL.  *Q.E.D.*  
> 特别地，当 ΣL = 1 ，也就是 W 列表的权重和等于1，pk = wk  
----
**问题3：**有一个长度未知但有限的链表K={k1, k2, k3...}，对任意 n 有f(kn)=Tn，Tn∈{t1,t2,...tm}。现在需要从 K 中选取一个元素 k，使得 f(k) 的概率权重为W={w1, w2, ..., wt}。f(k)相等的物体概率相等。    
举例：有若干张纸牌，每张纸牌归属于一个特定的花色Tn，需要按花色权重随机选取卡牌，同花色的卡牌概率相等。  
**解答3：**这种情况属于分层概率，每个元素有两种权重：跨类别权重和同类别的权重。可以结合前两种情况解决。**时间复杂度依然是O(n)。空间复杂度是O(m)，m是类别数。**  
  
> 依然从前到后遍历列表：  
> 1. 当每类元素第一次出现时，使用解决2的方案，以类别权重和选中元素对比，等于是以权重对类别抽样  
>
> 2. 当前元素不是第一次出现时，分两种情况：  
>>   2.1. 当前元素和选中元素类型一致，这时采用解决1的方案，在组内采样。需要记录每一组的已遍历的数量。  
>>   2.2. 当前元素和选中元素类型不一致，这时不做处理，选中元素不变。因为跨类型的抽样在该类型第一次出现时已经执行过了。  

```python
ret = k.first
crnt_node = k.first
summed_weight = 0
counts_by_type = {}
while crnt_node.next != None:
    crnt_type = crnt_node.type
    counts_by_type[crnt_type] = counts_by_type[crnt_type] + 1 if crnt_type in counts_by_type else 1
    if counts_by_type[crnt_type] == 1:
        # 类型第一次出现，以权重比较
        old_weight = summed_weight
        summed_weight = summed_weight + crnt_node.weight
        if rand() >= old_weight / summed_weight:
            ret = crnt_node
    elif crnt_node.type == ret.type:
        # 和当前结果id一致，机会均等
        if rand() < 1 / counts_by_type[crnt_type]:
            ret = crnt_node
    else:
        # 不做处理
        pass
    crnt_node = crnt_node.next
```
  
----
**问题3.1：**有一个长度未知但有限的链表K={k1, k2, k3...}，对任意 n 有f(kn)=Tn，Tn∈{v1,v2,...vt}。现在需要从 K 中选取一个元素 k，使得 f(k) 的概率权重为W={w1, w2, ..., wt}。f(k)相等的物体按照第二层概率采样。  
**解答3.1：**这种情况同样属于分层概率，区别在于第二层依然是按权重分配概率。可以参考解答3解决。