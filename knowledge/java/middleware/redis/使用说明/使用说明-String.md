##### 阅读该篇文章之前，需了解[使用说明](使用说明.md)中的内容
> 注：本篇文章中使用的redis客户端均为Jedis-Version3.1，且SpringTemplate的序列化方式为key、hashKey设置为StringRedisSerializer，value、hashValue设置为GenericToStringSerializer  
> 本文中示例只是列出部分情况。

string类型是Redis最常用的数据结构。其可以用来存储简单的字符串，也可以存储对象的Json串，如果value是一个整数，还可以使用自增操作实现简单的计数器功能。

string在redis中的内部实现是字符数组。内部结构可以类比于java中的ArrayList，是可以修改的字符串，采用预分配冗余空间的方式来减少内存的频繁分配。当字符串的长度小于1MB时，扩容都是加倍现有的空间，大于1MB时，每次最多扩容1MB。需要注意的是，字符串的最大长度为512MB。

## set key value-设置指定key的值
set key value [expiration EX seconds|PX milliseconds] [NX|XX]。EX代表时间单位为秒，PX为毫秒。NX代表不存在时set，XX代表存在时set。    
setnx key value   
setex key seconds value  
psetex key mulliseconds value	

	/** 设置成功返回OK，失败返回null */
    jedis.set("name", "coding life"); //设置name值为coding life
    //String set(String key, String value, SetParams params)
    jedis.set("name", "coding", new SetParams().ex(60).xx()); //当name值存在时，设置name的值为coding，有效期为60s
	jedis.setnx("name", "setnx"); //当name值不存在时，设置name的值为setnx
    jedis.setex("name", 10, "setex"); //设置name值为setex，且有效期为10s
    jedis.psetex("name", 500L, "psetex"); //设置name值为psetex，且有效期为500毫秒


	/** 无返回值 */
	//绑定key值。name相当于redisTemplate.opsForValue(key)
    BoundValueOperations<String, Object> name = redisTemplate.boundValueOps("name");
    name.set("cli"); //设置name值为cli
    name.set("cli", 1000L, TimeUnit.SECONDS); //设置name值为cli且缓存时间为1000s
    name.set("cli", Duration.ofSeconds(100L)); //设置name值为cli且缓存时间为100s

	/** 设置成功返回true，失败返回false */
    name.setIfAbsent("cli"); //name不存在时，设置name值为cli
    name.setIfAbsent("cli", 1000L, TimeUnit.SECONDS); //name不存在时，设置name值为cli且缓存时间为1000s
    name.setIfAbsent("cli", Duration.ofSeconds(100L)); //name不存在时，设置name值为cli且缓存时间为100s

	/** 设置成功返回true，失败返回false */
    name.setIfPresent("cli"); //name存在时，设置name值为cli
    name.setIfPresent("cli", 1000L, TimeUnit.SECONDS); //name存在时，设置name值为cli且缓存时间为1000s
    name.setIfPresent("cli", Duration.ofSeconds(100L)); //name存在时，设置name值为cli且缓存时间为100s	

## get key-获取指定key的字符串值，如果key不存在，返回null
	
	jedis.get("name"); //返回与键name相关联的字符串值。
    jedis.get("empty-name"); //返回null
	jedis.get("lname"); //获取key的值是非字符串的话，报JedisDataException: WRONGTYPE Operation against a key holding the wrong kind of value
	

	redisTemplate.opsForValue().get("name"); //返回与键name相关联的字符串值。
	
## getset key value-将key的值设置为value，并返回被设置之前的值。如果设置之前key不存在，则返回null
	
	jedis.getSet("name", "getset"); 
	

	redisTemplate.opsForValue().getAndSet("name", "getAndSet");

## strlen key-返回键key存储的字符串值的长度
	
	jedis.strlen("name"); 
	
	
	redisTemplate.opsForValue().size("name");

## append key value-键存在并且它的值是字符串，将value值追加到现有值的末尾。如果不存在，相当于set
		
	jedis.append("name", "append"); 

	
	redisTemplate.opsForValue().append("name", "append"); 

## settange key offset value-从偏移量offset开始，用value参数将覆写键key存储的字符串值。如果key不存在，当成空白字符串处理。该命令返回修改之后字符串的长度
setrange命令会确保字符串足够长以便将value设置到指定的偏移量上，如果key原本储存的字符串长度比偏移量小（比如字符串只有5个字符长，但要设置的偏移量为10），那么字符串和偏移量之间的空白将用零字节（"\x00"）进行填充。	

	127.0.0.1:6379> exists name
	(integer) 0
	127.0.0.1:6379> setrange name 3 setrange
	(integer) 11
	127.0.0.1:6379> get name
	"\x00\x00\x00setrange"
	127.0.0.1:6379> setrange name 1 three
	(integer) 11
	127.0.0.1:6379> get name
	"\x00threerange"
	

	jedis.setrange("name", 3, "setrange"); //jedis.setrange(key, offset, value)
	
	
	redisTemplate.opsForValue().set("name", "setrange", 3); //set(key, value, offset)

## getrange key start end-返回键key储存的字符串的指定部分。截取范围为start和end（包含start和end）
负偏移量表示从字符串的末尾开始，例如-1代表最后一个字符，-2代表倒数第二个字符。  
getrange通过保证子字符串的范围不超过实际字符串的值域来处理超出范围的值域请求
	
	127.0.0.1:6379> set name getrange
	OK
	127.0.0.1:6379> getrange name 0 2
	"get"
	127.0.0.1:6379> getrange name 3 100
	"range"
	127.0.0.1:6379> getrange name -3 -1
	"nge"
	127.0.0.1:6379> setrange name -1 -5 //不支持回绕操作
	(error) ERR offset is out of range
	127.0.0.1:6379> getrange name 0 -1
	"getrange"
	
	
	jedis.getrange("name", 0, 1);

	
	redisTemplate.opsForValue().get("name", 0L, 1L); //get(key, start, end)

## 	incr key-为键key所储存的数字值加一。返回增加后的值
如果key不存在，会先将值初始化为0，然后再执行incr命令  
如果key存储的值不是个数字，会报错  
本操作的值限制在64位（bit）(long)有符号数字表示之内	

	127.0.0.1:9527> set notint not
	OK
	127.0.0.1:9527> inct notint
	(error) ERR unknown command `inct`, with args beginning with: `notint`,


	jedis.incr("name");


	redisTemplate.opsForValue().increment("name");


## setbit-bitmap位图



