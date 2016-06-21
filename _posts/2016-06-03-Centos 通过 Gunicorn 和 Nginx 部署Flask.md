---
layout: post
title: " 阿里云部署 Flask"
date: 2016-06-04 10:34:37
tags: Linux Nginx Gunicorn 运维  
categories: wiki
---

## 一. 简介
 租的阿里云Centos 7.0, 使用 Nginx 和 Gunicorn部署 Flask，部署也是第一次，碰到好多问题，做个wiki 方便下次。

## 二. 过程

### 1. 挂载普通云盘
查看文档[挂载云盘](http://jingyan.baidu.com/article/90808022d2e9a3fd91c80fe9.html)

### 2. 初始化安装

> centos 默认 Python 版本是2.7.5和我使用的一样，故Python 没有升级。/etc/ssh/sshd_config 中设置 PermitRootLogin 设置为 without-password,当然设置之前要先添加 [ssh key](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2 添加ssh key)

    ``` 
    yum -y update
    yum -y install python-pip
    yum install python-devel
    yum install mysql-devel
    yum install git-all
    pip install virtualenv
    ```


### 3. 修改 host

> 不修改 hosts 名称也可以，不过自带的 host名称不好记住，换一个配置 nginx 的时候方便


    vim /etc/hosts
    hostname xxx


### 4. 安装 Mysql

[安装](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-centos-7)
Mysql 相关安全措施要做到位。

### 5. 部署代码

    git clone --depth 1 https://github.com/huangli/kuaidi.git
    cd kuaidi
    virtualenv .
    pip install -r requirement.txt

### 6. Gunicorn

> 如果出现问题可以去掉-D参数，调试可能出现的错误,如果要关闭进程 kill 'cat routes.pid'

    source ./bin/activate
    pip install gunicorn
    gunicorn -w 1 -b 127.0.0.1:8000 routes:app -p routes.pid -D

### 7. Nginx

> sites-enabled 和 sites-available 这个两个目录折腾了半天，原来只是遵循一个习惯而已，enabled 目录添加在配置文件中，available添加一个软连接到sites-enabled 目录，这样可以通过链接随时添加，比较灵活。

    yum install epel-release
    systemctl start nginx
    mkdir site-enabled
    vim /etc/nginx/sites-enabled/kuaidi
    systemctl start nginx
    
    在 /etc/nginx/nginx.conf 中添加 
    /etc/nginx/sites-enabled/*;
    检查配置语法
    nginx -t
    
    配置暂时如下：
    server {
        listen 80;
        server_name yckj;
        access_log  /home/kuaidi/log/access.log;
        error_log   /home/kuaidi/log/error.log;
        # Handle all locations
        location / {
                # Pass the request to Gunicorn
                proxy_pass http://127.0.0.1:8000;
                # Set some HTTP headers so that our app knows where the request really came from
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /static {
                root /home/kuaidi/;
                expires 30d;
        }
    }

### 8. Mysql

> 注意字符集统一用 utf-8

控制字符集，可以在/etc/my.cnf中添加两行

    character-set-server=utf8
    collation-server=utf8_general_ci

    CREATE DATABASE datebasename DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
    GRANT ALL PRIVILEGES ON datebasename.* TO 'username'@'localhost' WITH GRANT OPTION;
    FLUSH PRIVILEGES;

检查字符集

    show variables like '%colla%';
    show variables like '%charac%';

### 9. 问题

- 通过 python routes.py 运行没有问题，但是通过 gunicorn 和 nginx 就报错，后来把 debug 改为 True ，gunicorn -D 去掉看到了一个错误消息，Could not build url for endpoint 'admin.index'，这时才明白我把 admin 添加 view 的代码放在了 main 中，只需要把代码移出 main 即可。


## 三. 待优化

> 运维调优也是一个系统功能，了解的还太少

- 自动部署 fabric
- 系统监控 supervisor
- 如何增加服务器安全性

