java.util.concurrent.locks.LockSupport

类描述：Basic thread blocking primitives for creating locks and other synchronization classes  

- 用于创建锁和其他同步类的基本线程阻塞原语，用来阻塞和唤醒线程。  
- 每个使用LockSupport的线程都会与一个许可证关联，如果该许可证可用，则调用pack()方法将立即返回，否则可能会阻塞（中断的情况下也会返回）。如果许可证不可用，则可以调用unpack()使其可用。需要注意的是许可不可重入，比如调用两次unpack,再调用两次pack方法，那么该线程依然会被阻塞。  

----------

void park()：阻塞当前线程，如果调用unpark方法或者当前线程被中断，从能从park()方法中返回。  
    
	public static void park() {
    	UNSAFE.park(false, 0L);
	}
我们可以看到park方法最终调用了UNSAFE类中的park方法，下面对UNSAFE.park()方法进行分析。
	
	public native void park(boolean isAbsolute, long time);

UNSAFE.park()是一个native方法，会阻塞当前线程，isAbsolute参数代表是否是绝对时间，time参数代表的是等待时间值，单位为ms。  
只有当一下4中情况中的一种发生时，该方法才会返回。  
1. 与pack对应的unpack执行或已经执行时，“已经执行”是指unpark先执行，然后再执行park的情况。  
2. 线程被中断。  
3. 等待完time参数指定的毫秒数时。  
4. 异常现象发生时，这个异常现象没有任何原因。  

----------

void park(Object blocker)：阻塞当前线程，并设置阻塞该线程的阻塞对象（可以用来标记该线程是被谁阻塞的或者阻塞的原因，用于线程监控和分析工具来分析原因）
	
	public static void park(Object blocker) {
	    Thread t = Thread.currentThread();
	    setBlocker(t, blocker);
	    UNSAFE.park(false, 0L);
	    setBlocker(t, null);
	}
下面来看一下setBlocker方法。
	
	private static void setBlocker(Thread t, Object arg) {
	    // Even though volatile, hotspot doesn't need a write barrier here.
	    UNSAFE.putObject(t, parkBlockerOffset, arg);
	}
该方法调用UNSAFE.putObject方法，putObject是一个native方法，通过直接操作内存的方式对某个对象设置属性值。其中我们看到传入的“parkBlockerOffset”，该变量代表Thread对象中“parkBlocker”在内存中相对于对象首地址的偏移量
	
	private static final long parkBlockerOffset;
	parkBlockerOffset = UNSAFE.objectFieldOffset(tk.getDeclaredField("parkBlocker"));
可以通过调用LockSupport.getBlocker(Thread t)来获取parkBlocker的快照信息。

----------
其余的方法的实现在上述描述中均有涉及。