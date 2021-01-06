---
title: Nginx部署Hexo
date: 2021-01-06 15:13:48
tags: ['Hexo','Nginx']
categories: 工具教程
---
本文测试本地git push操作后，Nginx部署的网站是否更新。

<br />
<br />

说明：
1. 本文基于github pages搭建的博客，更换为自定义域名且支持https
2. Nginx服务器运行在腾讯云 window server 2019（nginx为1.18.0稳定版）
3. 静态页面保存在nginx-1.18.0/web/public目录下，配置的域名为 www.learnmap.website<span>（与github设置的域名相同）</span>



