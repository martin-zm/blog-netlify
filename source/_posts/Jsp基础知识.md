title: Jsp基础知识
author: 禾田
tags:
  - jsp
categories:
  - java
date: 2018-02-25 10:10:00
description: jsp相关基础知识
---
>通过Servlet进行整个网站的开发是可以的。不过在Servlet中输出html代码，特别是稍微复杂一点的html代码，就会给人一种很**酸爽**的感觉。

### Jsp文件开头
```
<%@page contentType="text/html; charset=UTF-8"  pageEncoding="UTF-8" import="java.util.*"%>
```
是JSP的<%@page指令

```
contentType="text/html; charset=UTF-8" 
```

相当于response.setContentType("text/html; charset=UTF-8"); 通知浏览器以UTF-8进行中文解码
``` 
pageEncoding="UTF-8" 
```
如果jsp文件中出现了中文，这些中文使用UTF-8进行编码
```
import="java.util.* 
```
导入其他类，如果导入多个类，彼此用,逗号隔开，像这样 import="java.util.*,java.sql.*"

### [为什么Jsp是Servlet?](http://how2j.cn/k/jsp/jsp-transfer/574.html)

### 页面元素

**jsp由这些页面元素组成：**

1）静态内容
就是html,css,javascript等内容

2）指令
以<%@开始 %> 结尾，比如<%@page import="java.util.*"%>

3）表达式 <%=%>
用于输出一段html

4）Scriptlet
在<%%> 之间，可以写任何java 代码

5）声明
在<%!%> 之间可以声明字段或者方法。但是不建议这么做。

6）动作
<jsp:include page="Filename" > 在jsp页面中包含另一个页面。在包含的章节有详细的讲解

7）注释 <%-- -- %>不同于 html的注释 <!-- --> 通过jsp的注释，浏览器也看不到相应的代码，相当于在servlet中注释掉了 
 
![image](http://stepimage.how2j.cn/1657.png)

### 指令include和动作include区别   

**指令include**  
```
<%@include file="footer.jsp" %>
```

footer.jsp的内容会被插入到 hello.jsp 转译 成的hello_jsp.java中，最后只会生成一个hello_jsp.java文件

**动作include** 
```
<jsp:include page="footer.jsp" />
```

footer.jsp的内容不会被插入到 hello.jsp 转译成的hello_jsp.java中，还会有一个footer_jsp.java独立存在。 hello_jsp.java 会在服务端访问footer_.jsp.java,然后把返回的结果，嵌入到响应中。

**传递参数**

因为指令<%@include 会导致两个jsp合并成为同一个java文件，所以就不存在传参的问题，在发出hello.jsp 里定义的变量，直接可以在footer.jsp中访问。

而动作<jsp:include />其实是对footer.jsp进行了一次独立的访问，那么就有传参的需要。

### cookie      

1）Cookie是一种浏览器和服务器交互数据的方式。  

2）Cookie是由服务器端创建，但是不会保存在服务器。  

3）创建好之后，发送给浏览器。浏览器保存在用户本地。  

4）下一次访问网站的时候，就会把该Cookie发送给服务器。    

![image](http://stepimage.how2j.cn/1675.png)

### session     

**Session对应的中文翻译是会话。**

会话指的是从用户打开浏览器访问一个网站开始，无论在这个网站中访问了多少页面，点击了多少链接，都属于同一个会话。 直到该用户关闭浏览器为止，都属于同一个会话。  

**session 原理**（和cookie配合工作）
![image](http://stepimage.how2j.cn/1673.png)

1）当在同一个浏览器中同时打开多个标签，发送同一个请求或不同的请求，仍是同一个session；

2）当不在同一个窗口中打开相同的浏览器时，发送请求，仍是同一个session；

3）当使用不同的浏览器时，发送请求，即使发送相同的请求，是不同的session；

4）当把当前某个浏览器的窗口全关闭，再打开，发起相同的请求时，就是本文所阐述的，是不同的session；

[**如果没有cookie，session如何工作**](http://how2j.cn/k/jsp/jsp-session/583.html#step1674)

**cookie和session区别和联系？**  
>具体来说cookie机制采用的是在客户端保持状态的方案，而session机制采用的是在服务器端保持状态的方案。两者存储的都是用户登录信息，操作行为等等的数据。

**访问同一个服务器**  
1） cookie是把用户的数据写在用户本地浏览器上,其他网站也可以扫描使用你的cookie，容易泄露自己网站用户的隐私，而且一般浏览器对单个网站站点有cookie数量与大小的限制。

2）Session是把用户的数据写在用户的独占session上，存储在服务器上，一般只将session的id存储在cookie中。但将数据存储在服务器对服务器的成本会高。  

3）session是由服务器创建的，开发人员可以在服务器上通过request对象的getsession方法得到session。

4）一般情况，登录信息等重要信息存储在session中，其他信息存储在cookie中。


### JSP有4个作用域 

1） pageContext 当前页面 

2）requestContext 一次请求  

**注意：** 服务端跳转算一次请求，而客户端跳转不算一次请求。

3）sessionContext 当前会话（当前用户）

4）applicationContext 全局，所有用户共享

### JSP九大隐式对象  

1）**request对象**

request 对象是 javax.servlet.httpServletRequest类型的对象。 该对象代表了客户端的请求信息，主要用于接受通过HTTP协议传送到服务器的数据。（包括头信息、系统信息、请求方式以及请求参数等）。request对象的作用域为一次请求。

2）**response对象**

response 代表的是对客户端的响应，主要是将JSP容器处理过的对象传回到客户端。response对象也具有作用域，它只在JSP页面内有效。

3）**session对象**

session 对象是由服务器自动创建的与用户请求相关的对象。服务器为每个用户都生成一个session对象，用于保存该用户的信息，跟踪用户的操作状态。session对象内部使用Map类来保存数据，因此保存数据的格式为 “Key/value”。 session对象的value可以使复杂的对象类型，而不仅仅局限于字符串类型。

4）**application对象**

application 对象可将信息保存在服务器中，直到服务器关闭，否则application对象中保存的信息会在整个应用中都有效。与session对象相比，application对象生命周期更长，类似于系统的“全局变量”。

5）**out 对象**

out 对象用于在Web浏览器内输出信息，并且管理应用服务器上的输出缓冲区。在使用 out 对象输出数据时，可以对数据缓冲区进行操作，及时清除缓冲区中的残余数据，为其他的输出让出缓冲空间。待数据输出完毕后，要及时关闭输出流。

6）**pageContext 对象**

pageContext 对象的作用是取得任何范围的参数，通过它可以获取 JSP页面的out、request、reponse、session、application 等对象。pageContext对象的创建和初始化都是由容器来完成的，在JSP页面中可以直接使用 pageContext对象。

7）**config 对象**

config 对象的主要作用是取得服务器的配置信息。通过 pageConext对象的 getServletConfig() 方法可以获取一个config对象。当一个Servlet 初始化时，容器把某些信息通过 config对象传递给这个 Servlet。 开发者可以在web.xml 文件中为应用程序环境中的Servlet程序和JSP页面提供初始化参数。

8）**page 对象**

page 对象代表JSP本身，只有在JSP页面内才是合法的。 page隐含对象本质上包含当前 Servlet接口引用的变量，类似于Java编程中的 this 指针。

9）**exception 对象**

exception 对象的作用是显示异常信息，exception对象只有当前页面的<%@page 指令设置为isErrorPage="true"的时候才可以使用。同时，在其他页面也需要设置 <%@page 指令 errorPage="" 来指定一个专门处理异常的页面。

### JSTL标准标签库    
**为了能够在JSP 中使用JSTL，首先需要两个jar包，分别是jstl.jar 和standard.jar**

### [EL表达式](http://how2j.cn/k/jsp/jsp-el/579.html#nowhere) 

### MVC架构  

M 代表 模型（Model），模型就是数据，就是dao,bean

V 代表 视图（View），就是网页, JSP，用来展示模型中的数据

C 代表 控制器（controller），控制器的作用就是把不同的数据(Model)，显示在不同的视图(View)上。（控制将数据转发到什么视图）