---
layout: post
title: "Git命令行总结-rebase"
date: 2015-10-17 22:11:24 +0800
comments: true
categories: Git rebase
---


[原文](http://git-scm.com/book/zh/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E8%A1%8D%E5%90%88)

使用git rebase的有一条不能去违背的rule，那就是已经提交的commit，不要做rebase操作。

rebase，就是衍合，就是将一个已经committed的commit重新在另一处做提交。它的意义在于将提交记录显得清晰，有条理，切整洁。这是他和merge最大的区别。

##rebase的命令格式如下:

```python
git rebase [-i | --interactive] [options][--exec <cmd>] [--onto <newbase>][<upstream>] [<branch>]
```

如果我们的开发分支dev_branch从master开出来，那么rebase的命令如下：`git rebase master dev_branch`
它的工作原理：从dev_branch和master的共同祖先开始，后续所有在dev_branch上面的提交都将以master当前最新的提交为新的基准点再次挨个提交。这是重新生成新的提交而非做合并。你会发现在dev_branch上的同样的commit在master上的SHA1校验码是不一样的。

此外rebase还可以做很多别的有意思的操作，假设如下的一种情况，我们有一个dev1分支，dev1-1分支，和master分支。dev1来源于master的A点，它上面有B，C，D三处提交，dev1-1则是来源于dev1的B点，它上面有E，F两处提交。当我们要将dev1－1 提交到master，而不提交dev1的时候，应该如何使用rebase呢？
我们用到的命令就是｀git rebase --onto master dev1 dev1-1`。这个命令就是从dev1-1和dev1的共同祖先开始之后的所有提交依次重新提交到master分支上。因此我们就跳过了dev1，直接将dev1-1合并回到master。
        

 
    
    
    
    





