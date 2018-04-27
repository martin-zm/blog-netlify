title: Servlet基本知识
author: 禾田
tags:
  - servlet
categories:
  - web
date: 2018-04-25 10:12:00
---
**注意：**开发Servlet需要导入servlet-api.jar包。

### get和post的区别   
**get**是form默认的提交方式。如果通过一个超链访问某个地址，是get方式 
如果在地址栏直接输入某个地址，是get方式 
提交数据会在浏览器显示出来不可以用于提交二进制数据，比如上传文件。

**哪些是get方式呢？**

1）form默认的提交方式
2）如果通过一个超链访问某个地址
3）如果在地址栏直接输入某个地址
4）ajax指定使用get方式的时候

**post**必须在form上通过 method="post" 显示指定。
提交数据不会在浏览器显示出来可以用于提交二进制数据，比如上传文件。
  
**哪些是post方式呢？**

1）在form上显示设置 method="post"的时候
2）ajax指定post方式的时候

### 生命周期  

Servlet 生命周期可被定义为从创建直到毁灭的整个过程。以下是 Servlet 遵循的过程： 
![image](http://stepimage.how2j.cn/1593.png)

1）实例化：当用户通过浏览器输入一个路径，这个路径对应的servlet被调用的时候，该Servlet就会被实例化。  
**注意：** 不管访问多少次servlet，只会实例化一次，这是由Tomcat服务器实例化的。


2）初始化：init方式是一个实例方法，所以会在构造方法执行后执行。  
**注意：** init初始化 只会执行一次。

3）提供服务：接下来就是执行service()方法，然后通过浏览器传递过来的信息进行判断，是调用doGet()还是doPost()方法。

4）销毁：调用destroy()方法。  

5） 被回收：当该Servlet被销毁后，就满足垃圾回收的条件了。当下一次垃圾回收GC来临的时候，就有可能被回收。

**在如下几种情况下，会调用destroy()：**  

1）该Servlet所在的web应用重新启动。 

2）关闭tomcat的时候 destroy()方法会被调用，但是这个一般都发生的很快，不易被发现。   

### Servlet三种实现方式

参考博客：（[Servlet三种实现方式](https://www.zhihu.com/question/38971252/answer/79042433)）

1）实现Servlet接口，然后实现接口中的五个方法；

2）继承GenericServlet，只需要实现一个方法：service；

3）继承HttpServlet,复写doGet和doPost方法；（最常用）  

### Servlet中页面的跳转     

1）在服务端进行页面跳转  

request.getRequestDispatcher("success.html").forward(request, response);  

2）在客户端进行页面跳转  

response.sendRedirect("fail.html");