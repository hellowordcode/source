---
title: SpringMVC笔记（1）
date: 2018-07-12 22:28:27
tags: SpringMVC笔记
categories: back-end
---
# SpringMVC操作笔记

## SpringMVC基础知识介绍

###SpringMVC功能部件

<!--springmvc是spring框架的一个模块，springmvc和spring无需通过中间整合层进行整合。-->





![](https://i.imgur.com/e7v40XO.jpg)

组件及其作用

- 前端控制器(DispatcherServlet)：接收请求，响应结果，相当于转发器，中央处理器。减少了其他组件之间的耦合度
- 处理器映射器(HandlerMapping)：根据请求的url查找Handler
- **Handler处理器**：按照HandlerAdapter的要求编写
- 处理器适配器(HandlerAdapter)：按照特定规则(HandlerAdapter要求的规则)执行Handler。
- 视图解析器(ViewResolver)：进行视图解析，根据逻辑视图解析成真正的视图(View)
- **视图(View)**：View是一个接口实现类试吃不同的View类型（jsp,pdf等等）

<!--*其中加粗的为需要程序员开发的，没加粗的为不需要程序员开发的*-->

### Spring工作流程

![](https://i.imgur.com/FriXTPC.png)

- 1.发起请求到前端控制器(`DispatcherServlet`)
- 2.前端控制器请求处理器映射器(`HandlerMapping`)查找`Handler`(可根据xml配置、注解进行查找)
- 3.处理器映射器(`HandlerMapping`)向前端控制器返回`Handler`
- 4.前端控制器调用处理器适配器(`HandlerAdapter`)执行`Handler`
- 5.处理器适配器(HandlerAdapter)去执行Handler
- 6.Handler执行完，给适配器返回ModelAndView(Springmvc框架的一个底层对象)
- 7.处理器适配器(`HandlerAdapter`)向前端控制器返回`ModelAndView`
- 8.前端控制器(`DispatcherServlet`)请求视图解析器(`ViewResolver`)进行视图解析，根据逻辑视图名解析成真正的视图(jsp)
- 9.视图解析器(ViewResolver)向前端控制器(`DispatcherServlet`)返回View
- 10.前端控制器进行视图渲染，即将模型数据(在`ModelAndView`对象中)填充到request域
- 11.前端控制器向用户响应结果
