---
date: 2024-10-14
---

## 远程工具

SSH

Xftp

## 用户管理

- `/etc/group`
- `/etc/passwd`
- `/etc/shadow`

## 文件和目录管理

## VIM

快捷命令

## 文件系统管理

简介

- 非索引式文件系统：block

- 索引式文件系统（效率高）：inode、block

### 磁盘分区

主分区（3个）、扩展分区（1个）、逻辑分区（N个）

### 配置分区

1. `fdisk -l``
2. ``fdisk 设备名`
   1. `fdisk /dev/sda`
3. `/etc/fstab`

`df`、`du`、`lsof`

## 网络管理

### 网络基础

设备、链路、接口、子网掩码、广播地址

### 配置网络接口

`ifconfig`

`ifconfig 接口 up`

- `/etc/sysconfig/network-scripts/ifcfg-*`

### 配置路由

`route`临时配置（测试用），开机使用需要修改配置文件

### 网络侦测

`ping -c 3 $IP`：ICMP协议

``traceroute`

## 进程管理与服务管理

### 查看和管理进程

`ps -l`、`ps aux`、`top`、`pstree`、`kill`、`killall`

### 任务管理

`jobs`

Ctrl+z

`fg %jobid`

``bg %jobid`

`crontab`周期执行计划

`at`定时计划任务

### 服务管理

`systemd`

`systemctl`

## 系统监控

### 监控

lspci

iostat

netstat

/proc

free内存使用情况

fdisk

uptime

uname -a

### 查看登录信息

`who`

`w`

`finger`

`last`登录过系统的信息

`lastlog`用户最近一次登录信息







```
docker run -itd -m 2GB --cpus 2 -u HwHiAiUser:HwHiAiUser --pid=host xxx /bin/bash
```

