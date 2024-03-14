---
layout: article
title: DualKeyDictionary
tags: C# Specification
date: 2021-12-29
category: c#
---
# C# 代码规范

大小写
- 名字空间、类名、属性字段、方法、枚举用Pascal（大驼峰）式
- 文件夹名字用Pascal式。尽量做到每个文件代表一个类/枚举/接口，或者至少是围绕一个核心组织的一组类/枚举/接口，文件名字和核心一致。
- 字段名用小驼峰式，私有成员不需要加 _ 或者 m_（不要匈牙利命名法）
- 常量用全大写加下划线如 const int BOOK_MAX_COUNT = 5，包括 const 和 readonly
- 少于等于两个字母的缩写词全大写，如 System.IO。多于两个字母的缩写词使用驼峰式，如HtmlReader。


空行和空格
- 类（接口、枚举）与类（接口、枚举）之间空两行
- 类内的方法、属性前后空一行
- 函数内的逻辑段之间（逻辑段过长的话更建议拆分函数）
- 使用空格而非 tab，现代IDE会自动把 tab 转成空格
- 行注释前空一行，之后紧贴要注释的内容
- 行尾注释前空两格，双斜线后空一格
- 如非文档级注释，不要使用 /**/
 

命名
- 过于长的单词可以缩写，以开发组范围内了解含义，不至于引起误解为准。比如Gui、Xml
- 接口前边加I，抽象类结尾加Base
- 不要用百度翻译得到的单词，开发组内部应该有公用词典
- 不要在名称中包括其上级名称，比如 Book.BookTitle、World.WorldCitys、School.SchoolClass.ClassStudents.StudentIds
- 互补的方法/变量应该给予对应的名称和格式，比如 BeginSample/EndSample、startTime/endTime
- 单字母变量基本仅限于循环变量(ijk)和坐标(xyzw)、颜色(rgba)、算法和数学公式(tkmnabc)，尽量不要用 l 和 o，以免和数字混淆
- 方法含义比较复杂时候，命名宁长勿短
- 如果使用了设计模式，命名尽量带上模式，比如 TimeObserver、OrderFactory
- public 方法尽量不要暴露内部实现(因为会变)，比如 GetStudentsDictionary

程序猿语法
- 布尔值：is/has/had/can + 形容词/动词分词，见 4.1
- 枚举：单数名词。可以进行位运算的枚举以 Flag 结尾。Flag 枚举通常要提供一个 None=0
- 数字值：有物理量的值可以使用对应的名词如 speed，或者带单位如dotPerInch/Dpi。单位不明确时就以 Count/Num 结尾
- 数组：名词复数形式如 users。如果是某种特定数据结构，需要单数名词加类型如 userList，userSet, userLinkedList, userMaxHeap。
- 函数：以动词为核心，类似于祈使句。
```C#
void DoSomething();
```
- 返回值是某种数据的，核心动词建议用 get；修改函数域外变量（副作用）的，核心词建议用set/modify。通常不建议函数既有返回值又有副作用。
- 返回值是布尔值的，参考布尔值采用 is/has/had/can +分词 形式。
- 动词可能带宾语比如 requestUserdata()
- 还可能带有时间/地点状语表示触发时机比如 playSoundOnUserClick()
- 如果核心动词就是 do，可以考虑省略，比如 doOnClick() → OnClick()
- 一组函数有类似的功能但是参数不一样，可以用 with/by/from 等介词短语表示区别
```C#
void getUserById(uint userId)
void getUserByName(string name)
void getUsersFromCache()
void getUsersFromDatabase()
```

需注意属性虽然是一种函数，但是其命名等同于变量。
- 需注意有些引起歧义的情况，如：
  - Open，究竟是形容词还是动词，所以如果是形容词，最好用 isOpen
  - read，究竟是一边现在时还是过去时，如果是过去时，最好使用 hasRead，或者不按规范语法写成 readedBooks。同理为免歧义，data 的复数可以写成 datas。
- 编程语言里的关键字、编程术语、项目术语，需要避免歧义，尽量和其通用意义保持一致。在同一模块中，一个词尽量保持一个意义不变，以不引起歧义为准。比如 tree 在 model 中可能是一种数据结构，但是不引起歧义的情况下（比如项目里就根本很少用到二叉树），也可能确实代表地图上的一棵树。两个意义都可以有但是不要交叉使用。如果有交集，应该采用一些重构手段把两个意义隔离。
- 含义相似功能相同的格式词，最好在项目里统一约定只采用一个，比如 refresh/update/reload，load/fetch/request， destroy/delete
- init/refresh
- data/item

其他
- 大括号换不换行都行，但是最好和项目组其他人保持一致
- 不要使用魔数

# 附记
## 命名布尔值
注：Prefix-前缀，Suffix-后缀，Alone-单独使用
|位置|单词|意义|例|  
----
|Prefix|is|对象是否符合期待的状态|isValid|
|Prefix|can|对象能否执行所期待的动作|canRemove|
|Prefix|should|调用方执行某个命令或方法是好还是不好,应不应该，或者说推荐还是不推荐|shouldMigrate|
|Prefix|has|对象是否持有所期待的数据和属性|hasObservers|
|Prefix|needs|调用方是否需要执行某个命令或方法|needsMigrate|
## 用来检查的方法
|单词|意义|例|
----
|ensure|检查是否为期待的状态，不是则抛出异常或返回error code|ensureCapacity|  
|validate|检查是否为正确的状态，不是则抛出异常或返回|error code|validateInputs|

## 按需求才执行的方法
|位置|单词|意义|例|
----
|Suffix|IfNeeded|需要的时候执行，不需要的时候什么都不做|drawIfNeeded|
|Prefix|might|同上|mightCreate|
|Prefix|try|尝试执行，失败时抛出异常或是返回errorcode|tryCreate|
|Suffix|OrDefault|尝试执行，失败时返回默认值|getOrDefault|
|Suffix|OrElse|尝试执行、失败时返回实际参数中指定的值|getOrElse|
|Prefix|force|强制尝试执行。error抛出异常或是返回值|forceCreate, forceStop|

## 异步相关方法
位置单词意义例Prefixblocking线程阻塞方法blockingGetUserSuffixInBackground执行在后台的线程doInBackgroundSuffixAsync异步方法sendAsyncSuffixSync对应已有异步方法的同步方法sendSyncPrefix or AlonescheduleJob和Task放入队列schedule, scheduleJobPrefix or Alonepost同上postJobPrefix or Aloneexecute执行异步方法（注：我一般拿这个做同步方法名）execute, executeTaskPrefix or Alonestart同上start, startJobPrefix or Alonecancel停止异步方法cancel, cancelJobPrefix or Alonestop同上stop, stopJob
4.5 回调方法/触发时机
位置单词意义例Prefixon事件发生时执行onCompletedPrefixbefore事件发生前执行beforeUpdatePrefixpre同上preUpdatePrefixwill同上willUpdatePrefixafter事件发生后执行afterUpdatePrefixpost同上postUpdatePrefixdid同上didUpdatePrefixshould确认事件是否可以发生时执行shouldUpdate
4.6 操作对象生命周期的方法
单词意义例initialize初始化。也可作为延迟初始化使用initializepause暂停onPause ，pausestop停止onStop，stopabandon销毁的替代abandondestroy同上destroydispose同上dispose
4.7 与集合操作相关的方法
单词意义例contains是否持有与指定对象相同的对象containsadd添加addJobappend添加appendJobinsert插入到下标ninsertJobput添加与key对应的元素putJobremove移除元素removeJobenqueue添加到队列的最末位enqueueJobdequeue从队列中头部取出并移除dequeueJobpush添加到栈头pushJobpop从栈头取出并移除popJobpeek从栈头取出但不移除peekJobfind寻找符合条件的某物findById
4.8 与数据相关的方法
单词意义例create新创建（动词）createAccountnew新创建的(形容词+名词)newAccountfrom从既有的某物新建，或是从其他的数据新建fromConfigto转换toStringupdate更新既有某物updateAccountload读取loadAccountfetch远程读取fetchAccountdelete删除deleteAccountremove删除removeAccountsave保存saveAccountstore保存storeAccountcommit保存commitChangeapply保存或应用applyChangeclear清除数据或是恢复到初始状态clearAllreset清除数据或是恢复到初始状态resetAll
4.9 成对出现的动词
get获取 set设置  
add 增加 remove 删除  
create 创建 destory 销毁  
start 启动 stop 停止  
open 打开 close 关闭  
read 读取 write 写入  
load 载入 save 保存  
create 创建 destroy 销毁  
begin 开始 end 结束  
backup 备份 restore 恢复  
import 导入 export 导出  
split 分割 merge 合并  
inject 注入 extract 提取  
attach 附着 detach 脱离  
bind 绑定 separate 分离  
view 查看 browse 浏览  
edit 编辑 modify 修改  
select 选取 mark 标记  
copy 复制 paste 粘贴  
undo 撤销 redo 重做  
insert 插入 delete 移除  
add 加入 append 添加  
clean 清理 clear 清除  
index 索引 sort 排序  
find 查找 search 搜索  
increase 增加 decrease 减少  
play 播放 pause 暂停  
launch 启动 run 运行  
compile 编译 execute 执行  
debug 调试 trace 跟踪
observe 观察 listen 监听  
build 构建 publish 发布  
input 输入 output 输出  
encode 编码 decode 解码  
encrypt 加密 decrypt 解密  
compress 压缩 decompress 解压缩  
pack 打包 unpack 解包  
parse 解析 emit 生成  
connect 连接 disconnect 断开  
send 发送 receive 接收  
download 下载 upload 上传  
refresh 刷新 synchronize 同步  
update 更新 revert 复原  
lock 锁定 unlock 解锁  
check out 签出 check in 签入  
submit 提交 commit 交付  
push 推 pull 拉  
expand 展开 collapse 折叠  
begin 起始 end 结束  
start 开始 finish 完成  
enter 进入 exit 退出  
abort 放弃 quit 离开  
obsolete 废弃 depreciate 废旧  
collect 收集 aggregate 聚集