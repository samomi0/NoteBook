---
title: 简易docker使用手册
date: 2023-11-04
authors: ["Misaki"]
categories: ["Docker"]
---

<!-- more -->

## 拉取与推送镜像

### 拉取镜像

通常可以在[DockerHub](https://hub.docker.com/)这里搜索你想要的镜像

~~~shell
docker pull ubuntu:20.04
~~~
或者指定sha来拉取特定版本(例如armv8的版本)
~~~shell
docker pull ubuntu:20.04@sha256:0b897358ff6624825fb50d20ffb605ab0eaea77ced0adb8c6a4b756513dec6fc
~~~

### 推送镜像

一般情况下都会需要先登录到希望推送的仓库(示例registry.myharbor.com)
~~~shell
docker login -u user registry.myharbor.com
~~~

~~~shell
docker push registry.myharbor.com/ubuntu:focal
~~~

### 如何换源

也许会需要用到代理，或者干脆使用国内的源

*目前国内源（例如阿里云）基本上只支持少量的公共镜像下载，如果是本地机器，建议走代理下载。*

这里以华为云举例，打开docker的配置文件，如果是新安装的docker，这里可能没有这个文件，创建一个就可以。

~~~shell
sudo vim /etc/docker/daemon.json
~~~

填入我们想加入的源，例如华为云：`https://mirrors.huaweicloud.com/`，像这样：

~~~json
{
  "registry-mirrors": [ 
    "https://mirrors.huaweicloud.com/"
  ]
}
~~~

然后重启docker服务：

~~~shell
sudo systemctl daemon-reload
sudo systemctl restart docker
~~~

这样就完成了换源操作。

## 进入容器开发

### 启动容器

​	简单的示例：`docker run -ti --rm --net=host --name=test image:tag`

​	示例常见参数：

| 参数名       | 功能                                                         |
| ------------ | ------------------------------------------------------------ |
| `-ti`        | t和i两个参数通常一起使用，交互式启动容器                     |
| `--rm`       | 停止容器后自动删除，常见于开发时，避免旧容器堆积             |
| `--net=host` | 与-p重叠，使用宿主机网卡，映射全部端口                       |
| `-p`         | -p port:port，映射容器内端口到宿主机端口，可以批量指定       |
| `--name`     | 为容器设置名称，默认会使用一个hash值作为容器名               |
| `-v`         | 挂载外部目录到容器内，会覆盖容器内已存在的目录。好处是可以外部修改或即时备份，可以利用挂载实现一些动态效果 |

还有很多有用参数可见[Docker官方文档](https://docs.docker.com/engine/reference/commandline/run/)

### 挂载目录

大多数时候我们还是希望即时的修改工程代码

*条件允许的话我猜你不会喜欢在容器里用终端写代码的*

所以我们可以把工程挂载到容器里

~~~shell
docker run -ti -v /path/to/project/folder:/path/to/folder image:tag
~~~

例如这样我们把宿主机上的`/path/to/project/folder`目录挂载到了容器里的`/path/to/folder`目录，相当于映射关系。

### 限制资源或传递默认环境变量

有时候我们并不希望这个容器拿走机器上全部的资源，*尤其是做性能测试的时候*

~~~shell
docker run -ti --cpus=16 image:tag
~~~

- 使用`--cpus=16`限制容器只能最多用到`1600%`的CPU核心资源，这并不意味着一定是16个核心，也可能是32核心每个用一半

- 使用`--cpu-shares`选项可以设置 CPU 共享权重，权重的默认值是1024，每个启动的容器在他们需要的时候，都会按权重抢占可用的CPU资源

- 使用`--cpu-period`和`--cpu-quota`选项可以限制容器的 CPU 使用时间。

  例如`--cpu-period=100000 --cpu-quota=25000`表示在每 100ms 的时间周期内，容器最多只能使用 25ms 的 CPU 时间。

- 使用`-m`或`--memory`选项可以指定容器的最大内存使用量。

  例如`-m 512m`或是`-m 1G`

~~~shell
docker run -ti -e MY_ENV=my_env image:tag
~~~

使用参数`-e`就可以为容器传递一个默认的环境变量，当然，进入容器后仍然可以改变他们

## 保存与构建镜像

### 保存

docker save 可以将一个镜像打包成tar包，以便于你发布它
~~~shell
docker save -o name_of_image.tar image:tag
~~~

docker commit 可以将一个正在运行的容器保存成镜像，当然这会保存这个容器当前的所有状态，请不要在发布镜像时这么做喔
~~~shell
docker commit container_id image:tag
~~~

### 构建镜像

利用dockerfile实现镜像打包，详细的使用方式可见[官方文档](https://docs.docker.com/engine/reference/builder/)

build时可以选择上传的缓存目录，像这样`docker build -f dockerfile -t image:new_tag .`，最后的`.`即表示使用当前目录作为缓存上传。

镜像构建通常用于去掉一些不必要的layer，来尽可能减小镜像体积，并且将变动层置后，有利于镜像仓库更好的管理。[资料](https://docs.docker.com/build/guide/layers/)

在自动化构建中，还可以利用多次FROM实现隔离，只选取必要的内容添加到最终的镜像里。
