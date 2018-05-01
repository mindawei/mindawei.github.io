---
title: Redis 的安装和使用
date: 2018-05-01 14:41:47
tags: [Redis,入门] 
categories: Redis
---

本文是安装和使用 Redis 时的一个简单记录，主要参考了 [Redis 教程](http://www.runoob.com/redis/redis-tutorial.html)。
<!-- more -->

# 安装与启动
参考 [Redis 安装](http://www.runoob.com/redis/redis-install.html)

mac 下安装也可以使用 homebrew，homebrew 是 mac 的包管理器。
1、执行 `brew install redis`
2、启动 redis，可以使用后台服务启动 `brew services start redis`。或者直接启动：`redis-server /usr/local/etc/redis.conf`。

# 设置
参考 [Redis 配置](http://www.runoob.com/redis/redis-conf.html)

打开配置文件 `/usr/local/etc/redis.conf` 进行配置。

requirepass  字段配置密码。

# 相关操作
## 客户端启动
1.默认启动
`redis-cli` 启动客户端，按照默认配置连接 Redis（127.0.0.1:6379）。

2.指定地址和端口号
`redis-cli -h 127.0.0.1 -p 6379`

## 关闭服务端

参考 [mac下redis安装、设置、启动停止](https://www.cnblogs.com/shoren/p/redis.html)

1.强行关闭
强行终止redis进程可能会导致数据丢失，因为redis可能正在将内存数据同步到硬盘中。
```
ps axu|grep redis  ## 查找redis-server的PID
kill -9 PID
```

2.命令关闭
向redis发送SHUTDOWN命令，即 `redis-cli SHUTDOWN`。Redis收到命令后，服务端会断开所有客户端的连接，然后根据配置执行持久化，最后退出。

```
# 启动redis-server，后台线程
AT8775:redis shoren$ redis-server /usr/local/etc/redis.conf 
12105:C 29 Apr 22:16:14.570 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
12105:C 29 Apr 22:16:14.571 # Redis version=4.0.9, bits=64, commit=00000000, modified=0, pid=12105, just started
12105:C 29 Apr 22:16:14.571 # Configuration loaded

## 启动成功
AT8775:redis shoren$ ps axu|grep redis
mindw            12108   0.0  0.0  2432804   1940 s002  S+   10:16下午   0:00.00 grep redis
mindw            12106   0.0  0.0  2458976   2248   ??  Ss   10:16下午   0:00.02 redis-server 127.0.0.1:6379

## 关闭服务器
mindwdeMacBook-Pro:~ mindw$ redis-cli
127.0.0.1:6379> AUTH 你设置的密码
OK
127.0.0.1:6379> shutdown
mindwdeMacBook-Pro:~ mindw$ ps axu|grep redis
mindw            12122   0.0  0.0  2432804   1940 s002  S+   10:19下午   0:00.00 grep redis
```

## 客户端操作 Redis 服务器

参考 [Redis 服务器](http://www.runoob.com/redis/redis-server.html)