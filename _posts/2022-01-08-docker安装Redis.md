---
​---
layout: post
title: "docker安装常用软件"
date:   2022-01-08
tags: [docker]
comments: true
author: xiaobai
---

docker安装常用软件

<!-- more -->

## docker 安装配置 redis

### 下载Redis镜像

```
docker search redis
docker pull
```



### 下载redis.conf文件

```
http://www.redis.cn/download.html 
下载Redis，解压获取redis.conf文件
```



### 上传redis.conf 到服务器

```
scp /redis.conf root@xx.xx.xx.xx:/
```

修改redis.conf配置文件

```
bind 127.0.0.1 #注释掉这部分，使redis可以外部访问
daemonize no#用守护线程的方式启动
requirepass 你的密码#给redis设置密码
appendonly yes#redis持久化　　默认是no
tcp-keepalive 300 #防止出现远程主机强迫关闭了一个现有的连接的错误 默认是300
```

把`配置文件`拷贝到：/data/redis/data/redis.conf



### 启动redis

```
sudo docker run -p 6379:6379 --name redis -v /data/redis/data/redis.conf:/etc/redis/redis.conf  -v /data/redis/data:/data -d redis redis-server /etc/redis/redis.conf --appendonly yes

-p 6379:6379:把容器内的6379端口映射到宿主机6379端口
-v /data/redis/redis.conf:/etc/redis/redis.conf：把宿主机配置好的redis.conf放到容器内的这个位置中
-v /data/redis/data:/data：把redis持久化的数据在宿主机内显示，做数据备份
redis-server /etc/redis/redis.conf：这个是关键配置，让redis不是无配置启动，而是按照这个redis.conf的配置启动
–appendonly yes：redis启动后数据持久化
```



### 连接Redis

```
docker exec -it redis /bin/bash  // 进入Redis镜像
redis-cli -p 6379 -a 123456  // 连接Redis， -a:redis 密码
ping 返回pong 说明连接成功
```

