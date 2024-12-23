---
title: 搭建Java服务器(Ubuntu20.04)运行环境
date: 2023-11-14
authors: ["Mireux"]
categories: ["Ubuntu", "Java"]
---

## 前言

第一次搭Ubuntu环境，记录一下大大小小的坑。关于端口开放，或者相关命令这里不会说明，默认都了解。

<!-- more -->

## docker环境

安装docker详见[从零开始的Ubuntu之旅](https://samomi0.github.io/docs/GetStartWithUbuntu/)。这里主要贴一下docker-compose,包含常用的一些镜像。
### docker-compose

> 移植到《简易docker使用手册》？

- 安装
>tips: 您可以在[docker-compose](https://github.com/docker/compose/releases)中查找官方版本。请确保使用最新版本。
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
- 授权可执行权限 
使用以下命令将docker-compose二进制文件标记为可执行文件：
`sudo chmod +x /usr/local/bin/docker-compose`
> tips: 运行 `docker-compose --version` 验证Docker Compose是否正确安装

- docker-compose.yml
``` yml
version: "3"
networks:
  es-net:
    driver: bridge

services:
  redis:
    image: redis
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - /data/redis/conf/redis.conf:/etc/redis/redis.conf
      - /data/redis/data:/data
    command: redis-server /etc/redis/redis.conf

  elasticSearch:
    image: elasticsearch:7.17.7
    container_name: es
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - /data/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      - /data/es/data:/usr/share/elasticsearch/data
      - /data/es/logs:/usr/share/elasticsearch/logs
      - /data/es/plugins:/usr/share/elasticsearch/plugins
      - /data/es/plugins:/usr/share/elasticsearch/plugins
    privileged: true
    networks:
      - es-net
    environment:
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"

  kibana:
    image: kibana:7.17.7
    container_name: kibana
    ports:
      - "5601:5601"
    volumes:
      - /data/kibana/data:/usr/share/kibana/data
      - /data/kibana/config:/usr/share/kibana/config
      - /data/kibana/plugins:/usr/share/kibana/plugins
      - /data/kibana/logs:/usr/share/kibana/logs
    privileged: true
    networks:
      - es-net
  minio:
    image: minio/minio
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - /data/minio/data:/data
    environment:
      - "MINIO_ACCESS_KEY=minioadmin"
      - "MINIO_SECRET_KEY=minioadmin"
    command: server /data --console-address ":9001"
```
> 小tips:
> - 挂载的config中都需要有xxx.conf文件，尤其是kibana会直接导致启动失败,其中kibana还需要配置`elasticsearch.hosts`,保证kibana能够被正常访问, 挂载目录同时需要777的权限
> - 上述全是单机模式，如果需要集群只需要复制即可
> - es记得配置ik分词器哦~


## mysql环境

### 安装

> 网上安装配置mysql教程纷繁杂乱。记录常用到的安装和配置mysql教程，作为笔记和踩坑日志，便复用。

在ubuntu中安装mysql只要一行命令就可以了 `sudo apt install mysql-server`。输入`service mysql status`查看服务运行状态。

> tips: 通过apt安装的mysql不需要像其他平台那样开启服务，因为已经默认开启，并且服务名叫做mysql，而不是mysqld

### 登录
`sudo mysql` 进入到mysql命令行。

- 重置密码

重置 root 账户密码
`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';`

刷新权限
`FLUSH PRIVILEGES;`

- 远程登录用户
如果需要远程登陆：
- - 创建一个 host 字段为 % 的 root 用户（创建用户的同时设置密码）
- - 授权所有数据库的访问权限
- - 刷新权限列表

新建一个 host 为 % 的 mireux 用户，密码随意 
`create user 'mireux'@'%' identified by 'yourpassword';`

授权
`GRANT ALL PRIVILEGES ON *.* TO 'mireux'@'%' WITH GRANT OPTION;`

刷新权限
`FLUSH PRIVILEGES;`
 
> 可以通过如下的方式查看我们的用户信息情况
> mysql> use mysql
> mysql> select host,user,authentication_string from user;

### 其他问题

如果按照上述步骤依然无法远程连接，可能需要修改mysql相关的配置文件。
1. 关停mysql服务 `sudo systemctl stop mysql`
2. 编辑mysql配置文件 `sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf`
3. 注释掉`#bind-address = 127.0.0.1` 
4. 重启mysql服务 `sudo service mysql start`

## Java环境

### 安装

> 安装的脚本 要是有什么需要或者更好的建议可以交流 ( 我也是个初学者:-( )
> 全部都是基于apt的安装

```bash
#!/bin/bash
# 可能需要根据实际情况调整~/.bashrc里的路径 需要指向jdk所在的目录 修改后记得source
echo "开始检测是否已经安装jdk..."
javaPath=`sudo update-alternatives --list java`
if [ -e $javaPath ]; then
        echo "java的路径是:${javaPath:0:${#javaPath}-9}"
else
        echo 'jdk不存在 开始下载jdk17'
        sudo apt update
        sudo apt install openjdk-17-jdk
        sudo update-alternatives --list java
        echo "export JAVA_HOME=${javaPath:0:${#javaPath}-9}" >> ~/.bashrc
        source ~/.bashrc
fi
# 一定要确保已经安装了jdk并且已经配置了JAVA_HOME环境变量 不然mvn -version会报错
# 下载后不要忘记修改mvn的配置文件 setting.xml中的localRepository和镜像哦~
echo "开始检测是否已经安装maven..."
mvnPath=`which mvn`
if [ -e $mvnPath ]; then
        echo "maven已经存在,$(whereis maven)"
        mvn -version
else
        echo "maven不存在 开始下载"
        sudo apt-get update
        sudo apt-get -y install maven
        mvn -version
fi

echo "开始检测是否已经安装git..."
gitPath=`which git`
if [ -e $gitPath ]; then
        echo "git已经存在,$(whereis git)"
        git --version
else
        echo "git不存在 开始下载"
        sudo apt update
        sudo apt install git
        git --version
fi
```
---
搞定环境！ 接下来就可以去尝试部署你的程序咯~