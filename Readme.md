
为天嵌E9交叉编译MySQL

目录
----
* [环境描述](#0-环境描述)
* [编译前准备](#1-编译前准备)
* [编译PC版本mysql](#2-编译PC版本MySQL)
* [编译arm版本mysql](#3-编译arm版本mysql)
* [部署到目标arm板](#4-部署到目标arm板)
* [一些其他操作](#5-一些其他操作)

# 0-环境描述
----
1. 硬件平台：天嵌E9-V3 iMXQ
1. 交叉编译器版本：arm-none-linux-gnueabihf
1. MySql版本：5.7.21
1. 上位机版本：Ubuntu 17.04

# 1-编译前准备  

##  1.1:主要参考：  

[https://blog.csdn.net/fhyocean/article/details/74960005](https://blog.csdn.net/fhyocean/article/details/74960005)


##  1.2:下载mysql源码：  

[https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.21.tar.gz](https://cdn.mysql.com//Downloads/MySQL-5.7/mysql-5.7.21.tar.gz)

##  1.3:下载boost源码：  

[https://jaist.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz](https://jaist.dl.sourceforge.net/project/boost/boost/1.59.0/boost_1_59_0.tar.gz)

##  1.4:下载ncurses源码：  

[https://ftp.gnu.org/pub/gnu/ncurses/ncurses-5.9.tar.gz](https://ftp.gnu.org/pub/gnu/ncurses/ncurses-5.9.tar.gz) 

**注意：5.7.21版本mysql对应的boost版本是1.59.0** 

# 2-编译PC版本MySQL

##  2.1:解压源码至mysql-5.7.21-pc并编译  

### 2.1.1:执行带参数camke命令

在mysql-5.7.21-pc目录下执行： 

`
cmake . -LH -DCMAKE_INSTALL_PREFIX=/home/velarn/work/mysql/build/ -DMYSQL_DATADIR=/home/velarn/work/mysql/build/data -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci  -DEXTRA_CHARSETS=all -DENABLED_LOCAL_INFILE=1  -DWITH_BOOST=/home/velarn/work/boost/boost_1_59_0
`


**编译选项对应说明：**

 选项  |  说明  
-------|------ 
 -DCMAKE_INSTALL_PREFIX |  mysql安装目录  
 -DMYSQL_DATADIR   |mysql数据库的安装目录  
 -DDEFAULT_CHARSET   |使用utf8字符  
 -DDEFAULT_COLLATION|   校验字符  
 -DEXTRA_CHARSETS   |安装所有扩展字符集  
 -DENABLED_LOCAL_INFILE  |  允许从本地导入数据  
 -DWITH_BOOST  |  Boost库的头文件路径（mysql5.7.18安装需要boost库支持）  

**报错 “No CMAKE_CXX_COMPILER could be found. ”：**

>No CMAKE_CXX_COMPILER could be found.
>Tell CMake where to find the compiler by setting either the environment 
>variable “CXX” or the CMake cache entry CMAKE_CXX_COMPILER to the full path 
>to the compiler, or to the compiler name if it is in the PATH.

**解决方法：**
sudo apt-get update
sudo apt-get install -y build-essential

**参考：**
[https://blog.csdn.net/xx352890098/article/details/78819852](https://blog.csdn.net/xx352890098/article/details/78819852)


### 2.1.2:再次执行cmake带参数命令

**报错 “找不到curses”：**

`找不到curses`

**解决方法：**

sudo apt install libncurses5-dev

### 2.1.3:再次执行cmake带参数命令

##  2.2:make  

cmake没有错误的情况下，执行make -j4开始编译。
无需make install。

**编译pc版本的目的**
为了产生：
1.  extra/comp_err 
2.  scripts/comp_sql 
3.  sql/gen_lex_hash 
4.  sql/gen_lex_token 
5.  extra/protobuf/protoc 
6.  libmysql/libmysql_api_test 

这六个文件，这6个文件在编译arm版本的mysql时需要替换过去并touch。否则arm版本不能编译通过。

**至此pc版本mysql编译完成**

# 3-编译arm版本mysql  

##  3.1:交叉编译ncurses  

`
./configure --prefix=/home/velarn/work/ncurses/build --host=arm-linux-gnueabihf CC=arm-linux-gnueabihf-gcc CXX=arm-linux-gnueabihf-g++  
`


**遇到问题：**

>In file included from ./curses.priv.h:325:0,
>                 from ../ncurses/lib_gen.c:19:_24273.c:843:15: error: expected ‘)’ before ‘int’
>../include/curses.h:1631:56: note: in definition of macro 'mouse_trafo' #define mouse_trafo(y,x,to_screen) wmouse_trafo(dscr,y,x,to_screen)


**解决方法：**

添加**CPPFLAGS=-P**选项  

`
./configure --prefix=/home/velarn/work/ncurses/build --host=arm-linux-gnueabihf CC=arm-linux-gnueabihf-gcc CXX=arm-linux-gnueabihf-g++ CPPFLAGS=-P
`

**参考：**
[https://stackoverflow.com/questions/37475222/ncurses-6-0-compilation-error-error-expected-before-int/37475223](https://stackoverflow.com/questions/37475222/ncurses-6-0-compilation-error-error-expected-before-int/37475223)

##  3.2:执行CMAKE  

将mysql源码压缩包解压至mysql-5.7.21-arm文件夹并执行:

`
cmake . -LH -DCMAKE_INSTALL_PREFIX=/usr/local/arm/mysql -DMYSQL_DATADIR=/usr/local/arm/mysql/data -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci  -DEXTRA_CHARSETS=all -DENABLED_LOCAL_INFILE=1  -DCMAKE_CXX_COMPILER=arm-linux-gnueabihf-g++ -DCMAKE_C_COMPILER=arm-linux-gnueabihf-gcc -DWITH_BOOST=/home/velarn/work/boost/boost_1_59_0 -DCURSES_INCLUDE_PATH=/home/velarn/work/ncurses/build/include/ncurses -DCURSES_LIBRARY=/home/velarn/work/ncurses/build/lib/libncurses.a
`

**需要注意的是**

`
-DCMAKE_INSTALL_PREFIX=/usr/local/arm/mysql
-DMYSQL_DATADIR=/usr/local/arm/mysql/data
`

**这两项会被填充到最终生成的mysql.server文件中，因此在arm环境中，务必使mysql所在文件夹与此目录相同。否则在arm端运行mysql时会出现问题。**

##  3.3:执行make  

cmake无错误，执行make

### 3.3.1:make时可能出现的错误

 **第一类：** ncurses文件的头文件包含路径问题

 **第二类：** 用于测试的文件不能执行 

 **第三类：** Unsupported platform错误

下面简单记述一番。

#### 第一类：ncurses文件的头文件包含路径问题

**出现错误：**

>In file included from /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/cmd-line-utils/libedit/terminal.c:60:0:
>/home/velarn/work/ncurses/build/include/ncurses/curses.h:60:33: fatal error: ncurses/ncurses_dll.h: No such file or directory
>compilation terminated.
>cmd-line-utils/libedit/CMakeFiles/edit.dir/build.make:602: recipe for target 'cmd-line-utils/libedit/CMakeFiles/edit.dir/terminal.c.o' failed
>make[2]: *** [cmd-line-utils/libedit/CMakeFiles/edit.dir/terminal.c.o] Error 1
>CMakeFiles/Makefile2:457: recipe for target 'cmd-line-utils/libedit/CMakeFiles/edit.dir/all' failed
>make[1]: *** [cmd-line-utils/libedit/CMakeFiles/edit.dir/all] Error 2
>Makefile:162: recipe for target 'all' failed
>make: *** [all] Error 2 


**解决方法：**
把/home/velarn/work/ncurses/build//include/ncurses/curses.h头文件的第60行的 

>#include &lt;nucrses/ncursed_dll.h&gt;  

改为

>#include "/home/velarn/work/ncurses/build/include/ncurses/ncurses_dll.h"  


此外还需要更改**三处**地方，恕不详加列展，具体问题可参考[主要参考链接](https://blog.csdn.net/fhyocean/article/details/74960005)。

**以上解决curses库头文件关联问题。**

#### 第二类：用于测试的文件不能执行

**make时，出现错误：**

> ./comp_err: 1: ./comp_err: Syntax error: word unexpected (expecting ")")
>extra/CMakeFiles/GenError.dir/build.make:63: recipe for target 'include/mysqld_error.h' failed
>make[2]: *** [include/mysqld_error.h] Error 2
>CMakeFiles/Makefile2:6410: recipe for target 'extra/CMakeFiles/GenError.dir/all' failed
>make[1]: *** [extra/CMakeFiles/GenError.dir/all] Error 2
>Makefile:162: recipe for target 'all' failed
>make: *** [all] Error 2 


>./libmysql_api_test: 1: ./libmysql_api_test: Syntax error: word unexpected (expecting ")")
>libmysql/CMakeFiles/libmysql_api_test.dir/build.make:95: recipe for target 'libmysql/libmysql_api_test' failed
>make[2]: *** [libmysql/libmysql_api_test] Error 2
>make[2]: *** Deleting file 'libmysql/libmysql_api_test'
>CMakeFiles/Makefile2:1431: recipe for target 'libmysql/CMakeFiles/libmysql_api_test.dir/all' failed
>make[1]: *** [libmysql/CMakeFiles/libmysql_api_test.dir/all] Error 2
>Makefile:162: recipe for target 'all' failed
>make: *** [all] Error 2 


以及**comp_sql、gen_lex_hash、gen_lex_token、protoc**四个文件。

**解决方法：**
这类问题的解决方法是，把方才编译好的mysql-5.7.21-PC目录下对应的文件拷贝到mysql-5.7.21-arm相同目录下，替换掉该文件之后首先要在终端下执行touch该文件以更新该的时间（这个是必需的），然后才能继续make。


#### 第三类：Unsupported platform错误

出现错误：
>error: #error "Unsupported platform"

此问题之详细描述请参考[主要参考链接](https://blog.csdn.net/fhyocean/article/details/74960005)。

主要解决思想就是修改os0atomic.h和os0atomic.ic两个文件的一些宏定义。

##  3.4:执行make-install  

执行make并在无错误后，执行完make install，之后会在/usr/local/arm/目录下生成mysql文件夹。

##  3.5:需要拷贝并touch的文件列表  

>$ cp extra/comp_err /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/extra/  
>$ cp scripts/comp_sql /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/scripts/  
>$ cp sql/gen_lex_hash /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/sql/  
>$ cp sql/gen_lex_token /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/sql/  
>$ cp extra/protobuf/protoc /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/extra/protobuf/    
>$ cp libmysql/libmysql_api_test /home/velarn/work/mysql/mysql-5.7.21-arm/mysql-5.7.21/libmysql/    

**至此ARM版的mysql交叉编译完成。**

# 4-部署到目标arm板  

##  4.1:拷贝文件：  

1. 将/usr/local下的arm目录打包压缩并上传至目标机器的/usr/local/中并解压.
2. 在/usr/local/arm/mysql目录下创建data文件夹。

##  4.2:初始化mysql  

执行：（注意目录）

`
./mysqld --user=root --basedir=/usr/local/arm/mysql --datadir=/usr/local/arm/mysql/data --initialize 
`

以初始化mysql环境。

显示信息：

>2018-04-04T06:28:01.660078Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
>2018-04-04T06:28:01.789800Z 0 [Warning] InnoDB: MySQL was built without a memory barrier capability on this architecture, which might allow a mutex/rw_lock violation under high thread concurrency. This may cause a hang.
>2018-04-04T06:28:07.048011Z 0 [Warning] InnoDB: New log files created, LSN=45790
>2018-04-04T06:28:07.821112Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
>2018-04-04T06:28:07.988962Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 56de1a61-37d1-11e8-b647-7ec40e970e0c.
>2018-04-04T06:28:07.994920Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
>2018-04-04T06:28:07.999044Z 1 [Note] A temporary password is generated for root@localhost: 1NSRjqXdTd!&

**配置成功**

**需要注意的是， “1NSRjqXdTd!&”是自动生成的密码，请记录下来。**

##  4.3:启动mysql  

执行：

`
./mysqld --user=root &
`

**启动成功**

至此可以在终端执行 ps -aux | grep mysql 来查看mysql的pid。

##  4.4:另外一种启动方式：  

网上诸多教程中，是通过执行./support-files/mysql.server start来启动mysql，但是会出现大名鼎鼎的**The server quit without updating PID file**错误。

例如，执行：

`
./support-files/mysql.server start
`

则显示信息：
>Starting MySQL.Logging to '/usr/local/arm/mysql/data/Embedsky.err'.
> ERROR! The server quit without updating PID file (/usr/local/arm/mysql/data/Embedsky.pid).

可以参考的链接：
[https://www.cnblogs.com/ivictor/p/6846017.html](https://www.cnblogs.com/ivictor/p/6846017.html)

~~我在执行./mysqld --user=root &成功的前提下，修改support_files文件中的mysql.server~~

~~将开始的basedir=和datadir=修改为：~~

>~~basedir=/usr/local/arm/mysql~~
>~~datadir=/usr/local/arm/mysql/data~~

~~后，执行./support-files/mysql.server start 运行成功。。。~~

~~具体之问题下次再分析吧。~~

嗯，现在把解决的过程记录一下：

分析mysql.server源码可得知，在使用“**./mysql.server start**”时，脚本会调用mysqld_safe这个可执行脚本，

>...
>#Give extra arguments to mysqld with the my.cnf file. This script
>#may be overwritten at next upgrade.
>**$bindir/mysqld_safe** --datadir="$datadir" --pid-file="$mysqld_pid_file_path" $other_args >/dev/null &
>wait_for_pid created "$!" "$mysqld_pid_file_path"; return_value=$?># Make lock for RedHat / SuSE
>...

也就是说，在执行wait_for_pid前先调用mysqld_safe。
wait_for_pid返回错误肯定是mysqld_safe没起来嘛。于是乎手动测试了一下:

>../bin/mysqld_safe --basedir=/usr/local/arm/mysql --datadir=/usr/local/arm/mysql/data --pid-file=/usr/local /arm/mysql/data/Embedsky.pid

返回：

>2018-04-08T06:33:42.158958Z mysqld_safe Logging to '/usr/local/arm/mysql/data/Embedsky.err'.
>2018-04-08T06:33:42.281441Z mysqld_safe Starting mysqld daemon with databases from /usr/local/arm/mysql/data
>2018-04-08T06:33:42.911509Z mysqld_safe mysqld from pid file /usr/local/arm/mysql/data/Embedsky.pid ended

然后查看了以下线程运行，发现mysql没起来。

在网上找了找，都是说没权限创建pid文件。

例如：[这个](http://d-prototype.com/archives/3793)

但是我没找到mysql的log文件，于是乎我就在/etc文件夹下建了一个my.cnf，内容如下：

>[mysqld_safe]
>basedir=/usr/local/arm/mysql/
>datadir=/usr/local/arm/mysql/data
>pid-file=/usr/local/arm/mysql/data/Embedsky.pid
>log-error=/usr/local/arm/mysql/data/mysqld_safe.log

然后测试了一下，还是不行。

那把用户加上呢？

>[mysqld_safe]
>basedir=/usr/local/arm/mysql/
>datadir=/usr/local/arm/mysql/data
>pid-file=/usr/local/arm/mysql/data/Embedsky.pid
>log-error=/usr/local/arm/mysql/data/mysqld_safe.log
>user=root

**成功启动了。。。**

原来是mysqld在默认的情况下是以mysql用户启动，由于在arm上无法创建其他用户，因此在执行命令行时，我用的都是root，比如：

>./mysqld --user=root ...

所以，加上权限就行了。

**至此arm版的mysql移植完成。**

## 4.5:添加开机启动，待续。


# 5-一些其他操作  

##  5.1:登陆mysql：  

执行：
`
./mysql -u root –p
`
显示


>Welcome to the MySQL monitor.  Commands end with ; or \g.
>Your MySQL connection id is 2
>Server version: 5.7.21>

>Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.>

>Oracle is a registered trademark of Oracle Corporation and/or its
>affiliates. Other names may be trademarks of their respective
>owners.>

>No entry for terminal type "cygwin";
>using dumb terminal settings.
>Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.>

>mysql>



进入mysql成功。

##  5.2:更改默认密码  

使用安装时生成的随机密码登陆mysql:  

`  
mysqld -u root -p
`  

输入原始随机密码后，采用如下命令重设密码：  

`  
set password = password('123456');
`  

[参考链接](https://www.jianshu.com/p/53ac2d55b279)  



##  5.3:添加软连接  

ln -s /usr/local/arm/mysql/bin/mysql /usr/bin 

ln -s /usr/local/arm/mysql/bin/mysqldump /usr/bin 

ln -s /usr/local/arm/mysql/bin/mysqladmin /usr/bin 

ln -s /usr/local/arm/mysql/lib/libmysqlclient.so.20 /lib 
