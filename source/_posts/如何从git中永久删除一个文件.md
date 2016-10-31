---
title: 如何从git中永久删除一个文件
date: 2016-10-28 09:24:12
tags: git
---
一不小心提交了不该提交的文件到版本库中该怎么办呢？戳进去就知道了
<!--more-->
## 步骤一: 从你的资料库中清除文件
```
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 要删除的文件(相对git的路径)' --prune-empty --tag-name-filter cat -- --all
```
## 步骤二: 推送我们修改后的repo
```
git push origin master --force
```
## 步骤三: 清理和回收空间
```
git reflog expire --expire=now --all
git gc --prune=now
git gc --aggressive --prune=now
```
