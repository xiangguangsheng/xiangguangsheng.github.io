+++
title = "docker"
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

# 1. 组件
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

# 2. 资源隔离

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

# 3. 常用命令

```shell
# 查看pod.spec的完整配置字段
kubectl explain pod.spec
```

# 4.调度管理

## 4.1 资源分配调度

###4.1.1 基于pod中容器request资源“总和”调度

- resources。limits影响pod的运行资源上限，不影响调度
- initContainer取最大值，container取累加值，最后取大者。即Max(Max(initContainers.requests),Sum(containers.requests))

> initContainer运行后会退出，所以不需要累加

- 未指定request，按0资源需求进行调度

### 4.1.2 基于资源申明的调度，而非实际占用

- 不依赖监控，系统不会过于敏感
- 能否调度成功：pod.request<node.allocatable -node.request;pod的请求量 小于（node的分配量-已经调度的pod的量）

### 4.1.3 资源分配的盒子模型

![image-20210822233428309](http://qiniu.anideal.site/img/image-20210822233428309.png)

###4.1.4资源分配相关算法

- generalPredicates(主要是PodFitsResources)，检查余量，cpu内存硬盘是否满足
- LeastRequestedPriotity，最少被调度pod的节点
- BalanceResourceAllocation,平衡节点cpu/mem消耗比例

## 4.2 高级调度

### 4.2.1 nodeSelector:将pod调度到指定的node上

> 匹配node.lables,完全匹配

### 4.2.2 nodeAffinity：nodeselector升级版

> 与nodeSelector关键差异，是
>
> 1. 引入运算符：In，NotIN（lableSelector语法）
>
> 2. 支持枚举lable可能的取值，如zone in [az1,az2 ...]
>
> 3. 支持硬性过滤和软性评分
>
> 4. 硬性过滤规则支持指定 多条件之间的逻辑或运算
>
> 5. 软性评分规则支持 设置条件权重值

### 4.2.3 podAffinity:让某些pod分布在同一组上Node上

> 与nodeAffinity的关键差异
>
> 1. 定义在PodSpec中，亲和和反亲和规则具有对称性
> 2. labelSelector的匹配对象为Pod
> 3. 对node分组依据label-key=topologKey，每个label-value取值为一组
> 4. 硬性过滤规则，条件之间只有逻辑与运算

### 4.2.4podAntiAffinity：避免某些pod分布在同一组Node上

> 与podAffinity的匹配过程相同，最终的调度结果取反

### 4.2.5 手动调度pod

1. nodeName:直接指定运行的nodeName；

   > 适用于调度器不工作，临时救急;
   >
   > 封装实现自定义的调度器

2. DaemonSet：每个节点来一份

   > 适用于每个节点都需要部署的agent等

### 4.2.6 Taints：避免Pod调度到特定Node上

   1. 带effect的特殊label，对Pod有排斥星

   > 硬性排斥：NoSchedule
   >
   > 软性排斥 PreferNoSchedule

   2. 系统创建的taint附带时间戳

   > effect为NoExecute
   >
   > 便于触发对pod的超时驱逐

   典型用法：预留特殊节点做特殊用途

   ```shell
   # 给node添加taint
   kubectl taint node node-n1 foo=bar:NoSchedule
   # 删除taint
   kubectl taint node node-n1 foo:Noschedule-
   ```

   ### 4.2.7 Tolerations：允许Pod调度到有特定taints的Node上

   ## 4.3 调度失败的原因分析

   1. 查看调度结果

   ```shell
   kubectl get pod [podname] -o wide
   ```

   

   1. 查看调度失败的原因

      ```shell
      kubectl describe pod [podname]
      ```

      ## 4.4 多调度器

      1. 使用场景：集群中存在多个调度器，分别处理不同类型的作业调度
      2. 使用限制：建议对弄的做资源池划分，避免调度结果写入冲突

      ## 4.5 自定义调度器配置

      --policy-config-file自定义调度器加载的算法，或者调整排序算法权重

      ``` --helpshell
      # 查询多调度器配置
      kube-scheduler --help
      ```

      
      
# 5.监控

      ```shell
      # 查看集群状态
      kubectl cluster-info 
      kubectl cluster-info  dump >info.txt
      # 查看pod详情
      kubectl describe pod
      
      #占用资源排序
      kubectl top pod ｛podName｝
      
      # 监控资源运行过程
      kubectl get pod ｛podName｝ --watch
      ```


​      
​      
~~~shell
  ```shell
  # 日志查看
  # 组件日志
  /var/log/kube-apiserver.log
  /var/log/kube-proxy.log
  /var/log/kube-controller-manager.log
  /var/log/kubelet.log
  
  # 使用systemd管理
  journalctl -u kubelet
  
  #使用k8s插件部署
  kubectl logs kube-proxy
  
  # k8s应用日志
  #获取容器的日志
  # 1.从容器标准输出截获
  kubectl logs -f {podname} -c {container name}
  #或者使用docker命令
  docker logs -f {container Name}
  
  # 2.直接进入容器内查看日志
  kubectl exec -it {pod} -c {container} /bin/sh
  docker exec -it  {container} /bin/sh
  
  # 3.挂载到主机上的日志
  tail -f /log #主机挂载的目录
  ```
~~~

 # 6 Deployment 升级与回滚

      ```shell
      #创建deployment
      #可以使用yaml，重点配置replicas和image字段
      kubectl run ｛deployment｝-image={image} -replicas={rep.}
      
      #升级deployment
      kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
      kubectl set resources deploument/nginx-deployment -c=nginx --limits=cpu=200m,memory=521Mi
      ```
      
      ```shell
      # 暂停deployment：不计入升级历史
      kubectl rollout pause deployment/nginx-deployment
      # 恢复deployment
      kubectl rollout resume deployment/nginx-deployment
      
      # 查询升级状态
      kubectl rollout status deployment/nginx-deployment
      
      # 查询升级历史
      kubectl rollout history deployment/nginx-deployment
      kubectl rollout history deployment/nginx-deployment --revision=2
      
      # 回滚
      kubectl rollout undo deployment/nginx-deployment --通-revision=2
      
      # 应用弹性伸缩
      kubectl scale deployment nginx-doployment --replicas=10
      
      #自动伸缩，需要对接监控
      kubectl autoscale deployment nginx-deployment--min=10 --max=15 --cpu-percent=80
      ```
      
      ```shell 
      # 应用自恢复（重启策略+探针） 使用restartPolicy + livenssProbe
      
      ```

# 7. 网络

## 7.1 pod网络

1. 一个pod一个ip

- 每个pod独立ip，pod内所有容器共享网络namespace（同一个ip）
- 容器之间直接通讯，不需要nat
- Node和容器直接通信，不需要nat
- 其他容器和容器自身看到的ip是一样的

2. 集群内访问走service，集群外访问走ingress
3. CNI（container network interface）用于篇日志pod网络，不支持docker网络

# 7.2 service

1. clusterIp 
2. nodePort :创建后也被分配了clusterip
3. headless service：类型为clusterIp ，但是clusterIp 为none

![image-20210824222942310](http://qiniu.anideal.site/img/image-20210824222942310.png)

![image-20210824223036236](http://qiniu.anideal.site/img/image-20210824223036236.png)

### ingress

> ingress是授权入站连接到达集群服务的规则集合
>
> 1. 支持通过url方式将service暴露到k8s集群外，service之上的L7访问入库
> 2. 支持自定义service的访问策略
> 3. 提供按域名访问的虚拟主机功能
> 4. 支持tls

### k8s dns

> 解析pod和service的域名，k8s集群内pod使用 
>
> 通过CoreDNS实现（kube-dns已经废弃）

#### 对Service 

1. A记录

```shell
# 普通service
my-svc.my-namespace.svc.cluster.local  #cluster ip
#headless service
my-svc.my-namespace.svc.cluster.local # 后端ip列表
```

2. SRV记录

```shell
_my-port-name._my-port-protocol.my-namespace.svc.cluster.local # service port
```

#### 对pod

1. A记录

```shell
pod-ip.my-namespace.pod.cluster.local  #pod ip

```

2. 在pod spec 指定hostname和subdomain

``` shell
hostname.subdomain.my-namespace.pod.cluster.local # pod ip
```

### 查看dns

1. nslookup

```shell
#需要安装 nslookup
wget https://kubernetes.io/examples/admin/dns/busybox.yaml
# 运行容器
kubectl  create -f busybox.yaml 
# 查看kubernetes.default域名
kubectl exec -it busybox -- nslookup kubernetes.default
```

```shell
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local

```

# 8. 存储管理

## 8.1 普通存储卷volume

> 存储没有单独资源对象，与pod的生命周期一起
>
> deployment应用建议使用普通卷

![image-20210824234118722](http://qiniu.anideal.site/img/image-20210824234118722.png)

# 8.2 持久化存储卷pv

> 存储系统与应用系统区分开，单独资源对象，它不直接和pod发生关系，通过另一个资源对象pvc来绑定
>
> statefulset应用建议使用持久卷

1. 静态模式

> 除了创建pvc外，还需要手动创建pv
>
> ![image-20210824234827809](http://qiniu.anideal.site/img/image-20210824234827809.png)

2. 动态模式

> 只需要创建pvc，系统根据pvc自动创建pv
>
> ![image-20210824235028384](http://qiniu.anideal.site/img/image-20210824235028384.png)

# 9. 安全管理

## 9.1 安全全景图

![image-20210827230851737](http://qiniu.anideal.site/img/image-20210827230851737.png)

1. 部署态的安全控制

> - 认证
> - 鉴权
> - Admission（准入控制）
> - Pod SecurityContext

1. 运行态的安全控制

> - Network policy

## 9.2 认证

![image-20210825003603124](http://qiniu.anideal.site/img/image-20210825003603124.png)

![image-20210825003629222](http://qiniu.anideal.site/img/image-20210825003629222.png)

![image-20210825003655282](http://qiniu.anideal.site/img/image-20210825003655282.png)



## 9.3 鉴权

![image-20210825003736994](http://qiniu.anideal.site/img/image-20210825003736994.png)

## 9.4 admission

![image-20210825003815340](http://qiniu.anideal.site/img/image-20210825003815340.png)

## 9.5 安全的持久化保存键值etcd

![image-20210825003449979](http://qiniu.anideal.site/img/image-20210825003449979.png)

## 9.6 安全上下文（pod securityContext）

![image-20210825004238224](http://qiniu.anideal.site/img/image-20210825004238224.png)

## 9.7 Network Policy

![image-20210825004644039](http://qiniu.anideal.site/img/image-20210825004644039.png)

![image-20210825005010880](http://qiniu.anideal.site/img/image-20210825005010880.png)

# 其他

1.preSet

在preSet中，同一个主机路径，不能挂载到2个路径上，必须使用2个volumes来挂载

Ps：单独创建pod是可以这么操作的

![image-20210731105844713](http://qiniu.anideal.site/img/image-20210731105844713.png)

错误的配置：如下图

![image-20210731110212588](http://qiniu.anideal.site/img/image-20210731110212588.png)

