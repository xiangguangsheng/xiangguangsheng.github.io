+++
title = "k8s kind"
date = "2021-07-19"
description = "docker"
featured = false
categories = [
  "k8s"
]
tags = [
  "k8s"]
series = [
  "k8s"
]
images = [
]

+++

# 1. node
## 1.1 master节点
1. kube-APIServer:集群的控制中枢，各个模块之间信息交互都需要经过Kube-APIServer，同时它也是集群管理、资源配置、整个集群安全机制的入口。
2. Controller-Manager:集群的状态管理器，保证Pod或其他资源达到期望值，也是需要和APIServer进行通信，在需要的时候创建、更新或删除它所管理的资源。
3. Scheduler:集群的调度中心，它会根据指定的一系列条件，选择一个或一批最佳的节点，然后部署我们的Pod。
4. Etcd:键值数据库，报错一些集群的信息，一般生产环境中建议部署三个以上节点（奇数个,建议和Master分开，单独部署）。

 ## 1.2 node节点
 1. kubelet:负责监听节点上Pod的状态，同时负责上报节点和节点上面Pod的状态，负责与Master节点通信，并管理节点上面的Pod。
 2. kube-proxy:负责Pod之间的通信和负载均衡，将指定的流量分发到后端正确的机器上。

> master节点也可以安装kubelet和kube-proxy，并配置污点，不运行node上的pod
> 

### 1.2.1查看Kube-proxy工作模式：
curl 127.0.0.1:10249/proxyMode
1. Ipvs：监听Master节点增加和删除service以及endpoint的消息，调用Netlink接口创建相应的IPVS规则。通过IPVS规则，将流量转发至相应的Pod上。
2. Iptables：监听Master节点增加和删除service以及endpoint的消息，对于每一个Service，他都会场景一个iptables规则，将service的clusterIP代理到后端对应的Pod。
## 1.3 其他组件
1. Calico：符合CNI标准的网络插件，给每个Pod生成一个唯一的IP地址，并且把每个节点当做一个路由器。(Cilium性能更好，可以替代calico)
2. CoreDNS：用于Kubernetes集群内部Service的解析，可以让Pod把Service名称解析成IP地址，然后通过Service的IP地址进行连接到对应的应用上。
3. Docker：容器引擎，负责对容器的管理。

# 2. pod

| 资源名称 | 是否namespace隔离 |
| -------- | ----------------- |
| Pod      | 是                |
| RC和RS   | 是                |
| pv       | 否                |
| RBAC     | 否                |
| pvc      | 是                |
|          |                   |
|          |                   |
|          |                   |
|          |                   |
|          |                   |
|          |                   |

----------------------------------------------------

![image-20210827235640899](http://qiniu.anideal.site/img/image-20210827235640899.png)

![image-20210827234932074](http://qiniu.anideal.site/img/image-20210827234932074.png)

# 其他

1.preSet

在preSet中，同一个主机路径，不能挂载到2个路径上，必须使用2个volumes来挂载

Ps：单独创建pod是可以这么操作的

![image-20210731105844713](http://qiniu.anideal.site/img/image-20210731105844713.png)

错误的配置：如下图

![image-20210731110212588](http://qiniu.anideal.site/img/image-20210731110212588.png)

