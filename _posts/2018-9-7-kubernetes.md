---
layout: post
title: kunernetes & docker新手村
---

从零开始学习kunernetes & docker

* 安装  
```yum install -y etcd kubernetes```  
-y : 默认yes
* 修改配置  
  1. 修改docker的配置文件/etc/sysconfig/docker，找到其中selinux-enabled位置  
  修改为：
  ```OPTIONS='--selinux-enabled=false --insecure-registry gcr.io'```  
  gcr.io是配置的镜像仓库地址，可能需要科学上网。  
  2. 修改kube-apiserver配置/etc/kubernetes/apiserver，删除--admission_control的ServiceAccount  
* 启动  
首先应该先关闭防火墙：  
   systemctl disable firewalld  
   systemctl stop firewalld  
服务列表：
```systemctl start etcd     
   systemctl start docker     
   systemctl start kube-apiserver     
   systemctl start kube-controller-manager     
   systemctl start kube-scheduler     
   systemctl start kubelet     
   systemctl start kube-proxy     
```
按顺序启动上述服务，etcd是k8s的主数据库，所以必须先于kube服务前启动。  

依赖关系：  
```etcd -> kube-apiserver  
   kube-apiserver ->  kube-controller-manager  
   kube-apiserver ->  kube-scheduler  
   docker -> kubelet  
   network -> kube-proxy  
```
可能出现的问题：
+ docker启动报错  
  ![_config.yml]({{ site.baseurl }}/images/k8s_docker_start.png)
解决方案：比较暴力点的就是卸载docker，然后重装k8s，会单独修复docker模块。  
```yum remove docker```  
```yum install -y kubernetes```  
+ 首次创建kube yaml文件创建rc时，出现"kubernetes启动容器时，容器一直是ContainerCreating不能running"，之后按照[解决方案1](https://blog.csdn.net/gezilan/article/details/80011905)执行到  
```docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest```  
报错"open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or"  
[解决方案2](https://blog.csdn.net/qq_15206589/article/details/81513178)
