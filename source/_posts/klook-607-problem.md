---
title: klook-607-problem
date: 2017-06-07 23:50:46
tags:
	- Question
categories: Question

---


> 年度问题日结,将今年每天遇到的问题做一个记录: <月.日 问题>

## 6.7 问题

1. Hashmap寻找的时间复杂度

	hashmap的实现是"数组+链表"的形式，
	```
	1. 先根据key值计算出hash值以及h值（h值是java实现中处理得到的更优的index索引值）
	2. 查找table数组中的h位置，得到相应的键值对链表
	3.	根据key值，遍历键值对链表，找到相应的键值对，
	4. 从键值对中取出value值。
	```

2. Golang 反射将数字转成float64

	`value.(float64)`

3. Golang map判断是否有键值

	if val,ok:=map[key];ok{
		// 有值
	}


<!--more-->

## 5.25 问题

1. 数据库无记录时插入，有记录时（设置的主键重复）更新

	`insert into city_th_tbl (id,name_col) values (8,'名字') on duplicate key update name_col='名字1';` ，当id=8插入的时候有重复的值的时候，将`name_col`更新为`名字1`，只针对于MySQL。

2. 将`docker info`之后显示的data space 修改大小`truncate -s 200G /var/lib/docker/devicemapper/devicemapper/data`

3. 一键停止所有正在运行的docker容器,`docker stop $(docker ps -a -q)` ,开启：`docker start $(docker ps -a -q) `

4. `grep “查的内容” “文件位置” -C 10` ; 使用grep查找的时候把匹配内容的上下10行也显示出来

<!--more-->


## 5.18 问题
1. 查看linux centos 某个软件是否安装

	答案:`rpm -qa | grep software`

2. 启动某个软件

	答案: 设置开机启动`sudo systemctl enable docker`，设置立即启动`sudo systemctl start docker`
3. 查看系统内核信息 `uname -a`
4. ubuntu安装软件出错，“Ubuntu unable to locate package”

	答案 `先更新 apt-get update ;
		 之后 apt-get upgrade`
5. vim 替换

	答案：`:s/old/new/g`,替换一行的所有old为new.`:`命令模式，`s`是指本行，`g`是指所有

6. 查看磁盘大小

	答案：`df -lh`,`fdisk -l`
7. virtual box 安装Ubuntu，之后使用本地命令行登录。

	答案： Ubuntu 装一个openssh-server . vb里面加一个hostonly-adapter. 之后将nat 端口转发

8. 查看当前电脑时区

	答案：`date`

9. 使用nginx ,实现多个tomcat 负载均衡。

	[原文地址](http://www.cnblogs.com/fengzheng/p/4995513.html);参考了这个博客，我也实现了。 不过我做了一些小改动。在tomcat上我直接`docker pull tomcat`镜像，之后根据tomcat镜像生成的容器，来做的实现。没有那么麻烦。nginx 主要在http模块增加了配置:

	```
	server{
		listen 80;
		server_name localhost;
		location / {
			proxy_pass http://blance;
		}
	}

	upstream blance{
		server localhost:3280 weight=5;
		server localhost:3380 weight=5;
	}

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;```

<!--more-->

## 5.17 问题
1. 怎么给mysql 唯一key 取名

 	答: index,key在MySQL指代的是同一个东西，不过在key有一个unique key 它会限制多行的该列不会有相同值。

 	```
 	-- 建表语句

 	create table language_tbl(
  		id int(11) unsigned NOT NULL AUTO_INCREMENT COMMENT '语言的数字id',
  		language_code varchar(20) NOT NULL COMMENT '语言显示名:en_US;zh_CN;zh_TW;ko_KR;th_TH',
   		create_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  		last_modify_time datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP ,
  		flag int(1) NOT NULL DEFAULT '0' COMMENT '',
  		PRIMARY KEY (id),unique key `idx_language_code_flag` (language_code,flag)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
 	```
可以看到我们自定义的unique key 名为`idx_language_code_flag `,可以用`show index from language_tbl;`查看一下表的index。另外如果表已经建立好:`create index idx_language_code on language_tbl(language_code);`，其中idx_langauge_code是自定义的名字。

2. 删除表

	答:`drop table  if EXISTS language_tbl;` 删除表,包括表数据。
		`delete from language_tbl;`删除表的所有已有数据。

3. 当访问一个URL时报404，nginx/1.10.2，这种情况查看 nginx 日志判断看是否Nginx访问到了。
   答: 查看日志，第一步要确定我们要查看的那个日志在那个位置，进入到`/etc/nginx`目录使用`grep`，找到log目录。

   ```
   grep 'log' nginx.conf
error_log  /var/log/nginx/error.log warn;
log_format  main  '$remote_addr - $remote_user [$time_local] $request_time "$request" '
access_log  /var/log/nginx/access.log  main;
   ```
<!--more-->

## 5.15 问题
1. Mysql 使用`group by`是否可以将多行中同列的不同值合并成一条

	答: 主要使用group_concat()函数，可以将同组id相同的值合并到一个值当中

	```
	SELECT genenic_id,
	       group_concat(LANGUAGE)
	FROM genenic_language_tbl
	GROUP BY genenic_id;

	SELECT genenic_id,
	       group_concat(LANGUAGE separator ';')
	FROM genenic_language_tbl
	GROUP BY genenic_id;                                

	SELECT genenic_id,
	       group_concat(LANGUAGE
	                    ORDER BY id DESC)
	FROM genenic_language_tbl
	GROUP BY genenic_id;                                

	SELECT genenic_id,
	       group_concat(DISTINCT LANGUAGE)
	FROM genenic_language_tbl
	GROUP BY genenic_id;
	```

	<!--more-->

2. 一条SQL更新两个表

	答: `update tbl_name1 a, tbl_name2 b set a.xx='',b.xx='';`这样有个不好的地方。这两个表的所有字段都会更新。所以这种情况的使用场景是两个表有唯一主键关联，一定要仔细检查。

	```
	begin;
update city_ch_tbl a,city_en_tbl b
	set a.desc_col = 'adsf',b.desc_col='testsss2'
where a.id = 2 or b.id=21231;
rollback;
commit;
update city_tbl a, city_detail_tbl b
set a.displayType_col=1 ,b.publish_status=?, 	a.publish_time=current_timestamp
where a.id =b.cityId_col and a.id=?

	```


3. redis Key 加密登录  

	答:如果redis配置了密码，修改了redis.cnf 的`requirepass chen`；

	```
	src/redis-server redis.conf
	auth password chen
	```

## 5.13 问题

1. mysql 替换数据库的某些值

	答案: 使用MySQL的`replace()`函数
	`update city_en_tbl set desc_col = REPLACE(desc_col, 'ne', 'ne1') where id = 3;`

2. mysql 字符相加

	答: 使用MySQL的`CONCAT(str1, str2, str3, ...)`



## 5.12 问题

1. mysql 增加唯一键
	`alter table tbl_name add unique key(col_name)`
	`alter table city_en_tbl add unique key(cityId_col)`


2. nginx 反向代理

3. robots.txt是啥？

	谷歌爬虫会根据robots的文件内容确定哪些爬，哪些不爬，可以减少网站的带宽。
4. Linux 查看软连接

	Linux增加软连接为`ln -s afile bfile`,查看为`ls -il`
5. nginx site-ennable


	## 5.10 问题

1. Curl 命令

	`curl [option] [url]`,平常使用最多的是前后端分离后，我们使用post，get请求某个接口。

```
	-- 头信息
	curl -H "Token:aadfdwd29b568918db" http://www.baidu.com/test/schedule_rule/autoextend
	-- 一个比较全面的例子 post请求
	curl -X POST \
 http://mywebsite.com/project/url/query \
 -H 'cache-control: no-cache'\
 -H 'content-type: application/json'\
 -H 'token: ce15-fba-ead1-e9d2-2a03041c9'\
 -d '{
       “id”: “22,23,76,12",
       “language”: “”

   }'
```
