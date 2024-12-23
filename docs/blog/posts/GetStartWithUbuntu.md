---
title: 从零开始的Ubuntu之旅
authors: ["Misaki"]
date: 2023-10-28
categories: ["Ubuntu", "虚拟机"]
---

## 从虚拟机开始

这里我使用`VirtualBox`，原因嘛，主要是免费+轻便好用。你可以在[Oracle下载页](https://www.virtualbox.org/wiki/Downloads)下载到它。当然，[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/virtualbox/)也有提供的虚拟机软件的下载。

<!-- more -->

然后需要下载一个Ubuntu的ISO映像文件，这里我们以20.04(focal)版本为例，同样可以在[清华大学开源软件镜像站](https://mirrors.tuna.tsinghua.edu.cn/ubuntu-releases/)下载到。

然后就可以根据提示来安装虚拟机了。

>安装系统时可能会遇到分辨率过低导致页面内容显示不完整的情况，可以用`xrandr`命令来切换分辨率。例如：`xrandr -s 7`，这里7表示第8个分辨率选项。

安装好虚拟机后进入操作系统，准备工作就完成了。

>① VirtualBox可能出现全屏时分辨率与屏幕不吻合的情况。
>
>1. 关闭虚拟机，在`控制器：IDE`处导入增强功能驱动
>2. 开启虚拟机，在屏幕底部（或顶部）的小工具栏选择`设备`->`安装增强功能`
>3. 可能需要重启
>
>② VirtualBox可能出现开启双向粘贴板/拖放但实际上失效的情况。
>
>1. 关闭虚拟机，在管理器选择`设置`->`存储`->`控制器：SATA`，勾选右侧的`使用主机输入输出(I/O)缓存`
>2. 选择`控制器：SATA`下方硬盘，勾选右侧的`固态驱动器`
>3. 在`控制器：IDE`处导入增强功能驱动
>4. 开启虚拟机，在文件系统打开驱动目录，运行可执行文件重新安装增强功能
>5. 可能需要重启
>6. *从虚拟机向宿主机拖放可能失败，暂无较好的解决办法*

## 从Docker开始

如果宿主机是Linux系统，那直接安装docker并拉取ubuntu镜像就可以了。

~~~shell
sudo docker pull ubuntu:latest
~~~

如果是其他系统，需要先安装docker desktop，你可以从[Docker官网](https://www.docker.com/products/docker-desktop/)下载到它。

安装好之后就可以搜索并拉取想要的镜像了。当然，也可以从[Docker Hub](https://hub.docker.com/_/ubuntu/tags)找你想要的镜像版本拉取。

*如何使用镜像？可以参考[简易Docker使用手册](https://samomi0.github.io/docs/简易docker使用手册/)。*

## 更换apt的下载源
现在我们有了一个最小的ubuntu系统，在一切工作之前，因为国内的特殊性，我们最好先解决下网络问题（源和代理）。

从[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)获取到ubuntu的apt源，并更新系统配置。

***注意页面上的提示**，ARM等版本需要使用[Ubuntu Ports 软件仓库](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/)*

>这里建议把选项`是否使用 HTTPS`取消勾选，即选择否，避免证书可能带来的问题。更换对应ubuntu版本选项后，就可以复制粘贴了。这里示例ubuntu20.04的源。
>
>~~~
>deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal main restricted universe multiverse
>deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-updates main restricted universe multiverse
>deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ focal-backports main restricted universe multiverse
>deb http://security.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
>~~~
>

使用`sudo vi /etc/apt/source.list`并把源写入，然后执行`sudo apt update`更新配置。

如果此时没有vi工具，尤其是官方提供的最小镜像，可以用echo命令追加写入文件的形式来操作，像这样：

~~~shell
echo "xxxxxx" >> /etc/apt/sources.list
~~~

**覆盖文件前记得备份！**

## 基础工具

这里会安装`vim`、`docker`、`curl`、`git`

~~~
sudo apt update && sudo apt install vim curl git ca-certificates curl gnupg lsb-release docker.io -y
~~~

如果docker安装有异常，也可以尝试下面的方式

~~~shell
sudo apt-add-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update && sudo apt install -y docker-ce
~~~

docker因为sudo权限的关系，平时使用可以将用户加入docker用户组，这里可能需要重启生效。

~~~shell
newgrp docker
sudo groupadd docker
sudo usermod -aG docker <username>
~~~

____

好了，现在你有了一个最基础的Ubuntu20.04开发环境了，享受代码的乐趣吧。以下是一些推荐的内容，以便于你有更好的开发体验。


## 更好的终端体验 —— zsh
接下来安装zsh + oh-my-zsh + p10k + 插件

~~~shell
sudo apt update && apt-get install zsh -y
wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh
sudo chmod +x install.sh
~~~

将`install.sh`中REPO和REMOTE

~~~
# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
~~~

替换为：

~~~
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
~~~

然后执行脚本，就会自动下载oh my zsh了，下载好之后会开始初始化。

然后我们下载p10k主题和插件：

~~~shell
git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

git clone https://gitee.com/phpxxo/zsh-autosuggestions.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://gitee.com/mirror-hub/zsh-syntax-highlighting $ZSH_CUSTOM/plugins/zsh-syntax-highlighting
~~~

在`.zshrc`（通常是home目录）中修改theme和添加插件：

~~~
ZSH_THEME="powerlevel10k/powerlevel10k"

plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
~~~

## Proxy

### for Docker

  ```shell
  sudo mkdir -p /etc/systemd/system/docker.service.d
  sudo touch /etc/systemd/system/docker.service.d/proxy.conf
  ```

  在这个`proxy.conf`文件（可以是任意`*.conf`的形式）中，添加以下内容：

  *这里`http://proxy.example.com:8080/`是自己的代理转发地址*

  ```
  [Service]
  Environment="HTTP_PROXY=http://proxy.example.com:8080/"
  Environment="HTTPS_PROXY=http://proxy.example.com:8080/"
  Environment="NO_PROXY=localhost,127.0.0.1,.example.com"
  ```
  
  接下来重启docker服务就可以了`sudo systemctl restart docker`

### for Terminal

  *如果你有Clash或者其他的工具，也许不需要这个*

  ~~~shell
  export http_proxy=xxx.xxx.xxx.xxx
  export https_proxy=xxx.xxx.xxx.xxx
  ~~~

  当然，也可以把它写进`.bashrc`里，让他开机就启动

  建议配合浏览器的`SwitchyOmega`插件实现页面的自动代理模式。

### for Git

  ~~~shell
  git config --global http.proxy http://proxy.example.com:8080/
  git config --global https.proxy https://proxy.example.com:8080/
  ~~~

## 更好的code体验

比如下载vscode，国内下载可以替换域名为：`vscode.cdn.azure.cn`

或者配置一个更棒的vim环境……

