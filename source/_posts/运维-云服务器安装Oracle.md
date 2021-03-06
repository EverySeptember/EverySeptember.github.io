---
title: 云服务器安装Oracle
date: 2018-01-28
tags: [运维,Oracle]
---
# 系统准备

1. 下载
   ```shell
   wget -c --http-user=用户名 --http-password=密码--output-document=输出名称 "下载地址" -o 日志名 &
   ```

   Oracle下载需要提供用户名密码，提前注册好，下载地址可以在本地浏览器获取

2. 配置内存

   首先查看内存是否满足使用要求：

   ```shell
   [root@lu opt]# free -m
                 total        used        free      shared  buff/cache   available
   Mem:           1839          81         713           0        1044        1593
   Swap:             0           0           0
   ```

   oracle的安装启动需要足够的交换分区，云服务器并没有配置交换分区，所以需要手动配置，如下便配置了一个2g的交换分区

   ```shell
   [root@lu opt]# dd if=/dev/zero of=/mnt/swap bs=2M count=1024
   1024+0 records in
   1024+0 records out
   2147483648 bytes (2.1 GB) copied, 19.0152 s, 113 MB/s
   [root@lu opt]# mkswap /mnt/swap
   Setting up swapspace version 1, size = 2097148 KiB
   no label, UUID=f017d5b7-ef9b-4f3f-95c3-5b503311416b
   ```

   启动交换分区

   ```shell
   [root@lu opt]# swapon /mnt/swap
   swapon: /mnt/swap: insecure permissions 0644, 0600 suggested.
   ```

   修改交换分区配置：

   ```shell
   在 /etc/fstab 文件中增加或修改swap行,
   #开机启动swap:
   /mnt/swap swap swap defaults 0 0
   #修改默认配置，防止ORA-00845: MEMORY_TARGET not supported on this system错误
   tmpfs /dev/shm tmpfs defaults,size=10240M 0 0
   ```

   重启系统

   ```shell
   reboot
   ```

3. 软件环境

   ```shell
   yum -y install binutils compat-libstdc++ compat-libstdc++-33elfutils-libelf-devel gcc gcc-c++ glibc-devel glibc-headers ksh libaio-devellibstdc++-devel make sysstat unixODBC-devel binutils-* compat-libstdc++*elfutils-libelf* glibc* gcc-* libaio* libgcc* libstdc++* make* sysstat*unixODBC* wget unzip
   
   yum clean all
   ```
   文件解压缩

   ```shell
   unzip 文件名
   ```

4. 内核参数配置

   在/etc/sysctl.conf添加

   ```vim
   kernel.shmmni = 4096
   kernel.sem = 250 32000 100 128
   net.ipv4.ip_local_port_range = 9000 65500
   net.core.rmem_default = 262144
   net.core.rmem_max = 4194304
   net.core.wmem_default = 262144
   net.core.wmem_max = 1048576
   ```

   加载参数

   ```shell
   sysctl -p
   ```

5. 用户配置

   1. 创建账号

      ```
      安装组
      groupadd 安装组名
      管理组
      groupadd 管理组名
      运行用户
      useradd -G 安装组名 -G 管理组名 运行用户名
      密码
      passwd 密码
      ```

   2. 调整用户变量

      ```vim
      在~用户名/.bash_profile最后添加
      umask 022
      export LC_ALL=zh_CN.UTF-8
      export PATH=$PATH:/oracle安装目标位置/product/11gr2/dbhome_1/bin
      export ORACLE_HOME=/oracle安装目标位置/product/11gr2/dbhome_1
      export ORACLE_SID=orcl
      
      加载文件
      source ~用户名/.bash_profile
      
      在/etc/profile添加
      export PATH=$PATH:/oracle安装目标位置/product/11gr2/dbhome_1/bin
      export ORACLE_HOME=/oracle安装目标位置/product/11gr2/dbhome_1
      export ORACLE_SID=orcl
      
      更新系统环境
      source /etc/profile
      ```

   3. 调整会话限制

      ```
      在/etc/pam.d/login添加
      session    required    pam_limits.so
      
      在/etc/security/limits.conf添加
      oracle        soft    nproc    8192
      oracle        hard    nproc    16384
      oracle        soft    nofile   32768
      oracle        hard    nofile   65536
      ```

# 安装步骤

1. 创建安装目录

   ```shell
   mkdir -p /data/oracle
   #给文件夹赋所属用户
   chown -R 用户名.安装用户组名 /data/oracle/
   #给文件夹赋权限
   chmod -R 775 /data/oracle/
   ```

2. 编辑应答文件

   ```vim
   #打开oracle解压目录下/response目录
   #编辑安装应答文件，vi db_install.rsp
   oracle.install.option=INSTALL_DB_SWONLY			    #安装模式：只装数据库软件
   UNIX_GROUP_NAME=安装用户组名						  #指定oracle inventory的所有者
   SELECTED_LANGUAGES=en,zh_CN                          #配置安装语言
   INVENTORY_LOCATION=/data/oracle/oraInventory         #指定产品清单oracle inventory目录的路径
   ORACLE_HOME=/data/oracle/product/11gr2/dbhome_1      #配置ORALCE_HOME目录
   ORACLE_BASE=/data/oracle                             #配置ORALCE_BASE目录
   oracle.install.db.InstallEdition=EE                  #配置安装版本：企业版
   oracle.install.db.DBA_GROUP=管理用户组名              #配置拥有dba权限的用户组
   oracle.install.db.OPER_GROUP=管理用户组名             #配置拥有oper权限的用户组
   DECLINE_SECURITY_UPDATES=true                        #配置是否开启安全更新
   ```

3. 安装软件

   ```shell
   #切换至oracle用户
   su oracle用户名
   #运行安装脚本
   ./runInstaller -silent -debug -force -ignorePrereq -responseFile /opt/database/response/db_install.rsp
   ```

4. 启动

   ```shell
   #启动监听
   netca /silent /responsefile /opt/database/response/netca.rsp
   
   #启动服务
   lsnrctl start
   
   #查看状态
   lsnrctl status
   
   #查看监听端口
   netstat -naptlu | grep 1521
   
   状态正常则可继续安装数据库
   ```

5. 安装数据库

   ```shell
   #在root权限下copy数据库安装响应文件至oracle用户目录下
   cp /opt/database/response/dbca.rsp ~oracle用户名
   
   #更改权限
   chown -R oracleuser.orainstall ~oracle用户名/dbca.rsp
   chmod -R 775 ~oracle用户名/dbca.rsp
   
   #修改响应文件
   GDBNAME = "全局数据库的名字=SID+名"
   SID="orcl" // SID
   CHARACTERSET="AL32UTF8" //编码
   NATIONALCHARACTERSET="UTF8" //编码
   SYSPASSWORD = "//密码"
   SYSTEMPASSWORD = "//密码"
   
   #切换至oracle用户，安装数据库
   dbca -silent -responseFile ~/dbca.rsp
   ```

6. 启动数据库

   ```shell
   #连接sqlplus
   sqlplus dba用户名/dba密码 as sysdba
   #启动数据库
   startup
   #修改用户密码有效期
   ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME UNLIMITED;
   ```

# 远程连接

在云上安装oracle之后，由于许多云服务默认将1521端口关闭，需配置打开才可以远程连接。

# 参考文章

1. [CentOS 无图形化安装oracle 11gr2](http://blog.51cto.com/haowen/1599042)
2. [Linux 使用 wget 下载 Oracle 软件说明](https://blog.csdn.net/tianlesoftware/article/details/7745667)
3. [linux 程序安装目录/opt目录和/usr/local目录](https://blog.csdn.net/w410589502/article/details/77848026)
4. [云服务器 ECS Linux SWAP 配置概要说明](https://help.aliyun.com/knowledge_detail/42534.html)
5. [db_install.rsp dbca.rsp netca.rsp 详解](http://www.cnblogs.com/chenjunjie/p/6116480.html)
6. [Oracle用户，权限，角色以及登录管理](http://blog.csdn.net/fuwencaho/article/details/21156243)
7. [阿里云添加安全组规则](https://help.aliyun.com/document_detail/25471.html)