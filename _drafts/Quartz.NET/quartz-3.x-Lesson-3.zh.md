# 第3课：关于工作和工作详情的更多信息

正如您在第2课中看到的那样，工作很容易实现。关于作业的性质，关于IJob接口的Execute（..）方法以及JobDetails，还需要了解更多内容。

虽然您实现的作业类具有知道如何处理特定类型作业的实际工作的代码，但Quartz.NET需要了解您可能希望该作业的实例具有的各种属性。这是通过JobDetail类完成的，该类在前面的章节中简要提及。

JobDetail实例是使用JobBuilder类构建的。JobBuilder允许您使用流畅的界面描述您的工作细节。

现在让我们花点时间讨论一下Quartz.NET中作业的“本质”和作业实例的生命周期。首先让我们回顾一下我们在第一课中看到的一些代码片段：

使用Quartz.NET

```C#
// define the job and tie it to our HelloJob class
IJobDetail job = JobBuilder.Create<HelloJob>()
    .WithIdentity("myJob", "group1")
    .Build();

// Trigger the job to run now, and then every 40 seconds
ITrigger trigger = TriggerBuilder.Create()
  .WithIdentity("myTrigger", "group1")
  .StartNow()
  .WithSimpleSchedule(x => x
      .WithIntervalInSeconds(40)
      .RepeatForever())
  .Build();
  
sched.ScheduleJob(job, trigger);
```

现在考虑将作业类HelloJob 定义为：

```C#
public class HelloJob : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        await Console.Out.WriteLineAsync("HelloJob is executing.");
    }
}
```

请注意，我们为调度程序提供了一个IJobDetail实例，并且它通过简单地提供作业的类来引用要执行的作业。调度程序执行作业的每个（和每个）时间，它在调用其Execute（..）方法之前创建该类的新实例。这种行为的一个后果是，工作必须有一个无争议的构造函数。另一个分支是在作业类上定义数据字段没有意义 - 因为它们的值不会在作业执行之间保留。

您现在可能想要问“如何为Job实例提供属性/配置？”以及“如何在执行之间跟踪工作状态？”这些问题的答案是相同的：关键是JobDataMap ，它是JobDetail对象的一部分。

## JobDataMap的

JobDataMap可用于保存您希望在作业实例执行时可用的任意数量（可序列化）对象。JobDataMap是IDictionary接口的一个实现，并且具有一些用于存储和检索基本类型数据的便利方法。

以下是在将作业添加到调度程序之前将数据放入JobDataMap的一些快速代码段：

在JobDataMap中设置值

```C#
// define the job and tie it to our DumbJob class
IJobDetail job = JobBuilder.Create<DumbJob>()
    .WithIdentity("myJob", "group1") // name "myJob", group "group1"
    .UsingJobData("jobSays", "Hello World!")
    .UsingJobData("myFloatValue", 3.141f)
    .Build();
```
以下是在作业执行期间从JobDataMap获取数据的一个简单示例：

从JobDataMap获取值

```C#
public class DumbJob : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        JobKey key = context.JobDetail.Key;

        JobDataMap dataMap = context.JobDetail.JobDataMap;

        string jobSays = dataMap.GetString("jobSays");
        float myFloatValue = dataMap.GetFloat("myFloatValue");

        await Console.Error.WriteLineAsync("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
}
```

如果您使用持久性JobStore（在本教程的JobStore部分中讨论），您应该谨慎地决定放置在JobDataMap中的内容，因为其中的对象将被序列化，因此它们容易出现类版本问题。显然，标准的.NET类型应该是非常安全的，但除此之外，每当有人更改您已为其序列化实例的类的定义时，必须注意不要破坏兼容性。

或者，您可以将AdoJobStore和JobDataMap置于只能在地图中存储基元和字符串的模式，从而消除以后序列化问题的任何可能性。

如果向作业类添加与JobDataMap中的键名称相对应的set accessor属性，那么Quartz的默认JobFactory实现将在作业实例化时自动调用这些setter，从而无需显式地获取值。在您的execute方法中映射。

触发器也可以有与之关联的JobDataMaps。如果您有一个存储在调度程序中的作业，以供多个触发器定期/重复使用，但每次独立触发，则希望为作业提供不同的数据输入。

在作业执行期间在JobExecutionContext上找到的JobDataMap作为一种便利。它是在JobDetail上找到的JobDataMap和在Trigger上找到的JobDataMap的合并，后者中的值覆盖前者中的任何相同名称的值。

以下是在作业执行期间从JobExecutionContext的合并JobDataMap获取数据的简单示例：

```C#
public class DumbJob : IJob
{
    public async Task Execute(IJobExecutionContext context)
    {
        JobKey key = context.JobDetail.Key;

        JobDataMap dataMap = context.MergedJobDataMap;  // Note the difference from the previous example

        string jobSays = dataMap.GetString("jobSays");
        float myFloatValue = dataMap.GetFloat("myFloatValue");
        IList<DateTimeOffset> state = (IList<DateTimeOffset>)dataMap["myStateData"];
        state.Add(DateTimeOffset.UtcNow);

        await Console.Error.WriteLineAsync("Instance " + key + " of DumbJob says: " + jobSays + ", and val is: " + myFloatValue);
    }
}
```

或者，如果您希望依赖JobFactory将数据映射值“注入”到您的类中，它可能会看起来像这样：

```C#
public class DumbJob : IJob
{
    public string JobSays { private get; set; }
    public float FloatValue { private get; set; }

    public async Task Execute(IJobExecutionContext context)
    {
        JobKey key = context.JobDetail.Key;

        JobDataMap dataMap = context.MergedJobDataMap;  // Note the difference from the previous example

        IList<DateTimeOffset> state = (IList<DateTimeOffset>)dataMap["myStateData"];
        state.Add(DateTimeOffset.UtcNow);

        await Console.Error.WriteLineAsync("Instance " + key + " of DumbJob says: " + JobSays + ", and val is: " + FloatValue);
    }
}
```

你会注意到类的整体代码更长，但Execute（）方法中的代码更清晰。人们还可以争辩说，虽然代码更长，但实际上它需要更少的编码，如果程序员的IDE用于自动生成属性，而不是必须手动编写单个调用以从JobDataMap检索值。这是你的选择。

## 作业“实例”

许多用户花时间对什么构成“工作实例”感到困惑。我们将尝试在这里和下面的部分中澄清关于作业状态和并发性的内容。

您可以创建单个作业类，并通过创建JobDetails的多个实例在调度程序中存储它的许多“实例定义”

每个都有自己的属性和JobDataMap - 并将它们全部添加到调度程序。
例如，您可以创建一个实现名为“SalesReportJob”的IJob接口的类。作业可能被编码以期望发送给它的参数（通过JobDataMap）指定销售报告应该基于的销售人员的名称。然后，他们可以创建作业的多个定义（JobDetails），例如“SalesReportForJoe”和“SalesReportForMike”，它们在相应的作业中具有在相应的JobDataMaps中指定的“joe”和“mike”。

当触发器触发时，它所关联的JobDetail（实例定义）将被加载，并且它引用的作业类将通过Scheduler上配置的JobFactory实例化。默认的JobFactory使用Activator.CreateInstance简单地调用作业类的默认构造函数，然后尝试在类上调用与JobDataMap中的键名匹配的setter属性。您可能希望创建自己的JobFactory实现来完成诸如让应用程序的IoC或DI容器生成/初始化作业实例等事情。

在“Quartz speak”中，我们将每个存储的JobDetail称为“作业定义”或“JobDetail实例”，并且我们将每个执行作业称为“作业实例”或“作业定义的实例”。通常，如果我们只使用“job”这个词，我们指的是命名定义或JobDetail。当我们提到实现作业接口的类时，我们通常使用术语“作业类型”。

## 作业状态和并发

现在，关于作业的状态数据（也称为JobDataMap）和并发性的一些附加说明。有几个属性可以添加到您的Job类中，这些属性会影响Quartz在这些方面的行为。

**DisallowConcurrentExecution**是一个可以添加到Job类的属性，它告诉Quartz不要同时执行给定作业定义的多个实例（指向给定的作业类）。请注意那里的措辞，因为它是非常谨慎地选择的。在上一节的示例中，如果“SalesReportJob”具有此属性，则只能在给定时间执行“SalesReportForJoe”的一个实例，但它可以与“SalesReportForMike”实例同时执行。约束基于实例定义（JobDetail），而不是基于作业类的实例。然而，决定（在Quartz的设计期间）具有类本身所承载的属性，因为它通常会对类的编码方式产生影响。

**PersistJobDataAfterExecution**是一个可以添加到Job类的属性，它告诉Quartz在Execute（）方法成功完成后（不抛出异常）更新JobDetail的JobDataMap的存储副本，以便下一次执行相同的作业（JobDetail） ）接收更新的值而不是原始存储的值。与DisallowConcurrentExecution属性类似，这适用于作业定义实例，而不是作业类实例，尽管决定让作业类携带属性，因为它通常会对类的编码方式产生影响（例如'有状态'将需要由execute方法中的代码明确地“理解”。

如果使用PersistJobDataAfterExecution属性，则应强烈考虑使用DisallowConcurrentExecution属性，以避免在同时执行同一作业（JobDetail）的两个实例时可能存在的数据混乱（竞争条件）。

## 工作的其他属性

以下是可以通过JobDetail对象为作业实例定义的其他属性的快速摘要：

持久性 - 如果作业不耐用，一旦不再有与之关联的活动触发器，它将自动从调度程序中删除。换句话说，非持久性工作的寿命由其触发器的存在所限制。
RequestsRecovery - 如果一个作业“请求恢复”，并且它正在调度程序的“硬关闭”期间执行（即它在崩溃中运行的进程，或者机器被关闭），那么它将被重新执行当调度程序再次启动时。在这种情况下，JobExecutionContext.Recovering属性将返回true。
JobExecutionException
最后，我们需要告诉您IJob.Execute（..）方法的一些细节。您应该从execute方法抛出的唯一异常类型是JobExecutionException。因此，您通常应该使用'try-catch'块来包装execute方法的全部内容。您还应该花一些时间查看JobExecutionException的文档，因为您的作业可以使用它来为调度程序提供有关如何处理异常的各种指令。