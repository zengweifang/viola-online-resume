---
layout: article
title:  "git 基本命令"
disqus: true
categories: basic
---

>简介：git 基本命令

---

A.提交分支
	1.提交到远程分支
	git push origin branchName:branchName

B.删除分支：
	1 删除远程分支
	$ git push origin :branchName
	2 删除本地分支，强制删除用-D
	$ git branch -d branchName

C.标记tag
	1.对当前分支打tag：
	$ git tag -a v1.4 -m 'my version 1.4'
	git tag tagContent
	2.然后push到远程即可：
	git push origin tagName
	3.删除本地tag
	git tag -d tagName

D.创建分支
	1 创建本地分支（建立分支后，仍停留在当前分支，切换分支：git checkout branchName）
	$ git branch branchName
	2 创建本地分支（建立分支后，立即切换到新分支）
	$ git branch -b branchName
E.查看分支
	1 查看本地分支
	git branch
	2 查看所有分支
	git branch -a
F.切换分支
	1 切换到本地分支
	git checkout  branchName
G.查看分支状态
	1 查看当前分支状态
	git status
H.查看提交记录
	1 查看当前分分支提交记录
	git log
I.查看分支修改内容
	1 查看修改内容
	git diff	
J.已缓存内容恢复到未缓存状态
	1 取消所有已缓存内容
	git reset HEAD .
	若取消某一个文件
	git reset HEAD src/index.html
K.git如何删除本地所有未提交的更改
	1. git clean -df
	这一个命令只删除所有untracked（未缓存）的文件

L.reset到merge前的版本
	$ git checkout 【行merge操作时所在的分支】
	$ git reset --hard 【merge前的版本号】

----

