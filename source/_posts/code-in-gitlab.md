---
title: gitlab 安装与配置
copyright: true
date: 2018-03-30 23:28:38
tags: gitlab
categories: git
---

## 使用 Docker 部署 gitlab

首先需要安装 Docker。推荐使用docker官网的安装方式：[Centos 安装docker](https://docs.docker.com/install/linux/docker-ce/centos/)，如果访问docker官网过于太慢，可以使用DaoCloud下载docker：[daocloud 安装docker](https://download.daocloud.io/Docker_Mirror/Docker)。有时候下载镜像，有一些比较显而易见的原因，所以可以使用[docker镜像加速器](https://www.daocloud.io/mirror#accelerator-doc)。

安装docker之后，下载镜像`gitlab/gitlab-ce:latest`，之后参照[安装文档](https://docs.gitlab.com/omnibus/docker/README.html#run-the-image)。

```shell
sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 --publish 22:22 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest
```
一个gitlab就搭建好了。默认的帐号为root。

### 配置邮件服务

在gitlab的配置目录有个文件：`/etc/gitlab/gitlab.rb`，编辑这个文件。

```shell
sudo docker exec -it gitlab vi /etc/gitlab/gitlab.rb
```

安装邮件的文档：[配置smtp]](https://docs.gitlab.com/omnibus/settings/smtp.html#doc-nav)

```properties
gitlab_rails['smtp_enable'] = true;
gitlab_rails['smtp_address'] = 'localhost';
gitlab_rails['smtp_port'] = 25;
gitlab_rails['smtp_domain'] = 'localhost';
gitlab_rails['smtp_tls'] = false;
gitlab_rails['smtp_openssl_verify_mode'] = 'none'
gitlab_rails['smtp_enable_starttls_auto'] = false
gitlab_rails['smtp_ssl'] = false
gitlab_rails['smtp_force_ssl'] = false
gitlab_rails['smtp_authentication'] = "login"
```


<!-->
gitlab 导入 svn

git 命令行修改登录账户和密码
<-->

