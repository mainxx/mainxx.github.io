# 第2课：工作和触发器

Quartz API
Quartz API的关键接口和类是：

* IScheduler - 与调度程序交互的主要API。
* IJob - 由您希望由调度程序执行的组件实现的接口。
* IJobDetail - 用于定义Jobs的实例。
* ITrigger - 一个组件，用于定义执行给定作业的计划。
* JobBuilder - 用于定义/构建定义作业实例的JobDetail实例。
* TriggerBuilder - 用于定义/构建触发器实例。
* 在本教程中，为了便于阅读，下面的术语可以互换使用：IScheduler和Scheduler，IJob和Job，IJobDetail和JobDetail，ITrigger和Trigger。

一个调度程序的生命周期是由它为界的创作，通过SchedulerFactory和其关断（）方法的调用。创建后，可以使用IScheduler接口添加，删除和列出作业和触发器，以及执行其他与调度相关的操作（例如暂停触发器）。但是，在使用Start（）方法启动调度程序之前，调度程序实际上不会对任何触发器执行（执行作业），如第1课所示。

Quartz提供定义域特定语言（或DSL，有时也称为“流畅接口”）的“构建器”类。在之前的课程中，您看到了一个例子，我们在这里再次介绍一部分：

```C#
    // define the job and tie it to our HelloJob class
    IJobDetail job = JobBuilder.Create<HelloJob>()
        .WithIdentity("myJob", "group1") // name "myJob", group "group1"
        .Build();

    // Trigger the job to run now, and then every 40 seconds
    ITrigger trigger = TriggerBuilder.Create()
        .WithIdentity("myTrigger", "group1")
        .StartNow()
        .WithSimpleSchedule(x => x
            .WithIntervalInSeconds(40)
            .RepeatForever())
        .Build();

    // Tell quartz to schedule the job using our trigger
    await sched.scheduleJob(job, trigger);
```

构建作业定义的代码块使用JobBuilder使用流畅的界面来创建产品IJobDetail。同样，构建触发器的代码块使用TriggerBuilder的流畅接口和扩展方法，这些方法特定于给定的触发器类型。可能的计划扩展方法是：

* WithCalendarIntervalSchedule
* WithCronSchedule
* WithDailyTimeIntervalSchedule
* WithSimpleSchedule

DateBuilder类包含各种方法，可以轻松地为特定时间点构建DateTimeOffset实例（例如表示下一个偶数小时的日期 - 或者换句话说，如果它当前是9:43:27，则为10:00:00）。

## 工作和触发器

Job是一个实现IJob接口的类，它只有一个简单的方法：

IJob接口

```C#
    namespace Quartz
    {
        public interface IJob
        {
            Task Execute(JobExecutionContext context);
        }
    }
```

当Job的触发器触发时（稍后会更多），执行（..）方法由调度程序的一个工作线程调用。传递给此方法的JobExecutionContext对象为作业实例提供有关其“运行时”环境的信息 - 执行它的调度程序的句柄，触发执行的触发器的句柄，作业的JobDetail对象以及其他一些项目。

JobDetail对象是在将Job添加到调度程序时由Quartz.NET客户端（您的程序）创建的。它包含Job的各种属性设置，以及JobDataMap，它可用于存储作业类的给定实例的状态信息。它本质上是作业实例的定义，将在下一课中进一步详细讨论。

触发器对象用于触发作业的执行（或“触发”）。当您希望计划作业时，可以实例化触发器并“调整”其属性以提供您希望的计划。触发器也可能有一个与之关联的JobDataMap - 这对于将参数传递给特定于触发器触发的Job非常有用。Quartz附带了一些不同的触发器类型，但最常用的类型是SimpleTrigger（接口ISimpleTrigger）和CronTrigger（接口ICronTrigger）。

如果您需要“一次性”执行（在某个特定时间只执行一项作业），或者如果您需要在给定时间启动作业，并重复执行N次，SimpleTrigger会很方便的T之间执行。如果您希望根据类似日历的时间表进行触发，例如“每个星期五，中午”或“每个月的第10天的10:15”，则CronTrigger非常有用。

为什么乔布斯和触发器？许多作业调度程序没有单独的作业和触发器概念。一些人将'工作'定义为执行时间（或计划）以及一些小型工作标识符。其他人很像Quartz的工作和触发对象的联合。在开发Quartz的过程中，我们决定在时间表和要在该时间表上执行的工作之间建立一个区分是有意义的。这（在我们看来）有很多好处。

例如，可以独立于触发器创建作业并将其存储在作业调度程序中，并且许多触发器可以与同一作业相关联。这种松散耦合的另一个好处是能够配置调度程序在其关联的触发器过期后保留的作业，以便稍后重新调度它，而不必重新定义它。它还允许您修改或替换触发器，而无需重新定义其关联作业。

## 身份

作业和触发器在向Quartz调度器注册时给出标识键。作业和触发器的键（JobKey和TriggerKey）允许它们被放置到“组”中，这可以用于组织作业和触发器，例如“报告作业”和“维护作业”等类别。作业或触发器的键的名称部分在组内必须是唯一的

换句话说，作业或触发器的完整键（或标识符）是名称和组的组合。
你现在有什么作业和触发器，您可以了解更多关于他们的总体思路 第3课：更多关于工作与JobDetails和第4课：更多关于触发器