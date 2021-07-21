+++
title = "docker"
date = "2021-07-19"
description = "docker"
featured = false
categories = [
  "linux"
]
tags = [
  "linux", "docker"]
series = [
  "docker"
]
images = [
]
+++

# 1. 基础环境搭建

## 1.1 mysql

```shell
mkdir -p /data/docker/mysql/data
docker run --name mysql -v /data/docker/mysql/data:/var/lib/mysql -p 3 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7.35
```

