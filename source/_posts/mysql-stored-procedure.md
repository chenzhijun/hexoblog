---
title: MySQL 存储过程
date: 2017-11-21 19:26:35
tags: 
	- MySQL
categories: Database
---

## MySQL 存储过程

MySQL存储过程是一个存储在MySQL数据库中的一段SQL代码，类似于我们平常在程序语言中的一个自定义函数。其实说到底，SQL也是一种语言，它也可以定义函数，定时器等等。只不过它是直接操作的数据库中的数据。

平常能用到存储过程的机会不多，以前在支付公司的时候更别说直接操作存储过程了，这根本不可能。任何需要做的处理逻辑都需要在代码中处理，所以这次接到写存储过程的任务，我还是有点忐忑的，一个是自己根本不会。O(∩_∩)O哈！完成的时间还比较紧。不过有挑战就有机遇，这个是没有错的。

整个一周下来，也算是对存储过程的时候有了一点小小的心得，所以做次记录，如果下次遇到，能提醒自己。

### 定义存储过程

定义一个存储过程是比较简单的，类似与我们写一个Java方法，定义的语法为：

```sql
    DELIMITER // 
    CREATE PROCEDURE proc4(IN param1 INT,OUT param2 varchar(50),INOUT param3 varchar(50))
    BEGIN
    SELECT max(order_type) INTO param1 FROM cncsen.order_info;
    select @param1; 
    select * from order_info orderInfo where orderInfo.ORDER_TYPE=param1; 
    END //
    DELIMITER ;
```

现在我们逐句来解析下：

`DELIMITER` : 这个是声明一个结束符，可以这样认为，解析器找到之前默认是找到`;`就认为这句话结束，而我们用`DELIMITER`将原来的`;`改为`//`,这样解析器只有遇到`//`才会认为当前句子结束；

`CREATE` : 这个和创建表，创建数据库是一个意思；

`PROCEDURE` : 这个是指创建存储过程，类似于`TABLE`,`DATABASE`；

`proc4` ：这个的意思是存储过程名字；

`OUT / IN / INOUT` : 相当于权限定义，OUT 是指该参数可以当做出参，不能作为入参；IN 是指该参数为可以作为入参，不能作为出参；INOUT 是指该参数既可以作为出参，也可以作为入参；（入参指作为参数传进来；出参是指作为返回值，传给其他人用）

`INT / VARCHAR `