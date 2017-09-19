---
title: message-rabbitmq
date: 2017-09-15 10:13:47
tags:
    - message
    - 消息中间件
categories: RabbitMQ
---

## 消息中间件 RabbitMQ

### 简述

RabbitMQ 是一个消息中间件，类似于传统的邮局，不过RabbitMQ充当了邮箱，邮局，邮递员的角色。和邮局不同的是，它使用消息接受，存储，发送二进制数据。

### RabbitMQ 的一些术语

`Producing`: 生产者，发送消息方，只发送消息。

`queue`: 消息队列，允许消息从RabbitMQ传送到应用程序，消息只能被存储在队列中。队列仅仅被宿主机的内存和磁盘空间限制，本质上它是一个大的缓存块。生产者可以往队列发消息，消费者也可以从队列中接受消息。

`Consuming`:消费者，类似于收信人，消费者程序通常是在等待接收消息。

> ps:生产者，消费者和中间件不应该在同一台宿主机上面，实际中大多数应用都不会这样做。

### Hello world

首先我们准备一个发送者，发送的特性就是发送消息，所以我们要知道有哪些消息，另外消息要发送到MQ队列，所以我们还要指明是哪个队列;

```java
package me.chenzhijun;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author chen
 * @version V1.0
 * @date 2017/9/15
 */
public class Sender {

    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws IOException, TimeoutException {

        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("localhost"); //连接本地的消息中间件，如果是其它机器换成ip就行了,也可以对连接进行授权，协议版本等控制
//        connectionFactory.setPassword();
        Connection connection = connectionFactory.newConnection();//socket连接, 大多数任务的完成都是调用connection的api
        Channel channel = connection.createChannel();

        //1 发送消息之前，必须先声明一个发送消息的队列，然后我们往队列里面发送消息
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);// 队列声明是幂等的，它只会在不存在的时候创建
        String message = "Hello,Rabbit,this is new Message";

        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());// 消息内容是一个字节数组，可以用使用任何编码

        //2 关闭
        channel.close();
        connection.close();

        System.out.println("[x] sent:" + message);

        //MQ 必须要有200M最低默认磁盘空间

    }
}

```

首先我们有一个连接工厂，我们对要生产的连接做一些设置，做完设置后我们就可以创建一个连接，连接创建完成后，我们就有了一个Connection，Connection包含了我们需要完成任务的api创建一个Channel，











































































































