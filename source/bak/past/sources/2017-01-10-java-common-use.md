---
layout:     post
title:      "开发总结"
subtitle:   "通用开发"
date:       2017-01-10 00:28:00
author:     "zhijun.chen/饺子"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 通用开发
---

> “这里开始，每天一篇”

## JSP页面常用到的代码总结

### 获取返回的 json 对象数据

```
 for(var key in data){
 		//遍历 json 对象获取数据
       console.log("key:"+key+",data:"+data[key]);           
 }
```
### js 代码片段
发现一个代码片段，总觉得有种很享受的感觉

```
       window.dom = document;

+function(){//静态参数
    var static_data = {
        //js常量
        event: 'click', //事件
        true: 'true',
        //接口
        IF_validatePasswordExist: '../validation',//验证密码是否存在
        
        //地址
        url_verify: 'view/verify',

        //常量
        card_type: 'CREDITCARD',

    };
    window.sd = static_data;
}();


var Fn = Object.create(null, {
    citiesHtml: {
        value: function(obj){
            var city = dom.querySelector('#city'),text = '';
            text += '<option value="" selected>选择城市</option>';
            for(var key in obj){
                text += '<option value="'+obj[key]+'" data-value="'+key+'">'+obj[key]+'</option>';
            }
            return text;
        }
    },
    getCity: {//获取城市信息
        value: function(provinceCode){
            Fn.ajax('getCityMap','GET',{province: provinceCode},function(response){
                dom.querySelector('#city').innerHTML = Fn.citiesHtml(response);
            });
        }
    },
    getInitCity: {//获取初始城市信息
        value: function(provinceCode){
            Fn.ajax('getCityMap','GET',{province: provinceCode},function(response){
                var city = dom.querySelector('#city');
                city.innerHTML = Fn.citiesHtml(response);
                function initCity(cityName){
                    city.querySelector('option[value="'+cityName+'"]').selected = true;
                }
                if(city.getAttribute('data-init')){
                    initCity(city.getAttribute('data-init'));
                }else if(sessionStorage.getItem('city')){
                    initCity(sessionStorage.getItem('city'));
                }

            });
        }
    },
    getInitBankBranch: {//获取初始支行信息
        value: function(bankCode){
        	Fn.ajax('queryHeadBank','GET',{bankCode: bankCode},function(response){
                var bankBranchName = dom.querySelector('#bankBranchName');
                branchLst = response;
                branchLstDefualt = response;
                function initCity(bankBranchNameval){
        			var branchSearchResult = [];
        	        for(var i = 0;i < branchLstDefualt.length;i++){
        	        	if(branchLstDefualt[i].bankName==bankBranchNameval){
        	        		var branchObject = new Object();
        	        		branchObject.bankCode = branchLstDefualt[i].bankCode;
        	        		branchObject.bankName = branchLstDefualt[i].bankName;
        	        		branchObject.errorCode = branchLstDefualt[i].errorCode;
        	        		branchSearchResult.push(branchObject);
        	        		break;
        	        	}
        	        }
        	        branchLst = branchSearchResult;
        	        Fn.bankBranchHtml(branchLst,1);
        	        $("#branchNameId").find("#chosenSearch").append($("#hidden_div").html());
                	bankBranchName.querySelector('option[value="'+bankBranchNameval+'"]').selected = true;
                	$("#bankBranchName").trigger("chosen:updated");
                	$("#branchNameId").find("#searchKey").val(bankBranchNameval);
                }
                if(bankBranchName.getAttribute('data-init')){
                    initCity(bankBranchName.getAttribute('data-init'));
                }else if(sessionStorage.getItem('bankBranchName')){
                    initCity(sessionStorage.getItem('bankBranchName'))
                }

            });
        }
    },
    bankBranchHtml:{
        value: function(response,num){
        	     page=num;
        	     $("#bankBranchName").html("");
        	     var text = '<option value="">请选择支行</option>';
                 if(response.length<=0||num<1){
                	 $("#bankBranchName").html(text);
                	 $("#bankBranchName").trigger("chosen:updated");
                	 $("#pageNum").html("&nbsp;&nbsp;0/0&nbsp;&nbsp;共0条记录");
                	 return;
                 }
                 var max = num*10;
                 if(response.length<num*10){
                	 max = response.length;
                 }
                 for(var i=(num-1)*10;i<max;i++){
                     var bank=response[i];
                     text +='<option value="'+bank.bankName+'">'+bank.bankName+'</option>';
                 }
                 $("#bankBranchName").append(text);
                 $("#bankBranchName").trigger("chosen:updated");
        		 $("#pageNum").html("&nbsp;&nbsp;"+num+"/"+Math.ceil(response.length/10)+"&nbsp;&nbsp;"+"共"+response.length+"条记录");
        }
     },
    ajax: {
        value: function (url,method, data, success, error, options) {
            $.ajax({
                timeout: 30000,
                url: url,
                method: method,
                data: data == null ? {} : data,
                dataType: "JSON",
                crossDomain: true,
                beforeSend: function (request) {
                    console.log('与服务器创建连接。发送数据：');
                    console.log(data);
                },
                success: function (result, status, xhr) {
                    if (console) {
                        console.log("访问接口[" + url + "]成功,结果如下:");
                        console.trace(result);
                    }
                    if (success) success(result, status, xhr);
                },
                error: function (data, type) {
                    if (error) error();
                    console.log("访问接口[" + url + "]失败！");
                    console.log(type + " 错误,请联系管理员");
                }
            })
        }

    }
});
var Style = Object.create(null, {
    provinceBindEvent: {//省份绑定查询事件
        value: +function(){
            var province = dom.querySelector('#province');
            if(province.value){
                for(var i=0;i<province.querySelectorAll('option').length;i++){
                    if(province.querySelectorAll('option')[i].value == province.value){
                        province.querySelectorAll('option')[i].selected = true;
                        Fn.getInitCity(province.querySelectorAll('option')[i].getAttribute('data-value'));
                    }
                }
            }
            province.addEventListener('change',function(){
                if(this.value){
                    Fn.getCity(this.querySelector('option[value='+this.value+']').getAttribute('data-value'));
                }else{
                    //dom.querySelector('#city option[value="select-city"]').selected = true;
                    dom.querySelector('#city').innerHTML = '<option value="" selected>选择城市</option>'
                }
            },false);
        }()
    },
    bankBindEvent: {//开户行绑定事件
        value: +function(){
            var bankCode = dom.querySelector('#bankCode');
            if(bankCode.value){
            	Fn.getInitBankBranch(bankCode.value);
            }
        }()
    }
});
```
### jQuery 
```
//绑定事件
$("id").change(function(){
	doSomething();
})
```
```
//判断为空对象
var isEmpty = $.isEmptyObject(data);

```
```
//清空 option
document.getElementById("optionId").length=0;
//是否有某个字符
var str = "abcdefGWechat";
var index = str.indexOf("chat");
index != -1;// 找到
```

### java

___在开发中，通常我们需要判断null，不然容易造成程序运行时报出 NullpointException ,切记。___

其实开发中，如果简单的 BS 模式程序可以抽象为：1用户上传文件，用户上传文字给程序；2程序展示文件，程序展示文字。而其中可以根据业务逻辑写出代码，也就是业务流程。if-else。简单的抽象可以成这样。所以什么是最重要的？对于用户来说，实现功能最重要。对于程序员来说，高效，高可用最重要。用户的功能千变万化，但是程序的本质不会变。业务的流程归根来说就是你给我什么数据，我按照你的逻辑处理数据，返回给你处理结果。为什么程序员很大多数都被称为码农，因为我们很大部分人都是按照别人给的逻辑，才能处理数据，归根到底，就是别人说你该怎么办，就得怎么办。业务由此而来。业务程序员由此而来。如果对代码的追求局限于实现功能，那么当然你可以成为一个非常出色的业务型程序员，在某一领域当专家。如果你梦想给程序员当程序员，那么中间件，组件开发是我们该选择的。每个程序员都有一个梦想，那就是让别人用我们开发的程序。中间件的开发，可以说归根到底是数据结构，算法了。所以，万变不离其宗。每个人都要选一个合适自己的道路。加油。


