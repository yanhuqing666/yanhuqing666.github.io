---
layout:     post
title:      Windows下使用VS2015调试MySQL
category: blog
description: Windows下使用VS2015调试MySQL。
---

最近为了实现mysql的半同步协议，有一个bug很难重现，特意研究了一下怎么debug mysql，此篇为windows篇。
***
<br />



## 0. 软件环境(mysql,vs,cmake都是当前最新版本)
MySQL:[mysql-5.7.11.zip Sorce Distribution](http://dev.mysql.com/downloads/mysql/)   
IDE:[Visual Studio Communty 2015(含update1)](https://www.visualstudio.com/zh-cn/products/visual-studio-community-vs.aspx)(根据官网的说法mysql5.7.11 源码调试至少需要VS2013以上版本)  
Cmake:cmake-3.5.0-rc3-win32-x86.msi 现官网已经提供稳定版[cmake-3.5.0-win32-x86.msi](https://cmake.org/files/v3.5/cmake-3.5.0-win32-x86.msi)  
Bison:[bison-2.4.1-setup](http://gnuwin32.sourceforge.net/packages/bison.htm)   
Editor:随意,能改文件编码格式就行  
Virtual ISO:UrtlaISO 9.6(Option) 或者其他虚拟光驱软件



## 1. 软件安装

### 1.1 安装Visual Studio Communty 2015

选择Visual C++环境。

这里遇到了2个坑
#### 1.1.1.Team Foundation Server 2015 Update 1 装不上，报错，安装退出。
原因：requirements： Windows 7 (minimum SP1) (Home Premium, Professional, Enterprise, Ultimate)   
解决办法，把自己的电脑从win7 home basic 升级一下，为防止有问题，我升到了旗舰版。
#### 1.1.2.BuildTools_MSBuild(86) 遇到严重错误，报错，安装退出。
原因：软件需求是 framework需求4.5.1以上  
解决办法： 解决办法： 手动单独安装了[.NET Framework 4.6.1](https://www.microsoft.com/zh-CN/download/details.aspx?id=49978),手动安装了[Microsoft Build Tools 2015](https://www.microsoft.com/zh-cn/download/details.aspx?id=48159)，之后再重装VS2015，PASS.  
可能安装了Framework 4.6.1，直接重装VS2015应该也能过，没试过，如果遇到了，可以试一下。

### 1.2 安装CMake, Bison
将各自bin目录加入到Path环境变量，文件目录不要有空格。


## 2.源码下载
 源码位置在 http://dev.mysql.com/downloads/mysql/
选择  Windows (Architecture Independent), ZIP Archive (mysql-5.7.11.zip)  
 configure的时候会提示你需要指定参数BOOST_INCLUDE_DIR和DOWNLOAD_BOOST，这样configure会自动下载BOOST。
 
 下载完成解压以后需要修改文件sql\sql_locale.cc的格式，改为Unicode就好了。
 否则编译时候会报错，改过之后也会报warning，但是warning就不影响我们调试了。
 
## 3.configure 
 打开Cmake(CMake-gui)程序，在where is the source code:文本框填入源码目录（G:\mysql\mysql-5.7.11），
 在where to build the binaries:填入make 目录（G:\mysql\mysql-server_make），这两个目录我是分开的，便于管理。
 
 勾选Advanced。
 点击Configure，在弹出的对话框中选择适当的平台（本处选择Visual Studio 14）.  然后点击Finish。
 点击Generate。一段时间后提示完成。  
可在MAKE目录下发现mysql.sln，双击可以用VS2015 打开了。

## 4.BUILD
VS龟速载入后，在ALL_BUILD 右键生成。。。然后就默默等吧。。  
如果碰到错误也不会停，最后生成完看下log对应一下。  
我这里生成没碰到什么异常

## 5.mysql服务启动
自己建立一个工作目录,eg: G:\mysql\mysql-server_debug.  
次目录下建立data,share以及lib,lib下建立plugin  
把make目录生成的一些文件copy过来，  
如share下的english 和plugin下的对应dll以及lib(没有lib行不？) ，我这里要debug半同步，所以把半同步的dll和lib拷贝过来了。  

写mysql.ini 的配置，略，保存到G:\mysql\mysql-server_debug\run\my.ini。记得配置里的目录都要建立好  

第一次启动mysql
打开命令行提示符
到了make目录下G:\mysql\mysql-server_make\sql\Debug>   
```mysqld.exe --initialize-insecure  --basedir=G:\\mysql\\mysql-server_debug   --datadir=G:\\mysql\\mysql-server_debug\\data``` 生成初始文件
这样会在data下生成mysql的系统db (mysql,performance_schema,sys).
如果有报错缺失文件，可以自己从make目录里copy过来。

以后启动mysql可以用```mysqld.exe   --defaults-extra-file=G:\mysql\mysql-server_debug\run\my.ini``` 

mysql客户端可以在另一个命令提示符里打开client目录（G:\mysql\mysql-server_make\client\Debug）下执行 ```mysql -uroot```  

停止服务可以在另一个命令提示符里打开client目录（G:\mysql\mysql-server_make\client\Debug）下执行```mysqladmin -uroot  shutdown```

## 6.本地vs调试mysql
### 方式一附加到进程
打开VS2015调试-附加到进程，在弹出对话框中选择mysqld.exe进程，点击附加。  
就可以在VS中添加断点调试mysql了。
 
### 方式二 
将mysqld设置为默认启动项目，点击调试-mysql属性，在弹出对话框中配置属性中点击选择调试，在命令参数里面添加```--defaults-extra-file=G:\mysql\mysql-server_debug\run\my.ini```。  
启动本地windows调试器，这样就可以增加断点调试了
  
 


### 参考资料
[Windows下使用VS2010调试MySQL](http://www.omgdba.com/debugging-mysql-on-windows-using-vs2010.html)  
[Source Installation System Requirements](http://dev.mysql.com/doc/refman/5.7/en/source-installation.html)

 <br /> 
***     
## 姊妹篇
 [linux下用gdb调试mysql](/debug-mysql-with-gdb) 
 


 