	import lombok.NonNull;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.data.redis.connection.RedisConnection;
	import org.springframework.data.redis.connection.RedisStringCommands;
	import org.springframework.data.redis.core.StringRedisTemplate;
	import org.springframework.data.redis.core.script.DefaultRedisScript;
	import org.springframework.data.redis.core.types.Expiration;
	import org.springframework.stereotype.Component;
	import org.springframework.util.Assert;
	
	import java.nio.charset.StandardCharsets;
	import java.util.Arrays;
	import java.util.concurrent.TimeUnit;
	
	/**
	 * @Description: redis分布式锁实现
	 * @Author: LiG
	 * @Date: 2019/6/18 9:22
	 * @Version: 1.0
	 */
	@Component
	public class DistributedLockUtil {
	    @Autowired
	    private StringRedisTemplate stringRedisTemplate;
	
	    @Autowired
	    private DefaultRedisScript<Long> defaultRedisScript;
	
	    private static final Long UNLOCK_SUCCESS = 1L;
	
	    /**
	     * 分布式锁前缀
	     */
	    public static final String LOCK_PREFIX = "distributed:lock:";
	
	    /**
	     *  获取分布式锁，失败直接返回
	     * @param key 分布式锁的key（后缀）
	     * @param value 随机value
	     * @param lockTime 锁定失效时间（毫秒）
	     * @return
	     */
	    public boolean lock(@NonNull String key, @NonNull String value, @NonNull Long lockTime) {
	        return lock(key, value, lockTime, 0L, 0L);
	    }
	
	    /**
	     *  获取分布式锁，设置获取超时时间，超时返回false
	     * @param key 分布式锁的key （后缀）
	     * @param value 随机value
	     * @param lockTime 锁定失效时间（毫秒）
	     * @param waitTime 最长等待时间（毫秒）
	     * @param sleepTime 重试休眠时间（毫秒）
	     * @return
	     */
	    public boolean lock(@NonNull String key, @NonNull String value, @NonNull Long lockTime, Long waitTime, Long sleepTime) {
	        String lock = LOCK_PREFIX + key;
	        Long end = System.currentTimeMillis() + waitTime;
	        return stringRedisTemplate.execute((RedisConnection connection) -> {
	            while (true) {
	                boolean acquire = connection.set(lock.getBytes(StandardCharsets.UTF_8), value.getBytes(StandardCharsets.UTF_8),
	                        Expiration.milliseconds(lockTime), RedisStringCommands.SetOption.SET_IF_ABSENT);
	                if (acquire) {
	                    return true;
	                } else if(System.currentTimeMillis() > end) {
	                    return false;
	                }
	                try {
	                    //休眠
	                    TimeUnit.MILLISECONDS.sleep(sleepTime);
	                } catch (InterruptedException e) {
	                    return false;
	                }
	            }
	        });
	    }
	
	
	    /**
	     *  删除分布式锁
	     * @param key 分布式锁的key （后缀）
	     * @param value 加锁时传入的随机值
	     * @return
	     */
	    public boolean unLock(@NonNull String key, @NonNull String value) {
	        String lock = LOCK_PREFIX + key;
	        long result = stringRedisTemplate.execute(defaultRedisScript, Arrays.asList(lock, value));
	        return UNLOCK_SUCCESS.equals(result);
	    }
	
	    @Bean
	    public DefaultRedisScript<Long> defaultRedisScript() {
	        DefaultRedisScript<Long> defaultRedisScript = new DefaultRedisScript<>();
	        defaultRedisScript.setResultType(Long.class);
	        defaultRedisScript.setScriptText("if redis.call('get', KEYS[1]) == KEYS[2] then return redis.call('del', KEYS[1]) else return 0 end");
	        return defaultRedisScript;
	    }
	}