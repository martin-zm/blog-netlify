title: SpringAop日志埋点
author: 禾田
tags:
  - springAop
  - 日志
categories:
  - 日志
date: 2018-06-08 11:13:00
---
> 最近项目中使用了springAop来统一处理日志，总结下用法

## springAop用法

### maven导入aspectj jar包

```
	<dependency>
		<groupId>org.aspectj</groupId>
		<artifactId>aspectjweaver</artifactId>
		<version>1.8.9</version>
	</dependency>
```

### 定义切面类

```
@Aspect
@Component
public class LoggingAspect {
........
}
```
注意：需要通过@Component注解把切面类添加进容器中。

### 定义切面类中的相关方法

#### 定义切入点表达式

```
/**
 * 定义一个方法, 用于声明切入点表达式. 一般地, 该方法中再不需要添入其他的代码. 
 * 使用 @Pointcut 来声明切入点表达式. 
 * 后面的其他通知直接使用方法名来引用当前的切入点表达式. 
 */
@Pointcut("execution(public int com.atguigu.spring.aop.ArithmeticCalculator.*(..))")
public void declareJointPointExpression(){}
```

定义切入点表达式是为了方便后面“通知方法的调用”，也相当于在写代码时候定义一个变量，来避免后面冗余的代码。

==定义切入点表达式有个小技巧：==

可以定义一个注解类：
```
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RUNTIME)
public @interface AopLog {}
```
在需要植入通知的方法上加上该注解
```
@RequestMapping(value = "/api/text/trans", method = RequestMethod.POST)
@AopLog
public ResponseResult transText(@RequestBody TransArticleAO transArticleAO,
        @RequestHeader("appkey") String appKey) {}
```

定义切入点表达式

```
@Pointcut("@annotation(com.atguigu.spring.annotation.AopLog)")
public void OpenApiAopLogPoint() {
}
```

这样可以更加灵活的给需要植入通知的方法植入通知

#### 定义前置通知方法

```
/**
 * 在 com.atguigu.spring.aop.ArithmeticCalculator 接口的每一个实现类的每一个方法开始之前执行一段代码
 */
@Before("declareJointPointExpression()")
public void beforeMethod(JoinPoint joinPoint){
	String methodName = joinPoint.getSignature().getName();
	Object [] args = joinPoint.getArgs();
	System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
}
```
#### 定义后置通知方法

```
/**
 * 在方法执行之后执行的代码. 无论该方法是否出现异常
 */
@After("declareJointPointExpression()")
public void afterMethod(JoinPoint joinPoint){
	String methodName = joinPoint.getSignature().getName();
	System.out.println("The method " + methodName + " ends");
}
```

#### 定义返回通知方法

```
/**
 * 在方法法正常结束后执行的代码
 * 返回通知是可以访问到方法的返回值的!
 */
@AfterReturning(value="declareJointPointExpression()",
		returning="result")
public void afterReturning(JoinPoint joinPoint, Object result){
	String methodName = joinPoint.getSignature().getName();
	System.out.println("The method " + methodName + " ends with " + result);
}
```

#### 定义异常通知方法

```
/**
 * 在目标方法出现异常时会执行的代码.
 * 可以访问到异常对象; 且可以指定在出现特定异常时再执行通知代码
 */
@AfterThrowing(value="declareJointPointExpression()",
		throwing="e")
public void afterThrowing(JoinPoint joinPoint, Exception e){
	String methodName = joinPoint.getSignature().getName();
	System.out.println("The method " + methodName + " occurs excetion:" + e);
}
```

#### 定义环绕通知方法

```
/**
 * 环绕通知需要携带 ProceedingJoinPoint 类型的参数. 
 * 环绕通知类似于动态代理的全过程: ProceedingJoinPoint 类型的参数可以决定是否执行目标方法.
 * 且环绕通知必须有返回值, 返回值即为目标方法的返回值
 */
/*
@Around("execution(public int com.atguigu.spring.aop.ArithmeticCalculator.*(..))")
public Object aroundMethod(ProceedingJoinPoint pjd){
	
	Object result = null;
	String methodName = pjd.getSignature().getName();
	
	try {
		//前置通知
		System.out.println("The method " + methodName + " begins with " + Arrays.asList(pjd.getArgs()));
		//执行目标方法
		result = pjd.proceed();
		//返回通知
		System.out.println("The method " + methodName + " ends with " + result);
	} catch (Throwable e) {
		//异常通知
		System.out.println("The method " + methodName + " occurs exception:" + e);
		throw new RuntimeException(e);
	}
	//后置通知
	System.out.println("The method " + methodName + " ends");
	
	return result;
}
```
环绕通知是前面几种通知的融合。但是需要注意的是环绕通知方法是有返回值的，如果不设置返回值，会影响目标方法的结果返回。

上面几种方法可以根据实际情况在代码中使用。

## log4j配置日志单独打印到独立文件中

在log4j配置文件中添加如下配置

```
//配置单独打印到的日志目录别名，日志级别等
log4j.logger.com.netease.mind.nmt.aop=${log4j.level}, aop
log4j.additivity.com.netease.mind.nmt.aop=false
log4j.appender.aop=org.apache.log4j.DailyRollingFileAppender
log4j.appender.aop.DatePattern='_'yyyy-MM-dd
log4j.appender.aop.File=${log4j.path}/nmtp_aop.log
log4j.appender.aop.Append=true
log4j.appender.aop.Threshold=${log4j.level}
log4j.appender.aop.layout=org.apache.log4j.PatternLayout
log4j.appender.aop.layout.ConversionPattern=%m%n
```

日志具体参数参考：https://blog.csdn.net/foreverling/article/details/51385128