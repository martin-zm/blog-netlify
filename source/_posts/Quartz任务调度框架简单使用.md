title: Quartz任务调度框架简单使用
author: 禾田
tags:
  - Quartz
categories:
  - Quartz
date: 2018-05-31 16:59:00
---
## Quartz是什么？
Quartz是一个任务调度框架！<br> 
Quartz是一个任务调度框架!!<br> 
Quartz是一个任务调度框架!!!  


## 为什么使用Quartz？

你可能遇到这样的问题：

1. 想每月25号，信用卡自动还款
2. 想每年4月1日自己给当年暗恋女神发一封匿名贺卡
3. 想每隔1小时，备份一下自己的学习笔记到云盘  

这些问题总结起来就是：在某一个有规律的时间点干某件事。并且时间的触发的条件可以非常复杂（比如每月最后一个工作日的17:50），复杂到需要一个专门的框架来干这个事。 Quartz就是来干这样的事，你给它一个触发条件的定义，它负责到了时间点，触发相应的Job起来干活。

## 搭建一个简单demo？

### maven导入jar包
```maven
    <dependencies>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz-jobs</artifactId>
            <version>2.2.1</version>
        </dependency>
    </dependencies>
```

### 编写测试用例
```java
package top.miao;

import static org.quartz.JobBuilder.newJob;
import static org.quartz.SimpleScheduleBuilder.simpleSchedule;
import static org.quartz.TriggerBuilder.newTrigger;

import org.quartz.JobDetail;
import org.quartz.Scheduler;
import org.quartz.Trigger;
import org.quartz.impl.StdSchedulerFactory;
/**
 * @author zhangmiao3
 * @Description: quartz测试
 * @date 11:47 2018/5/30
 */
public class QuartzTest {
    public static void main(String[] args) {
        try {
            //创建scheduler
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

            /* 定义一个Trigger
               1、定义name/group
               2、一旦加入scheduler，立即生效
               3、使用SimpleTrigger
               4、一直执行，奔腾到老不停歇
             */
            Trigger trigger = newTrigger().withIdentity("trigger1", "group1")
                    .startNow()
                    .withSchedule(simpleSchedule()
                            .withIntervalInSeconds(1)
                            .repeatForever())
                    .build();

            /* 定义一个JobDetail
               1、定义Job类为HelloQuartz类，这是真正的执行逻辑所在
               2、定义name/group
               3、定义属性
             */
            JobDetail job = newJob(HelloQuartz.class)
                    .withIdentity("job1", "group1")
                    .usingJobData("name", "quartz")
                    .build();

            //加入这个调度
            scheduler.scheduleJob(job, trigger);

            //启动之
            scheduler.start();

            //运行一段时间后关闭
            Thread.sleep(10000);
            scheduler.shutdown(true);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

```
package top.miao;
import java.util.Date;

import org.quartz.Job;
import org.quartz.JobDetail;
import org.quartz.JobExecutionContext;

/**
 * @author zhangmiao3
 * @Description:
 * @date 11:49 2018/5/30
 */

public class HelloQuartz implements Job {
    @Override
    public void execute(JobExecutionContext context) {
        JobDetail detail = context.getJobDetail();
        String name = detail.getJobDataMap().getString("name");
        System.out.println("say hello to " + name + " at " + new Date());
    }
}
```

### 运行结果

![image](http://owq01tqh9.bkt.clouddn.com/quartz1.png)

job每秒执行一次


### 分析说明

这个例子很好的覆盖了Quartz最重要的3个基本要素：

1. Scheduler：调度器。所有的调度都是由它控制。
2. Trigger： 定义触发的条件。例子中，它的类型是SimpleTrigger，每隔1秒中执行一次（什么是SimpleTrigger下面会有详述）。
3. JobDetail & Job： JobDetail 定义的是任务数据，而真正的执行逻辑是在Job中，例子中是HelloQuartz。 为什么设计成JobDetail + Job，不直接使用Job？这是因为任务是有可能并发执行，如果Scheduler直接使用Job，就会存在对同一个Job实例并发访问的问题。而JobDetail & Job 方式，sheduler每次执行，都会根据JobDetail创建一个新的Job实例，这样就可以规避并发访问的问题。

参考： [Quartz使用总结](https://www.cnblogs.com/drift-ice/p/3817269.html)