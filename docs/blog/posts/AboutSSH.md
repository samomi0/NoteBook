---
title: SSH的配置
authors: ["Misaki"]
date: 2024-07-14
categories: ["SSH"]
---

<!-- more -->

## 安装

~~~sh
sudo apt-get install openssh-server  # Debian/Ubuntu
sudo yum install openssh-server      # CentOS/RHEL
~~~

启动
~~~sh
sudo systemctl start ssh
sudo systemctl enable ssh

sudo systemctl status ssh       # 查看状态

sudo systemctl restart ssh      # 重启
~~~

## 配置

配置文件存放在`/etc/ssh`下，一般是`sshd_config`文件

~~~
#Port 22
# SSH服务监听的端口号，默认是22。可以更改为其他端口以增强安全性。

#AddressFamily any
# 指定SSH监听的地址族，可以是 "inet" (IPv4) 或 "inet6" (IPv6) 或 "any" (默认)。

#ListenAddress 0.0.0.0
#ListenAddress ::
# 指定SSH服务监听的地址，可以配置特定的IP地址或使用默认的所有地址。

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key
# SSH服务使用的主机密钥文件，用于不同的加密算法（RSA、ECDSA、Ed25519）。

# Ciphers and keying
#RekeyLimit default none
# 设置重新加密数据传输的频率。可以指定数据量和时间限制。

# Logging
#SyslogFacility AUTH
# 设置日志的设施，默认是AUTH。

#LogLevel INFO
# 设置日志的详细级别，可以是QUIET、FATAL、ERROR、INFO、VERBOSE、DEBUG1、DEBUG2、DEBUG3。

# Authentication:

#LoginGraceTime 2m
# 登录宽限时间，默认是2分钟。在这个时间内如果用户未能成功登录，连接将被关闭。

#PermitRootLogin prohibit-password
# 是否允许root用户登录，默认是 "prohibit-password"（禁止使用密码登录）。

#StrictModes yes
# 检查用户主目录和SSH相关文件的权限设置，默认是启用的。

#MaxAuthTries 6
# 允许的最大认证尝试次数，默认是6次。

#MaxSessions 10
# 每个网络连接允许的最大并发会话数，默认是10。

#PubkeyAuthentication yes
# 启用公钥认证，默认是启用的。

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
# 指定存储公钥的文件，默认是 ".ssh/authorized_keys"。

#AuthorizedPrincipalsFile none
# 指定存储授权主体的文件，默认是没有。

#AuthorizedKeysCommand none
# 指定获取公钥的命令，默认是没有。

#AuthorizedKeysCommandUser nobody
# 指定运行获取公钥命令的用户，默认是 "nobody"。

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# 是否启用基于主机的认证，默认是禁用的。

# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# 是否忽略用户的 "known_hosts" 文件，默认是不忽略。

# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes
# 是否忽略用户的 ".rhosts" 和 ".shosts" 文件，默认是忽略。

# To disable tunneled clear text passwords, change to no here!
#PasswordAuthentication yes
# 是否允许密码认证，默认是启用的。为了安全，可以设置为 "no"。

#PermitEmptyPasswords no
# 是否允许空密码，默认是不允许。

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no
# 是否启用挑战-响应密码认证，默认是禁用的。

# Kerberos options
#KerberosAuthentication no
# 是否启用Kerberos认证，默认是禁用的。

#KerberosOrLocalPasswd yes
# 是否允许使用Kerberos或者本地密码进行认证，默认是启用的。

#KerberosTicketCleanup yes
# 是否在用户注销时清除Kerberos票据，默认是启用的。

#KerberosGetAFSToken no
# 是否在认证过程中获取AFS令牌，默认是禁用的。

# GSSAPI options
#GSSAPIAuthentication no
# 是否启用GSSAPI认证，默认是禁用的。

#GSSAPICleanupCredentials yes
# 是否在用户注销时清除GSSAPI凭证，默认是启用的。

#GSSAPIStrictAcceptorCheck yes
# 是否启用GSSAPI严格的接受者检查，默认是启用的。

#GSSAPIKeyExchange no
# 是否启用GSSAPI密钥交换，默认是禁用的。

# AllowAgentForwarding yes
# 允许代理转发，默认是启用的。

# AllowTcpForwarding yes
# 允许TCP转发，默认是启用的。

# GatewayPorts no
# 是否允许远程主机连接转发端口，默认是不允许的。

X11Forwarding yes
# 允许X11转发，用于图形化界面应用的远程使用。

# X11DisplayOffset 10
# X11显示偏移，默认值是10。

# X11UseLocalhost yes
# 仅允许本地回环地址连接到X11服务器。

# PermitTTY yes
# 允许TTY（终端）分配，默认是启用的。

PrintMotd no
# 登录时不显示系统消息（Message of the Day），可以减少噪音信息。

# PrintLastLog yes
# 登录时显示上次登录信息，默认是启用的。

# TCPKeepAlive yes
# 允许TCP连接保持活跃，默认是启用的。

# PermitUserEnvironment no
# 是否允许用户自定义环境变量，默认是不允许的。

# Compression delayed
# 启用压缩，但延迟直到用户认证完成。

# ClientAliveInterval 0
# 设置服务器向客户端发送保持活动消息的时间间隔，默认是0（禁用）。

# ClientAliveCountMax 3
# 设置服务器发送保持活动消息的最大次数，如果客户端没有响应，连接将被关闭。

# UseDNS no
# 是否使用DNS反向解析客户端的主机名，默认是启用的。

# PidFile /var/run/sshd.pid
# SSH守护进程的PID文件路径。

# MaxStartups 10:30:100
# 设置同时进行的未认证连接的最大数量。

# PermitTunnel no
# 是否允许通过SSH创建网络隧道，默认是不允许的。

# ChrootDirectory none
# 设置用户的Chroot目录，默认是没有。

# VersionAddendum none
# 添加自定义的版本字符串，默认是没有。

# no default banner path
# Banner none
# 不显示登录横幅信息。

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*
# 允许客户端传递语言环境变量（locale）。

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server
# 设置SFTP子系统的路径。
~~~

## 正向与反向连接

指定远程连接服务器的端口(1234)
~~~sh
ssh -p 1234 user@10.1.10.10
~~~

将本地的端口(4321)映射到远程服务器的端口(1234)
~~~sh
ssh -NCR 1234:127.0.0.1:4321 user@10.1.10.10
~~~

这里参数`-R`即反向隧道；

`-N`表示不会在远程主机上运行任何命令，只建立隧道而不打开远程 shell；

`-C`表示在传输过程中压缩数据，在带宽有限的网络上也许能提高传输效率。

## 基于ssh用scp来传输文件

~~~sh
scp -P 1234 file.tar.gz user@10.1.10.10:/home/user/
~~~

表示利用端口(1234)建立连接，并将本地的文件传输到目标服务器的`/home/user/`目录下