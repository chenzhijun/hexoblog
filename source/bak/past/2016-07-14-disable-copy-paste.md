---
layout:     post
title:      "禁止复制粘贴网页内容"
subtitle:   "一个自我安慰的需求"
date:       2016-07-14 03:19:00
author:     "chenzhijun"
header-img: "img/post-bg-js-module.jpg"
catalog: true
tags:
    - HTML
    - JavaScript
---

##网页页面禁止复制粘贴

最近看到有些网站对页面有要求控制，比如在某些敏感的地方，页面的数据不能粘贴复制（这只不过是障眼法罢了）。下面先讲一下原理：

###页面内容禁止复制
其实很好想，要想复制内容，必须选中内容，然后进行copy。因此让用户无法用鼠标进行选中，其实就是获取焦点的时候，让它失效：
```
onfocus=this.blur();
```
###页面禁用右键
详情看代码。
###重新启用右键
详情看代码。

disable_copy_paste.js:

```js
//给需要禁止复制的input标签加入class="nocopy"
$(".nocopy").attr("onfocus", "this.blur()");
$("textarea.nocopy").attr("oncopy","return false").attr("oncut","return false").attr("onselectstart","return false");

// 屏蔽右键菜单
document.oncontextmenu = function(event) {
	if (window.event) {
		event = window.event;
	}
	try {
		var the = event.srcElement;
		if (!((the.tagName == "input" && the.type.toLowerCase() == "text") || the.tagName == "textarea")) {
			return false;
		}
		return true;
	} catch (e) {
		return false;
	}
}

// 屏蔽复制
document.oncopy = function(event) {
	if (window.event) {
		event = window.event;
	}
	try {
		var the = event.srcElement;
		if (!((the.tagName == "input" && the.type.toLowerCase() == "text") || the.tagName == "textarea")) {
			return false;
		}
		return true;
	} catch (e) {
		return false;
	}
}

// 屏蔽选中
document.onselectstart = function(event) {
	if (window.event) {
		event = window.event;
	}
	try {
		var the = event.srcElement;
		if (!((the.tagName == "input" && the.type.toLowerCase() == "text") || the.tagName == "textarea")) {
			return false;
		}
		return true;
	} catch (e) {
		return false;
	}
}
```
test.html

```

	<input type="text" value="value  1"/>
	<br>
	<input type="text" value="value 2" onfocus="this.blur();"/>
	<br>
	<input type="text" value="this is test value 3" readonly="true" onfocus="this.blur();"/><br>
	<input type="text" class="nocopy" value="value 4" ><br>
	<input type="text" class="nocopy" value="value 5" id="text" ><br>
	<textarea class="nocopy"></textarea>
```
revert.js

```javasript
$('input').each(function(i){
	$(this).removeAttr("readonly");
	$(this).removeClass("nocopy");
	$(this).removeAttr("onfocus");
});
$('textarea').each(function(i){
	$(this).removeClass("nocopy");
	$(this).removeAttr("readonly");
	$(this).removeAttr("onfocus");
	$(this).removeAttr("oncopy");
	$(this).removeAttr("oncut");
	$(this).removeAttr("onselectstart");
});
document.oncontextmenu=null;
document.oncopy=null;
document.onselectstart=null;
```