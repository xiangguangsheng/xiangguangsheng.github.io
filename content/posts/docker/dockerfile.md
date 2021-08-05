+++
title = "docker"
date = "2021-07-19"
description = "docker"
featured = false
categories = [
  "linux"
]
tags = [
  "dockerfile", "docker"]
series = [
  "docker"
]
images = [
]
+++
# 1.制作小镜像
# 1.1基础镜像的选择

1. alpine
2. busybox
3. scratch：空的镜像，什么都没有
4. debian
5. 如果用到glibc，可以使用各种slim镜像版本：例如node:slim

# 1.2使用多阶段构建
> 编译和最终的操作分开
> 多阶段构建时，如果使用了自定义的镜像，自定义的镜像被修改了，可以在build的时候添加参数 --pull来拉取最新的版本