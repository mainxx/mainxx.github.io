# Collect设计_UML学习之路

```plantuml

@startuml
interface  ICollectBeganManagerFactory<T>{
   ..创建一个ICollectBegan..
   IDisposableDependencyObjectWrapper<ICollectBegan> Create(CollectTypes collectClassify);
}
note right:采集开始管理工厂接口

interface IDisposableDependencyObjectWrapper{
    Object Object{get;}
}
note left of IDisposableDependencyObjectWrapper:ABP内置接口：对象包装器

IDisposableDependencyObjectWrapper <.. ICollectBeganManagerFactory : 依赖

class CollectBeganManagerFactory<T>

ICollectBeganManagerFactory <|.. CollectBeganManagerFactory : 实现

interface ICollectBegan{
    --校验ExtensionData--
    bool VerifyExtensionData(BaseCollectSource collectSource);
    ..执行采集函数..
    T Begin<T, TPrimaryKey>(BaseCollectSource<TPrimaryKey> collectSource);
    Task<T> BeginAsync<T, TPrimaryKey>(BaseCollectSource<TPrimaryKey> collectSource);
}
note right:采集开始接口

enum CollectTypes{
    HTTP=1,
}

abstract class BaseCollectSource<TPrimaryKey>{
    public CollectTypes CollectType { get; set; }
    public string ExtensionData { get; set; }
}
interface IExtendableObject
abstract class AuditedAggregateRoot
CollectTypes <.. BaseCollectSource : 依赖
AuditedAggregateRoot <|-- BaseCollectSource : 继承
IExtendableObject <|.. BaseCollectSource : 实现
BaseCollectSource <..  ICollectBegan : 依赖

note top of AuditedAggregateRoot:ABP带审计得聚合根对象


interface IHttpCollectBegan{
}
ICollectBegan <|--  IHttpCollectBegan : 继承

interface ITextCollectBegan
ICollectBegan <|--  ITextCollectBegan : 继承

IHttpCollectBegan <|.. HttpCollectBegan : 实现

class HttpExBaseCollectSource  <<BaseCollectSource>> {
    --设置ExtensionData必须数据--
    string Url
    string RequestType
    string ResultType
}

HttpExBaseCollectSource <.. IHttpCollectBegan

class CollectSource{
- Title:string
- Desc:string
- CollectPlanId:long
- RunStatus:RunStatuses
- CollectOption:CollectOptions
- IsSaveResult:bool
- Source:string
- ParserName:string
}
note left of CollectSource
     采集源
end note

BaseCollectSource <|-- CollectSource : 继承

@enduml
```
