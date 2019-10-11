---
layout: post
title : Docker 搭建Nginx环境
#二级标题
subtitle : Nginx环境的搭建
author : Bert
header-img: img/post-bg-2015.jpg
catalog : true
date : 2019-10-11 15:00:00
tags :
 - Linux
 - Nginx
 - Docker
---



# Docker 搭建Nginx环境

### 1. 获取nginx 镜像

- `docker pull nginx ` 默认获取最新版

- 拉取镜像下来后，用`docker images `查看镜像列表，我们就可以在本地镜像列表里查到　REPOSITORY 为 nginx 的镜像。

### 2. 启动nginx容器

- 我们用 ngxin 的默认配置来启动一个Ngxin的实例

- `docker run --name bert_nginx  -dp 8081:80 nginx` 

  - bert_nginx 是容器的名称
  - -d 设置容器在后台运行，-p 是端口映射，将本地的8081端口映射到内部的80端口
  - 运行以上命令会生成一个字符串，类似于839c10a353a2d53acd68086bf72ee123d6386656c4f50efcf069f44968ff8005，是容器的id号。

- 我们可以用docker ps 查看我们刚才的容器是否有在运行

  - ```shell
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
    839c10a353a2        nginx               "nginx -g 'daemon of…"   23 minutes ago      Up 23 minutes        0.0.0.0:8081->80/tcp   bert_nginx
    
    ```

### 3.nginx 环境部署

- 在当前目录下创建目录nginx,用于存放后面需要的东西
  - mkdir	-p	./nginx/www	./nginx/logs	./nginx/conf
- 将容器内的配置文件拷贝到外部nginx文件下
  - docker cp 839c10a353a2:/etc/nginx/nginx.conf ./nginx/conf
  - www目录将为nginx容器配置的虚拟目录
  - logs 目录将为容器配置的日志目录
  - conf目录将为容器的配置文件

### 4. 容器部署

- ` docker run -d -p 8082:80 --name bert-nginx-test-web -v ~/nginx/www:/usr/share/nginx/html -v ~/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v ~/nginx/logs:/var/log/nginx nginx`	

  - `~/nginx/www:/usr/share/nginx/html ` 将外部的www目录挂载到容器内的/usr/share/nginx/html。
  - ...

- 进入www 目录

  - `cd ~/nginx/www`

- 创建一个index.html ,内容如下

- ```html
  <!DOCTYPE html>
  <html>
  <head>
  <meta charset="utf-8">
  <title>Docker搭建nginx</title>
  </head>
  <body>
      <h1>Bert</h1>
      <p>搭建成功</p>
  </body>
  </html>
  ```

- 输出结果为：



<!-- ![图片]{img/nginx.png} -->
![图片](img/nginx.png)

