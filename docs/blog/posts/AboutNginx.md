---
title: nginx的配置
date: 2024-07-14
authors: ["Misaki"]
categories: ["Nginx"]
---

<!-- more -->

## 安装

~~~sh
sudo apt update && apt install nginx                # ubuntu

sudo yum install epel-release && yum install nginx  # centos
~~~

## 启动

~~~sh
sudo systemctl start nginx

sudo systemctl stop nginx

sudo systemctl restart nginx

sudo systemctl status nginx
~~~

## 配置

默认配置文件位于 `/etc/nginx/nginx.conf`

### 1. 配置 HTTP 服务器

```
http {
    include       mime.types;
    default_type  application/octet-stream;

    server {
        listen       80;
        server_name  your_domain_or_IP;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

        error_page  404              /404.html;
        location = /404.html {
            root   /usr/share/nginx/html;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

以上配置会在服务器的根目录（`/usr/share/nginx/html`）下查找 `index.html` 或 `index.htm` 文件并返回。

### 2. 配置反向代理

在 `http` 块中添加一个新的服务器块：

```
http {
    include       mime.types;
    default_type  application/octet-stream;

    server {
        listen       80;
        server_name  your_domain_or_IP;

        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        error_page  404              /404.html;
        location = /404.html {
            root   /usr/share/nginx/html;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```

以上配置会将所有请求代理到本地的 8080 端口。

### 3. 配置 HTTPS

要配置 HTTPS，你需要拥有 SSL 证书。可以通过 `certbot` 获取免费的 Let’s Encrypt 证书。

首先，安装 Certbot：

```sh
sudo apt install certbot python3-certbot-nginx
```

然后，获取证书：

```sh
sudo certbot --nginx -d your_domain
```

这会自动为你的域名配置 SSL，并修改 Nginx 配置文件以使用 HTTPS。

### 4. 跨域问题

~~~
http {
    # 其他配置...

    server {
        # 其他配置...

        location / {
            # 其他配置...
            add_header 'Access-Control-Allow-Origin' '*';
            add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            add_header 'Access-Control-Allow-Headers' 'Origin, Content-Type, Accept, Authorization';
        }
    }
}
~~~