---
title: SpringMVC笔记（2）
date: 2018-07-12 22:28:27
tags: SpringMVC笔记
categories: back-end
---
## SpringMVC(MAVEN工程创建)

### IDEA创建过程

#### ①创建MAVEN（web工程）

`new->project->maven`，建一个裸的maven工程

#### ②添加依赖（pom.xml文件）

```xml
<dependency>

    <groupId>org.springframework</groupId>

    <artifactId>spring-webmvc</artifactId>

    <version>4.2.4.RELEASE</version> 
           <!--版本号4.2.4-->
</dependency>
```

#### ③配置文件--前端控制器（web.xml）

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- contextConfigLocation配置springmvc加载的配置文件(配置处理器映射器、适配器等等)
      若不配置，默认加载WEB-INF/servlet名称-servlet(springmvc-servlet.xml)
    -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
                   <!--Springmvc.xml文件的地址-->
    </init-param>
</servlet>
```

```xml
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <!--
    第一种:*.action,访问以.action结尾，由DispatcherServlet进行解析
    第二种:/,所有访问的地址由DispatcherServlet进行解析，对静态文件的解析需要配置不让DispatcherServlet进行解析，
            使用此种方式和实现RESTful风格的url
    第三种:/*,这样配置不对，使用这种配置，最终要转发到一个jsp页面时，仍然会由DispatcherServlet解析jsp地址，
            不能根据jsp页面找到handler，会报错
    -->
    <url-pattern>*.action</url-pattern>
</servlet-mapping>
```

![配置后的web.xml](https://i.imgur.com/FOoGJXA.png)

#### ⑤配置Handler

将编写Handler在spring容器加载

```xml
<!-- 配置Handler -->
<bean name="/queryItems.action" class="com.iot.ssm.controller.ItemsController"/>
```

#### ⑥配置处理器映射器

在classpath下的springmvc.xml中配置处理器映射器

```xml
<!-- 处理器映射器
    将bean的name作为url进行查找，需要在配置Handler时指定beanname(就是url)
-->
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>
```

#### ⑦配置处理器适配器

所有处理器适配器都实现了`HandlerAdapter`接口

`<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter">`

源码

```java
public boolean supports(Object handler) {
        return handler instanceof Controller;
}
```

此适配器能执行实现`Controller`接口的Handler

#### ⑧配置视图解析器

需要配置解析jsp的视图解析器

```xml
 <!-- 视图解析器
    解析jsp,默认使用jstl,classpath下要有jstl的包
    -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"/>
```

在springmvc.xml中视图解析器配置前缀和后缀：

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <!-- 配置jsp路径的前缀 -->
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <!-- 配置jsp路径的后缀 -->
        <property name="suffix" value=".jsp"/>
</bean>
```

程序中不用指定前缀和后缀：

```java
//指定视图
//下边的路径，如果在视图解析器中配置jsp的路径前缀和后缀，修改为items/itemsList
//modelAndView.setViewName("/WEB-INF/jsp/items/itemsList.jsp");

//下边的路径配置就可以不在程序中指定jsp路径的前缀和后缀
modelAndView.setViewName("items/itemsList");
```

### 部署调试

`HTTP Status 404 -` 
处理器映射器根据url找不到Handler,说明url错误

`HTTP Status 404 -/springmvc/WEB-INF/jsp/items/itemsLists.jsp` 
处理器映射器根据url找到了Handler，转发的jsp页面找不到