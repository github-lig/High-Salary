> ThreadLocal，线程变量，是一个以ThreadLocal对象为键，任意对象为值得存储结构。这个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。

首先看一个ThreadLocal的例子  
	
	public class SimpleThreadLocal {
	    private static Random random = new Random(99);
	    private static ThreadLocal<Integer> simpleThreadLocal = ThreadLocal.withInitial(() -> random.nextInt());
	
	    public static Integer get() {
	        return simpleThreadLocal.get();
	    }
	
	    public static void incr() {
	        simpleThreadLocal.set(simpleThreadLocal.get() + 1);
	    }
	
	    public static void main(String[] args) {
	        Thread t1 = new Thread(new MyRunnable(1));
	        Thread t2 = new Thread(new MyRunnable(2));
	
	        t1.start();
	        t2.start();
	    }
	}

	class MyRunnable implements Runnable {
	    private int id;
	
	    public MyRunnable(int id) {
	        this.id = id;
	    }
	
	    @Override
	    public void run() {
	        System.out.println("初始化值:" + this);
	        SimpleThreadLocal.incr();
	        System.out.println("递增:" + this);
	    }
	
	    @Override
	    public String toString() {
	        return "MyRunnable{" +
	                "id=" + id + ":" + SimpleThreadLocal.get() +
	                '}';
	    }
	}

运行结果
	
	初始化值:MyRunnable{id=1:-1192035722}
	递增:MyRunnable{id=1:-1192035721}
	初始化值:MyRunnable{id=2:1672896916}
	递增:MyRunnable{id=2:1672896917}

由此，我们可以看到同一个ThreadLocal变量被不同的线程使用时，初始化值不同，并且在修改的时候互不影响。    

---
下面是对ThreadLocal的解析

> java.lang.ThreadLocal

	
	public class ThreadLocal<T>
	public void set(T value)
	public T get()
首先我们看到ThreadLocal类是一个泛型类，从set和get方法可以看出，ThreadLocal线程本地变量实际存储的是我们在实例化的时候指定的泛型，故我们可以通过ThreadLocal存储任意类型的值。  
  
而作为一个变量，首先想到的肯定是set和get方法，下面我们先看一下ThreadLocal类的set方法。  
	
	public void set(T value) {
        Thread t = Thread.currentThread(); //取得当前线程
        ThreadLocalMap map = getMap(t); //获取当前线程对象的ThreadLocalMap类型的变量
        if (map != null)	//如果当前线程对象的threadLocals变量不为null，set值
            map.set(this, value);
        else
            createMap(t, value); //如果为null，则为当前线程对象初始化一个ThreadLocalMap对象。
    }
	
	ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
由上可以看出，ThreadLocal是通过ThreadLocalMap进行值的存储，下面看一下ThreadLocalMap
>java.lang.ThreadLocal.ThreadLocalMap  

ThreadLocalMap是ThreadLocal的一个静态内部类

	
	static class ThreadLocalMap {
		static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
		
		private Entry[] table; //实际存储数据的结构
		
		ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
            table = new Entry[INITIAL_CAPACITY];
            int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
            table[i] = new Entry(firstKey, firstValue);
            size = 1;
            setThreshold(INITIAL_CAPACITY);
        }

		private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                    e.value = value;
                    return;
                }

                if (k == null) {
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold)
                rehash();
        }

从上述代码我们可以看出，ThreadLocalMap的实现类似于HashMap的思路（有兴趣的可以查看源码，上述只是一部分内容）。  
以Entry为数据节点，其中ThreadLocal对象为key，存储在ThreadLocalMap对象中的Entry[] table数组变量中。在set方法中，对相同key的value进行覆盖。  

接下来，我们来看一下get()方法  
	
	public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

从线程对象的ThreadLocalMap中取得以当前ThreadLoca为key的值，如果为null，则进行初始化。初始化方法为
	
	protected T initialValue() {
        return null;
    }
一般需要使用ThreadLocal的话，我们都会覆盖此方法来实现默认值的设置。

---
总结：ThradLocal线程本地变量，从名字中我们可以看出，其是线程安全的，而线程安全的原因在于，每一个线程Thread对象中均存在一个ThreadLocalMap的数据结构，当当前线程使用ThreadLocal变量时，会以该ThreadLocal为key存储到线程的ThreadLocalMap中，则其存储的value值只能被当前线程访问。  
  

---
在jdk1.8版本之后，ThreadLocal为我们提供了函数式的初始化方式。
	
	ThreadLocal<String> threadLocal = ThreadLocal.withInitial(() -> "初始值");

---
ThreadLocal的使用需要注意以下几点：  
- ThreadLocal变量类似于全局变量，它会降低代码的可重用性，并在类之间引入隐含的耦合性。  
- 线程复用的话会产生脏数据。由于线程池会复用Thread对象，那么与Thread绑定的静态ThreadLocal变量也会被复用。  
- 容易产生线程泄露的问题。每次使用完之后尽量通过remove()方法进行删除。
	
