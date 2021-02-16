---
title: Nginx部署Hexo并配置域名
date: 2021-01-06 15:13:48
tags: ['Hexo','Nginx']
categories: 工具教程
---
使用nginx部署博客网站，并在国外网站(cloudflare)配置域名解析。

<br />
<br />

# ###写在前面
1. 本文的内容全部基于《搭建个人博客网站》
2. 为确保网站运行正常，除了基于github外，又使用Nginx服务器部署，网站内容完全相同
3. Nginx服务器绑定的域名是www.example.com<span>，基于github绑定的域名是blog.example.com，并且都支持https（本文不涉及cloudflare如何申请https密钥和github如何设置使用https）</span>
4. 先前Nginx服务器运行在腾讯云 window server 2019（nginx为1.18.0稳定版），且阿里云购买的域名，域名解析也在阿里云，后来网站无法运行，提示需要域名备案
5. 域名备案过程中，提示“备案和云服务器提供商必须一致”。有两种方式解决：1）购买腾讯云服务器，平时服务器使用不多，不打算购买；2）将阿里云域名转入腾讯云，不过域名需强制续费一年，但我使用的是朋友的腾讯云，备案实名认证多少有些不方便。最后我选择在GoDaddy购买域名，cloudflare进行域名解析，与此同时，我有一台VPS服务器(购买的是伦敦的，ubuntu)
6. Nginx服务器运行在ubuntu，静态页面保存在根目录/web/public目录

<br />
<br />


# 一、Nginx配置

1、配置http网站

1）XShell远程连接ubuntu，Xftp可视化操作ubuntu文件夹，在根目录(/)新建文件夹web；

2）执行`yarn build`，生成的public文件夹保存到web目录下，静态页面保存的路径是/web/public

3）编辑/etc/nginx/nginx.conf文件：

```bash
server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        # root         /usr/share/nginx/html;
        root         /web/public;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
	        root	/web/public;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
<br />

2、配置https

1）在cloudflare上申请https 密钥（申请过程省略），即生成两个PEM密钥文件

2）在/etc/nginx路径下新建文件夹conf.d，再新建文件夹cert（即/etc/nginx/conf.d/cert），将生成的PEM文件放在该路径下

3）编辑/etc/nginx/nginx.conf文件：

```bash
# Settings for a TLS enabled server.
#
server {
    listen       443 ssl http2 default_server;
    listen       [::]:443 ssl http2 default_server;
    server_name  _;
    root         /web/public; #/usr/share/nginx/html;

    ssl_certificate "/etc/nginx/conf.d/cert/cert.pem"; #"/etc/pki/nginx/server.crt";
    ssl_certificate_key "/etc/nginx/conf.d/cert/key.pem"; #"/etc/pki/nginx/private/server.key";
    ssl_session_cache shared:SSL:1m;
    ssl_session_timeout  10m;
    ssl_ciphers PROFILE=SYSTEM;
    ssl_prefer_server_ciphers on;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    location / {
	    root	/web/public;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```

<!-- ————————————————————————————————————————————————————————————————————————————————
# 二、配置域名解析


![cloudflare配置域名解析](https://cdn.jsdelivr.net/gh/YuliaScott/blogpic/img/.png)
<br />
输入<span>www.example.com</span>或example.com或blog.example.com，可同时访问。 -->


————————————————————————————————————————————————————————————————————————————————
# 附：国外域名备案说明

四种组合：

国外域名+国外服务器，不需要备案；

国外域名+国内服务器，无法直接解析；

国内域名+国内服务器，需要；

国内域名+国外服务器，好像不太这么用；

<br />

（完）










