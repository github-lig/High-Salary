##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis-Version3.1，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer  
> 本文中示例只是列出部分情况。

相当于HashSet（基于HashMap实现），内部的键值对是无序的、唯一的。它的内部实现相当于一个特殊的HashMap，所有的Value值都是NULL。当集合中最后一个元素没移除之后，数据结构被自动删除，内部被回收。  
可以做一些差集、并集、交集的操作