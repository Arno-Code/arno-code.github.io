---
---
title: 研习RocketMQ01之高可用与消息存储
author:
name: Leoz
link: https://gitee.com/Arno-Code
date: 2022-10-10 14:10:00 +0800
categories: [踩坑日记, 持久层]
tags: [mybatis-plus]
render_with_liquid: false
---
* content
  {:toc}


# mybatis-plus踩坑日记


* mybatis-plus的`@TableField`注解中的`exist`属性默认为`true`，如果数据库中没有对应的字段，会报错。

* mybatis-plus的`@TableLogic`注解修饰的字段不能通过updateById修改。
