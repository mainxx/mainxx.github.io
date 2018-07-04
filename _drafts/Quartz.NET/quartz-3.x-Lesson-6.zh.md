第6课：CronTrigger
CronTriggers通常比SimpleTrigger更有用，如果您需要基于类似日历的概念而不是精确指定的SimpleTrigger间隔重复发生的作业计划。

使用CronTrigger，您可以指定触发时间表，例如“每周五中午”，或“每个工作日和上午9:30”，甚至“每周一，周三上午9:00到上午10:00之间每隔5分钟”和星期五”。

即便如此，像SimpleTrigger一样，CronTrigger有一个startTime，用于指定计划何时生效，以及一个（可选的）endTime，用于指定何时停止计划。

克伦表达式
Cron-Expressions用于配置CronTrigger的实例。Cron-Expressions是由七个子表达式组成的字符串，它们描述了计划的各个细节。这些子表达式用空格分隔，并表示：

秒
分钟
小时
日的日
月
某一天的周
年份（可选字段）
完整的cron表达式的一个例子是字符串“0 0 12？* WED“ - 表示”每周三中午12点“。

单独的子表达式可以包含范围和/或列表。例如，前一周中的星期字段（其中显示“WED”）示例可以替换为“MON-FRI”，“MON，WED，FRI”或甚至“MON-WED，SAT”。

通配符（' '字符）可用于表示该字段的“每个”可能值。因此，前一个例子的“月”字段中的' '字符仅表示“每个月”。周日字段中的'*'显然意味着“一周中的每一天”。

所有字段都有一组可以指定的有效值。这些值应该相当明显 - 例如秒和分钟的数字0到59，以及数小时的值0到23。每月的日期可以是0-31的任何值，但您需要注意一个月内的天数！可以将月份指定为0到11之间的值，或者使用字符串JAN，FEB，MAR，APR，MAY，JUN，JUL，AUG，SEP，OCT，NOV和DEC。星期几可以指定为1到7之间的值（1 =星期日）或使用字符串SUN，MON，TUE，WED，THU，FRI和SAT。

'/'字符可用于指定值的增量。例如，如果在“分钟”字段中输入“0/15”，则表示“每隔15分钟，从零分钟开始”。如果您在“分钟”字段中使用“3/20”，则表示“每小时每20分钟一次，从第3分钟开始” - 或者换句话说，它与在“分钟”中指定“3,23,43”相同领域。

'？' 允许使用字符表示日期和星期几字段。它用于指定“无特定值”。当您需要在两个字段之一中指定某些内容而不是另一个字段时，这非常有用。请参阅下面的示例（和CronTrigger API文档）以获得说明。

“L”字符允许用于日期和星期几字段。这个字符是“last”的简写，但它在两个字段的每一个中都有不同的含义。例如，日期字段中的值“L”表示“月份的最后一天” - 1月31日，非闰年2月28日。如果在星期几字段中单独使用，则仅表示“7”或“SAT”。但如果在星期几字段中使用另一个值后，则表示“该月的最后一个xxx日” - 例如“6L”或“FRIL”均表示“该月的最后一个星期五”。使用“L”选项时，重要的是不要指定列表或值范围，因为您会得到令人困惑的结果。

'W'用于指定距离指定日最近的星期几（星期一至星期五）。例如，如果您要将“15W”指定为月份日期字段的值，则其含义为：“距离本月15日最近的工作日”。

'＃'用于指定该月的第n个“XXX”工作日。例如，星期几字段中的“6＃3”或“FRI＃3”的值表示“该月的第三个星期五”。

示例Cron表达式
以下是表达式及其含义的更多示例 - 您可以在CronTrigger的API文档中找到更多

CronTrigger示例1 - 创建触发器的表达式，每5分钟触发一次

"0 0/5 * * * ?"
CronTrigger示例2 - 一个表达式，用于创建在分钟后10秒（即上午10:00:10，上午10:05:10等）每5分钟触发一次的触发器。

"10 0/5 * * * ?"
CronTrigger示例3 - 一个表达式，用于创建在每周三和周五的10:30，11:30，12:30和13:30时触发的触发器。

"0 30 10-13 ? * WED,FRI"
CronTrigger示例4 - 一个表达式，用于创建触发器，每个月的第5天和第20天上午8点至上午10点之间每隔半小时触发一次。请注意，触发器不会在上午10点，仅在8点，8点，9点和9点30分开火

"0 0/30 8-9 5,20 * ?"
请注意，某些时间安排要求过于复杂，无法用单一触发器表示 - 例如“上午9点到上午10点之间每5分钟一次，下午1点到10点之间每20分钟一次”。这种情况下的解决方案是简单地创建两个触发器，并将它们注册为运行相同的作业。

建设CronTriggers
CronTrigger实例是使用TriggerBuilder（用于触发器的主要属性）和WithCronSchedule扩展方法（用于特定于CronTrigger的属性）构建的。

您还可以使用CronScheduleBuilder的静态方法来创建计划。

每天早上8点到下午5点建立一个触发器，每隔一分钟就会触发一次：

trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithCronSchedule("0 0/2 8-17 * * ?")
    .ForJob("myJob", "group1")
    .Build();
建立一个触发器，每天在上午10:42开枪：

// we use CronScheduleBuilder's static helper methods here
trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(10, 42))
    .ForJob(myJobKey)
    .Build();
要么 -

trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithCronSchedule("0 42 10 * * ?")
    .ForJob("myJob", "group1")
    .Build();
构建一个触发器，将在星期三上午10:42在除系统默认值之外的TimeZone中触发：

trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithSchedule(CronScheduleBuilder
        .WeeklyOnDayAndHourAndMinute(DayOfWeek.Wednesday, 10, 42)
        .InTimeZone(TimeZoneInfo.FindSystemTimeZoneById("Central America Standard Time")))
    .ForJob(myJobKey)
    .Build();
要么 -

trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithCronSchedule("0 42 10 ? * WED", x => x
        .InTimeZone(TimeZoneInfo.FindSystemTimeZoneById("Central America Standard Time")))
    .ForJob(myJobKey)
    .Build();
CronTrigger失火指示
以下指令可用于通知Quartz在CronTrigger发生失火时应该执行的操作。（本教程的更多关于触发器部分介绍了失火情况）。这些指令以常量定义（并且API文档具有对其行为的描述）。说明包括：

MisfireInstruction.IgnoreMisfirePolicy
MisfireInstruction.CronTrigger.DoNothing
MisfireInstruction.CronTrigger.FireOnceNow
所有触发器都有可用的MisfireInstrution.SmartPolicy指令，该指令也是所有触发器类型的默认指令。CronTrigger将“智能策略”指令解释为MisfireInstruction.CronTrigger.FireOnceNow。CronTrigger.UpdateAfterMisfire（）方法的API文档解释了此行为的确切详细信息。

在构建CronTriggers时，您将失火指令指定为cron调度的一部分（通过WithCronSchedule扩展方法）：

trigger = TriggerBuilder.Create()
    .WithIdentity("trigger3", "group1")
    .WithCronSchedule("0 0/2 8-17 * * ?", x => x
        .WithMisfireHandlingInstructionFireAndProceed())
    .ForJob("myJob", "group1")
    .Build();