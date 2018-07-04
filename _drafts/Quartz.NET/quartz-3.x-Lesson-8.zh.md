第8课：SchedulerListeners
SchedulerListeners非常类似于ITriggerListeners和IJobListeners，除了它们接收调度程序本身内事件的通知 - 不一定是与特定触发器或作业相关的事件。

与调度程序相关的事件包括：添加作业/触发器，删除作业/触发器，调度程序中的严重错误，调度程序关闭的通知等。

ISchedulerListener接口

public interface ISchedulerListener
{
	Task JobScheduled(Trigger trigger);

	Task JobUnscheduled(string triggerName, string triggerGroup);

	Task TriggerFinalized(Trigger trigger);

	Task TriggersPaused(string triggerName, string triggerGroup);

	Task TriggersResumed(string triggerName, string triggerGroup);

	Task JobsPaused(string jobName, string jobGroup);

	Task JobsResumed(string jobName, string jobGroup);

	Task SchedulerError(string msg, SchedulerException cause);

	Task SchedulerShutdown();
} 
SchedulerListeners在调度程序的ListenerManager中注册。SchedulerListeners几乎可以是任何实现ISchedulerListener接口的对象。

添加SchedulerListener：

scheduler.ListenerManager.AddSchedulerListener(mySchedListener);
删除SchedulerListener：

scheduler.ListenerManager.RemoveSchedulerListener(mySchedListener);