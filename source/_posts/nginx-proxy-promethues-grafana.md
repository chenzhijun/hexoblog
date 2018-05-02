---
title: nginx 设置prometheus和grafana的反向代理
copyright: true
date: 2018-05-02 22:57:50
tags: ["nginx","prometheus","grafana"]
categories: monitor
---

# nginx 设置promethues和grafana的反向代理

在配置完promethues，和grafana之后，可能需要上生产环境，这个时候如果有下面两种情况，那么就可能需要用到代理；
1. 端口只开发80,或者8080等特别的几个端口，端口数量有限；
2. 不希望暴露给外部端口号，使用子路经来区分；eg: http://{ip}/prometheus,http://{ip}/grafana

nginx 配置：

```conf
server {
	listen 80;

	server_name example.com;

	root /var/www/example.com;
	index index.html;

	location / {
		try_files $uri $uri/ =404;
	}
}

```