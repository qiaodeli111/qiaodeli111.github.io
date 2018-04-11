---
layout: post
title: Linux学习笔记
category: Server
date: 2015-03-09
---

	echo $SHELL
	##永久改变shell
	chsh -s /bin/bash [username]

	##临时改变shell
	/bin/bash

<!-- more -->

### Linux命令包括内部命令和系统命令

	内部命令作为shell程序自身的子程序运行。Bash shell内部命令包括echo，exit，history。

	系统命令是作为独立文件存在的程序，通过键入命令或文件名来执行。

	帮助： info、help和man

	help 可显示所有内部命令列表

	man 提供系统命令的信息。man -k xxx 可以模糊搜索命令

	一行输入;多个命令;用分号

	!! 重复上个命令； !ma 可以执行上个以ma开头的命令

	用一条命令作为另一命令的参数  echo `date`



### 后台与前台切换命令

直接在命令后面加 & 开启后台任务

如果有前台任务，Ctrl+Z可以挂起作业，然后输入bg把作业移入后台。

输入 fg %jobnumber 把后台作业调到前台

输入 kill %jobnumber 删除后台作业


### 文件操作

	ls
	-c 按修改时间排列

	mkdir -p abc/def/eft 连续创建子目录

	rmdir -p abc/def/* 删除包括abc/def/下面的所有空子目录，如果abc/def也为空，删除这个目录。

	设备到设备复制文件： dd xxxx

	参数：
	if=filename	源文件
	of=filename	输出文件
	bs=blocksize	每次读或写多少字节


	mv 移动目录时，如果移动的是目录，目标目录已存在，则源目录会移动成为目标目录的子目录


### 信息显示命令

	wc -c/-w/-l filename(s)

	file xxxx 查看文件类型

	touch 建新文件或修改现有文件的上次访问或修改时间。
	-a 修改访问时间
	-m 修改改变时间
	-t 使用你指定的时间
	-d 更新修改


