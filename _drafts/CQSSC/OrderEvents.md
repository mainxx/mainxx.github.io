# Order事件设计

* CreateCaipiaoOrderEventData:订单创建事件
* CaipiaoOrderWiningEventData:订单中奖事件
* CaipiaoOrderOpenedEventData:开奖后 事件
* CaipiaoOrderLosingEventData:订单不中奖事件
* CaipiaoOrderCanceledEventData:订单取消事件

## 关联

* 采集系统定时采集到结果：将会创建 CqsscOpenRecord
* 订阅CqsscOpenRecord创建事件，调用CaipiaoOrder.Open()函数[后期优化]
* 其它领域，订阅CaipiaoOrder各个事件触发

