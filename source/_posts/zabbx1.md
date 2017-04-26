﻿---
title: Zabbix架构的改进方案(一)
date: 2016-03-03 21:02:24
tags: [zabbix,架构]
---
最近要使用Zabbix做二次开发, 实在无力吐槽..  

1, 界面丑陋(3版本的还好些), 二版本的简直就是上个世纪的界面, 看着就没有做的心情. 

2, 用户体验差,选择个下列框界面都要刷一次, 几乎所有的操作都是用提交本页刷新来完成. 

3, ui框架是用php后台做的, 面向对象的类继承结构,这个思想倒是不错, 让我想起了java的zk框架,但是做的远没有zk完善, 特别是对事件的处理上, 绝大多数还是要依赖嵌入的js文件来对dom操作. 组件的功能绝大多只是对对html元素做了个封装,实用的功能很少. 基本上也是鸡肋. 这也造成了前端后台耦合非常的深, 非常不利于前后端分离开发.

4,后台控制器实现繁琐. 系统中的后台请求分两种模式, 一种是在单个php文件中根据参数的不同用if来判断走不同的分枝,非常的乱,再加上php的调试模式, 调试起来非常的麻烦. 另一种是写一个类文件继承CController, 一个请求要对应一个php文件,还要在CRouter中配置, 也比较繁琐



针对以上情况,我们采取的方式是,原有功能仍保持zabbix原来的模式,新增加的功能则使用前后台分离的模式,要采用我们自己的前端js框架,前端只负责前端界面展示及操作,后台只负责后台操作,然后返回给前端json数据,两者之间只有数据的交互. 为了使用这种模式,除了引入前端的框架,还要对后台的mvc架构进行改进, 原先的后台模式分两种,一种是非mvc模式,直接在php文件中根据传来的参数简单的判断,非常的乱. 另一种是mvc模式,缺点一个请求要对应一个控制器文件,还要在手配置文件中手动的配置一下,开发比较繁琐. 针对以上两种模式的缺点, 在第二种mvc模式基础之上改进了一个新的mvc模式,即一个模块只要建一个前端js文件和后台的一个继承了BaseController的控制器即可,该模块里所有的操作都是做为该控制器的成员方法存在的,类似java的struts2.


新模式中,一个模块就是一个页面,包括头,尾和中间内容部分,头和尾都是固定的, 中间界面部分需要建一个对应的js来渲染界面,该js继承要自TSWidget. 访问时通过 模块名.do来访问．该模块中还会有很多其他请求,比如新增, 删除等,这些请示对应的后台是继承自BaseController的一个文件名为 模块名 + Controller.php的文件, 该模块中有对应操作的各方法, 在各方法中操作数据库, 并将结果赋值给$this->data即可，访问时通过 模块名!方法名.do来访问,返回一个由Controller中方法中$this->data转成的json字符, 返回到前台处理．





下节再见