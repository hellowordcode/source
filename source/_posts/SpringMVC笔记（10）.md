---
title: SpringMVC笔记（10）
date: 2018-07-12 23:28:27
tags: SpringMVC笔记
categories: back-end
---

## 修改范例

操作流程：

1.进入商品查询列表页面

2.点击修改，进入商品修改页面，页面中显示了要修改的商品。要修改的商品从数据库查询，根据商品id(主键)查询商品信息

3.在商品修改页面，修改商品信息，修改后，点击提交

### 开发mapper

mapper：

根据id查询商品信息

根据id更新Items表的数据

不用开发了，使用逆向工程生成的代码。

### 开发service

在`com.iot.learnssm.firstssm.service.ItemsService`中添加两个接口

```java
  //根据id查询商品信息
    /**
     *
     * <p>Title: findItemsById</p>
     * <p>Description: </p>
     * @param id 查询商品的id
     * @return
     * @throws Exception
     */
     ItemsCustom findItemsById(Integer id) throws Exception;

    //修改商品信息
    /**
     *
     * <p>Title: updateItems</p>
     * <p>Description: </p>
     * @param id 修改商品的id
     * @param itemsCustom 修改的商品信息
     * @throws Exception
     */
     void updateItems(Integer id,ItemsCustom itemsCustom) throws Exception;
12345678910111213141516171819202122
```

在`com.iot.learnssm.firstssm.service.impl.ItemsServiceImpl`中实现接口，增加`itemsMapper`属性

```java
@Autowired
private ItemsMapper itemsMapper;

public ItemsCustom findItemsById(Integer id) throws Exception {
    Items items = itemsMapper.selectByPrimaryKey(id);
    //中间对商品信息进行业务处理
    //....
    //返回ItemsCustom
    ItemsCustom itemsCustom = new ItemsCustom();
    //将items的属性值拷贝到itemsCustom
    BeanUtils.copyProperties(items, itemsCustom);

    return itemsCustom;
}

public void updateItems(Integer id, ItemsCustom itemsCustom) throws Exception {
    //添加业务校验，通常在service接口对关键参数进行校验
    //校验 id是否为空，如果为空抛出异常

    //更新商品信息使用updateByPrimaryKeyWithBLOBs根据id更新items表中所有字段，包括 大文本类型字段
    //updateByPrimaryKeyWithBLOBs要求必须转入id
    itemsCustom.setId(id);
    itemsMapper.updateByPrimaryKeyWithBLOBs(itemsCustom);
}
```

### 开发controller

方法：

​    商品信息修改页面显示

​    商品信息修改提交

```java
//使用@Controller来标识它是一个控制器
@Controller
//为了对url进行分类管理 ，可以在这里定义根路径，最终访问url是根路径+子路径
//比如：商品列表：/items/queryItems.action
//@RequestMapping("/items")
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


    //商品信息修改页面显示
    @RequestMapping("/editItems")
    //限制http请求方法，可以post和get
    //@RequestMapping(value="/editItems",method={RequestMethod.POST, RequestMethod.GET})
    public ModelAndView editItems()throws Exception {

        //调用service根据商品id查询商品信息
        ItemsCustom itemsCustom = itemsService.findItemsById(1);

        // 返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();

        //将商品信息放到model
        modelAndView.addObject("itemsCustom", itemsCustom);

        //商品修改页面
        modelAndView.setViewName("items/editItems");

        return modelAndView;
    }

    //商品信息修改提交
    @RequestMapping("/editItemsSubmit")
    public ModelAndView editItemsSubmit(HttpServletRequest request, Integer id, ItemsCustom itemsCustom)throws Exception {

        //调用service更新商品信息，页面需要将商品信息传到此方法
        itemsService.updateItems(id, itemsCustom);

        //返回ModelAndView
        ModelAndView modelAndView = new ModelAndView();
        //返回一个成功页面
        modelAndView.setViewName("success");
        return modelAndView;
    }

}
```

### @RequestMapping`

- url映射

定义controller方法对应的url，进行处理器映射使用。

- 窄化请求映射

```java
//使用@Controller来标识它是一个控制器
@Controller
//为了对url进行分类管理 ，可以在这里定义根路径，最终访问url是根路径+子路径
//比如：商品列表：/items/queryItems.action
@RequestMapping("/items")
public class ItemsController {
```

- 限制http请求方法

出于安全性考虑，对http的链接进行方法限制。

```java
//商品信息修改页面显示
    //@RequestMapping("/editItems")
    //限制http请求方法，可以post和get
    @RequestMapping(value="/editItems",method={RequestMethod.POST, RequestMethod.GET})
    public ModelAndView editItems()throws Exception {
```

如果限制请求为post方法，进行get请求，即将上面代码的注解改为`@RequestMapping(value="/editItems",method={RequestMethod.POST})`

报错，状态码405：

![](https://i.imgur.com/yrIoRte.png) 

controller方法的返回值

返回`ModelAndView`

需要方法结束时，定义ModelAndView，将model和view分别进行设置。

返回string

如果controller方法返回string

1.表示返回逻辑视图名。

真正视图(jsp路径)=前缀+逻辑视图名+后缀

```java
@RequestMapping(value="/editItems",method={RequestMethod.POST,RequestMethod.GET})
//@RequestParam里边指定request传入参数名称和形参进行绑定。
//通过required属性指定参数是否必须要传入
//通过defaultValue可以设置默认值，如果id参数没有传入，将默认值和形参绑定。
//public String editItems(Model model, @RequestParam(value="id",required=true) Integer items_id)throws Exception {
public String editItems(Model model)throws Exception {

    //调用service根据商品id查询商品信息
    ItemsCustom itemsCustom = itemsService.findItemsById(1);

    //通过形参中的model将model数据传到页面
    //相当于modelAndView.addObject方法
    model.addAttribute("itemsCustom", itemsCustom);

    return "items/editItems";
}
```

2.redirect重定向

商品修改提交后，重定向到商品查询列表。

redirect重定向特点：浏览器地址栏中的url会变化。修改提交的request数据无法传到重定向的地址。因为重定向后重新进行request（request无法共享）

```
//重定向到商品查询列表
//return "redirect:queryItems.action";

```

3.forward页面转发

通过forward进行页面转发，浏览器地址栏url不变，request可以共享。

```java
//页面转发
return "forward:queryItems.action";
```

- 返回void

在controller方法形参上可以定义request和response，使用request或response指定响应结果：

1.使用request转向页面，如下：

`request.getRequestDispatcher("页面路径").forward(request, response);`

2.也可以通过response页面重定向：

`response.sendRedirect("url")`

3.也可以通过response指定响应结果，例如响应json数据如下：

```java
response.setCharacterEncoding("utf-8");
response.setContentType("application/json;charset=utf-8");
response.getWriter().write("json串");
```

## 注解开发之集合类型参数绑定

本文主要介绍注解开发的集合类型参数绑定，包括数组绑定，list绑定以及map绑定

### 数组绑定

### 需求

商品批量删除，用户在页面选择多个商品，批量删除。

### 表现层实现

关键：将页面选择(多选)的商品id，传到controller方法的形参，方法形参使用数组接收页面请求的多个商品id。

- controller方法定义：

```java
// 批量删除 商品信息
@RequestMapping("/deleteItems")
public String deleteItems(Integer[] items_id) throws Exception
```

- 页面定义：

```jsp
<c:forEach items="${itemsList }" var="item">
<tr>
    <td><input type="checkbox" name="items_id" value="${item.id}"/></td>
    <td>${item.name }</td>
    <td>${item.price }</td>
    <td><fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/></td>
    <td>${item.detail }</td>

    <td><a href="${pageContext.request.contextPath }/items/editItems.action?id=${item.id}">修改</a></td>

</tr>
</c:forEach>
```

## list绑定

### 需求

通常在需要批量提交数据时，将提交的数据绑定到`list<pojo>`中，比如：成绩录入（录入多门课成绩，批量提交），

本例子需求：批量商品修改，在页面输入多个商品信息，将多个商品信息提交到controller方法中。

### 表现层实现

- controller方法定义：
  - 1、进入批量商品修改页面(页面样式参考商品列表实现)
  - 2、批量修改商品提交

使用List接收页面提交的批量数据，通过包装pojo接收，在包装pojo中定义`list<pojo>`属性

```java
public class ItemsQueryVo {

    //商品信息
    private Items items;

    //为了系统 可扩展性，对原始生成的po进行扩展
    private ItemsCustom itemsCustom;

    //批量商品信息
    private List<ItemsCustom> itemsList;
```

```java
// 批量修改商品提交
// 通过ItemsQueryVo接收批量提交的商品信息，将商品信息存储到itemsQueryVo中itemsList属性中。
@RequestMapping("/editItemsAllSubmit")
public String editItemsAllSubmit(ItemsQueryVo itemsQueryVo) throws Exception {

    return "success";
}
```

- 页面定义：

```jsp
<c:forEach items="${itemsList }" var="item" varStatus="status">
    <tr>

        <td><input name="itemsList[${status.index }].name" value="${item.name }"/></td>
        <td><input name="itemsList[${status.index }].price" value="${item.price }"/></td>
        <td><input name="itemsList[${status.index }].createtime" value="<fmt:formatDate value="${item.createtime}" pattern="yyyy-MM-dd HH:mm:ss"/>"/></td>
        <td><input name="itemsList[${status.index }].detail" value="${item.detail }"/></td>

    </tr>
</c:forEach>
```

name的格式：

**对应包装pojo中的list类型属性名[下标(从0开始)].包装pojo中List类型的属性中pojo的属性名**

例子：

`"name="itemsList[${status.index }].price"`

*可以和包装类型的参数绑定归纳对比一下，其实就是在包装类的pojo基础上多了个下标。只不过包装类参数绑定时，要和包装pojo中的pojo类性的属性名一致，而list参数绑定时，要和包装pojo中的list类型的属性名一致。*

### map绑定

也通过在包装pojo中定义map类型属性。

在包装类中定义Map对象，并添加get/set方法，action使用包装对象接收。

- 包装类中定义Map对象如下：

```java
Public class QueryVo {
private Map<String, Object> itemInfo = new HashMap<String, Object>();
  //get/set方法..
}
```

- 页面定义如下：

```jsp
<tr>
<td>学生信息：</td>
<td>
姓名：<inputtype="text"name="itemInfo['name']"/>
年龄：<inputtype="text"name="itemInfo['price']"/>
.. .. ..
</td>
</tr>
```

- Contrller方法定义如下：

```java
public String useraddsubmit(Model model,QueryVo queryVo)throws Exception{
System.out.println(queryVo.getStudentinfo());
}
```

## springmvc校验

本文主要介绍springmvc校验，包括环境准备，校验器配置，pojo张添加校验规则，捕获和显示检验错误信息以及分组校验简单示例。

### 校验理解

项目中，通常使用较多是前端的校验，比如页面中js校验。对于安全要求较高点建议在服务端进行校验。

服务端校验：

- 控制层conroller：校验页面请求的参数的合法性。在服务端控制层conroller校验，不区分客户端类型（浏览器、手机客户端、远程调用）
- 业务层service（使用较多）：主要校验关键业务参数，仅限于service接口中使用的参数。
- 持久层dao：一般是不校验的。

### springmvc校验需求

springmvc使用hibernate的校验框架validation(和hibernate没有任何关系)。

校验思路：

页面提交请求的参数，请求到controller方法中，使用validation进行校验。如果校验出错，将错误信息展示到页面。

具体需求：

商品修改，添加校验（校验商品名称长度，生产日期的非空校验），如果校验出错，在商品修改页面显示错误信息。

### 环境准备

我们需要三个jar包：

- hibernate-validator.jar
- jboss-logging.jar
- validation-api.jar

这里我们添加maven依赖

```xml
<!-- hibernate 校验 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.2.4.Final</version>
</dependency>
```

可以看到，另外两个jar包被`hibernate-validator`依赖，所以不用再额外添加了。

### 配置校验器

- 在springmvc.xml中添加

  ```xml
  <!-- 校验器 -->
  <bean id="validator"
        class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
      <!-- hibernate校验器-->
      <property name="providerClass" value="org.hibernate.validator.HibernateValidator" />
      <!-- 指定校验使用的资源文件，在文件中配置校验错误信息，如果不指定则默认使用classpath下的ValidationMessages.properties -->
      <property name="validationMessageSource" ref="messageSource" />
  </bean>
  <!-- 校验错误信息配置文件 -->
  <bean id="messageSource"
        class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
      <!-- 资源文件名-->
      <property name="basenames">
          <list>
              <value>classpath:CustomValidationMessages</value>
          </list>
      </property>
      <!-- 资源文件编码格式 -->
      <property name="fileEncodings" value="utf-8" />
      <!-- 对资源文件内容缓存时间，单位秒 -->
      <property name="cacheSeconds" value="120" />
  </bean>
  ```

  校验器注入到处理器适配器中

  ```xml
  <mvc:annotation-driven conversion-service="conversionService" validator="validator">
  </mvc:annotation-driven>
  ```

 在CustomValidationMessages.properties配置校验错误信息：

```properties
#添加校验的错误提示信息
items.name.length.error=请输入1到30个字符的商品名称
items.createtime.isNUll=请输入商品的生产日期
```

### 在pojo中添加校验规则

在ItemsCustom.java中添加校验规则：

```java
public class Items {
    private Integer id;
    //校验名称在1到30字符中间
    //message是提示校验出错显示的信息
    //groups：此校验属于哪个分组，groups可以定义多个分组
    @Size(min=1,max=30,message="{items.name.length.error}")
    private String name;

    private Float price;

    private String pic;

    //非空校验
    @NotNull(message="{items.createtime.isNUll}")
    private Date createtime;
```

### 捕获和显示校验错误信息

```java
@RequestMapping("/editItemsSubmit")
public String editItemsSubmit(
        Model model,
        HttpServletRequest request,
        Integer id,
        @Validated ItemsCustom itemsCustom,
        BindingResult bindingResult)throws Exception {
```

- 在controller中将错误信息传到页面即可

```java
//获取校验错误信息
if(bindingResult.hasErrors()){
    // 输出错误信息
    List<ObjectError> allErrors = bindingResult.getAllErrors();

    for (ObjectError objectError :allErrors){
        // 输出错误信息
        System.out.println(objectError.getDefaultMessage());
    }
    // 将错误信息传到页面
    model.addAttribute("allErrors", allErrors);

    //可以直接使用model将提交pojo回显到页面
    model.addAttribute("items", itemsCustom);

    // 出错重新到商品修改页面
    return "items/editItems";
}
```

- 页面显示错误信息：

```jsp
<!-- 显示错误信息 -->
<c:if test="${allErrors!=null }">
    <c:forEach items="${allErrors }" var="error">
        ${ error.defaultMessage}<br/>
    </c:forEach>
</c:if>
```

### 分组校验

- 需求：

  在pojo中定义校验规则，而pojo是被多个controller所共用，当不同的controller方法对同一个pojo进行校验，但是每个controller方法需要不同的校验

  解决方法：

  定义多个校验分组（其实是一个java接口），分组中定义有哪些规则

  每个controller方法使用不同的校验分组

1.校验分组

```java
public interface ValidGroup1 {
    //接口中不需要定义任何方法，仅是对不同的校验规则进行分组
    //此分组只校验商品名称长度

}
```

2.在校验规则中添加分组

```java
//校验名称在1到30字符中间
//message是提示校验出错显示的信息
//groups：此校验属于哪个分组，groups可以定义多个分组
@Size(min=1,max=30,message="{items.name.length.error}",groups = {ValidGroup1.class})
private String name;
```

3.在controller方法使用指定分组的校验

```java
// value={ValidGroup1.class}指定使用ValidGroup1分组的校验
@RequestMapping("/editItemsSubmit")
public String editItemsSubmit(
        Model model,
        HttpServletRequest request,
        Integer id,
        @Validated(value = ValidGroup1.class)ItemsCustom itemsCustom,
        BindingResult bindingResult)throws Exception {
```

## 数据回显

本文介绍springmvc中数据回显的几种实现方法

数据回显：提交后，如果出现错误，将刚才提交的数据回显到刚才的提交页面。

### pojo数据回显方法

1.springmvc默认对pojo数据进行回显。

**pojo数据传入controller方法后，springmvc自动将pojo数据放到request域，key等于pojo类型（首字母小写）**

使用`@ModelAttribute`指定pojo回显到页面在request中的key

2.`@ModelAttribute`还可以将方法的返回值传到页面

在商品查询列表页面，通过商品类型查询商品信息。在controller中定义商品类型查询方法，最终将商品类型传到页面。

```
 // 商品分类
//itemtypes表示最终将方法返回值放在request中的key
@ModelAttribute("itemtypes")
public Map<String, String> getItemTypes() {

    Map<String, String> itemTypes = new HashMap<String, String>();
    itemTypes.put("101", "数码");
    itemTypes.put("102", "母婴");

    return itemTypes;
}

```

页面上可以得到itemTypes数据。

```
<td>
    商品名称：<input name="itemsCustom.name" />
    商品类型：
    <select name="itemtype">
        <c:forEach items="${itemtypes}" var="itemtype">
            <option value="${itemtype.key }">${itemtype.value }</option>
        </c:forEach>
    </select>
</td>

```

3.使用最简单方法使用model，可以不用`@ModelAttribute`

```
//可以直接使用model将提交pojo回显到页面
//model.addAttribute("items", itemsCustom);

```

### 简单类型数据回显

使用最简单方法使用model

`model.addAttribute("id", id);`

## 异常处理器

本文主要介绍springmvc中异常处理的思路，并展示如何自定义异常处理类以及全局异常处理器的配置

### 异常处理思路

系统中异常包括两类：

- 预期异常
- 运行时异常RuntimeException

前者通过捕获异常从而获取异常信息，后者主要通过规范代码开发、测试通过手段减少运行时异常的发生。

系统的dao、service、controller出现都通过throws Exception向上抛出，最后由springmvc前端控制器交由异常处理器进行异常处理，如下图：



![](https://i.imgur.com/dnlo4hb.png) 

### 自定义异常类

对不同的异常类型定义异常类，继承Exception。

```java
package com.iot.learnssm.firstssm.exception;

/**
 * Created by brian on 2016/3/7.
 *
 * 系统 自定义异常类，针对预期的异常，需要在程序中抛出此类的异常
 */
public class CustomException  extends  Exception{
    //异常信息
    public String message;

    public CustomException(String message){
        super(message);
        this.message = message;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }
}
```

### 全局异常处理器

思路：

系统遇到异常，在程序中手动抛出，dao抛给service、service给controller、controller抛给前端控制器，前端控制器调用全局异常处理器。

全局异常处理器处理思路：

解析出异常类型

- 如果该异常类型是系统自定义的异常，直接取出异常信息，在错误页面展示
- 如果该异常类型不是系统自定义的异常，构造一个自定义的异常类型（信息为“未知错误”）

springmvc提供一个`HandlerExceptionResolver`接口

```java
   public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
        //handler就是处理器适配器要执行Handler对象（只有method）
        //解析出异常类型
        //如果该 异常类型是系统 自定义的异常，直接取出异常信息，在错误页面展示
        //String message = null;
        //if(ex instanceof CustomException){
            //message = ((CustomException)ex).getMessage();
        //}else{
            ////如果该 异常类型不是系统 自定义的异常，构造一个自定义的异常类型（信息为“未知错误”）
            //message="未知错误";
        //}

        //上边代码变为
        CustomException customException;
        if(ex instanceof CustomException){
            customException = (CustomException)ex;
        }else{
            customException = new CustomException("未知错误");
        }

        //错误信息
        String message = customException.getMessage();

        ModelAndView modelAndView = new ModelAndView();

        //将错误信息传到页面
        modelAndView.addObject("message", message);

        //指向错误页面
        modelAndView.setViewName("error");

        return modelAndView;

    }
}
```

### 错误页面

```jsp
<%--
  Created by IntelliJ IDEA.
  User: Brian
  Date: 2016/3/4
  Time: 10:51
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>错误提示</title>
</head>
<body>
${message}
</body>
</html>
```

### 在springmvc.xml配置全局异常处理器

```xml
<!-- 全局异常处理器
只要实现HandlerExceptionResolver接口就是全局异常处理器
-->
<bean class="com.iot.learnssm.firstssm.exception.CustomExceptionResolver"></bean>
```

全局异常处理器只有一个，配置多个也没用。

### 异常测试

在controller、service、dao中任意一处需要手动抛出异常。如果是程序中手动抛出的异常，在错误页面中显示自定义的异常信息，如果不是手动抛出异常说明是一个运行时异常，在错误页面只显示“未知错误”。

- 在商品修改的controller方法中抛出异常 .

```java
public String editItems(Model model,@RequestParam(value="id",required=true) Integer items_id)throws Exception {

    //调用service根据商品id查询商品信息
    ItemsCustom itemsCustom = itemsService.findItemsById(items_id);

    //判断商品是否为空，根据id没有查询到商品，抛出异常，提示用户商品信息不存在
    if(itemsCustom == null){
        throw new CustomException("修改的商品信息不存在!");
    }

    //通过形参中的model将model数据传到页面
    //相当于modelAndView.addObject方法
    model.addAttribute("items", itemsCustom);

    return "items/editItems";
}
```

- 在service接口中抛出异常：

```java
public ItemsCustom findItemsById(Integer id) throws Exception {
    Items items = itemsMapper.selectByPrimaryKey(id);
    if(items==null){
        throw new CustomException("修改的商品信息不存在!");
    }
    //中间对商品信息进行业务处理
    //....
    //返回ItemsCustom
    ItemsCustom itemsCustom = null;
    //将items的属性值拷贝到itemsCustom
    if(items!=null){
        itemsCustom = new ItemsCustom();
        BeanUtils.copyProperties(items, itemsCustom);
    }

    return itemsCustom;
}
```

- 如果与业务功能相关的异常，建议在service中抛出异常。
- 与业务功能没有关系的异常，建议在controller中抛出。

## 上传图片

在修改商品页面，添加上传商品图片功能。

在页面form中提交`enctype="multipart/form-data"`的数据时，需要springmvc对multipart类型的数据进行解析。

在springmvc.xml中配置multipart类型解析器。

```xml
<!-- 文件上传 -->
<bean id="multipartResolver"
      class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <!-- 设置上传文件的最大尺寸为5MB -->
    <property name="maxUploadSize">
        <value>5242880</value>
    </property>
</bean>
```

### 加入上传图片的jar

添加依赖

```xml
<!-- 文件上传 -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

依赖树

```properties
[INFO] \- commons-fileupload:commons-fileupload:jar:1.3.1:compile
[INFO]    \- commons-io:commons-io:jar:2.2:compile12
```

可以看到，其实还间接依赖了`commons-io:commons-io:jar`

### 创建图片虚拟目录存储图片IDEA

我使用的版本为

- tomcat 8.0.30
- intellij 15.0.2
- jdk 1.8.0_25

已经部署好了一个web应用，并且已经在IDEA中添加好了tomcat容器，现在想为这个web应用添加一个图片虚拟目录

#### 1.点击工具栏的运行配置`Edit Configurations`

![](https://i.imgur.com/qy1gIOI.png) 

#### 2.在弹出的`Run/debug Configurations`中选中tomcat容器，选择`deployment`这个tab

![](https://i.imgur.com/X02XsEN.png)

#### 3.添加物理目录和并设置虚拟目录路径

![](https://i.imgur.com/EoI2QPe.png)

这里我选择了D盘下面的tmp文件夹作为物理目录，虚拟目录设为了`/pic`,我试了下，虽然斜杠少了也没什么影响，一样能访问，不过还是建议加上吧。

#### 4.运行web应用，访问图片资源

这里需要接上具体访问资源的文件名，不然后访问不到的，如下图

![](https://i.imgur.com/33h4wXP.png) 

### tomcat配置虚拟目录映射

#### 一、在Server.xml中进行配置

在<Host>元素中添加子元素<Context path=" ...  "     docBase=" ... "/> 并重启服务器即可；

path表示虚拟目录，docBase表示真实的web应用所在目录；

比如在C盘中存在a这个web应用,则 <Context path="/test" docBase="C:\a"/>

则输入 <http://localhost:8888/test/1.html> 就能访问到a文件夹下的 1.html

注意：这种方法需要重启服务器才能够生效，所以不适用，因为每次添加一个web应用都需要重启服务器。

#### 二、最佳配置方法

`$CATALINA_BASE/conf/catalina/localhost/ 文件夹下创建一个xml文件，任意文件名都可以，但是此文件名是web应用发布后的虚拟目录；`

`比如创建一个test.xml ，在文件中添加 <Context docBase="C:\a"/>`

不需要重启服务器，只需要在浏览器中输入 <http://localhost:8888/test/1.html> 即可访问C:\a\1.html   ；

#### 三、配置默认web应用

一般，输入 [http://localhost:8080](http://localhost:8080/) 后都会跳出 tomcat的主页，因为这个tomcat的web应用就是默认的web应用，如果想将自己的web应用配置成默认的web应用，只需要在Server.xml中的<Context>元素中为 <Context path="" docBase="C:\a"/> 

或者将test.xml改成 ROOT.xml 即可；

输入 <http://localhost:8080/1.html> 就能访问C:\a\1.html ;

注意：在图片虚拟目录中，一定将图片目录分级创建（提高i/o性能），一般我们采用按日期(年、月、日)进行分级创建。

### 上传图片代码

```jsp
<tr>
    <td>商品图片</td>
    <td>
        <c:if test="${items.pic !=null}">
            <img src="/pic/${items.pic}" width=100 height=100/>
            <br/>
        </c:if>
        <input type="file"  name="items_pic"/>
    </td>
</tr>
```

- controller方法

修改：商品修改controller方法：

```java
@RequestMapping("/editItemsSubmit")
    public String editItemsSubmit(
            Model model,
            HttpServletRequest request,
            Integer id,
            @ModelAttribute("items")
            @Validated(value = ValidGroup1.class)ItemsCustom itemsCustom,
            BindingResult bindingResult,
            MultipartFile items_pic
    )throws Exception {12345678910
 //原始名称
String originalFilename = items_pic.getOriginalFilename();
//上传图片
if(items_pic!=null && originalFilename!=null && originalFilename.length()>0){

    //存储图片的物理路径
    String pic_path = "D:\\tmp\\";


    //新的图片名称
    String newFileName = UUID.randomUUID() + originalFilename.substring(originalFilename.lastIndexOf("."));
    //新图片
    File newFile = new File(pic_path+newFileName);

    //将内存中的数据写入磁盘
    items_pic.transferTo(newFile);

    //将新图片名称写到itemsCustom中
    itemsCustom.setPic(newFileName);

}
```

## json数据交互

本文主要介绍如何在springmvc中进行json数据的交互，先是环境准备和配置，然后分别展示了“输入json串，输出是json串”和“输入key/value，输出是json串”两种情况下的交互

### springmvc进行json交互

json数据格式在接口调用中、html页面中较常用，json格式比较简单，解析还比较方便。

比如：webservice接口，传输json数据.

![](https://i.imgur.com/efNlNLt.png) 

- 请求json、输出json，要求请求的是json串，所以在前端页面中需要将请求的内容转成json，不太方便。

- 请求key/value、输出json。此方法比较常用。

  ### 环境准备

  #### 添加json转换的依赖

  最开始我少了`jackson-databind`依赖，程序各种报错。

  ```xml
  <!-- json 转换-->
  <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.7.2</version>
  </dependency>
  
  <dependency>
      <groupId>org.codehaus.jackson</groupId>
      <artifactId>jackson-mapper-asl</artifactId>
      <version>1.9.13</version>
  </dependency>
  ```

  依赖树 

  ```properties
  [INFO] +- com.fasterxml.jackson.core:jackson-databind:jar:2.7.2:compile
  [INFO] |  +- com.fasterxml.jackson.core:jackson-annotations:jar:2.7.0:compile
  [INFO] |  \- com.fasterxml.jackson.core:jackson-core:jar:2.7.2:compile
  [INFO] \- org.codehaus.jackson:jackson-mapper-asl:jar:1.9.13:compile
  [INFO]    \- org.codehaus.jackson:jackson-core-asl:jar:1.9.13:compile
  ```

  #### 配置json转换器

  在注解适配器中加入`messageConverters`

```xml
<!--注解适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
    <list>
    <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter"></bean>
    </list>
    </property>
</bean>
```

**注意：如果使用<mvc:annotation-driven />则不用定义上边的内容。** 

#### json交互测试

显示两个按钮分别测试

- jsp页面

```jsp
<%--
  Created by IntelliJ IDEA.
  User: brian
  Date: 2016/3/7
  Time: 20:49
  To change this template use File | Settings | File Templates.
--%>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <title>json交互测试</title>
    <script type="text/javascript" src="${pageContext.request.contextPath }/js/jquery-1.4.4.min.js"></script>
    <script type="text/javascript">
        //请求json，输出是json
        function requestJson(){     省略    }
        //请求key/value，输出是json
        function responseJson(){    省略    }
    </script>
</head>
<body>
<input type="button" onclick="requestJson()" value="请求json，输出是json"/>
<input type="button" onclick="responseJson()" value="请求key/value，输出是json"/>
</body>

```

- controller

```java
@Controller
public class JsonTest {
    省略
}
```

- 测试结果

#### 输入json串，输出是json串

使用jquery的ajax提交json串，对输出的json结果进行解析。

- jsp页面

```javascript
//请求json，输出是json
function requestJson(){

    $.ajax({
        type:'post',
        url:'${pageContext.request.contextPath }/requestJson.action',
        contentType:'application/json;charset=utf-8',
        //数据格式是json串，商品信息
        data:'{"name":"手机","price":999}',
        success:function(data){//返回json结果
            alert(data);
        }

    });

}
```

- controller

```java
 //请求json串(商品信息)，输出json(商品信息)
//@RequestBody将请求的商品信息的json串转成itemsCustom对象
//@ResponseBody将itemsCustom转成json输出
@RequestMapping("/requestJson")
public @ResponseBody ItemsCustom requestJson(@RequestBody ItemsCustom itemsCustom){

    //@ResponseBody将itemsCustom转成json输出
    return itemsCustom;
}
```

![](https://i.imgur.com/JaRziR3.png) 

可以看到，request和response的HTTP头的Content-Type都是`application/json;charset=utf-8`

![请求json，返回json,response的body](http://7xph6d.com1.z0.glb.clouddn.com/springmvc_json-request-json-2.png)

### 输入key/value，输出是json串

使用jquery的ajax提交key/value串，对输出的json结果进行解析

- jsp页面

```javascript
//请求key/value，输出是json
function responseJson(){

    $.ajax({
        type:'post',
        url:'${pageContext.request.contextPath }/responseJson.action',
        //请求是key/value这里不需要指定contentType，因为默认就 是key/value类型
        //contentType:'application/json;charset=utf-8',
        //数据格式是json串，商品信息
        data:'name=手机&price=999',
        success:function(data){//返回json结果
            alert(data.name);
        }

    });

}
```

- controller

```java
 //请求key/value，输出json
@RequestMapping("/responseJson")
public @ResponseBody ItemsCustom responseJson(ItemsCustom itemsCustom){

    //@ResponseBody将itemsCustom转成json输出
    return itemsCustom;
}
```

![](https://i.imgur.com/BWxt2wC.png) 

可以看到，key/value键值对的默认Content-Type是`application/x-www-form-urlencoded`,同时，我们收到了响应“手机” 