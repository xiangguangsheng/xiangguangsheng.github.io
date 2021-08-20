+++
title = "shell"
date = "2021-07-19"
description = "shell编程"
featured = false
categories = [
  "linux"
]
tags = [
  "linux", "ubuntu"]
series = [
  "linux"
]
images = [
]

+++
# 1.安装后配置

## 1.1 更新系统

``` shell
# 列出可更新软件列表
sudo apt update
# 更新软件
sudo apt upgrade
# 清除旧的组件
sudo apt autoremove
#更新系统版本
sudo apt-get dist-upgrade
```

## 1.2 修改ip

``` shell
sudo vi /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: no
      addresses: [192.168.124.200/24]
      optional: true
      gateway4: 192.168.124.1
      nameservers:
       addresses: [223.5.5.5,223.6.6.6]
  version: 2

```

```shell
sudo netplan apply
```



## 1.3 修改hostname

``` shell
vim  /etc/cloud/cloud.cfg
# 修改  preserve_hostname: true
sudo hostnamectl set-hostname master
sudo reboot
```



## 1.4 安装docker

```shell
##文档路径 https://docs.docker.com/engine/install/ubuntu/
# 1.安装依赖工具
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
  # 2.安装 Docker’s official GPG key  
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# 3.设置标准仓库
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  # 4.安装
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io
 
  # 指定安装版本（可选）
 # apt-cache madison docker-ce
 # sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io

#5.查看docker信息
 sudo docker info
 
 #6. 配置docker
 
 sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors":["https://5m8lizc8.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
 
sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker

#7.解决docker info 报错WARNING: No swap limit support
vim /etc/default/grub
#GRUB_CMDLINE_LINUX=配置项，原有的内容切记不要删除，在双引号内添加cgroup_enable=memory swapaccount=1
update-grub
reboot
```

## 1.5 安装kubelet kubeadm kubectl

1. 更新 `apt` 包索引并安装使用 Kubernetes `apt` 仓库所需要的包：

   ```shell
   sudo apt-get update
   sudo apt-get install -y apt-transport-https ca-certificates curl
   ```

1. 下载 Google Cloud 公开签名秘钥：

   ```shell
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
   ```

1. 添加 Kubernetes `apt` 仓库：

   ```shell
   cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   ```

1. 更新 `apt` 包索引，安装 kubelet、kubeadm 和 kubectl，并锁定其版本：

   ```shell
   sudo apt-get update
   #列出所有的版本
   sudo apt-cache  madison   kubeadm
   sudo apt-get install -y kubelet=1.21.4-00 kubeadm=1.21.4-00 kubectl=1.21.4-00
   ##sudo apt-mark hold kubelet kubeadm kubectl
   systemctl daemon-reload
   systemctl enable kubelet 
   systemctl start kubelet
   ```

## 1.6  安装集群

### 1.6.1 创建集群安装配置文件kubeadm-config.yaml（master节点）

``` shell
#通过如下指令创建默认的kubeadm-config.yaml文件
kubeadm config print init-defaults  > kubeadm-config.yaml
```

```yaml
#默认文件如下
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 1.2.3.4
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: node
  taints: null
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: 1.21.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}

```

```yaml
#修改后如下
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s #加入集群token的过期时间
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.124.200 #本机的ip，主节点的ip
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master #主机名
  taints: #添加污点，master节点不部署容器
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "192.168.124.200:6443"    # 主节点的地址，高可用的时候为虚拟IP和haproxy端口
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers #替换为阿里云镜像
kind: ClusterConfiguration
kubernetesVersion: 1.21.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12 #service的网段
  podSubnet: 172.168.0.0/12 #pod的网段
scheduler: {}
```

```shell
#配置文件的版本和kubeadm版本不一致时可以使用
kubeadm config migrate --old-config kubeadm-config.yaml --new-config kubeadm-config-new.yaml

# 查看需要拉取的镜像
kubeadm config images list --config kubeadm-config.yaml
# 手动提前拉取要拉取的镜像，这步可以不操作，kubeadm init 时会自动拉取
kubeadm config images pull --config kubeadm-config.yaml

#安装集群
kubeadm init --config kubeadm-config.yaml --upload-certs 

# 如果出错了，可以重置集群 kubeadm reset
```

``` shell
## 初始化成功的提示信息
Your Kubernetes control-plane has initialized successfully!
## 普通用户管理集群，需要拷贝 /etc/kubernetes/admin.con 到.kube/config
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
## root管理集群，只需要配置环境变量即可
Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:
# 主节点加入到集群中的命令，--control-plane 只有主节点才会有
  kubeadm join 192.168.124.200:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:7a3bec57076a1dd4b7f9c3f7b5acebab29f669e6dc9dc94bc3e0b489e504915c \
	--control-plane --certificate-key b127516b8d4c04890a84f600549d117a5d7b606e4b092c9869ad248ca0aa41cb

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:
# node节点加入到集群中的命令，
kubeadm join 192.168.124.200:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:7a3bec57076a1dd4b7f9c3f7b5acebab29f669e6dc9dc94bc3e0b489e504915c 

```



``` shell
#admin.conf 保存的是集群的信息，kubectl 操作过去集群信息，就是通过这个文件
cat <<EOF >> ~/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
EOF
source ~/.bashrc

root@master:~# kubectl get node
NAME     STATUS     ROLES                  AGE     VERSION
master   NotReady   control-plane,master   9m56s   v1.21.4

root@master:~# kubectl get pod -A -o wide
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE   IP                NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-59d64cd4d4-7x2zl         0/1     Pending   0          12m   <none>            <none>   <none>           <none>
kube-system   coredns-59d64cd4d4-cbdk6         0/1     Pending   0          12m   <none>            <none>   <none>           <none>
kube-system   etcd-master                      1/1     Running   0          12m   192.168.124.200   master   <none>           <none>
kube-system   kube-apiserver-master            1/1     Running   0          12m   192.168.124.200   master   <none>           <none>
kube-system   kube-controller-manager-master   1/1     Running   0          12m   192.168.124.200   master   <none>           <none>
kube-system   kube-proxy-72zjr                 1/1     Running   0          12m   192.168.124.200   master   <none>           <none>
kube-system   kube-scheduler-master            1/1     Running   0          12m   192.168.124.200   master   <none>           <none>

#coredns 的STATUS为Pending，是因为网络组建没有装，装了网络组件后，状态变为正常
```

### 1.6.2 node节点加入到集群（node节点）

```shell
kubeadm join 192.168.124.200:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:7a3bec57076a1dd4b7f9c3f7b5acebab29f669e6dc9dc94bc3e0b489e504915c 
```



### 1.6.3  组建集群的token过期处理

``` shell
# 生成node节点加入集群的token
root@master:~# kubeadm token create --print-join-command
kubeadm join 192.168.124.200:6443 --token 6vtaym.uus1ojfhejme92c7 --discovery-token-ca-cert-hash sha256:7a3bec57076a1dd4b7f9c3f7b5acebab29f669e6dc9dc94bc3e0b489e504915c 

#生成master节点加入的token，master节点比node节点多了个--control-plane的key
root@master:~# kubeadm init phase  upload-certs --upload-certs
I0819 01:03:18.692929   28537 version.go:254] remote version is much newer: v1.22.0; falling back to: stable-1.21
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
8d8911281338fd20979a7bf4be9c103dd1499404afad592718efad837a8d9ac9

#组建主节点的加入命令
kubeadm join 192.168.124.200:6443 --token 6vtaym.uus1ojfhejme92c7 --discovery-token-ca-cert-hash sha256:7a3bec57076a1dd4b7f9c3f7b5acebab29f669e6dc9dc94bc3e0b489e504915c --control-plane --certificate-key 8d8911281338fd20979a7bf4be9c103dd1499404afad592718efad837a8d9ac9
```

``` shell
#其他命令
#查看所有的sectet
kubectl get secret -A
#输出密钥信息
kubectl get secret -n kube-system  bootstrap-token-6vtaym  -oyaml=
```

### 1.6.4  安装网络组建calico

```shell
##文档位置
# https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises
# 安装calico
curl https://docs.projectcalico.org/manifests/calico-etcd.yaml -o calico.yaml


```

```yaml
 #取消下面两行的注释，并将192.168.0.0/16 替换为kubeadm-config.yaml中的podSubnet 172.168.0.0/12
 - name: CALICO_IPV4POOL_CIDR
   value: "172.168.0.0/12" 

#修改etcd的地址
#etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"
etcd_endpoints: "https://192.168.124.200:2379"
#etcd的证书配置
```

```yaml
data:
  # Configure this with the location of your etcd cluster.
  #etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"
  etcd_endpoints: "https://192.168.124.200:2379" #etcd的地址，要使用https
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: "/calico-secrets/etcd-ca"  #打开注释
  etcd_cert: "/calico-secrets/etcd-cert" #打开注释
  etcd_key: "/calico-secrets/etcd-key" #打开注释

```

```yaml
# 使用命令 替换密钥 cat <file> | base64 -w 0
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: calico-etcd-secrets
  namespace: kube-system
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  etcd-key: LS0tLS1CRUdJT...VORCBSU0EgUFJJVkFURSBLRVktLS0tLQo= #/etc/kubernetes/pki/etcd/server.key
  etcd-cert: LS0tLS1CRUdJTiBDRVJU...LS0tCg== #/etc/kubernetes/pki/etcd/server.crt
  etcd-ca: LS0tLS1CRUdJ...tLS0tLQo= #/etc/kubernetes/pki/etcd/ca.crt

```

### 1.6.4  coredns安装失败

calico安装后，coredns镜像拉取失败

``` shell
# 查看错误信息
kubectl describe pod  coredns-59d64cd4d4-n46p6  -n kube-system
# 手动拉取镜像地址，每个node节点都要执行 然后改名
docker pull registry.aliyuncs.com/google_containers/coredns:1.8.0
docker tag  registry.aliyuncs.com/google_containers/coredns:1.8.0  registry.aliyuncs.com/google_containers/coredns:v1.8.0

# 删除错误的pod，k8s会自动重启pod，拉取镜像
kubectl delete pod coredns-59d64cd4d4-n46p6  -n kube-system
```

