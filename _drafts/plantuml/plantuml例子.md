# 使用plantuml例子

PlantUML官网：<http://plantuml.com>

## UML表现

```plantuml
@startuml

note "UML表现接口，类，抽象类，枚举形式" as N1
class 类{
}
interface 接口{
}
abstract 抽象类{
}
abstract class 抽象类2{
}
enum 枚举{
  A=1,
  B=2,
}
abstract class AbstractList
abstract AbstractCollection
interface List
annotation SuppressWarnings
enum TimeUnit
@enduml
```

## 类图

```plantuml
类名 <|-- 类名2
class 类名{
  ..静态..
  {static} String id
  ..
  string Name
  __
  Task<int>  methods()
  -void private()
  #void protected()
  ~void packagePrivate()
  +void public()
  --抽象--
  {abstract} void methods()
}
class 类名2{
  ..静态..
  {static} String id
  ..
  string Name
  __
  Task<int>  methods()
  -void private()
  #void protected()
  ~void packagePrivate()
  +void public()
  --抽象--
  {abstract} void methods()
}
```

> asdasdv

```plantuml
@startuml

Class01 "1" *-- "many" Class02 : contains
Class03 o-- Class04 : aggregation

Class05 --> "1" Class06

@enduml

```

### 关系描述

```plantuml
@startuml
interface 接口
class 人{
  +List<tool> tools{get;set;}
}

汽车 <|-- 吉普车: **Extension** //泛化,适用于继承
接口 <|.. 实现类: **实现**
tool "0..*" <--  "1" 人:**关联**
Class15 ..> Class16:**依赖**
@enduml
```

### 箭头描述

```plantuml
@startuml
汽车 <|-- 吉普车: **Extension** //泛化,适用于继承
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 -- Class10
@enduml
```

```plantuml
@startuml
interface Class11{

}
Class11 <|.. Class12:**实现**
Class13 "1" --> "0..*" Class14:**关联**
Class15 ..> Class16:**依赖**
Class17 ..|> Class18
Class19 <--* Class20
@enduml
```

```plantuml
@startuml
Class21 #-- Class22
Class23 x-- Class24
Class25 }-- Class26
Class27 +-- Class28
Class29 ^-- Class30
@enduml
```

```plantuml
@startuml

Class01 "1" *-- "many" Class02 : contains

Class03 o-- Class04 : aggregation

Class05 --> "1" Class06

@enduml
```

```plantuml
@startuml
class Object << general >>
Object <|--- ArrayList

note top of Object : In java, every class\nextends this one.

note "This is a floating note" as N1
note "This note is connected\nto several objects." as N2
Object .. N2
N2 .. ArrayList

class Foo
note left: On last defined class

@enduml