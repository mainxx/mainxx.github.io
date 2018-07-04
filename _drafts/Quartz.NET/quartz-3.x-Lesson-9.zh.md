第9课：JobStores
JobStore负责跟踪您为调度程序提供的所有“工作数据”：作业，触发器，日历等。为Quartz调度程序实例选择适当的IJobStore实现是一个重要的步骤。幸运的是，一旦你理解了它们之间的差异，选择应该是一个非常简单的选择。您声明您的调度程序应在您提供给用于生成调度程序实例的SchedulerFactory的属性文件（或对象）中使用哪个JobStore（以及它的配置设置）。

切勿在代码中直接使用JobStore实例。出于某种原因，许多人试图这样做。JobStore用于Quartz本身的幕后使用。您必须告诉Quartz（通过配置）使用哪个JobStore，但是您应该只使用代码中的Scheduler接口。

RAMJobStore
RAMJobStore是最简单的JobStore，它也是性能最高的（就CPU时间而言）。RAMJobStore以明显的方式得名：它将所有数据保存在RAM中。这就是它闪电般快速的原因，也是配置如此简单的原因。缺点是，当您的应用程序结束（或崩溃）时，所有调度信息都将丢失 - 这意味着RAMJobStore无法遵守作业和触发器上的“非易失性”设置。对于某些应用程序，这是可以接受的 - 甚至是期望的行为，但对于其他应用程序，这可能是灾难性的。

配置Quartz以使用RAMJobStore

quartz.jobStore.type = Quartz.Simpl.RAMJobStore, Quartz
要使用RAMJobStore（假设您正在使用StdSchedulerFactory），您不需要做任何特殊的事情。Quartz.NET的默认配置使用RAMJobStore作为作业存储实现。

ADO.NET Job Store（AdoJobStore）
AdoJobStore也被恰当地命名 - 它通过ADO.NET将所有数据保存在数据库中。因此，配置比RAMJobStore要复杂一些，而且速度也不快。但是，性能缺点并不是非常糟糕，特别是如果您使用主键上的索引构建数据库表。

要使用AdoJobStore，必须首先为Quartz.NET创建一组数据库表以供使用。您可以在Quartz.NET发行版的“database / dbtables”目录中找到表创建SQL脚本。如果还没有适用于您的数据库类型的脚本，只需查看其中一个现有脚本，然后以您的数据库所需的任何方式对其进行修改。需要注意的一点是，在这些脚本中，所有表都以前缀“QRTZ_”开头，例如表“QRTZ_TRIGGERS”和“QRTZ_JOB_DETAIL”。这个前缀实际上可以是你想要的任何东西，只要你通知AdoJobStore前缀是什么（在你的Quartz.NET属性中）。对于在同一数据库内为多个调度程序实例创建多组表，使用不同的前缀可能很有用。

目前，作业存储的内部实现的唯一选择是JobStoreTX，它自己创建事务。这与Quartz的Java版本不同，后者还可以选择使用J2EE容器管理事务的JobStoreCMT。

最后一个难题是设置一个数据源，AdoJobStore可以从中获得与数据库的连接。数据源在Quartz.NET属性中定义。数据源信息包含连接字符串和ADO.NET委托信息。

配置Quartz以使用JobStoreTx

quartz.jobStore.type = Quartz.Impl.AdoJobStore.JobStoreTX, Quartz
接下来，您需要为JobStore选择要使用的IDriverDelegate实现。DriverDelegate负责执行特定数据库可能需要的任何ADO.NET工作。StdAdoDelegate是一个使用“vanilla”ADO.NET代码（和SQL语句）来完成其工作的委托。如果没有专门为您的数据库创建的另一个委托，请尝试使用此委托 - 特殊委托通常具有更好的性能或变通方法来解决数据库特定问题。其他委托可以在“Quartz.Impl.AdoJobStore”命名空间中找到，也可以在其子命名空间中找到。

注意：如果您使用默认的StdAdoDelegate，Quartz.NET将发出警告，因为当您有大量可供选择的触发器时，它的性能很差。特定委托具有特殊的SQL来限制结果集长度（SQLServerDelegate使用TOP n，PostgreSQLDelegate LIMIT n，OracleDelegate ROWCOUNT（）<= n等）。

选择委托后，将其类名设置为AdoJobStore要使用的委托。

配置AdoJobStore以使用DriverDelegate

quartz.jobStore.driverDelegateType = Quartz.Impl.AdoJobStore.StdAdoDelegate, Quartz
接下来，您需要通知JobStore您正在使用的表前缀（如上所述）。

使用表前缀配置AdoJobStore

quartz.jobStore.tablePrefix = QRTZ_
最后，您需要设置JobStore应该使用哪个数据源。还必须在Quartz属性中定义指定的数据源。在这种情况下，我们指定Quartz应该使用数据源名称“myDS”（在配置属性中的其他位置定义）。

使用要使用的数据源的名称配置AdoJobStore

quartz.jobStore.dataSource = myDS
配置所需的最后一件事是设置数据源连接字符串信息和数据库提供程序。连接字符串是标准的ADO.NET连接，它是特定于驱动程序的。数据库提供程序是数据库驱动程序的抽象，用于在数据库驱动程序和Quartz之间创建松耦合。

设置数据源的连接字符串和数据库提供程序

 quartz.dataSource.myDS.connectionString = Server=localhost;Database=quartz;Uid=quartznet;Pwd=quartznet
 quartz.dataSource.myDS.provider = MySql
目前支持以下数据库提供程序：

SqlServer - .NET Framework 2.0的SQL Server驱动程序
OracleODP - Oracle的Oracle驱动程序
OracleODPManaged - Oracle的Oracle 11托管驱动程序
MySql - MySQL Connector / .NET
SQLite - SQLite ADO.NET Provider
SQLite-Microsoft - Microsoft SQLite ADO.NET Provider
Firebird - Firebird ADO.NET提供程序
Npgsql - PostgreSQL Npgsql
如果有更新的驱动程序，您可以而且应该使用最新版本的驱动程序，只需创建程序集绑定重定向即可

如果您的调度程序非常繁忙（即几乎总是执行与线程池大小相同的作业数，那么您应该将数据源中的连接数设置为大约线程池的大小+ 1。这通常在ADO.NET连接字符串中配置 - 有关详细信息，请参阅驱动程序实现。

“quartz.jobStore.useProperties”配置参数可以设置为“true”（默认为false），以指示AdoJobStore JobDataMaps中的所有值都是字符串，因此可以存储为名称 - 值对，而不是存储BLOB列中序列化形式的更复杂对象。从长远来看，这样更安全，因为您可以避免将非String类序列化为BLOB时出现的类版本问题。

配置AdoJobStore以将字符串用作JobDataMap值（推荐）

quartz.jobStore.useProperties = true