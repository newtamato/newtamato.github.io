# 根据git log得到版本间文件的变化

标签（空格分隔）： git log format output

---

```
git log --name-only --oneline 8692a53f1..HEAD| grep -v '.{7} '  | uniq 
```
#格式化文件输出#
```
git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative --name-status
```
>1.将某一个commit之后的提交做成patch
```
git format-patch -M HEAD~1
```
>1.将patch打到我们的开发分支
```
git am xxx.patch
```
>2.vim中格式化json
```
:%!python -m json.tool       
```







