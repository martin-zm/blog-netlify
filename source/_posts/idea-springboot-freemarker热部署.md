title: idea+springboot+freemarker热部署
author: 禾田
tags:
  - springboot
  - freemarker
categories:
  - 技巧
date: 2018-02-15 10:02:00
description: idea+springboot+freemarker热部署提高代码调试效率
---
> 今天在学习springboot集成freemarker模板引擎修改代码时，发现每次修改一次freemarker文件时，都必须重启下应用，浏览器刷新才能显示修改后的内容，这样效率太低，每次启动一次应用都需要耗费大量时间。通过参考网上的资料终于解决了该问题，将部署步骤整理如下，方便后续参考。

**第一步：在maven中加入devtools的依赖（这里我使用的是maven来管理项目）**
![图片飞走了~](http://owq01tqh9.bkt.clouddn.com/1.png)

![图片飞走了~](http://owq01tqh9.bkt.clouddn.com/2.png)

**第二步：在application.properties中设置禁用模板引擎缓存**

```
spring.freemarker.cache=false
spring.freemarker.settings.template_update_delay=0
```
**第三步：修改IDEA的设置**

1. 打开 Settings --> Build-Execution-Deployment --> Compiler，将 Build project automatically.勾上。

![图片飞走了~](http://owq01tqh9.bkt.clouddn.com/3.png)

2. 点击 Help --> Find Action..，或使用快捷键 Ctrl+Shift+A来打开 Registry...，将其中的compiler.automake.allow.when.app.running勾上。

![图片飞走了~](http://owq01tqh9.bkt.clouddn.com/4.png)

全部设置完毕，重启一下IDEA。现在你就不必每次都手动的去点停止和启动了。