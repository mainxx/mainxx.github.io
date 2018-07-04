# 第5课：SimpleTrigger

如果您需要在特定时刻执行一次作业，或者在特定时刻执行作业，然后按特定时间间隔重复，则SimpleTrigger应满足您的日程安排需求。或者更简洁的英语，如果你想让触发器在2005年1月13日上午11:23:54发射，那么每10秒再发射5次。

通过此描述，您可能不会觉得奇怪的是，发现SimpleTrigger的属性包括：开始时间和结束时间，重复计数和重复间隔。所有这些属性都与您期望的完全相同，只有几个与结束时间属性相关的特殊注释。

重复计数可以是零，正整数或常量值SimpleTrigger.RepeatIndefinitely。重复间隔属性必须是TimeSpan.Zero或正TimeSpan值。请注意，重复间隔为0会导致触发器的“重复计数”触发同时发生（或者与调度器可以同时处理的同时发生）。

如果您还不熟悉DateTime类，则可能会发现它有助于计算触发器触发时间，具体取决于您尝试创建的startTimeUtc（或endTimeUtc）。

EndTimeUtc属性（如果已指定）覆盖重复计数属性。如果您希望创建一个触发器（例如每隔10秒触发一次，直到给定的时刻 - 而不是必须计算它在开始时间和结束时间之间重复的次数，这可能很有用，可以简单地指定结束时间，然后使用RepeatIndefinitely的重复计数（你甚至可以指定一个巨大数字的重复计数，肯定会超过触发器在结束时间到来之前实际触发的次数） 。

SimpleTrigger实例是使用TriggerBuilder（用于触发器的主要属性）和WithSimpleSchedule扩展方法（用于SimpleTrigger特定属性）构建的。

**在特定的时间内建立触发器，无需重复**：

```C#
// trigger builder creates simple trigger by default, actually an ITrigger is returned
ISimpleTrigger trigger = (ISimpleTrigger) TriggerBuilder.Create()
    .WithIdentity("trigger1", "group1")
    .StartAt(myStartTime) // some Date 
    .ForJob("job1", "group1") // identify job with name, group strings
    .Build();
```

**在特定时间内建立触发器，然后每十秒钟重复十次**：

```C#
trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .StartAt(myTimeToStartFiring) // if a start time is not given (if this line were omitted), "now" is implied
    .WithSimpleSchedule(x => x
        .WithIntervalInSeconds(10)
        .WithRepeatCount(10)) // note that 10 repeats will give a total of 11 firings
    .ForJob(myJob) // identify job with handle to its JobDetail itself
    .Build();
```

**构建一个触发器，将在未来五分钟内触发一次**：

```C#
trigger = (ISimpleTrigger) TriggerBuilder.Create()
    .WithIdentity("trigger5", "group1")
    .StartAt(DateBuilder.FutureDate(5, IntervalUnit.Minute)) // use DateBuilder to create a date in the future
    .ForJob(myJobKey) // identify job with its JobKey
    .Build();
```

**建立一个现在会触发的触发器，然后每隔五分钟重复一次，直到22:00**：

```C#
trigger = TriggerBuilder.Create()
    .WithIdentity("trigger7", "group1")
    .WithSimpleSchedule(x => x
        .WithIntervalInMinutes(5)
        .RepeatForever())
    .EndAt(DateBuilder.DateOf(22, 0, 0))
    .Build();
```

**建立一个触发器，在下一个小时的顶部触发，然后每2小时重复一次**：

```C#
trigger = TriggerBuilder.Create()
    .WithIdentity("trigger8") // because group is not specified, "trigger8" will be in the default group
    .StartAt(DateBuilder.EvenHourDate(null)) // get the next even-hour (minutes and seconds zero ("00:00"))
    .WithSimpleSchedule(x => x
        .WithIntervalInHours(2)
        .RepeatForever())
    // note that in this example, 'forJob(..)' is not called 
    //  - which is valid if the trigger is passed to the scheduler along with the job  
    .Build();

await scheduler.scheduleJob(trigger, job);
```

花一些时间看所有的由所定义的语言可用的方法TriggerBuilder及其扩展方法WithSimpleSchedule，这样你可以熟悉可能没有在上面的例子证明提供给您的选项。

SimpleTrigger失火指示
SimpleTrigger有几条指令可用于告知Quartz.NET发生失火时它应该做什么。（本教程的更多关于触发器部分介绍了失火情况）。这些指令在MisfirePolicy.SimpleTrigger上定义为常量（包括描述其行为的API文档）。说明包括：

SimpleTrigger的Misfire指令常量

MisfireInstruction.IgnoreMisfirePolicy
MisfirePolicy.SimpleTrigger.FireNow
MisfirePolicy.SimpleTrigger.RescheduleNowWithExistingRepeatCount
MisfirePolicy.SimpleTrigger.RescheduleNowWithRemainingRepeatCount
MisfirePolicy.SimpleTrigger.RescheduleNextWithRemainingCount
MisfirePolicy.SimpleTrigger.RescheduleNextWithExistingCount
您应该从早期的课程中回忆一下，所有触发器都可以使用MisfirePolicy.SmartPolicy指令，并且此指令也是所有触发器类型的默认指令。

如果使用'智能策略'指令，则SimpleTrigger会根据给定SimpleTrigger实例的配置和状态，在其各种MISFIRE指令之间动态选择。SimpleTrigger.UpdateAfterMisfire（）方法的文档解释了此动态行为的确切详细信息。

构建SimpleTriggers时，您可以将简单的时间表（通过SimpleSchedulerBuilder）指定为失火指令：

```C#
trigger = TriggerBuilder.Create()
    .WithIdentity("trigger7", "group1")
    .WithSimpleSchedule(x => x
        .WithIntervalInMinutes(5)
        .RepeatForever()
        .WithMisfireHandlingInstructionNextWithExistingCount())
    .Build();
```