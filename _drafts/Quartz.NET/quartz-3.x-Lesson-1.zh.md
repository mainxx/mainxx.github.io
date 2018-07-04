# 第1课：使用Quartz

在您使用调度器之前，它需要被实例化（谁已经猜到了？）要做到这一点，您需要使用ISchedulerFactory的实现。

一旦调度程序被实例化，它就可以启动，进入待机模式并关机。请注意，一旦调度程序关闭，它不能重新启动而不被重新实例化。在调度程序启动前，触发器不会触发（作业不会执行），也不会在调度程序处于暂停状态时触发。

以下是一段代码，它实例化并启动一个调度程序，并安排一个作业执行：

## 使用Quartz.NET

```C#
    // 构造一个 scheduler factory（调度器工厂）
    NameValueCollection props = new NameValueCollection
    {
        { "quartz.serializer.type", "binary" }
    };
    StdSchedulerFactory factory = new StdSchedulerFactory(props);

    // 取得 scheduler
    IScheduler sched = await schedFact.GetScheduler();
    await sched.Start();

    // 定义工作并将其与我们的HelloJob类联系起来
    IJobDetail job = JobBuilder.Create<HelloJob>()
        .WithIdentity("myJob", "group1")
        .Build();

    // 现在触发启动运行这个工作，每40秒执行一次
    ITrigger trigger = TriggerBuilder.Create()
      .WithIdentity("myTrigger", "group1")
      .StartNow()
      .WithSimpleSchedule(x => x
          .WithIntervalInSeconds(40)
          .RepeatForever())
      .Build();

    await sched.ScheduleJob(job, trigger);
```

如你所见，使用石英。网络是非常简单的。在**第2课**中，我们将简要介绍作业和触发器，以便您能够更全面地理解这个示例。