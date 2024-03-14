---
layout: article
title: git/svn 锦囊
tags: git svn tips
date: 2023-06-19
category: tips
---
# git/svn 锦囊

## git

查看日志
```shell
# 显示 b1 分支上没有合并到 master 的提交
git log master..b1

# 显示从 c869c18cd 到 HEAD 之间的提交
git log c869c18cd..HEAD

# 查看某段时间的日志 + 特定格式
# https://git-scm.com/docs/git-log, format:<format-string>
git log --since='2022-02-15' --pretty=format:'%s'

# 列出某次提交中某个文件的修改
git show commit-id file-path

# blame
git blame <filename> -L <start>[,<end>]
```

分支
```shell
# 查看所有分支
git branch

# 创建分支b1
git branch b1

# 切换到已有的分支b1
git checkout b1
git switch b1

# 创建分支b1并切换到新分支
git checkout -b b1
git switch -c b1

# 切换到上一次的分支
git switch -
```

## svn

```shell
# 在 svn 命令后加 --xml 参数，会以 xml 格式输出，便于进一步解析
svn log --xml

# 查看某段时间内的日志
svn log -r {2021-5-1}:{2021-12-31}

# 查看某两个版本之间的日志
svn log -r 1024:2048

# 按提交者和时间筛选日志
svn log --search fugui.wang -r {2020-07-01}:{2020-08-01}
```