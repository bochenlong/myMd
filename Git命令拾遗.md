> 查看Git版本   

```
~ git --version
```
> 初始化仓库   

```
~ git init
```
> 远程仓库   

```
~ git remote -v
~ git remote add origin git@github.com:bochenlong/study-demo.git
~ git remote remove origin // 取消关联
```
> 提交到暂存区   

```
~ git add 1.txt ...
~ git add ./ 
```
> 提交到版本库

```
~ git commit -m "add the README.md"
~ git commit ./1.txt -m "modify 1.txt"
~ git commit ./ -m "modify 1.txt"
```
> 查看仓库当前状态

```
~ git status
```
> 对比文件

```
~ git diff 1.txt
```
<!-- more -->
> 查看Git日志   
```
~ git log --pretty=oneline
~ git log 1.txt ./
~ git log --graph --pretty=oneline --abbrev-commit //查看分支合并图
```
> 还原文件   

```
~ git reset --hard HEAD^
// HEAD^上一个版本 HEAD^^上上一个版本 HEAD~100上100个版本 
~ git reset --hart commitid
~ git reset HEAD 1.txt   // 还原版本信息，配合git checkout -- 1.txt可还原已删除文件
```
> 查看Git commit日志

```
~ git relog
```
> 检出   

```
~ git checkout -- filename
// 一种是file自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一样的状态；
// 一种是file已添加到暂存区后，做了修改，现在，撤销修改就回到添加到暂存区后的状态；
~ git checkout -b dev // 创建且检出切换到dev分支
~ git checkout dev // 检出切换到dev分支
```
> 删除暂存区

```
~ git rm file.txt ...
~ git rm ./*
```
> 仓库推送到远程端   

```
~ git push orign master
首次关联远程仓库： git push -u --force orign master
~ git push origin :dev // 删除远程分支

~ git push origin tagid // 推送标签到远程
~ git push origin --tags // 推送全部未推送到远程的本地标签
~ git push origin :refs/tags/v0.9 // 删除远程标签
```
> 克隆

```
~ git clone git@github.com:bochenlong/study-demo.git
```
> 分支操作

```
~ git branch // 查看当前分支
~ git branch dev // 新建分支
~ git branch -d dev  // 删除分支
~ git branch -D dev // 强制删除分支
~ git merge dev // dev合并到当前分支
~ git branch --set-upstream branch-name origin/branch-name 
// 本地分支与远程分支关联
```
> 储藏

```
~ git stash //储藏当前工作区
~ git stash list //查看储藏列表
~ git stash apply stashid //恢复工作区
~ git stash drop stashid //删除储存内容
~ git stash pop //恢复工作区同事删除存储内容
```
> 更新本地库

```
~ git pull
```
> 标签

```
~ git tag v1.0 // 打标签
~ git tag v0.9 commitid // 指定节点打标签
~ git tag -a v0.9 -m 'version 0.9' commitid // 指定标签、描述
~ git tag // 查看标签
~ git show tagid // 查看标签信息
~ git tag -d v0.9 // 删除标签
```







