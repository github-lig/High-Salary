##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis-Version3.1，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer  
> 本文中示例只是列出部分情况。

相当于HashMap，无序字典。内部存储键值对。结构和HashMap一致，数组+链表的组合