# Dapper

今天收到一个需求

```text
功能: 入库组盘检查

根据箱号目标去向验证托盘号是否可用
入参：目标去向   托盘号
1. 根据入参托盘号在组盘记录表中查询该托盘是否正在使用？
1.1. 没有正在使用（空托盘），直接返回托盘可用，且为空托盘。
1.2. 正在使用，验证入参目标去向是否能组在该托盘上？
1.2.1. 不能，返回错误信息：“该托盘目标去向不一致，不能组盘！”
1.2.2. 可以使用，返回托盘可用，且为正在使用的托盘
备注：
1. VNA/层网区/平置区只能与对应去向组在一起
2. 目标去向包含拆零的才能组盘一起
```

好吧，好像也比较容易理解，毕竟是产品那边整理过来的。好像帮我们把逻辑梳理好了。

由此我写了一个BDD

```specflow

背景:
    假如 我传入了一个目标去向'VNA区',托盘号:'00X2'

@入库组盘
场景: 入库组盘检查:没有在使用(空托盘)
    假如 该托盘没有在使用
    当 我点击检查
    那么 返回结果应该是可以使用
    而且 箱子是空的


场景大纲: 入库组盘检查:正在使用，目标去向一致（除拆零）
    假如 目标去向是:<目标去向类型>
    假如 该托盘正在使用
    假如 托盘上的箱子也是<目标去向类型>
    当 我点击查询
    那么 返回结果应该是可以使用
    而且 箱子不是空的
    但是 两个目标去向不一致
    当 我点击查询
    那么 返回结果应该是不可以使用

    例子:
        | 目标去向类型 |
        | C类品调拨  |
        | VNA区   |
        | 层网区    |
        | 平置区    |


场景大纲: 入库组盘检查:正在使用，目标去向包含拆零
    假如 目标去向是:<目标去向类型>
    假如 该托盘正在使用
    假如 托盘上的箱子的目标对象属于
        | 托盘箱子目标去向类型        |
        | 拆零区           |
        | 大件拆零CG91      |
        | 大件拆零CG91-TEST |
        | 大件拆零CG92      |
        | 混SKU去拆零       |
        | 混SKU去大件拆零     |
        | 混SKU去平置拆零     |
        | 紧急上架（混SKU去拆零） |
        | 平置拆零区         |
    当 我点击查询
    那么 返回结果应该是可以使用
    而且 箱子不是空的
    假如 托盘上的箱子的目标对象属于
    | 托盘箱子目标去向类型 |
    | C类品调拨      |
    | VNA区       |
    | 层网区        |
    | 平置区        |
    当 我点击查询
    那么 返回结果应该是不可以使用

    例子:
        | 目标去向类型        |
        | 拆零区           |
        | 大件拆零CG91      |
        | 大件拆零CG91-TEST |
        | 大件拆零CG92      |
        | 混SKU去拆零       |
        | 混SKU去大件拆零     |
        | 混SKU去平置拆零     |
        | 紧急上架（混SKU去拆零） |
        | 平置拆零区         |
```

目标去向一致及包含拆零，逻辑非常简单

```C#
/// <summary>
/// 相同目的地
/// </summary>
private bool SameDestination(string target, string destination)
{
    return (target.Contains("拆零") && destination.Contains("拆零")) || target.Equals(destination);
}
```

由于我需要去 组盘记录表中查询该托盘是否正在使用？所以我需要查询数据库

但我发现都是简单的查询，而且目前DAL层好像没有可用的接口

so..我写了一个基于Dapper的仓储模式(Repository)

* 使用到的库
1. Dapper:一个轻量级的ORM框架 开源地址：<https://github.com/StackExchange/Dapper>
2. Dapper-Extensions:添加一个实体类映射,添加基本的CRUD操作补充Dapper高级查询方案.开源地址:<https://github.com/tmsmith/Dapper-Extensions>

如果只是要使用，可以跳过下面一节

## 实现DapperRepository

```C#
    /// <summary>
    /// Dapper仓储接口
    /// 使用第三方库
    /// Dapper:https://github.com/StackExchange/Dapper
    /// Dapper-Extensions:https://github.com/tmsmith/Dapper-Extensions
    /// </summary>
    /// <typeparam name="TEntity">实体</typeparam>
    public interface IDapperRepository<TEntity> where TEntity : EntityBase
    {
        IBetweenPredicate Between(Expression<Func<TEntity, object>> expression, BetweenValues values, bool not = false);

        IExistsPredicate Exists<TSub>(IPredicate predicate, bool not = false) where TSub : class;

        IFieldPredicate Field(Expression<Func<TEntity, object>> expression, Operator op, object value, bool not = false);

        IPredicateGroup Group(GroupOperator op, params IPredicate[] predicate);

        ISort Sort(Expression<Func<TEntity, object>> expression, bool ascending = true);

        /// <summary>
        /// 添加
        /// </summary>
        /// <param name="entity"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        void Insert(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null);

        /// <summary>
        /// 查询条数
        /// </summary>
        /// <param name="predicate">谓语词组 <see cref="Field"/>,<see cref="Group"/></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        int Count(object predicate, IDbTransaction transaction = null, int? commandTimeout = null);
        /// <summary>
        /// 取得列表
        /// </summary>
        /// <param name="predicate">谓语词组 <see cref="Field"/>,<see cref="Group"/></param>
        /// <param name="sort"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <param name="buffered"></param>
        /// <returns></returns>
        IEnumerable<TEntity> GetList(object predicate = null, IList<ISort> sort = null, IDbTransaction transaction = null, int? commandTimeout = null, bool buffered = false);
        /// <summary>
        /// 修改
        /// </summary>
        /// <param name="entity"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <param name="ignoreAllKeyProperties"></param>
        /// <returns></returns>
        bool Update(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null, bool ignoreAllKeyProperties = false);
        /// <summary>
        /// 根据ID取得
        /// </summary>
        /// <param name="id"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        TEntity Get(object id, IDbTransaction transaction = null, int? commandTimeout = null);
        /// <summary>
        /// 删除
        /// </summary>
        /// <param name="entity"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        bool Delete(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null);
        /// <summary>
        /// 条件谓语删除
        /// </summary>
        /// <param name="predicate"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        bool Delete(object predicate, IDbTransaction transaction = null, int? commandTimeout = null);
        /// <summary>
        /// 分页查询
        /// </summary>
        /// <param name="predicate"></param>
        /// <param name="sort"></param>
        /// <param name="page"></param>
        /// <param name="resultsPerPage"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <param name="buffered"></param>
        /// <returns></returns>
        IEnumerable<TEntity> GetPage(object predicate, IList<ISort> sort, int page, int resultsPerPage, IDbTransaction transaction = null, int? commandTimeout = null, bool buffered = false);
    }
```

以上是基于Dapper-Extensions做了一个简单的仓储接口。

实现:

```C#
    /// <summary>
    /// Dapper仓储实现基类
    /// 使用数据库连接：KYE.CWMS.Common.Keys.ConnectionString
    /// 使用框架：https://github.com/tmsmith/Dapper-Extensions
    /// </summary>
    /// <typeparam name="TEntity"></typeparam>
    public class DapperRepository<TEntity> : IDapperRepository<TEntity> where TEntity : EntityBase
    {
        protected DapperRepository()
        {
        }


        public static IDapperRepository<TEntity> Create()
        {
            return new DapperRepository<TEntity>();
        }
        protected void UsingDb(Action<IDbConnection> action)
        {
            using (SqlConnection cn = new SqlConnection(Keys.ConnectionString))
            {
                cn.Open();
                action(cn);
                cn.Close();
            }
        }

        public IBetweenPredicate Between(Expression<Func<TEntity, object>> expression, BetweenValues values, bool not = false)
        {
            return Predicates.Between<TEntity>(expression, values, not);
        }
        public IExistsPredicate Exists<TSub>(IPredicate predicate, bool not = false) where TSub : class
        {
            return Predicates.Exists<TSub>(predicate, not);
        }
        public IFieldPredicate Field(Expression<Func<TEntity, object>> expression, Operator op, object value, bool not = false)
        {
            return Predicates.Field<TEntity>(expression, op, value, not);
        }
        public IPredicateGroup Group(GroupOperator op, params IPredicate[] predicate)
        {
            return Predicates.Group(op, predicate);
        }

        public ISort Sort(Expression<Func<TEntity, object>> expression, bool ascending = true)
        {
            return Predicates.Sort<TEntity>(expression, ascending);
        }


        public IEnumerable<TEntity> GetList(object predicate = null, IList<ISort> sort = null, IDbTransaction transaction = null, int? commandTimeout = null, bool buffered = false)
        {
            IEnumerable<TEntity> list = null;
            UsingDb(cn =>
            {
                var result = cn.GetList<TEntity>(predicate, sort, transaction, commandTimeout, buffered);
                list = result.ToList();
            });
            return list;
        }

        public int Count(object predicate, IDbTransaction transaction = null, int? commandTimeout = null)
        {
            int count = 0;
            UsingDb(cn =>
            {
                count = cn.Count<TEntity>(predicate, transaction, commandTimeout);
            });
            return count;
        }

        public void Insert(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null)
        {
            dynamic result = null;
            UsingDb(cn =>
            {
                result = cn.Insert<TEntity>(entity, transaction, commandTimeout);
            });

        }
        /// <summary>
        /// 修改
        /// </summary>
        /// <param name="entity"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <param name="ignoreAllKeyProperties"></param>
        /// <returns></returns>
        public bool Update(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null, bool ignoreAllKeyProperties = false)
        {
            bool result = false;
            UsingDb(cn =>
            {
                result = cn.Update(entity, transaction, commandTimeout, ignoreAllKeyProperties);
            });
            return result;
        }
        /// <summary>
        /// 根据ID获取一个实体
        /// </summary>
        /// <param name="id"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        public TEntity Get(object id, IDbTransaction transaction = null, int? commandTimeout = null)
        {
            TEntity entity = null;
            UsingDb(cn =>
            {
                entity = cn.Get<TEntity>(id, transaction, commandTimeout);
            });
            return entity;
        }
        /// <summary>
        /// 
        /// </summary>
        /// <param name="entity"></param>
        /// <param name="transaction"></param>
        /// <param name="commandTimeout"></param>
        /// <returns></returns>
        public bool Delete(TEntity entity, IDbTransaction transaction = null, int? commandTimeout = null)
        {
            bool result = false;
            UsingDb(cn =>
            {
                result = cn.Delete<TEntity>(entity, transaction, commandTimeout);
            });
            return result;
        }
        public bool Delete(object predicate, IDbTransaction transaction = null, int? commandTimeout = null)
        {
            bool result = false;
            UsingDb(cn =>
            {
                result = cn.Delete<TEntity>(predicate, transaction, commandTimeout);
            });
            return result;
        }

        public IEnumerable<TEntity> GetPage(object predicate, IList<ISort> sort, int page, int resultsPerPage, IDbTransaction transaction = null, int? commandTimeout = null, bool buffered = false)
        {
            IEnumerable<TEntity> result = null;
            UsingDb(cn =>
            {
                result = cn.GetPage<TEntity>(predicate, sort, page, resultsPerPage, transaction, commandTimeout, buffered).ToList();
            });
            return result;
        }

    }
```

## 使用IDapperRepository

好了，根据Dapper-Extensions文档，我需要写一个实体类，

```C#
using DapperExtensions.Mapper;
using System;

namespace KYE.CWMS.Local.DAL.Entities
{
    public class WMSQCAssembleMapper : ClassMapper<WMSQCAssemble>
    {
        public WMSQCAssembleMapper()
        {
            //指定表名称
            Table("TA_WMSQCAssemble");
            //主键
            Map(x => x.UniqueID).Key(KeyType.Identity);
            //自动映射
            AutoMap();
        }
    }
    public class WMSQCAssemble : Repositorys.Core.EntityBase
    {

        /// <summary>
        /// 0.组盘中 1.已释放
        /// </summary>
        public byte Status { get; set; }
        public DateTime? SaveTime { get; set; }
        public DateTime? MoveTime { get; set; }
        public long UniqueID { get; set; }
        public string BoxNo { get; set; }
        public string SKU { get; set; }
        public string PalletNo { get; set; }
        public string Position { get; set; }
        public string DeskNo { get; set; }
        public string OperateBy { get; set; }
        public string Forward { get; set; }
        public string MoveBy { get; set; }
        public string Addre { get; set; }
        public string ForwardArea { get; set; }

    }
}
```

上面是TA_WMSQCAssemble表映射成实体

接下来写一个测试

```C#
private IDapperRepository<WMSQCAssemble> WMSQCAssembleRepository;

public Repository_Tests()
{
    //创建 IDapperRepository<WMSQCAssemble>
    WMSQCAssembleRepository = DapperRepository<WMSQCAssemble>.Create();
}
```

```C#
        [Fact(DisplayName = "Insert_Test")]
        public void Insert_Test()
        {
            //造假数据
            var faker = new Faker<WMSQCAssemble>()
                .RuleFor(x => x.Addre, x => x.Address.ZipCode());
            var entity = faker.Generate();
            entity.PalletNo = "joventest11";

            //插入实体
            WMSQCAssembleRepository.Insert(entity);

            //获取数量
            var count = WMSQCAssembleRepository.Count(WMSQCAssembleRepository.Field(x => x.PalletNo, Operator.Eq, "joventest11"));

            //断言
            count.ShouldBe(1);
        }
```

好了，基本我是使用IDapperRepository<WMSQCAssemble\>就可以对表TA_WMSQCAssemble做简单的CRUD

总算摆脱了写简单的sql

## API V2 的初步版本

然后接下来，我不想使用之前的方式写接口

我想拥有一个明确的Input以及Output，来表达接个接口传参跟返回结果

```C#
    /// <summary>
    /// 传入参数
    /// </summary>
    public class IntoWarehouseAssemblyDishInput
    {

        /// <summary>
        /// 目标去向
        /// </summary>
        public string TargetDestination { get; set; }
        /// <summary>
        /// 托盘号
        /// </summary>
        public string PalletNo { get; set; }
    }
```

```C#
    /// <summary>
    /// 返回结果
    /// </summary>
    public class IntoWarehouseAssemblyDishOutput
    {
        /// <summary>
        /// 可用的
        /// </summary>
        public bool IsUsable { get; set; }
        /// <summary>
        /// 托盘是空的
        /// </summary>
        public bool IsEmpty { get; set; }
    }
```

那么我的接口应该是这样写

```C#
    public class AssemblyDishController : APIV2ControllerBase, IAssemblyDishAppService
    {
        private readonly IDapperRepository<WMSQCAssemble> assembleRepository;
        public AssemblyDishController()
        {
            //创建仓储对象
            assembleRepository = DapperRepository<WMSQCAssemble>.Create();
        }
        /// <summary>
        /// 入库组盘检查
        /// </summary>
        /// <param name="input">入参</param>
        /// <returns>结果</returns>
        [HttpPost]
        public IntoWarehouseAssemblyDishOutput IntoWarehouseAssemblyDishCheck(IntoWarehouseAssemblyDishInput input)
        {
            // 查字段:Status = 0
            var statusField = assembleRepository.Field(x => x.Status, DapperExtensions.Operator.Eq, 0);
            // 查字段:PalletNo = input.PalletNo
            var palletNoField = assembleRepository.Field(x => x.PalletNo, DapperExtensions.Operator.Eq, input.PalletNo);

            //对以上两个条件结合为组,并查询
            var list = assembleRepository.GetList(assembleRepository.Group(DapperExtensions.GroupOperator.And, statusField, palletNoField));

            //查询出来的数量为0,代表该托盘为空 状态为0，表示正在组盘,(托盘不是空的)
            if (list.Count() == 0)
            {
                return new IntoWarehouseAssemblyDishOutput() { IsEmpty = true, IsUsable = true };
            }

            //检查第一个箱子目标去向是否属于相同目的地
            if (SameDestination(input.TargetDestination, list.First().Forward))
            {
                return new IntoWarehouseAssemblyDishOutput() { IsEmpty = false, IsUsable = true };
            }
            return new IntoWarehouseAssemblyDishOutput() { IsEmpty = false, IsUsable = false };
        }
        /// <summary>
        /// 相同目的地
        /// </summary>
        private bool SameDestination(string target, string destination)
        {
            return (target.Contains("拆零") && destination.Contains("拆零")) || target.Equals(destination);
        }
    }
```

### 测试

使用postman,Post如下参数到<http://10.8.80.33:2211/AssemblyDish/IntoWarehouseAssemblyDishCheck>

```json
{
"Forward":"拆零区",
"PalletNo":"TP00221"
}
```

得到的Json,因需要兼容V1版本的返回结果，所以部分使用了小写开头

```json
{
    "result": {
        "IsUsable": true,
        "IsEmpty": true
    },
    "errCode": null,
    "retStatus": "1",
    "errMsg": null,
    "Success": true,
    "Error": null
}
```

如果要使用新的接口方式，需要继承：APIV2ControllerBase

```C#
    [Filters.ApiResultFilter]
    [Filters.ApiExceptionFilter]
    public abstract class APIV2ControllerBase : ApiController
    {

    }
```

原理：重写过滤器,同时也重写了异常过滤器（可能过滤器没有考虑到那么完整,需要大家继续完善）

好处如下

1. 统一返回结果
2. 避免滥用try.catch(重写了异常过滤器)
3. 使用UserFriendlyException,来为前端提供友好提示
4. 还有其它好处不一一列举