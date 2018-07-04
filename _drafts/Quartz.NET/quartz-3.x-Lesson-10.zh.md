第10课：配置，资源使用和SchedulerFactory
Quartz以模块化方式构建，因此要使其运行，需要将几个组件“拼接”在一起。幸运的是，有一些助手可以实现这一目标。

在Quartz可以完成其工作之前需要配置的主要组件是：

线程池
作业存储
DataSources（如有必要）
调度程序本身
自从引入基于任务的作业以来，线程池已经发生了很大变化。TODO文档更多

JobStores和DataSrouces在本教程的第9课中进行了讨论。值得注意的是，所有JobStore都实现了IJobStore接口 - 如果其中一个捆绑的JobStore不能满足您的需求，那么您可以创建自己的。

最后，您需要创建Scheduler实例。需要为Scheduler本身指定一个名称并交给JobStore和ThreadPool的实例。

StdSchedulerFactory
StdSchedulerFactory是ISchedulerFactory接口的实现。它使用一组属性（NameValueCollection）来创建和初始化Quartz Scheduler。这些属性通常存储在文件中并从文件中加载，但也可以由程序创建并直接传递给工厂。只需在工厂调用getScheduler（）就可以生成调度程序，初始化它（及其ThreadPool，JobStore和DataSources），并返回其公共接口的句柄。

Quartz发行版的“docs / config”目录中有一些示例配置（包括属性的描述）。您可以在Quartz文档的“参考”部分下的“配置”手册中找到完整的文档。

DirectSchedulerFactory
DirectSchedulerFactory是另一个SchedulerFactory实现。对于那些希望以更加程序化的方式创建Scheduler实例的人来说，它非常有用。由于以下原因，通常不鼓励使用它：（1）它要求用户更好地理解他们正在做什么，以及（2）它不允许进行声明性配置 - 换句话说，你最终难以 - 编码所有调度程序的设置。

记录
Quartz.NET使用LibLob库来满足其所有日志记录需求。Quartz不会产生太多的日志信息 - 通常只是在初始化期间的一些信息，然后只有在执行作业时有关严重问题的消息。为了“调整”日志记录设置（例如输出量以及输出的位置），您需要实际配置您选择的日志框架，因为LibLog主要将工作委托给更成熟的日志框架，如log4net，血清等

有关更多信息，请参阅LibLog Wiki。