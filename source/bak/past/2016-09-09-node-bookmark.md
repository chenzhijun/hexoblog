---
layout:     post
title:      "Node.js 笔记"
subtitle:   "个人笔记"
date:       2016-09-09 02:08:00
author:     "chenzhijun"
header-img: "img/post-bg-js-module.jpg"
catalog: true
tags:
    - JavaScript
---

## Node.js 初体验
==========================

***这只是个人笔记***

## 简单介绍
Node.js是一个开放源代码、跨平台的、可用于服务器端和网络应用的运行环境。

## 实例讲解
我还是觉得 everything 最好是有实例，然后讲解实例，才能学得快。

* 简单实例

```javaScript
'use strict'

console.log('Hello,world.');

```
从hello.js中可以看出，node使用的是strict模式。现在的js文件中也推荐使用 strict 模式。其实际语法和javaScript语法一致。可以将JavaScript的语法直接写过来，前台工程师很快可以上手。

* 内置对象

所有的浏览器都会有 window 对象，node.js 也提供了很多内置对象给我们使用.可以在node环境中输入 global 来查询。

* 模块

node不但有自己的内置模块，也允许用户创建自己的自定义模块。这个可以非常爽的功能。从此妈妈再也不用担心我会变量覆盖或者重名了。因为模块的所有变量相当于局部变量。下面举出一个模块实例：
module：hello.js

```javaScript
'use strict'

//console.log('Hello,world.');
var s='Hello';

function greet(name){
	console.log(s + ','+ name+';');
}

module.exports=greet;
```

引入module：main.js

```javaScript
'use strict'

//引入hello模块
var greet=require('./hello');

var s='chenzhijun';

greet(s); // Hello,chenzhijun;

```

主要看到上面的 ```module.exprots=greet``` 这里是导出一个自定义模块，并且可以模块名为greet。
在需要引入模块的 main.js 中，```var greet=require('./hello');``` 这里的require需要使用模块的地址。

* 内置模块-filestream

首先我们思考一下：文件操作有哪些？文件读-read，文件写-write，异步读取，同步读取，返回值，读取文件的类型。这些应该是filestream应该都涵盖的基础范围。node当然也提供了这些：

fs_easy.js：简单的文件读取操作，err表示读取状态结果，data表示读取之后的数据。如果读取成功，err应该为 null 。

```javaScript
'user strict'

var fs=require('fs');

fs.readFile('test.txt','utf-8',function(err,data){//test.txt 必须同一级目录
	if(err){
		console.log(err);
	}else{
		console.log(data);
	}
});
```

fs_file_type.js:读取非字符文件-转换成二进制文件读取

```javaScript
'use strict'

var fs=require('fs');

fs.readFile('test.png',function(err,data){
	var text=data.toString('utf-8');
	console.log("text: "+text);

	var buf=new Buffer(text,'utf-8');
	console.log("buffer:"+buf);

	if(err){//正确的时候err为null，data有值，错误的时候err为一个错误对象，data为undefined
		console.log(err);
	}else{
		console.log(data);
		console.log(data.length+'bytes');
	}
});
```
fs_sync.js:同步方法读取文件。

```javaScript
'use strict'
try{
	var fs=require('fs');

	var data=fs.readFileSync('test.txt','utf-8');

	console.log(data);
}catch(err){
	//出错了
}
```
fs_write.js:写文件

```javaScript
'use strict'
var fs=require('fs');
var data='Hello,Node.js';
fs.writeFile('output.txt',data,function(err){
	if(err){
		console.log(err);
	}else{
		console.log('ok.');
	}
});
```

fs_write_sync.js:同步写文件

```javaScript
'use strict'
var fs=require('fs');
var data='Hell,Node.js';
fs.writeFileSync('output_sync.txt',data);

```
* http
一款后台软件怎么可以没有http呢？node也有相应的http模块来对应。
server.js:简单的请求与响应

```javascript
'use strict'

//import http module
var http=require('http');

//create http server, callback function as the param
var server = http.createServer(function(request,response){
	// receive the response and request object
	// achieve the http request's method and url
	console.log(request.method+' : '+request.url);

	//set the http 200 into the response, and set Content-Type:text/html:
	response.writeHead(200,{'Content-Type':'text/html'});

	//set html content into http response；
	response.end('<h1>Hello world!</h1>');
}); 

//make the server listen the port 2314
server.listen(2314);

console.log('Server is runing at http://127.0.0.1:2314/');
```

server_file.js: 请求的响应，如果有文件返回文件，如果为 / ，转换为index.css

```
'use strict'

var fs = require('fs'),
    url = require('url'),
    path = require('path'),
    http = require('http');

// from the command get the root dir, the defult is current dir;
var root = path.resolve(process.argv[2] || '.');

console.log('Static root dir: '+root);

//create web server
var server = http.createServer(function (request,response){
	//get the url's path, eg: '/css/bootstrap.css';
	var pathname = url.parse(request.url).pathname;
	console.log('pathname:'+pathname);
	
	//get the local address, eg:'/src/www/css/bootstrap.csss';
	var filepath = path.join(root,pathname);
	console.log('filepath:'+filepath);

	if(pathname==='/'){
		response.writeHead(200);
		//send the file stream to the response
		fs.createReadStream('./index.css').pipe(response);
		//response.end();
		return;
	}
	
	//get the file satus
	fs.stat(filepath,function(err,stats){
		if(!err && stats.isFile()){
			//no error, the file is exist
			console.log('200 '+ request.url);
			
			//send 200 response
			response.writeHead(200);

			//send the file stream to the response
			fs.createReadStream(filepath).pipe(response);
		}else{
			//error ,or the file is not exist
			console.log('404 '+request.url);
			
			//send 404 response
			response.writeHead(404);
			response.end('404 NOT Found');
		}
	});
});

server.listen(8080);

console.log('Server is running at http://127.0.0.1:8080/');
//http://127.0.0.1:8080/index.html   是否有index.html页面看返回值
//http://127.0.0.1:8080/   返回index.css 如果没有当前文件This webpage is not available

ERR_CONNECTION_REFUSED报错
```






 
