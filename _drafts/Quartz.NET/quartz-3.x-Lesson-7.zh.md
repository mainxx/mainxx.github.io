第7课：TriggerListeners和JobListeners
侦听器是您创建的对象，用于根据调度程序中发生的事件执行操作。正如您可能猜到的，TriggerListeners接收与触发器相关的事件，JobListeners接收与作业相关的事件。

与触发器相关的事件包括：触发器触发，触发错误触发（在本文档的“触发器”部分中讨论）和触发器完成（触发器触发的作业完成）。

ITriggerListener接口

public interface ITriggerListener
{
	 string Name { get; }
	 
	 Task TriggerFired(ITrigger trigger, IJobExecutionContext context);
	 
	 Task<bool> VetoJobExecution(ITrigger trigger, IJobExecutionContext context);
	 
	 Task TriggerMisfired(ITrigger trigger);
	 
	 Task TriggerComplete(ITrigger trigger, IJobExecutionContext context, int triggerInstructionCode);
}
与作业相关的事件包括：作业即将执行的通知，以及作业完成执行时的通知。

IJobListener接口

public interface IJobListener
{
	string Name { get; }

	Task JobToBeExecuted(IJobExecutionContext context);

	Task JobExecutionVetoed(IJobExecutionContext context);

	Task JobWasExecuted(IJobExecutionContext context, JobExecutionException jobException);
} 
使用自己的听众
要创建一个侦听器，只需创建一个实现ITriggerListener和/或IJobListener接口的对象。然后在运行时向调度程序注册监听器，并且必须为其指定名称（或者更确切地说，他们必须通过其Name属性来通告自己的名称。

为方便起见，您的类可以扩展JobListenerSupport或TriggerListenerSupport类，而不是实现那些接口，而只是覆盖您感兴趣的事件。

监听器与调度程序的ListenerManager一起注册，并附带一个Matcher，用于描述监听器想要接收事件的作业/触发器。

监听器在运行时在调度程序中注册，并且不与作业和触发器一起存储在JobStore中。这是因为侦听器通常是与应用程序的集成点。因此，每次运行应用程序时，都需要使用调度程序重新注册侦听器。

添加对特定作业感兴趣的JobListener：

scheduler.ListenerManager.AddJobListener(myJobListener, KeyMatcher<JobKey>.KeyEquals(new JobKey("myJobName", "myJobGroup")));
添加对特定组的所有作业感兴趣的JobListener：

scheduler.ListenerManager.AddJobListener(myJobListener, GroupMatcher<JobKey>.GroupEquals("myJobGroup"));
添加对两个特定组的所有作业感兴趣的JobListener：

scheduler.ListenerManager.AddJobListener(myJobListener,
	OrMatcher<JobKey>.Or(GroupMatcher<JobKey>.GroupEquals("myJobGroup"), GroupMatcher<JobKey>.GroupEquals("yourGroup")));
添加对所有作业感兴趣的JobListener：

scheduler.ListenerManager.AddJobListener(myJobListener, GroupMatcher<JobKey>.AnyGroup());
Quartz.NET的大多数用户都没有使用监听器，但是当应用程序需求创建了事件通知的需要时，监听器就很方便，而Job本身并没有明确地通知应用程序。