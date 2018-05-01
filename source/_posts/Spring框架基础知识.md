title: Spring框架基础知识
author: 禾田
tags:
  - spring
categories:
  - spring
date: 2018-02-23 10:09:00
description: Spring框架相关知识，包括IOC AOP等
---
## IOC/DI     

**IOC—Inversion of Control** ，即“控制反转”，不是什么技术，而是一种设计思想。在Java开发中，Ioc意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制。

**DI—Dependency Injection** ，即“依赖注入”：组件之间依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。

**理解：** 

在平时的java应用开发中，我们要实现某一个功能或者说是完成某个业务逻辑时至少需要两个或以上的对象来协作完成，在没有使用Spring的时候，每个对象在需要使用他的合作对象时，自己均要使用像new object() 这样的语法来将合作对象创建出来，这个合作对象是由自己主动创建出来的，创建合作对象的主动权在自己手上，自己需要哪个合作对象，就主动去创建，创建合作对象的主动权和创建时机是由自己把控的，而这样就会使得对象间的耦合度高了，A对象需要使用合作对象B来共同完成一件事，A要使用B，那么A就对B产生了依赖，也就是A和B之间存在一种耦合关系，并且是紧密耦合在一起，而使用了Spring之后就不一样了，创建合作对象B的工作是由Spring来做的，Spring创建好B对象，然后存储到一个容器里面，当A对象需要使用B对象时，Spring就从存放对象的那个容器里面取出A要使用的那个B对象，然后交给A对象使用，至于Spring是如何创建那个对象，以及什么时候创建好对象的，A对象不需要关心这些细节问题(你是什么时候生的，怎么生出来的我可不关心，能帮我干活就行)，A得到Spring给我们的对象之后，两个人一起协作完成要完成的工作即可。

DI其实就是IOC的另外一种说法。DI是由Martin Fowler 在2004年初的一篇论文中首次提出的。他总结：**控制的什么被反转了？就是：获得依赖对象的方式反转了**。

## Bean的生命周期  

**1）Spring IOC 容器对 Bean 的生命周期进行管理的过程：** 

1）通过构造器或工厂方法创建 Bean 实例  
2）为 Bean 的属性设置值和对其他 Bean 的引用  
3）调用 Bean 的初始化方法  
4）Bean 可以使用了  
5）当容器关闭时, 调用 Bean 的销毁方法  

**说明：**在Bean的声明里设置 init-method 和 destroy-method 属性, 为Bean指定初始化和销毁方法。    

**2）（添加bean的后置处理器后）Spring IOC 容器对 Bean 的生命周期进行管理的过程：** 
 
1）通过构造器或工厂方法创建 Bean 实例
2）为 Bean 的属性设置值和对其他 Bean 的引用
3）将 Bean 实例传递给 Bean 后置处理器的postProcessBeforeInitialization 方法
4）调用 Bean 的初始化方法
5）将 Bean 实例传递给 Bean 后置处理器的postProcessAfterInitialization方法
6）Bean 可以使用了  
7）当容器关闭时, 调用 Bean 的销毁方法  

## 使用注解来注入对象   

**@Autowired 注解**  （推荐使用）  

说明： 

1）注解自动装配具有兼容类型的单个 Bean属性。

2）构造器, 普通字段(即使是非 public), 一切具有参数的方法都可以应用@Authwired 注解

**@Resource 注解**（要求提供一个 Bean名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名
称）  

**@Inject 注解**（和@Autowired 注解一样也是按类型匹配注入的Bean，但没有 required 属性）


## 在classpath中扫描特定组件
组件扫描(component scanning): Spring 能够从classpath下自动扫描，侦测和实例化具有特定注解的组件。

通过在配置文件中声明：   

```
<context:component-scan base-package="com.atguigu.spring.beans">
```

**特定组件包括：**  

**@Repository**

Spring在容器初始化时将自动扫描base-package指定的包及其子包下的所有class文件，所有标注了 @Repository 的类都将被注册为Spring Bean。

为什么 **@Repository** 只能标注在DAO类上呢？

这是因为该注解的作用不只是将类识别为Bean，同时它还能将所标注的类中抛出的数据访问异常封装为 Spring 的数据访问异常类型。 Spring本身提供了一个丰富的并且是与具体的数据访问技术无关的数据访问异常结构，用于封装不同的持久层框架抛出的异常，使得异常独立于底层的框架。


**@Component** 是一个泛化的概念，仅仅表示一个组件 (Bean) ，可以作用在任何层次。  

**@Service** 通常作用在业务层，但是目前该功能与 @Component 相同。
  
**@Constroller** 通常作用在控制层，但是目前该功能与 @Component 相同。  

## Spring AOP  

**代理设计模式的原理:**   使用一个代理将对象包装起来，然后用该代理对象取代原始对象。任何对原始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。     

```Java
package com.atguigu.spring.aop.helloworld;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

public class ArithmeticCalculatorLoggingProxy {
	
	//要代理的对象
	private ArithmeticCalculator target;
	
	public ArithmeticCalculatorLoggingProxy(ArithmeticCalculator target) {
		this.target = target;
	}
	
	public ArithmeticCalculator getLoggingProxy() {
		ArithmeticCalculator proxy = null;
		
		//代理对象由哪一个类加载器进行加载，getClass()运行时加载
		ClassLoader loader = target.getClass().getClassLoader();
		//代理对象的类型，即其中有哪些方法，".calss"编译时加载
		Class [] interfaces = new Class[]{ArithmeticCalculator.class};
		//当调用代理对象其中的方法时，该执行的代码
		InvocationHandler h = new InvocationHandler() {
			/**
			 * proxy: 正在返回的那个代理对象。一般情况下，在invoke方法中都不使用该对象。
			 * method: 正在被调用的方法
			 * args: 调用方法时，传入的参数
			 */
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				String methodName = method.getName();
				//日志
				System.out.println("The method" + methodName + "begins with " + Arrays.asList(args));
				//执行方法
				Object result = null;
				
				try {
					//前置通知
					//target:执行这个方法的对象;
					//args:执行这个方法的参数;
					result = method.invoke(target, args);
					//返回通知，可以访问到方法的返回值
				} catch (Exception e){
					e.printStackTrace();
					//异常通知，可以访问到方法出现的异常
				}
				//后置通知，因为方法可能会出现异常，所以访问不到方法的返回值
				//日志
				System.out.println("The method" + methodName + " ends with " + result);
				return result;
			}
		};
		proxy = (ArithmeticCalculator) Proxy.newProxyInstance(loader, interfaces, h);
		return proxy;
	}
}
```
参考博客：[AOP通俗理解](http://blog.csdn.net/qukaiwei/article/details/50367761)

**AOP 的好处:**  

1）每个事物逻辑位于一个位置, 代码不分散, 便于维护和升级；

2）业务模块更简洁, 只包含核心业务代码；  

**两种动态代理（JDK动态代理和CGLIB动态代理）区别：**  

1）前一种兄弟模式，spring会使用JDK的java.lang.reflect.Proxy类，它允许Spring动态生成一个新类来实现必要的接口，织入通知，并且把对这些接口的任何调用都转发到目标类。

2）后一种父子模式，spring使用CGLIB库生成目标类的一个子类，在创建这个子类的时候，spring织入通知，并且把对这个子类的调用委托到目标类。

相比之下，还是兄弟模式好些，他能更好的实现松耦合，尤其在今天都高喊着面向接口编程的情况下，父子模式只是在没有实现接口的时候，也能织入通知，应当做一种例外。

## Spring事务管理  

事务管理是企业级应用程序开发中必不可少的技术,用来确保数据的**完整性和一致性**。 

事务就是一系列的动作, 它们被当做一个单独的工作单元. 这些动作要么全部完成, 要么全部不起作用。

**事务的四个关键属性(ACID)：**

**1. 原子性(atomicity)：** 事务是一个原子操作, 由一系列动作组成. 事务的原子性确保动作要么全部完成要么完全不起作用。  

**2. 一致性(consistency)：** 一旦所有事务动作完成, 事务就被提交。数据和资源就处于一种满足业务规则的一致性状态中。  

**3. 隔离性(isolation)：** 可能有许多事务会同时处理相同的数据, 因此每个事物都应该与其他事务隔离开来, 防止数据损坏。  

**4. 持久性(durability)：** 一旦事务完成, 无论发生什么系统错误, 它的结果都不应该受到影响. 通常情况下, 事务的结果被写到持久化存储器中。   

**Spring 支持编程式事务管理，也支持声明式事务管理：**  

**1. 编程式事务管理:** 将事务管理代码嵌入到业务方法中来控制事务的提交和回滚。在编程式管理事务时, 必须在每个事务操作中包含额外的事务管理代码。   

**2. 声明式事务管理:** 大多数情况下比编程式事务管理更好用。它将事务管理代码从业务方法中分离出来, 以声明的方式来实现事务管理。事务管理作为一种横切关注点, 可以通过 AOP 方法模块化。Spring 通过 Spring AOP 框架支持声明式事务管理。  

###并发事务所导致的问题： 

参考博客：

[数据库事务隔离级别--脏读、不可重复读、幻读（清晰解释）](http://blog.csdn.net/jiesa/article/details/51317164)  

[数据事务四种隔离机制和七种传播行为代码示例](http://www.cnblogs.com/hq-123/p/6023359.html)


**1）脏读：** 脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据，之后如果未提交的事务回滚了，则事务读取的数据就是无效的。   

**2）不可重复读：** 是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。（即不能读到相同的数据内容）。  

例如，一个编辑人员两次读取同一文档，但在两次读取之间，作者重写了该文档。当编辑人员第二次读取文档时，文档已更改。原始读取不可重复。如果只有在作者全部完成编写后编辑人员才可以读取文档，则可以避免该问题。

**3）幻读：** 是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。  

例如，一个编辑人员更改作者提交的文档，但当生产部门将其更改内容合并到该文档的主复本时，发现作者已将未编辑的新材料添加到该文档中。如果在编辑人员和生产部门完成对原始文档的处理之前，任何人都不能将新材料添加到文档中，则可以避免该问题。    

### 事务隔离级别
![image](http://owq01tqh9.bkt.clouddn.com/7.png)

从理论上来说, 事务应该彼此完全隔离, 以避免并发事务所导致的问题。然而, 那样会对性能产生极大的影响, 因为事务必须按顺序运行。在实际开发中, 为了提升性能, 事务会以较低的隔离级别运行。  


Oracle Sql Server默认隔离级别：Read committed

Mysql默认隔离级别：Repeatable read   

###事务传播属性：

当事务方法被另一个事务方法调用时, 必须指定事务应该如何传播。例如: 方法可能继续在现有事务中运行, 也可能开启一个新事务, 并在自己的事务中运行。  

Spring 有7种，常用如下两种：

REQUIRED: 如果有事务在运行，当前方法就在这个事务内运行，否则，就启动一个新的事务，并在自己的事务内运行。
    
REQUIRED_NEW：当前方法必须启动新事务，并在它自己的事务内运行。如果有事务正在运行，应该将它挂起。

###面试题目：Spring是如何控制事务的？  

Spring的事务，可以说是 Spring AOP 的一种实现。

AOP面向切面编程，即在不修改源代码的情况下，对原有功能进行扩展，通过代理类来对具体类进行操作。 

spring是一个容器，通过spring这个容器来对对象进行管理，根据配置文件来实现spring对对象的管理。

spring的事务声明有两种方式，编程式和声明式。spring主要是通过“声明式事务”的方式对事务进行管理，即在配置文件中进行声明，通过AOP将事务切面切入程序，最大的好处是大大减少了代码量。