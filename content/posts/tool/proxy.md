+++
title = "代理"
date = "2021-07-19"
description = "关于代理相关的工具"
featured = false
categories = [
  "tool"
]
tags = [
  "proxy","github","SwitchHosts","Shadowsocks"]
series = [
  "tool"
]
images = [
]

+++

# 1. Github访问慢处理办法

## 1.1 方法1：GitHub520(推荐)

1. 下载[SwitchHosts](https://github.com/oldj/SwitchHosts/releases)
2. 按照下图配置SwitchHosts

url为：https://raw.hellogithub.com/hosts

![image-20210720001305528](http://qiniu.anideal.site/img/image-20210720001305528.png)

## 1.2 方法2：手动修改host方式

1. 通过站长工具查询github的ip，http://tool.chinaz.com/dns

![image-20210720001744633](http://qiniu.anideal.site/img/image-20210720001744633.png)

2. 修改host文件（C:\Windows\System32\drivers\etc\hosts），选择上图中速度最快的ip

   ```
   52.192.72.89 github.com
   ```

3. github.com的ip是动态变化的，经常需要重复上述步骤修改

# 2. Shadowsocks安装

> centos8,使用了新的包管理工具dnf来代替yml，使用podman来代理docker

## 2.1 使用podman安装Shadowsocks的服务端

```shell
dnf install -y podman
podman pull shadowsocks/shadowsocks-libev
podman run -e PASSWORD=Td123456 -e METHOD=aes-256-gcm  -p28080:8388 -p28080:8388/udp  -d shadowsocks/shadowsocks-libev

```

## 2.2 安装Shadowsocks的客户端

1. 下载并安装Shadowsocks客户端：https://github.com/shadowsocks/shadowsocks-windows/releases

2. 如下图配置Shadowsocks客户端

![image-20210720004044003](http://qiniu.anideal.site/img/image-20210720004044003.png)
