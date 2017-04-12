---
layout: timemachine
title: 如何把 java 转化成 smali
tags: others android
date: 2017-04-12
category: android
---

# 如何把 java 转化成 smali

最近研究 android 拆包，发现一个问题：拆包之后有新的 res 文件添加进来，特别是有新的 id ，需要根据重排的 public.xml 重新编译所有的 R.smali

前置任务：首先是资源合并要正确，没有重复id之类

然后拢共分四步：
1. 根据重排的资源重建R.java
1. .java -> .class
1. .class -> .dex
1. .dex -> .smali

----
## 0. 前置任务
 - 首先准备一个「加好资源的apk」，反编译到一个目录——姑且称为 TEMP_FOLDER——这个 apk 的资源ID是重排好的
 - 如果只有一个apk和一堆要加的资源，那么做法是这样：
   - 用 apktool 反编译 apk，把资源放进去，重新编译，如果遇到 id 冲突 ——非常有可能—— 请选好你更爱哪一个让他留下，解决完了打出「加好资源的apk」，再反编译成我们要的「TEMP_FOLDER」
 - TEMP_FOLDER 的下一级目录是 assets、res、smali、AndroidManifest.xml 等等等等，不要把目录层级搞错

 - 准备一些临时目录输出中间文件，比如 JAVA_OUTPUT_DIR、CLASS_OUTPUT_DIR

## 1. 根据重排的资源重建R.java
先上命令：
```shell
 aapt.exe package -m -J $JAVA_OUTPUT_DIR -S $TEMP_FOLDER/res -I $ANDROID_JAR -M $TEMP_FOLDER/AndroidManifest.xml
```

```shell
 aapt.exe package -m -J $JAVA_OUTPUT_DIR -S $TEMP_FOLDER/res -I $ANDROID_JAR -M $TEMP_FOLDER/AndroidManifest.xml --custom-package $PACKAGE_NAME
```

说明：
 - aapt 是一个工具，在 android sdk 的  _build-tools/版本号_  目录下面，注意有 exe 和 linux 两个版本
 - ANDROID_JAR 是android.jar的位置 在 android sdk 的  _platforms/版本号_  目录下
 - 有关 --custom-package 参数：
   - 不带这个参数时，打出来的R文件所在包是你的清单文件所指定的包名，也即你的apk包名
   - 带这个参数时，所打出来的R文件是你指定包下的R文件
   - 如果你的包里不只有一处R文件——如果你引用了第三方的库是有可能的——那么就需要把所有有R文件的地方都更新一遍，方法是搜索你的TEMP_FOLDER，看哪些文件夹里有 R.smali——每一处 R.smali 都需要更新一下


执行完这步之后，JAVA_OUTPUT_DIR 里面应该有N多包，每个包里都有重排好的 R.java

## 2. .java -> .class
核心命令：javac
```shell
 javac -d $CLASS_OUTPUT_DIR -encoding utf-8 -source 1.7 -target 1.7 $JAVA_FILE
```
说明:
 - javac:jdk里的工具，不解释
 - 核心就是 javac -d $CLASS_OUTPUT_DIR $JAVA_FILE 这部分，其余的 _-encoding utf-8 -source 1.7 -target 1.7_ 三个参数是为了避免一些错误，如果你在 java 1.8 下编译出错请加上，如果你的代码里有中文编译出错请加上

执行完这步之后，CLASS_OUTPUT_DIR 目录里应该有一堆.class文件

## 3. .class -> .dex
核心命令：
```shell
 java -jar DX_JAR --dex --output=R.dex --input-list=dex_list.txt
```

说明：
 - DX_JAR:android sdk 里的工具，位置在 android sdk 的 _build-tools/版本号/lib_ 目录下
 - output 参数：输出的dex的位置，这里写定成 R.dex，可以随便改
 - dex_list.txt:要打入dex的.class文件列表，里面每一行是一个.class文件的路径

## 4. .dex -> .smali
核心命令：
```shell
 java -jar baksmali.jar d R.dex
```
说明：
 - [baksmali.jar](https://github.com/JesusFreke/smali)是一个github上的项目，apktool 用的也是他
 - 作用是在 R.dex 的同级目录下生成一个smali目录，里面有所有smali文件，就是我们最后想要的，拷贝到 TEMP_FOLDER 对应位置即可
