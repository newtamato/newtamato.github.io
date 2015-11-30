---
layout: post
title: "git如何在本地和远端都删除相关的分支"
date: 2015-11-22 17:45:01 +0800
comments: true
categories: git
---

```
git branch -r #查看本地和远端同步的分支
origin/change_avatar_cloth
origin/develop
origin/master
```

<!--more-->

通过命令`git branch -rd  branch_name`，将存在自己本地的remote分支删除。但是并没有通知远端。因此还需要如下的命令`git push origin: branch_name` 将本地的分支同步出去。
如果此时想要让别人同步服务器的分支，也就是和服务器保持一样，如果服务器的分支被删除了，本地的分支也照样删除，可以使用如下命令：

```
git pull --prune
```

后来到stack overflow当中找到了高票答案

> As of Git v1.7.0, you can delete a remote branch using
> git push origin --delete <branchName>
> which is easier to remember than
> git push origin :<branchName>
> which was added in Git v1.5.0 "to delete a remote branch or a tag."


也就是说如果是1.7.0版本的Git，可以通过`git push origin --delete branch_name` 的方法去删除一个远端分支，而在1.5.0版本的Git重则可以直接使用`git push origin: branch_name` 的方法

或者就是先批量通过删除本地remote的仓库，然后通过如下的方法`git push --prune origin`直接和远端和本地仓库做同步。也就是说如果本地remote分支不存在，那么远端也就删除掉。因此这个命令如同放大招，最好慎用。

