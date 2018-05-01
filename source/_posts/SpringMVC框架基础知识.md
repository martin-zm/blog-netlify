title: SpringMVC框架基础知识
author: 禾田
tags:
  - springMvc
categories:
  - springmvc
date: 2018-02-20 10:05:00
description: SpringMVC的执行过程
---
**SpringMVC运行流程**

![image](http://owq01tqh9.bkt.clouddn.com/5.png)  

![image](http://owq01tqh9.bkt.clouddn.com/6.jpg)

1）用户发送请求至前端控制器DispatcherServlet。  

2）DispatcherServlet收到请求调用HandlerMapping处理器映射器。 
 
3）处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。   

4）DispatcherServlet调用HandlerAdapter处理器适配器。  

5）HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。   

6）Controller执行完成返回ModelAndView。  

7）HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。  

8）DispatcherServlet将ModelAndView传给ViewReslover视图解析器。  

9）ViewReslover解析后返回具体View。  

10）DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。  

11）DispatcherServlet响应用户。  

**说明：**  

Handler: 也就是处理器，直接对应着MVC中的C也就是控制层。SpringMVC中用@RequestMapping标注的方法都可以看成一个Handler。也就是只要可以实际处理的请求就是Handleer。  

HandlerMapping: 用来查找Handler。

HandlerAdapter: 适配器。因为SpringMVC中的Handler可以是任意形式，只要能处理请求就可以，但是Servlet需要的处理方法的结构是固定的，都是用request和response为参数的方法（比如doService方法）。如何让固定的Servlet处理方法调用灵活的Handler来进行处理？这就是HandlerAdapter所做的事情。   

View: 用来展示数据。 

ViewResolver: 用来查找View。