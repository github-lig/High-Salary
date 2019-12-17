##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis-Version3.1，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer

string类型是Redis最常用的数据结构。其可以用来存储简单的字符串，也可以存储对象的Json串，如果value是一个整数，还可以使用自增操作实现简单的计数器功能。

string在redis中的内部实现是字符数组。内部结构可以类比于java中的ArrayList，是可以修改的字符串，采用预分配冗余空间的方式来减少内存的频繁分配。当字符串的长度小于1MB时，扩容都是加倍现有的空间，大于1MB时，每次最多扩容1MB。需要注意的是，字符串的最大长度为512MB。

## set-设置指定key的值
set key value [expiration EX seconds|PX milliseconds] [NX|XX]。EX代表时间单位为秒，PX为毫秒。NX代表不存在时set，XX代表存在时set。  
	
	//String set(String key, String value)
    jedis.set("name", "coding life"); //设置name值为coding life
    //String set(String key, String value, SetParams params)
    jedis.set("name", "coding", new SetParams().ex(60).xx()); //当name值存在时，设置name的值为coding，有效期为60s






