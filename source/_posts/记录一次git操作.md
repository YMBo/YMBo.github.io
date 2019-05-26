---
title: 记录一次git操作（git远程仓库地址变更）
date: 2019-05-26 11:55:36
tags: git
---
### 背景
项目远程仓库地址是`A`，将项目以名称为`app`clone在本地，而且在服务器上也以项目名称为`app`clone。由于一些原因，将远程仓库`A`迁移到远程仓库`B`,所以需要将远程服务器`app`的仓库地址**由A改为B**，但由于服务器的一些程序的限制，`app`文件名称不可以变，只需要变动里面的项目内容
1. 远程仓库，有三次提交记录
![远程仓库](/images/记录一次git操作/1.png)
2. 本地仓库
![本地仓库](/images/记录一次git操作/2.png)
![仓库地址](/images/记录一次git操作/3.png)

### 一、仓库迁移
#### 1. 将A仓库镜像到B仓库
名称为`old`仓库迁移到了名称为`new`仓库里
![新的仓库地址](/images/记录一次git操作/4.png)
#### 2.本地额外提交到old仓库两次记录
![old仓库的提交记录](/images/记录一次git操作/5.png)
![new仓库的提交记录](/images/记录一次git操作/6.png)
**可以看到old仓库领先new仓库两次提交**
![远程服务器仓库的提交记录](/images/记录一次git操作/5.png)

#### 3. 将new仓库内容与old仓库内容同步
因为old里面内容较新，所以讲old文件同步到new上，并且同步后再提交一次记录到new上
![远程服务器仓库的提交记录](/images/记录一次git操作/7.png)
#### 4. 将服务器仓库的地址变更为new仓库地址并更新
##### 4.1. 服务器地址切换
![远程服务器仓库地址切换](/images/记录一次git操作/8.png)
##### 4.2. 服务器git pull
![远程服务器git pull](/images/记录一次git操作/9.png)
可以看到更新失败需要手动处理冲突，`其实关键就在这里如果服务跑一个程序比如说web服务，这些文件是web页面，那么此时web页面就会展示错误，因为文件里有待解决的冲突，那怎么解决呢？`


### 二、解决方案
其实下面1、2方法在一定程度上是一个意思
#### 1. 方法1
服务器clone最新的new仓库到app的同级目录，然后将本来存在的app删除，并将new仓库改名为app即可（mv也可以改名）
#### 2. 方法2
服务器clone最新的new仓库到app的同级目录，将new文件下的.git 隐藏文件（工作区）移动到app文件下替换app的.git，git reset --hard HEAD^后git pull即可
#### 3 git remote -set-url origin <新地址>
1.  首先git remote -set-url origin <新地址>
2.  git fetch（拉取更新但不合并）
3.  git reset --hard origin/master （用远程服务器的origin/master替换本地、暂存区、版本库）

> tip: 当更新仓库的时候用 git pull ，但是git pull 包含了两个操作 ，git fetch 和git merge,
>git fetch 是将远程的master（默认）分支存储到本地的origin/master命名空间中，不会进行合并
>但是有时候我们想用远程仓库的内容完全替换到本地的容：
>git reset --hard origin/master 
>撤销本地、暂存区、版本库(用远程服务器的origin/master替换本地、暂存区、版本库)


### 总结
上面说了这么一大堆，其实都是我实际中遇到的坑坑，差点坑死我，不过上面说的确实是很麻烦，所以再总结一下
**问题**：切换远程分支避免遇到`merge`，或者说怎么更好的切换远程分支并更新
**解决方案**：上面三种




