---    
layout:     post    
title:      Linux下使用gdb调试MySQL    
category: blog    
description: Linux下使用gdb调试MySQL    
---    
      
 最近为了实现mysql的半同步协议，有一个bug很难重现，特意研究了一下怎么debug mysql，此篇为linux篇。    
***    
<br />     
      
##   0. 环境     
###   0.1 操作系统    
linux:SUSE Linux Enterprise Server 11 SP3  (x86_64) - Kernel     
(通过 ```cat /proc/version``` 或者```cat /etc/issue```查看)    
这个版本有点老，包括glibc和gcc以及默认包都比较旧    
此操作系统可以使用默认zypper install xx 来安装软件，但实际使用发现包都比较旧，不推荐。推荐源码安装。    
    
    
###   0.2 软件环境/依赖    
MySQL:[Generic Linux , Compressed TAR Archive Includes Boost Headers](http://dev.mysql.com/downloads/mysql/)    
Cmake :cmake3.5.0-rc3.tar.gz,现官网提供稳定版[cmake-3.5.0.tar.gz](https://cmake.org/files/v3.5/cmake-3.5.0.tar.gz) (需要源码安装)    
Make :make3.81 (mysql 官网需求make 3.75 or newer）    
GCC: gcc5.3.0 (本机版本gcc4.3.4，mysql官网需求GCC 4.4.6 or later，所以需要升级 ）    
Boost: mysql官网需求Boost 1.59.0 or newer（下载源码的时候可以选择带boost的就无需安装了）    
Bision: mysql官网需求bison 2.1 or newer(zypper install bison)    
GDB: gdb-7.11.tar.gz(需要源码安装)    
     
##   1. 软件安装    
###   1.1 cmake 安装    
源码安装3.5.0-rc3，    
下载解压打开    
```./configure```    
```make```    
```make install```    
    
通过```zypper install cmake``` 也可以安装,装的是2.6.2版本的不知道 会不会版本过低    
    
###   1.2 make     
默认版本make 3.81，不用再安装    
    
###   1.3 bison    
```zypper install bison ```   
    
###   1.4升级gcc    
在保留原系统gcc的情况下，安装了一个高级版本的gcc    
我这里打算安装gcc5.3.0,    
    
####   1.4.1 [GCC环境依赖](https://gcc.gnu.org/install/prerequisites.html)    
GNU Multiple Precision Library (GMP) version 4.3.2 (or later)    
MPFR Library version 2.4.2 (or later)    
MPC Library version 0.8.1 (or later)    
    
#####   1.4.1.1. 安装[gmp-6.1.0](https://gmplib.org/download/gmp/gmp-6.1.0.tar.bz2)    
下载解压打开    
```./configure```    
```make```    
```make check``` 这一步用来查看有没有文件不匹配或缺失,然后安装：    
```make install```    
    
#####   1.4.1.2. 安装 [mpfr-3.1.3](http://www.mpfr.org/mpfr-current/#download)    
下载解压打开    
配置：    
```./configure --with-gmp-include=/usr/local/include --with-gmp-lib=/usr/local/lib```    
```make```    
```make check```     
```make install```    
    
#####   1.4.1.3.安装 [mpc-1.0.3](http://www.multiprecision.org/index.php?prog=mpc&page=download)    
下载解压打开    
```./configure```    
```make```    
```make install```    
    
####   1.4.2 安装gcc    
    
#####   1.4.2.1 下载gcc-5.3.0.tar.gz     
在[gcc官网](https://gcc.gnu.org/) 上下载gcc5.3.0    
    
#####   1.4.2.2 解压缩    
    
```tar xzvf gcc-5.3.0.tar.gz```    
新生成的gcc-5.3.0这个目录为源目录。    
    
    
#####   1.4.2.3 建立目标目录    
     
目标目录是用来存放编译结果的地方。我 建立了一个叫 gcc-build ，作为目标目录（与源目录gcc-5.3.0是同级目录）：    
    
```mkdir gcc-build```    
```cd gcc-build```    
    
以下的操作主要是在gcc-build下进行。    
    
#####   1.4.2.4 配置    
    
```../gcc-4.4.2/configure --prefix=/usr/local/gcc-5.3.0 --disable-multilib```       
/usr/local/gcc-5.3.0是自定的安装目录    
--disable-multilib是指定不用编译32位    
    
将GCC安装在/usr/local/gcc-5.3.0目录下。    
    
#####   1.4.2.5  编译安装    
    
```make```    
```make install```    
    
至此，GCC 安装过程就完成了。    
    
#####   1.4.2.6 环境设置（将gcc的头文件和库文件指向新的版本）    
```vi /etc/profile```    
向其中添加以下语句。    
GCCHOME=/usr/local/gcc-5.3.0              （指定gcc的搜索路径）    
PATH=$GCCHOME/bin:$PATH    
LD_LIBRARY_PATH=$GCCHOME/lib    
export GCCHOME PATH LLD_LIBRARY_PATH    
    
重新引导，查看gcc版本    
```source /etc/profile```    
     
本部分参考资料[OpenSUSE 11.1下更新gcc版本为4.4.2过程](http://blog.lehu.shu.edu.cn/maglee/A152690.html)    
     
###   1.5  源码下载     
如果选择 Includes Boost Headers 的源码版本，将来configure 的时候就不需要单独下载Boost的头文件了，configure的时候会提示你需要指定参数BOOST_INCLUDE_DIR 。如果不是Includes Boost Headers的源码版本还需要指定DOWNLOAD_BOOST，这样configure会自动下载BOOST    
解压    
    
    
###   1.6 配置    
如果你想保持安装目录干净，新建一个子目录或者在外面建一个同级目录    
    
我们这里建立了子目录    
`````````mkdir bld_debug`````````    
```cd bld_debug```    
### 1.6.1 cmake    
```cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr/local/mysql5.7.11_debug -DMYSQL_DATADIR=/usr/local/mysql5.7.11_debug/data -DMYSQL_TCP_PORT=11004 -DMYSQL_UNIX_ADDR=/tmp/mysql_11004.sock          -DSYSTEMD_PID_DIR=/tmp/mysql_11004.pid  -DWITH_BOOST=/tmp/mysql-5.7.11_source/boost/boost_1_59_0 -DWITH_DEBUG=1```    
    
这里最重要的参数是**DWITH_DEBUG**，有了这个选项这样我们一会儿才能用gdb去debug    
具体可通过 mysqld --help 查看 --debug  的说明    
    
这里遇到了两个错误    
我是这么解决的    
####   1.6.1.1报错说gcc版本不对    
添加环境变量    
```export CC=/usr/local/gcc-5.3.0/bin/gcc```    
```export CXX=/usr/local/gcc-5.3.0/bin/g++```    
    
####   1.6.1.2 /usr/lib64/libstdc++.so.6: version ```GLIBCXX_3.4.21' not found    
    
```cp /usr/local/gcc-5.3.0/lib64/libstdc++.so.6.0.21 /usr/lib64/```    
    
复制后，修改系统默认动态库的指向，即：重建默认库的软连接。    
    
切换工作目录至/usr/lib64：    
    
```cd /usr/lib64```    
    
删除原来软连接：    
    
```rm -rf libstdc++.so.6```    
    
将默认库的软连接指向最新动态库：    
    
```ln -s libstdc++.so.6.0.21 libstdc++.so.6```    
    
本问题 [参考资料](http://itbilu.com/linux/management/NymXRUieg.html)      
     
####   1.6.2 后续安装    
```make```    
```make install```    
    
到此为止,源码编译安装就结束了，下面安装步骤与正常二进制包安装mysql没有太大什么不同    
    
####   1.6.3 mysql安装    
打开我们之前指定的目录 ```cd /usr/local/mysql5.7.11_debug```    
修改/添加 my.cnf ,如果my.cnf 里有指定的目录，记得添加.    
下面是5.7.6以后比较通用的办法    
```chown -R mysql .```    
```chgrp -R mysql .```    
```bin/mysqld --initialize --basedir=/usr/local/mysql5.7.11_debug --datadir=/usr/local/mysql5.7.11_debug/data```    
这里会在mysql_err.log 里产生一个初始密码，把它记下来备用    
```bin/mysql_ssl_rsa_setup --defaults-file=/usr/local/mysql5.7.11_debug/my.cnf```    
    
修改mysql.server    
```vi support-files/mysql.server```     
编辑basedir和 datadir    
拷贝到启动目录下    
```cp support-files/mysql.server /etc/init.d/mysql57_debug.server```    
启动mysql     
```service mysql57_debug.server start```    
    
    
修改root密码    
```mysql -uroot -p -P 11004 -S /tmp/mysql_11004.sock```    
    
```SET PASSWORD = PASSWORD('yourpassword');```    
    
完成    
    
    
###   1.7  安装 gdb-7.11    
下载解压 gdb-7.11.tar.gz    
```cd gdb-7.11```    
```./configure```    
```make```    
```make install```    
    
    
     
###   1.8使用gdb调试mysql    
     
#### 1.8.1 方式一    
 首先查看mysql进程号    
 ```ps -ef|grep mysql```    
 比如知道进程号 2046    
 启动gdb    
 ```gdb```    
 附加线程    
 ```attach 2046```    
 这样就可以添加断点进行调试了。    
    
#### 1.8.1 方式二    
```gdb --args /usr/local/mysql5.7.11_debug/bin/mysqld --defaults-extra-file=/usr/local/mysql5.7.11_debug/my.cnf ```    
在sql_show里加个断点    
```break /tmp/mysql-5.7.11_source/sql/sql_show.cc:3414```    
然后```run```    
这时候，去客户端执行show database，就会停在断点处了     
    
 至于使用gdb的用法，那又是另外一个故事了，可以参考[gdb用户手册](https://sourceware.org/gdb/current/onlinedocs/gdb/) 或者简明一点的[Linux gdb调试器用法全面解析](https://segmentfault.com/a/1190000003733061)    
     
     
####   参考资料    
 mysql的源码安装部分基本参考官网，就不一一列出了    
 部分官网资料    
 [source-configuration-options](http://dev.mysql.com/doc/refman/5.7/en/source-configuration-options.html)    
 [source-installation](http://dev.mysql.com/doc/refman/5.7/en/source-installation.html)    
 <br />  
***   
 
## 姊妹篇    
 [Windows下使用VS2015调试MySQL](/debug-mysql-with-vs2015) 