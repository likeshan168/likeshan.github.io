---
title: windows下的linux系统ip自动绑定在hosts文件中
date: 2022-06-14 16:28:53
tags: 技术杂谈
---

## 前言

windows系统的linux，在每一次启动的时候ip总是变动的，导致在windows系统中连接linux下的mysql服务总是失败，那是否可以在启动linux系统的时候就将ip地址绑定到windows的hosts文件中呢？这样通过本地的域名就能连接到linux系统的服务了。

## 编写脚本

编写shell脚本：modify_hosts.sh

```shell
#!/bin/bash

params[1]=$1
# get ip address
ip_addr=$(ip addr|grep eth0|grep inet|awk '{print $2}'|cut -d / -f 1)
# ifconfig eth0|sed -n '2p'|awk '{print $2}' #该命令获取ip地址更为简洁
# 判断参数是否为空
if [ -z ${params[1]} ]
then
        #为空，则获取系统的名称
        sys_name=$(cat /etc/lsb-release|grep ID|cut -d = -f 2)
else
        #不为空，则取第一个参数名
        sys_name=${params[1]}
fi

host_name=$sys_name".wsl"

win_host_path=/mnt/c/Windows/System32/drivers/etc/HOSTS
#获取行号
line_no=$(nl -b a $win_host_path|grep $host_name|awk '{print $1}')

for line in $line_no
do
        #删除该行的内容
        sed -i $line'd' $win_host_path
done

#追加ip的映射
echo $ip_addr' '$host_name >> $win_host_path

[ -f "$win_host_path" ] && echo "windows host:"$(nl $win_host_path|grep $host_name) && echo 'linux ip addr:'$ip_addr
exit 0
```

## 自动执行

为了能让脚本每次启动的时候自动运行，修改~/.bashrc文件，并在最后添加如下代码：

```shell
bash /root/shell_scripts/modify_host.sh myubuntu
# 下面的代码是想每次启动的时候确保mysql服务也启动了
service mysql status|grep -w stopped
if [ $? -eq 0 ]
then
        service mysql start
fi
```

这样，只需要在windows下使用myubuntu.wsl域名就能连接到linux的服务器

![image-20220614165923164](windows-linux-ip/image-20220614165923164.png)
