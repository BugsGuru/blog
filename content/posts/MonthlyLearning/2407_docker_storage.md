---
title: "Docker Storage"
date: 2024-07-25

categories: [learning note]
tags: ["docker"]
keywords: ["docker storage"]

description: "Manage data in Docker"
---

## Q1:docker 数据管理有哪几种方式，各有什么特点
1. volume
    - 由docker管理，不应被其他进程更改
    - 持久化的
    - 可由多个容器复用
    - 支持卷驱动（可远程存储）
2. bind mount
    - 可在宿主机任意位置
    - 可被其他进程更改
    - 持久化的 
    - 可由多个容器复用
3. tmpfs mount
    - 存储在宿主机内存中，可快速读写，不会写入持久存储设备
    - 容器停止或重启时，tmpfs的数据可以自动清除


## Q2:每种数据管理方式使用场景举一个例子
1. volume\
   挂载数据库数据文件
2. bind mount \
   挂载配置文件
3. tmpfs mount \
   用做缓存目录，如nginx的proxy_cache_path

## 参考
- [https://docs.docker.com/storage/](https://docs.docker.com/storage/)
