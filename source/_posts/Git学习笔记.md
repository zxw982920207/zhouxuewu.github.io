---

title: Git学习笔记
date: 2016-05-10
tags: [git]

---

![](http://pccmxww5q.bkt.clouddn.com/git.png?imageView2/0/w/560/h/380/q/100)



### 将某个文件夹设置为git仓库：

在这个文件夹下打开命令行

```
git init
```
### 添加文件到git本地仓库
第一步

```
git add file1
git add file2
git add file3
```
一般使用

```
git add .
```
第二步

*git命令行commit不支持中文提示*

```
git commit -m "add 3 files"
```

### 时光穿梭
查看仓库状态

```
git status
```
查看文件修改了哪些内容

```
git diff file1
```

---

### 版本回退
---

### 创建与合并分支

查看分支
```
git branch
```
创建分支
```
git branch name
```
切换分支
```
git checkout name
```
创建并切换分支
```
git checkout -b name
```
合并某分支到当前分支
```
git merge name
```
删除分支
```
git checkout -d name
```

---

### 多人协作
• 查看远程库信息，使⽤
```
git remote -v
```
• 本地新建的分⽀如果不推送到远程，对其他⼈就是不可⻅的；
• 从本地推送分⽀，使⽤
```
git push origin branch-name
```
，如果推送失败，先⽤git pull抓取远程的新提交；
• 在本地创建和远程分⽀对应的分⽀,使用
```
git checkout -b branch-name origin/branchname
```
，本地和远程分⽀的名称最好⼀致；
• 建⽴本地分⽀和远程分⽀的关联，使⽤
```
git branch --set-upstream branch-name origin/branch-name
```
• 从远程抓取分⽀，使⽤git pull，如果有冲突，要先处理冲突。














