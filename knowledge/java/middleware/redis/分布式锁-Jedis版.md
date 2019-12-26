	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.stereotype.Component;
	import redis.clients.jedis.Jedis;
	
	import java.util.Collections;
	import java.util.Random;

	public class DistributedLockUtil {
	    private static final Long UNLOCK_SUCCESS = 1L;
	    private static final String SET_IF_NOT_EXIST = "NX";
	    private static final String SET_WITH_EXPIRE_TIME = "PX";
	
	    /**
	     *  获取分布式锁，失败直接返回
	     * @param key 分布式锁key
	     * @param value 最好是随机value
	     * @param lockTime 锁超时时间 (毫秒)
	     * @return
	     */
	    public boolean lock(String key, String value, Long lockTime) {
	        return lock(key, value, lockTime, 0L, 0L);
	    }
	
	    /**
	     * 获取分布式锁，设置获取超时时间，超时返回false
	     * @param key 分布式锁key
	     * @param value 最好是随机value
	     * @param lockTime 锁超时时间 (毫秒)
	     * @param waitTime  最长等待时间（尝试获取锁的最长时间） (毫秒)
	     * @param sleepTime 重试间隔 (毫秒)
	     * @return
	     */
	    public boolean lock(String key, String value, Long lockTime, Long waitTime, Long sleepTime) {
	        try (Jedis jedis = new Jedis("127.0.0.1")) {
	            Long endTime = System.currentTimeMillis() + sleepTime;
	
	            while (true) {
	                String result = jedis.set(key, value, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, lockTime);
	
	                if ("OK".equalsIgnoreCase(result)) {
	                    return true;
	                } else if (System.currentTimeMillis() > endTime) {
	                    return false;
	                }
	
	                try {
	                    Thread.sleep(sleepTime);
	                } catch (InterruptedException e) {
	                    e.printStackTrace();
	                }
	            }
	        }
	    }
	
	    /**
	     * 解锁
	     * @param key 分布式锁 key
	     * @param value 加锁时所用的value
	     * @return
	     */
	    public boolean unLock(String key, String value) {
	        try (Jedis jedis = new Jedis("127.0.0.1")) {
	            String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
	            Object result = jedis.eval(script, Collections.singletonList(key), Collections.singletonList(value));
	
	            if (UNLOCK_SUCCESS.equals(result)) {
	                return true;
	            }
	            return false;
	        }
	    }
	
	    /**
	     *  随机value的简单实现
	     * @return
	     */
	    public static String randomValue() {
	        return new Random(33).nextInt(99) + "";
	    }
	}