# 第4课：关于触发器的更多信息

与作业一样，触发器相对容易使用，但是在您可以充分利用Quartz.NET之前，确实需要了解和理解各种可自定义的选项。此外，如前所述，您可以选择不同类型的触发器来满足不同的调度需求。

## 常见触发器属性

除了所有触发器类型都具有用于跟踪其身份的TriggerKey属性之外，还有许多其他属性对所有触发器类型都是通用的。在构建触发器定义时，使用TriggerBuilder设置这些常用属性（后面将举例说明）。

以下是所有触发器类型共有的属性列表：

该JobKey属性指示应触发火灾时，执行作业的身份。
该StartTimeUtc属性指示当触发的时间表首先进入的影响。该值是DateTimeOffset对象，用于定义给定日历日期的时刻。对于某些触发类型，触发器实际上会在开始时触发，对于其他触发器类型，它只是标记应该开始遵循调度的时间。这意味着您可以存储一个触发器，其中包含一个计划，例如1月份的“每月的第5天”，如果StartTimeUtc属性设置为4月1日，则会在第一次触发前几个月。
该EndTimeUtc属性指示当触发的时间表应该不再有效。换句话说，具有“每月的第5天”和7月1日结束时间表的触发器将在6月5日的最后一次触发。
其他属性需要更多解释，将在以下小节中讨论。

## 优先

有时，当您有许多触发器（或Quartz.NET线程池中的少数工作线程）时，Quartz.NET可能没有足够的资源来立即触发计划同时触发的所有触发器。在这种情况下，您可能希望控制哪些触发器在可用的Quartz.NET工作线程中首先破解。为此，您可以在Trigger上设置priority属性。如果N触发器同时触发，但当前只有Z工作线程可用，则首先执行具有最高优先级的第一个Z触发器。如果未在触发器上设置优先级，则它将使用默认优先级5.任何整数值都允许优先级，正或负。

注意：仅当触发器具有相同的触发时间时才会比较优先级。计划在10:59开火的触发器将始终在计划在11:00开火之前触发。

注意：当检测到触发器的作业需要恢复时，其恢复的调度优先级与原始触发器相同。

## 失火说明

触发器的另一个重要特性是它的“失火指令”。如果持久性触发器由于调度程序被关闭而“错过”其触发时间，或者因为Quartz.NET的线程池中没有可用于执行作业的线程，则会发生失败。不同的触发类型可以使用不同的失火指令。默认情况下，它们使用“智能策略”指令 - 该指令具有基于触发类型和配置的动态行为。当调度程序启动时，它会搜索任何已失效的持久触发器，然后根据各自配置的失火指令更新每个触发器。当您在自己的项目中开始使用Quartz.NET时，您应该熟悉在给定触发器类型上定义的失火指令，并在其API文档中进行了解释。有关失火指令的更多具体信息将在特定于每种触发类型的教程课程中给出。

## 日历

实现ICalendar接口的Quartz.NET Calendar对象可以在触发器存储在调度程序中时与触发器相关联。日历可用于从触发器的发射计划中排除时间块。例如，您可以创建一个触发器，在每个工作日上午9:30触发作业，但随后添加一个排除所有业务假期的日历。

Calendar可以是任何实现ICalendar接口的可序列化对象，如下所示：

```C#
namespace Quartz
{
    public interface ICalendar
    {
        string Description { get; set; }

        ICalendar CalendarBase { set; get; }

        bool IsTimeIncluded(DateTimeOffset timeUtc);

        DateTime GetNextIncludedTimeUtc(DateTimeOffset timeUtc);

        ICalendar Clone();
    }
}
```

即使日历可以“阻挡”时间段只有一毫秒，但最有可能的是，你会对整天的“封锁”感兴趣。为方便起见，Quartz.NET包含了类HolidayCalendar，它就是这样做的。

必须通过AddCalendar（..）方法实例化日历并向调度程序注册日历。如果您使用HolidayCalendar，则在实例化之后，您应该使用其AddExcludedDate（DateTime date）方法，以便使用您希望从计划中排除的日期填充它。相同的日历实例可以与多个触发器一起使用，例如：

日历示例

```C#
    HolidayCalendar cal = new HolidayCalendar();
    cal.AddExcludedDate(someDate);

    await sched.AddCalendar("myHolidays", cal, false);

    ITrigger t = TriggerBuilder.Create()
        .WithIdentity("myTrigger")
        .ForJob("myJob")
        .WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(9, 30)) // execute job daily at 9:30
        .ModifiedByCalendar("myHolidays") // but not on holidays
        .Build();

    // .. schedule job with trigger

    ITrigger t2 = TriggerBuilder.Create()
        .WithIdentity("myTrigger2")
        .ForJob("myJob2")
        .WithSchedule(CronScheduleBuilder.DailyAtHourAndMinute(11, 30)) // execute job daily at 11:30
        .ModifiedByCalendar("myHolidays") // but not on holidays
        .Build();

    // .. schedule job with trigger2
```

触发器构造/构建的细节将在接下来的几节课中给出。现在，只要相信上面的代码创建了两个触发器，每个触发器计划每天触发。但是，将跳过在日历排除的期间内发生的任何发射。

有关可能满足您需求的许多ICalendar实现，请参阅Quartz.Impl.Calendar命名空间。