---
layout: post
title: "git命令行总结-log"
date: 2015-10-17 22:08:49 +0800
comments: true
categories: Git log
---

```python
git log --name-only --oneline 8692a53f1..HEAD| grep -v '.{7} '  | uniq 
```

##格式化文件输出

```python
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative --name-status
```

##将某一个commit之后的提交做成patch

```python
git format-patch -M HEAD~1
```

##将patch打到我们的开发分支

```python
git am xxx.patch
```

##vim中格式化json

```python
:%!python -m json.tool       
```
