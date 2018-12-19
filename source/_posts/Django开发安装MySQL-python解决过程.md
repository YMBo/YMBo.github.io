---
title: Django开发安装MySQL-python解决过程
date: 2018-12-19 17:52:13
tags: [Django,Python,Mysql]
---

### 一、前言
这篇文章写的是我安装 `MySQL-python`
遇到的问题，因为使用django开发所以我要和MySQL数据库连接，然后就死活安装不上，各种报错，折腾了一天多，终于解决了，趁着现在还有点印象，赶紧写下来，做个记录
#### 1.问题描述
django新建一个mysite项目，将数据库设置为MySQL，然后执行 **pip install MySQL-python** 安装数据库模块开始遇到的问题


#### 2. 环境
>macOS 10.13.6
>django 1.11.17
>python 2.7.10


### 二、解决问题历程
1. 执行 **pip install MySQL-python** 报错
    ![安装MySQL-python](/images/Django开发安装MySQL-python解决过程/1.png)
2. 百度后说需要安装 **mysql-connector-c**
    ```
    brew install mysql-connector-c
    ```
    如果有这种报错
    ![安装mysql-connector-c](/images/Django开发安装MySQL-python解决过程/2.png)
    那就按提示的输入命令解决（应该是**brew unlink mysql**），然后再次安装mysql-connector-c安装完后记得
    安装成功后执行一次**brew link mysql**
3. 安装 **Command Line Tools** 这个去苹果官网下就可以了 100多兆，网上说要下`xCode`，但是我没下`xCode`也成功了安装上这个了，所以不用下`xCode`就可以，终端输入 **which gcc** 查看 
    ![安装Command Line Tools成功](/images/Django开发安装MySQL-python解决过程/3.png)

4. 这时安装MySQL-python 肯定依然报错，反正我是这样 ，然后执行
>export CC='/usr/bin/gcc'
>export CFLAGS='-isysroot/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk -I/opt/X11/include -arch i386 -arch x86_64'
>export LDFLAGS='-arch i386 -arch x86_64'
>export ARCHFLAGS='-arch i386 -arch x86_64'

    我也不知道这啥意思，惭愧惭愧。。。
5. 安装 MySQL-python，还会报这个错误
    ![报错](/images/Django开发安装MySQL-python解决过程/4.png)
    
    >这里要说明一下 之前我的**mysql**是通过下载dmg包那种方式安装的，但是现在没办法我卸载**mysql**后又用brew方式安装了一下，版本为 `8.0.12 Homebrew`

    修改mysql配置文件：**mysql_config**
    ```
    112 # Create options
    113 libs="-L$pkglibdir"
    114 #libs="$libs -l "           ##注释掉源代码
    115 libs="$libs -lmysqlclient -lssl -lcrypto "  ##修改成这样
    ```
6. 安装 MySQL-python 如果你的还报错，那我只能说再百度吧 ，离安装成功不远了，
    因为我安装的时候也是疯狂报错，现在我回想的可    能不太全，因为我解决了好久

7. 好了 现在安装好了MySQL-python ,然后执行 **pip install MySQL-python**，
    妈的依然提示`Did you install mysqlclient or MySQL-python?` 好，在django根目录__init__.py文件里写上
    ``` python
    import pymysql
    pymysql.install_as_MySQLdb()
    ```
    执行**pip install MySQL-python**
    报错 `ImportError: No module named pymysql`
    然后安装 **pip install pymysql**
    再次执行 **python manage.py migrate**
    终于成功了，我差点哭出来
    去数据库看，对应的表已经创建了
    ![成功啦](/images/Django开发安装MySQL-python解决过程/5.jpeg)