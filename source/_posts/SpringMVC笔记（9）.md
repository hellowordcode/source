---
title: SpringMVC笔记（9）
date: 2018-07-12 22:28:27
tags: SpringMVC笔记
categories: back-end
---

## 定义service接口

```java
public interface ItemsService {
    //商品查询列表
    List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception;

}
```

```java
public class ItemsServiceImpl implements ItemsService {

    @Autowired
    private ItemsMapperCustom itemsMapperCustom;

    public List<ItemsCustom> findItemsList(ItemsQueryVo itemsQueryVo) throws Exception {
        return itemsMapperCustom.findItemsList(itemsQueryVo);
    }
}
```

### 在spring容器配置service

在`resources/spring`下创建applicationContext-service.xml，文件中配置service。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

    <!-- 商品管理的service -->
    <bean id="itemsService" class="com.iot.learnssm.firstssm.service.impl.ItemsServiceImpl"/>

</beans>
```

### 事务控制

在`resources/spring`下创建applicationContext-transaction.xml，在applicationContext-transaction.xml中使用spring声明式事务控制方法。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">


<!-- 事务管理器
        对mybatis操作数据库事务控制，spring使用jdbc的事务控制类
    -->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <!-- 数据源
        dataSource在applicationContext-dao.xml中配置了
         -->
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 通知 -->
    <tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!-- 传播行为 -->
            <tx:method name="save*" propagation="REQUIRED"/>
            <tx:method name="delete*" propagation="REQUIRED"/>
            <tx:method name="insert*" propagation="REQUIRED"/>
            <tx:method name="update*" propagation="REQUIRED"/>
            <tx:method name="find*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="get*" propagation="SUPPORTS" read-only="true"/>
            <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!-- aop -->
    <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.iot.learnssm.firstssm.service.impl.*.*(..))"/>
    </aop:config>
</beans>
```

## 定义controller

### springmvc.xml

在`resources/spring`文件下下创建springmvc.xml文件，配置处理器映射器、适配器、视图解析器。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"

       xmlns:context="http://www.springframework.org/schema/context"

       xmlns:mvc="http://www.springframework.org/schema/mvc"

       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd

    http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd

    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

   <!-- 对于注解的Handler 可以单个配置

    实际开发中加你使用组件扫描

    -->

    <!-- 可以扫描controller、service、...

    这里让扫描controller，指定controller的包

     -->

    <context:component-scan base-package="com.iot.learnssm.firstssm.controller">/context:component-scan

    <!-- 使用mvc:annotation-driven代替上面两个注解映射器和注解适配的配置

     mvc:annotation-driven默认加载很多的参数绑定方法，

     比如json转换解析器默认加载了，如果使用mvc:annotation-driven则不用配置上面的RequestMappingHandlerMapping和RequestMappingHandlerAdapter

     实际开发时使用mvc:annotation-driven

     -->

  <mvc:annotation-driven></mvc:annotation-driven>

    <!-- 视图解析器

    解析jsp,默认使用jstl,classpath下要有jstl的包

    -->

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">

        <!-- 配置jsp路径的前缀 -->

        <property name="prefix" value="/WEB-INF/jsp/"/>

        <!-- 配置jsp路径的后缀 -->

        <property name="suffix" value=".jsp"/>

    </bean>

</beans>

```

### 配置web.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         id="WebApp_ID" version="3.0">
    <display-name>firstssm</display-name>

    <!-- 加载spring容器 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/classes/spring/applicationContext-*.xml</param-value>
        <!--  <param-value>classpath:spring/applicationContext-*.xml</param-value>-->
      </context-param>
    <listener>
      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>


<!-- springmvc 前端控制器  -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- contextConfigLocation配置springmvc加载的配置文件(配置处理器映射器、适配器等等)
          若不配置，默认加载WEB-INF/servlet名称-servlet(springmvc-servlet.xml)
        -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!--
        第一种:*.action,访问以.action三结尾，由DispatcherServlet进行解析
        第二种:/,所有访问的地址由DispatcherServlet进行解析，对静态文件的解析需要配置不让DispatcherServlet进行解析，
                使用此种方式和实现RESTful风格的url
        第三种:/*,这样配置不对，使用这种配置，最终要转发到一个jsp页面时，仍然会由DispatcherServlet解析jsp地址，
                不能根据jsp页面找到handler，会报错
        -->
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>


    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
        <welcome-file>default.html</welcome-file>
        <welcome-file>default.htm</welcome-file>
        <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

这个文件有两个作用：

- 配置前端控制器(`DispatcherServlet`)

- 加载spring容器：添加spring容器监听器，加载spring容器，使用通配符加载<u>spring/</u>

  下的配置文件：

  **applicationContext-dao.xml**

  **applicationContext-service.xml**

  **applicationContext-transaction.xml**

### 编写Controller(就是Handler)

```java
package com.iot.learnssm.firstssm.controller;
import com.iot.learnssm.firstssm.po.Items;
import com.iot.learnssm.firstssm.po.ItemsCustom;
import com.iot.learnssm.firstssm.service.ItemsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;
import java.util.ArrayList;
import java.util.List;
/**
 * Created by brian on 2016/3/2.
 */
//使用@Controller来标识它是一个控制器
@Controller
//为了对url进行分类管理 ，可以在这里定义根路径，最终访问url是根路径+子路径
//比如：商品列表：/items/queryItems.action
public class ItemsController {
    @Autowired
    private ItemsService itemsService;
    //商品查询列表
    @RequestMapping("/queryItems")
    //实现 对queryItems方法和url进行映射，一个方法对应一个url
    //一般建议将url和方法写成一样
    public ModelAndView queryItems() throws Exception{
        //调用service查找数据库，查询商品列表
        List<ItemsCustom> itemsList = itemsService.findItemsList(null);
        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //相当于request的setAttribute方法,在jsp页面中通过itemsList取数据
        modelAndView.addObject("itemsList",itemsList);
        //指定视图
        //下边的路径，如果在视图解析器中配置jsp的路径前缀和后缀，修改为items/itemsList
        //modelAndView.setViewName("/WEB-INF/jsp/items/itemsList.jsp");
        //下边的路径配置就可以不在程序中指定jsp路径的前缀和后缀
        modelAndView.setViewName("items/itemsList");
        return modelAndView;
    }
}
```

### 编写jsp

服务器路径为`WEB-INF/jsp/items/itemsList.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt"  prefix="fmt"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>查询商品列表</title>
</head>
<body> 
<form action="${pageContext.request.contextPath }/item/queryItem.action" method="post">
查询条件：
<table width="100%" border=1>
<tr>
<td><input type="submit" value="查询"/></td>
</tr>
</table>
商品列表：
<table width="100%" border=1>
<tr>
    <td>商品名称</td>
    <td>商品价格</td>
    <td>生产日期</td>
    <td>商品描述</td>
    <td>操作</td>
</tr>
<c:forEach items="${itemsList }" var="item">
<tr>
    <td>${item.name }</td>
    <td>${item.price }</td>
    <td><fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/></td>
    <td>${item.detail }</td>
    <td><a href="${pageContext.request.contextPath }/item/editItem.action?id=${item.id}">修改</a></td>
</tr>
</c:forEach>
</table>
</form>
</body>
</html>
```

## 