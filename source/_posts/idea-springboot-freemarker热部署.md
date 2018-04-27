title: idea+springboot+freemarker热部署
author: 禾田
tags:
  - springboot
  - freemarker
categories:
  - 技巧
date: 2018-04-25 10:02:00
---
> 今天在学习springboot集成freemarker模板引擎修改代码时，发现每次修改一次freemarker文件时，都必须重启下应用，浏览器刷新才能显示修改后的内容，这样效率太低，每次启动一次应用都需要耗费大量时间。通过参考网上的资料终于解决了该问题，将部署步骤整理如下，方便后续参考。

**第一步：在maven中加入devtools的依赖（这里我使用的是maven来管理项目）**
![这里写图片描述](http://img.blog.csdn.net/20170915172654952?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTTIwMTY3MjM4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170915173237914?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTTIwMTY3MjM4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**第二步：在application.properties中设置禁用模板引擎缓存**

```
spring.freemarker.cache=false
spring.freemarker.settings.template_update_delay=0
```
**第三步：修改IDEA的设置**

1. 打开 Settings --> Build-Execution-Deployment --> Compiler，将 Build project automatically.勾上。

![这里写图片描述](http://img.blog.csdn.net/20170915175046788?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTTIwMTY3MjM4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2. 点击 Help --> Find Action..，或使用快捷键 Ctrl+Shift+A来打开 Registry...，将其中的compiler.automake.allow.when.app.running勾上。

![这里写图片描述](http://img.blog.csdn.net/20170915175243305?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTTIwMTY3MjM4OQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

全部设置完毕，重启一下IDEA。现在你就不必每次都手动的去点停止和启动了。