##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis-Version3.1，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer  
> 本文中示例只是列出部分情况。

类似于SortedSet和HashMap的结合体。一方面是一个set，另一方面为每个value赋予一个socre，代表这个value的排序权重。内部实现是一种叫作跳跃列表的数据结构。