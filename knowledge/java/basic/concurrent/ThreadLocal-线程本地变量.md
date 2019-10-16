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

