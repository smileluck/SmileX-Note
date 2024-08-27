[TOC]

---

# 问题记录

### type longstr but current is none

lternateExchange是RabbitMQ中自定义的备用交换机的名称 

```shell
received the value 'dead_exchange' of type 'longstr' but current is none, class-id=50, method-id=10
```

原因：queue已经存在，但是启动时试图设定一个 x-dead-letter-exchange 参数，这和服务器上的定义不一样，server 不允许所以报错。

解决方案：删除 queue 重新 declare 则不会有问题。
