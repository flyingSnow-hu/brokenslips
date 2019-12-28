---
layout: article
title: 《游戏编程模式》笔记
tags: others design-pattern
date: 2017-04-20
category: internal-strength
---
# 《游戏编程模式》笔记
## 基本设计模式
 * 命令模式：do/undo
 * 享元模式：重复的实例可以共享
 * 观察者模式：C#的event
 * 原型模式：
 * 单例模式: 避免使用单例的方法
  - 作为参数传递
  - 放在基类中
  - 合并到其他全局变量里
  - 服务定位器模式
 * 状态模式：状态机、层次状态机、下推状态机

## 序列型模式
 * 双缓冲：解决「一边访问一边修改」的问题
 * 游戏循环：
   * [Fix Your Timestep](http://gafferongames.com/game-physics/fix-your-timestep)
   * [Game loops](http://www.koonsolo.com/news/dewitters-gameloop)
   * [Unity 的游戏循环](http://www.richardfine.co.uk/2012/10/unity3d-monobehaviour-lifecycle)
 * 更新方法

## 行为型模式
 * 字节码：写游戏之前先设计个语言吧
 * 子类沙盒：使用基类提供的操作集合来定义子类中的行为
 * 类型对象：把继承改为组合，父类改为组件

## 解耦型模式
 * 组件模式：ECS
 * 事件队列
 * 服务定位器：多个单例封装到一起，可以兼用装饰器模式灵活修改

## 优化型模式
 * 数据局部性
 * 脏标记
 * 对象池
 * 空间分区
