##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer

String类型是Redis最常用的数据结构。其可以用来存储简单的字符串，也可以存储对象的Json串，如果value是一个整数，还可以使用自增操作实现简单的计数器功能。


set-设置指定key的值



