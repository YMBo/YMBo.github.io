---
title: linux命令nohup和&区别
date: 2019-01-15 14:27:22
tags:  [linux]
---

### 一、前言
因为我想把一个程序一直运行在centos系统上，后来的知 `nohup` ，`&` 命令都可以，在网上查找资料后，对这些资料进行整理，让这两条命令的异同更直观的展现出来

### 二、nohup命令
nohup 是 no hang up 的缩写,不挂断的意思

语法：nohup Command [ Arg ... ] [　& ]
作用：不挂断的运行指定程序
下面我会按照 ** nohup command > myout.file 2>&1 ** 这条命令进行拆分讲解

#### 1、nohup command
> 不挂起的执行 `command` 程序,当 `ctrl+c` 的时候会退出 `command` 程序 （因为对SIGINT信号不免疫）

#### 2、 > myout.file 2>&1 
1. 标准输出和标准错误都打印到当前目录下的myout.file文件里
2. 如果直接执行 ** nohup command ** 没有后面的一串，那么将在当前目录自动创建 `nohup.out`文件，并shell中提示 `appending output to nohup.out`，输出都将附加到这个文件里。
3. 如果没有当前目录创建文件的权限，那么输出重定向到跟目录下的 /nohup.out 文件中，
4. 如果没有文件能创建或打开以用于追加，那么那么 Command 参数指定的命令不可调用（执行失败）

##### 2.1、问题1：  2>&1 什么意思？

操作系统中有三个常用的流：
　　0：标准输入流 stdin
　　1：标准输出流 stdout
　　2：标准错误流 stderr
一般当我们用 > console.txt，实际是 1>console.txt的省略用法；< console.txt ，实际是 0 < console.txt的省略用法。
`补充` :**> console.txt** 每次执行会覆盖文件内容，使用 **>> console.txt** 进行追加，则console.txt不会被覆盖

有时候希望将错误的信息重新定向到输出，就是将2的结果重定向至1中就有了"2>1"这样的思路，如果按照上面的写法，系统会默认将错误的信息（STDERR）2重定向到一个名字为1的文件中，而非所想的（STDOUT）中。因此需要加&进行区分。就有了 2>&1 这样的用法


> 这句话意思是把标准错误（2）重定向到标准输出中（1），而标准输出又导入文件myout.file里面，所以结果是标准错误和标准输出都导入文件myout.file里面了

##### 2.2、问题2：  为何2>&1要写在>myout.file后面？
` command > file 2>&1  `
       首先是command > file将标准输出重定向到file中， 2>&1 是标准错误拷贝了标准输出的行为，也就是同样被重定向到file中，最终结果就是标准输出和错误都被重定向到file中。 
` command 2>&1 >file `
      2>&1 标准错误拷贝了标准输出的行为，但此时标准输出还是在终端。>file 后输出才被重定向到file，但标准错误仍然保持在终端。

#### 3、例子：执行jenkins服务
`nohup java -jar jenkins.war --ajp13Port=-1 --httpPort=8088 > /Users/admin/nohup.out 2>&1`  启动jenkins,并将输出导入到文件nohup.out里

![执行结果](/images/linux命令nohup和&区别/1.png)

可以看到执行成功了，**新建窗口** 再看看日志文件打开 `/Users/admin/nohup.out` 这个文件：
![文件内容](/images/linux命令nohup和&区别/2.jpeg)

`结论`：**控制台没有输入jenkins的启动信息**

#### 4、关闭当前终端
![关闭启动jenkins的终端](/images/linux命令nohup和&区别/4.gif)

`结论` ：**可以看到关掉终端不会关闭jenkins进程**

#### 5、注意注意！ctrl+c
可以从`执行结果`图看到，任务开始后终端是不能输入的，那我执行 ** ctrl+c ** 让终端编程可输入状态会发生什么呢？
![执行ctrl+c后任务结束了](/images/linux命令nohup和&区别/3.png)
可以看到任务结束！！ 

`结论`：**`ctrl+c`使jenkins进程结束了**


#### 6、 nohup总结
** nohup Command [ Arg ... ] [　& ]: 
程序运行不挂起，默认会将输出重定向nohup.out文件中，也可以自定义输出文件，` ctrl+c `的话会退出a.sh进程（因为对SIGINT信号不免疫），` 关闭Command `, Command进程还是存在的（对SIGHUP信号免疫） **

### 三、&命令
#### 1、command &
让command程序在后台运行

#### 2、还以jenkins为例子测试
![启动jenkins](/images/linux命令nohup和&区别/5.png)

`结论`：**控制台输出了jenkins的启动信息**

#### 3、执行`ctrl+c`
![执行ctrl+c](/images/linux命令nohup和&区别/6.png)

`结论`：**ctrl+c不会使jenkins进程停止**

#### 4、关闭当前终端
![关闭当前终端](/images/linux命令nohup和&区别/7.gif)

`结论`：**关闭终端会使jenkins进程停止**

#### 5、补充
这时我突然想到了，上面介绍了 `>a.txt` 这样可以把信息输入到a.txt文件里，那可不可以 `Command >a.txt &`这样用呢？
`java -jar jenkins.war --ajp13Port=-1 --httpPort=8088 > /Users/admin/nohup.out 2>&1 &`
![猜想测试](/images/linux命令nohup和&区别/8.png)

`结论`：**command >a.txt &可以不在终端输出信息**

#### 6、&总结
** Command & : 
&的意思是在后台运行， 当你在执行 Command & 的时候， 即使你用 `ctrl+C`, 那么Command照样运行（因为对SIGINT信号免疫）。 但是要注意， 如果你直接 `关掉终端` 后， 那么，Command进程同样消失（因为对SIGHUP信号不免疫）。**


### 四、表格对比
| 命令方式 | ctrl+c关闭后进程是否关闭 | 直接关闭终端后进程是否关闭 | 是否终端输出信息 |
| ------ | ------ | ------ | ------ |
| nohup | 关闭 | 不关闭 | 不输出，无论是否加上 >nohup.out，信息都被导入到nohup.out（指定）文件 |
| & | 不关闭 | 关闭 | 默认输出，但加上>nohup.out则会不输出，信息被导入到nohup.out（指定）文件 |

### 五、实际应用
鉴于它们个自的优缺点，一般都是这样用
**nohup command > myout.file 2>&1 &**