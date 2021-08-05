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

## 1.2 安装docker

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

#查看docker信息
 sudo docker info

```

