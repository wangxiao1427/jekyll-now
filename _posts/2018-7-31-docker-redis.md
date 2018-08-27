---
layout: post
title: 在docker中安装运行使用redis!
---


假设已经在centOS中安装好docker

- 拉取镜像
  在docker官方仓库搜索已经存在的镜像文件(`https://hub.docker.com`)，选择星级最高的镜像，如redis。  
  找到拉取当前镜像需要执行的命令，如docker pull redis，在centOS中启动docker环境后执行docker pull redis 拉取最新镜像到本地。
- 启动redis  
  >docker run --name my-redis -d redis  
  >--name:为容器指定一个名称  
  >-d:后台运行容器，并返回容器ID  
- 查看容器启动情况  
  >docker ps  
  >container-id  image command  ...  
  >74404d727c07   redis "docker-entrypoint..."  
- 使用镜像执行redis-cli命令连接到刚启动的容器  
  >docker exec -it 74404d727c07 redis-cli  
  >127.0.0.1:6379> set name shaw_wg  
  >127.0.0.1:6379> get name  
  >"shaw_wg"  
- 已经存在的未启动/正在运行的镜像 启动或重新启动  
  >docker start \<image id>
  >docker restart \<image id>
- redis设置密码  
  可以在redis环境下设置密码，  
  >127.0.0.1:6379> config set requirepass \<password>  
  >OK  
  >127.0.0.1:6379> config get requirepass  
  >(error) NOAUTH Authentication required.  
  >127.0.0.1:6379> auth \<password>  
  >OK  
  >127.0.0.1:6379> set key1 shaw_wg  
  >OK  
  >127.0.0.1:6379> get key1  
  >"shaw_wg"  
- 远程访问
  暂无
- 基础操作
  增/改  
  >127.0.0.1:6379> set \<key> \<value>  
  删  
  >127.0.0.1:6379> del \<key>  
  查  
  >127.0.0.1:6379> get \<key>  
  
