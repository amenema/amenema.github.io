---
layout:     post
title:      "centos下搭建shadowsocks服务"
subtitle:   " \"科学上网\""
date:       2017-03-15 12:00:00
author:     "Mzx"
header-img: "img/post-bg-2015.jpg"
catalog: true
keyword: shadowsocks,vpn,科学上网,翻墙,梯子,服务器
description: 本文描述了如何在centos服务器上搭建shadowsocks服务,进行科学上网。
tags:
    - OSX
---


# **CentOS**搭建**shadowsocks**服务  

> 厌倦了莆田系的广告？想得到更准确的搜索结果？有问题希望Google一下？却不知如何翻墙？本文将送你一个科学的梯子。  

## 购买一台服务器  

首先需要购买一台国外的**VPS**,推荐买[Tzhost](https://idc.ac/aff.php?aff=57)。便宜，稳定。购买完成后选择系统为**CentOS** 64位。  

## 安装**shadowsocks**  

### 安装**python**  

``` 
[root@vps ~]# yum install python-setuptools && easy_install pip
```  

### 安装**shadowsocks**  

``` 
[root@vps ~]# yum install shadowsocks
```   

### 添加配置文件  
```
[root@vps ~]# vi /etc/shadowsocks.json 
//在文件中输入如下内容
{
"server":"128.1.208.139", //本机的外网ip地址，
"server_port":7623, //开放给shadowsocks的端口号
"local_address": "127.0.0.1", //本机内网地址，使用“127.0.0.1”即可
"local_port":1080, //内网端口，使用“1080”即可，也可改成其它
"password":"yourPassword", //你的链接密码
"timeout":600, //超时时间 500~5000皆可
"method":"ace-256-cfb" //密码加密方式有如下选择：“ace-256-cfb”,"table","rc4-md5","salsa20",等等
}
```  

### 启动与停止**shadowsocks**  
```
//启动
[root@vps ~]# ssserver -c /etc/shadowsocks.json -d start
//停止 
[root@vps ~]# ssserver -c /etc/shadowsocks.json -d stop
```  

## 配置开机启动  

### 在**/etc/init.d/**下添加一个shadowsocks服务  

```
[root@vps ~]# vi /etc/init.d/shadowsocks
//输入如下内容

#!/bin/sh
# chkconfig: 2345 90 10
# description: Start or stop the Shadowsocks server
#
### BEGIN INIT INFO
# Provides: Shadowsocks
# Required-Start: $network $syslog
# Required-Stop: $network
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Description: Start or stop the Shadowsocks server
### END INIT INFO
# Author: xju <qing0991@163.com>
name=shadowsocks
BIN=/usr/bin/ssserver
conf=/etc/shadowsocks.json
start(){
    $BIN -c $conf -d start
    RETVAL=$?
    if [ "$RETVAL" = "0" ]; then
        echo "$name start success"
    else
        echo "$name start failed"
    fi
}
stop(){
    pid=`ps -ef | grep -v grep | grep -v ps | grep -i "${BIN}" | awk '{print $2}'`
    if [ ! -z $pid ]; then
        $BIN -c $conf -d stop
        RETVAL=$?
        if [ "$RETVAL" = "0" ]; then
            echo "$name stop success"
        else
            echo "$name stop failed"
        fi
    else
        echo "$name is not running"
        RETVAL=1
    fi
}
status(){
    pid=`ps -ef | grep -v grep | grep -v ps | grep -i "${BIN}" | awk '{print $2}'`
    if [ -z $pid ]; then
        echo "$name is not running"
        RETVAL=1
    else
        echo "$name is running with PID $pid"
        RETVAL=0
    fi
}
case "$1" in
'start')
    start
    ;;
'stop')
    stop
    ;;
'status')
    status
    ;;
'restart')
    stop
    start
    RETVAL=$?
    ;;
*)
    echo "Usage: $0 { start | stop | restart | status }"
    RETVAL=1
    ;;
esac
exit $RETVAL
```  

### 添加执行权限  

```
[root@vps ~]# chmod a+x /etc/init.d/shadowsocks
```  

### 启动停止服务  

```
查看服务状态
[root@vps ~]# service shadowsocks status

启动服务
[root@vps ~]# service shadowsocks start

停止服务
[root@vps ~]# service shadowsocks stop

重启服务
[root@vps ~]# service shadowsocks restart

```


### 添加开机启动服务  
```  
[root@vps ~]# chkconfig --add shadowsocks
[root@vps ~]# chkconfig --list shadowsocks
输出：
shadowsocks    	0:off	1:off	2:on	3:on 4:on 5:on 6:off
```  

## 编写自动修改**shadowsocks**配置信息脚本  
> 目前的密码和端口是写死的，每次修改都需要登录到机器上手动修改，如果需要经常修改就会很麻烦。所以编写一个自动修改的脚本，定期修改，修改完成后发送一个邮件到你的邮箱即可。  

### 编写修改配置信息的脚本  

```
[root@vps ~]# vi change_pwd.sh

//添加如下内容
#!/bin/sh 
source /etc/profile
pwd=$(openssl rand -base64 10)
rp_str='"password":"'${pwd}'",'
sed -i '6c'${rp_str} /etc/shadowsocks.json
service shadowsocks restart
echo '密码已更新：'${pwd}|mail -s 'vps 密码更新邮件' abc@163.com
```  
#### 注： 
* **source /etc/profile**  是为了解决使用**crontab**执行定时任务导致环境不一致问题。
* **openssl rand -base64 10**  是生成一个随机密码，长度为10，可以替换为别的密码生成方式
* **sed -i '6c'${rp_str} /etc/shadowsocks.json**  是将配置文件的第六行替换为当前的内容，因为**shadowsocks.json**中密码在第六行，所以这里替换的是第六行，如果不是第六行则需要改成别的如在第五行，则改为**'5c'**  
* **service shadowsocks restart**  修改完成后需要重启才能够生效  
* **echo '密码已更新：'${pwd}\|mail -s 'vps 密码更新邮件' abc@163.com**  这里是使用**mail**服务发送邮件  

### 安装邮件 
这里使用外部的邮箱服务发送邮件，具体配置如下：

```
//安装mailx
[root@vps ~]# yum install mailx -y 

//配置外部服务器
[root@vps ~]# vi /etc/mail.rc
//添加如下内容
set from=abc@163.com smtp=smtp.163.com
set smtp-auth-user=abc@163.com smtp-auth-password=123456 smtp-auth=login

//from 是写信人：即你的邮箱账号，smtp 为固定地址“smtp.163com”
//smtp-auth-user 是你的邮箱账号， smtp-auth-password 是你的邮箱密码 smtp-auth 是认证方式 为固定值“login”
//配置完成后即可通过mail发邮件
```
### 添加可执行权限  

```
[root@vps ~]# chmod a+x change_pwd.sh
[root@vps ~]# ./change_pwd.sh
```

### 配置**crontab** 
目前我们只是完成了修改密码并发送邮件脚本，仍然需要手动执行。为了完成定时执行的任务，需要使用**crontab**

```
[root@vps ~]# crontab -e 0 9 */7 * * /root/change_pwd.sh
[root@vps ~]# crontab -l 
//输出 0 9 */7 * * /root/change_pwd.sh

//表示每七天早晨9点执行一次脚本。
``` 


## 下载**shadowsocks**客户端  

下载地址： [https://shadowsocks.com.hk/client.html](https://shadowsocks.com.hk/client.html)
下载对应系统的客户端，添加上你的服务器配置即可。


