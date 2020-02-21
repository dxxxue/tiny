[TOC]
## 容器redis相关配置

### 容器启动redis的案例
```bash
docker run --privileged=true -p 6379:6379 --name redis_master \
		-v /Users/dongdxxue/docker/redis/master/conf/redis.conf:/etc/redis/redis.conf \
		-v /Users/dongdxxue/docker/redis/master/data:/data \
		-d redis:5.0 \
		redis-server /etc/redis/redis.conf --appendonly yes
```
#### 对应的相关说明
- --privileged=true 提升权限为root, 容器的root权限在宿主机只是普通权限
- --appendonly yes 开启redis的持久化


### 容器启动redis服务注意事项
#### 容器使用主机上的服务(redis/mongodb)
- 原因: docker运行的网络环境默认是桥接模式(默认), 会连接不上127.0.0.1
- 解决: 1. 切换网路模式, 使用host模式, 主机与容器共享Ip \
       2. 将容器的ip添加到对应的服务中, 允许容器的Ip访问 