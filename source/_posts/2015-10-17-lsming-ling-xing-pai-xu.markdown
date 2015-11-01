---
layout: post
title: "ls命令行排序"
date: 2015-10-17 22:26:44 +0800
comments: true
categories: 命令行
---

##按文件大小降序排列
>ls -lSr 

##查看当前电脑上硬盘的使用情况
> df -lh

##这是按兆（M）来排序
> du -sh * |sort -rm

##选出排在前面的10个
>du -sh * | sort -rm | head

##选出排在后面的10个
>du -sh * | sort -rm | tail
