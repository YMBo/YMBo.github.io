---
title: git版本回退
date: 2019-05-29 11:36:19
tags: git
---
### 一、版本回退
> file可以用“.”代替，表示所有文件
#### 1.丢弃工作区修改
`git checkout -- file`
#### 2.丢弃暂存区
`git reset HEAD <file>`后，会将指定文件回退到工作区
#### 3.回退版本库
##### 3.1、回退且暂存区、工作区保存修改
`git reset --soft commit`
--soft参数告诉Git重置HEAD到另外一个commit，但也到此为止。如果你指定--soft参数，Git将停止在那里而什么也不会根本变化。这意味着`暂存区`,`工作区`都不会做任何变化，所有的在original HEAD和你重置到的那个commit之间的所有变更集都放在`暂存区`区域中。
**工作区=暂存区!=版本库**
> tip:仅把改动内容回退到暂存区

![soft](/images/git版本回退/1.png)
##### 3.2、回退且丢弃修改
`git reset --hard commit`
回退到commit版本且，从`HEAD`到另外一个`commit`之间的所有修改都会被丢弃，现在暂存区，工作区，版本库的内容都是commit这个点的内容
**工作区=暂存区=版本库**
![hard](/images/git版本回退/2.png)

#### 3.3、回退且保存修改到工作区
`git reset --mixed commit`参数默认，可以不写
原理同上，只不过修改会保存在工作区，暂存区会跟版本库一致
**工作区!=暂存区=版本库**
![mixed](/images/git版本回退/3.png)