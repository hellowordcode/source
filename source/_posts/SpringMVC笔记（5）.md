---
title: SpringMVC笔记（5）
date: 2018-07-12 22:28:27
tags: SpringMVC笔记
categories: back-end
---
## 前端控制器

1.前端控制器接收请求

调用`doDispatch`方法

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
        HttpServletRequest processedRequest = request;
        HandlerExecutionChain mappedHandler = null;
        boolean multipartRequestParsed = false;
}
```

2.前端控制器调用`HandlerMapping`（处理器映射器）根据url查找Handler

```
// Determine handler for the current request.
mappedHandler = getHandler(processedRequest);
```

```java
/**

- Return the HandlerExecutionChain for this request.
- <p>Tries all handler mappings in order.
- @param request current HTTP request
- @return the HandlerExecutionChain, or {@code null} if no handler could be found
  **/
      protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
          for (HandlerMapping hm : this.handlerMappings) {
              if (logger.isTraceEnabled()) {
     logger.trace(
             "Testing handler map [" + hm + "] in DispatcherServlet with name '" + getServletName() + "'");
              }
              HandlerExecutionChain handler = hm.getHandler(request);
              if (handler != null) {
     return handler;
              }
          }
          return null;
      }

```

3.调用处理器适配器执行Handler,得到执行的结果`ModelAndView mv`

在`doDispatch`方法中

```java
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

4.视图渲染，将model数据填充到request域

`doDispatch`->`processDispatchResult`->`render`

在`render`方法中,视图解析得到view

```java
// We need to resolve the view name.
view = resolveViewName(mv.getViewName(), mv.getModelInternal(), locale, request);
123
```

5.调用view的渲染方法，将model数据填充到request域

在`render`方法中,调用`View`接口的`render`方法

```java
view.render(mv.getModelInternal(), request, response);
```

## 