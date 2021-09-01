+++
title = "hugo"
date = "2021-07-19"
description = "静态网站发布工具"
featured = false
categories = [
  "tool"
]
tags = [
  "hugo", "github pages"]
series = [
  "tool"
]
images = [
]

+++



# 1. hugo安装配置

## 1.1 下载hugo

1. 下载  [hugo]( https://github.com/gohugoio/hugo/releases),并配置环境变量

  ![image-20210720005626447](http://qiniu.anideal.site/img/image-20210720005626447.png)
  
  2. 运行demo
  
  ``` shell
  cd  e:workspace/workspace-notes/
  hugo new site notes
  cd notes
  git submodule add https://github.com/razonyang/hugo-theme-bootstrap themes/hugo-theme-bootstrap
  mkdir config
  cp -a themes/hugo-theme-bootstrap/exampleSite/config/* ./config
  #复制exampleSite\content\zh-cn 中的目录到content\下,然后按照自己的需求修改
  ```
  
  ## 1.2 hugo配置
  
  配置文件位置为项目的根目录下的`config.toml`
  
  ``` toml
  baseURL = "https://xiangguangsheng.github.io/"
  languageCode = "zh-cn"
  title = "james的个人主页"
  publishDir = "docs" #修改了默认的发布目录
  ```
  
  ## 1.3 GitHub pages配置
  
  1. 新建仓库
  2. 添加pages配置
  
  > 下图1中的仓库名称，必须为：xxx.github.io,其中xxx为您的账号名称
  
  ![image-20210720011437764](http://qiniu.anideal.site/img/image-20210720011437764.png)
  
  ![image-20210720011224855](http://qiniu.anideal.site/img/image-20210720011224855.png)
  
   

##  1.4 访问页面

1. 提交代码到GitHub

2. 直接访问 [https://xiangguangsheng.github.io/](https://xiangguangsheng.github.io/)

## 1.5 页面更新

```shell
# 1.执行hugo命令，自动编译页面，然后提交到github
hugo
# 2. 本地查看页面
hugo server

```

