# 首先回顾一下final的基本作用  #
在java中，final关键字代表“这是无法改变的”。可以被用来修饰对象、方法、域。  

**final域**，代表数据时恒定不变的。比如一个永不改变的编译时常量、一个运行时被初始化的值，但是不需要它被改变，编译器只需保证在使用前final域必须被初始化，这样就可以做到对于一个类中的final实例域在每个对象中又有所不同。  
对于编译时常量，编译器可以将该常量值代入到任何使用到它的地方，即在编译时只写计算，但是这种常量必须是基本数据类型（byte, boolean, short, char, int, float, long, double）。比如下述代码   
	
	我们定义了如下类，
	public class CompileTimeConstant {
	    private final int i = 1;
	    private int j = i + 1;
	
	    private int m = 1;
	    private int n = m + 1;
	}
	
将该类编译之后，通过工具查看反编译之后代码
	
	public class CompileTimeConstant {
	    private final int i = 1;
	    private int j = 2;
	    private int m = 1;
	    private int n;
	
	    public CompileTimeConstant() {
	        this.n = this.m + 1;
	    }
	}
而一个既是static又是final的域只占据一段不能改变的存储空间。  
对于基本类型，final使数值恒定不变；对于对象引用，final使引用恒定不变。一旦一个引用被初始化只想一个对象，就无法再把它改为指向另一个对象，然而对象其本身是可以被修改的。  

**final方法**，代表方法不可被覆盖。将方法锁定，确保在继承中使方法的行为保持不变，并且不会被覆盖。  
类中所有的private修饰的方法都隐式地指定为是final的。如下代码  
	
	public class FinalMethod {
	    void t1() {
	        System.out.println("FinalMethod : t1");
	    }
	
	    final void t2() {
	        System.out.println("FinalMethod : t2");
	    }
	
	    private void t3() {
	        System.out.println("FinalMethod : t3");
	    }
	}
我们建立一个子类继承上述FinalMethod，并试图重写t1、t2、t3三个方法  
	
	public class FinalMethodChild extends FinalMethod {
	    @Override
	    void t1() {
	        System.out.println("FinalMethodChild : t1");
	    }
	    
	    //'t2()' cannot override 't2()' in 'com.thinking.concurrent.thinkFinal.FinalMethod'; overridden method is final
	    void t2() {
	        System.out.println("FinalMethodChild : t2");
	    }
	
	    void t3() {
	        System.out.println("FinalMethodChild : t3");
	    }
	}
如上，对于t2()方法，子类中不可进行覆盖。对于t3()方法而言，根据我们上述的说法，private方法被隐式的指定为final，咋一看，发现t3()方法被覆盖，但是其实不然，覆盖，只有在某方法是基类的接口的一部分时才会出现，必须能将一个对象向上转换为其基类并调用相同的方法。如果某方法为private，它就不是基类接口的一部分，仅仅是一些隐藏于类中的程序代码。  
	
	public static void main(String[] args) {
	    FinalMethod finalMethod = new FinalMethodChild();
	    finalMethod.t1();
	}

	output：FinalMethodChild : t1


**final类**，表明该类不可继承。当某个类定义为final时，该类中的所有方法都被隐式的定义为final的，因为无法覆盖它们。


# final在并发中的作用 #
当我们需要安全发布一个域时，我们可以简单的将该域定义为final的，因为final的特性，在多线程的环境下，不会出现并发安全问题。

下面说一下final域的重排序规则  
首先，我们需要知道，当我们实例化一个对象并赋值给一个变量时，会分为三步，比如对于Instance instance = new Instance();可以分解为以下三步伪代码。  
	
	memory = allocate();  //1.分配对象的内存空间
	ctorInstance(memory); //2.初始化对象
	instance = memory;    //3.设置instance指向刚分配的内存地址
上述三行伪代码中的2和3之间，可能会被重排序。导致我们在并发环境下使用instance对象的时候，判断其不为空，但是使用的时候还没有初始化对象。  
比如我们在程序中如果需要单例对象，如果我们按照如下写法
	
	public class DoubleCheckedLocking {
	    private static Instance instance;
	    
	    public static Instance getInstance() {
	        if (instance == null) {
	            synchronized(DoubleCheckedLocking.class) {
	                if (instance == null) {
	                    instance = new Instance();
	                }
	            }
	        }
	        return instance;
	    }
	}
当程序执行到第5行时，instance判断不为null，但是此时，instance引用的对象可能还没有进行初始化。  
我们可以通过将instance变量设置为volatile来避免2和3的重排序。关于volatile以及基于类初始化的方式实现单例会在其它文章中说明。  

下面继续说一下final域。  
对于final域，编译器和处理器要遵守两个重排序规则。  
1. 在构造函数中对于一个final域的写入，与随后将这个被构造的对象赋值给一个引用变量，这两个操作之间不可以重排序。  
2. 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序。 
 
对于1来说，即使初始化对象和设置instance指向刚分配的内存地址之间发生了重排序，导致另一个线程看到一个只被部分构造的函数，但是final域肯定已经被初始化过了。  
对于2来说，由于初次读对象的引用和初次读对象包含的final域，这两个操作存在间接依赖关系，因此编译器不会重排序这两个操作。对于大部分处理器同样如此，但是少数处理器允许对这两个操作进行重排序。在此我们不做深入了解。  

在实际开发过程中，我们可以根据final的特性，将其应用到我们的代码中，比如以下样例。  
我们在接口中存储最后访问的用户的信息  
	
	public class CachedFactorizer implements Servlet {
	    private String name; //姓名
	    private int hash; //年龄
	    
	    public void service(ServletRequest req, ServletResponse resp) {
	        String name = req.getAttribute("name");
	        
	        if (name.equals(this.name)) {
	            System.out.println("name" + name + ",hash:" + this.hash);
	        }else {
	            this.name = name;
	            this.hash = name.hashCode();
	            System.out.println("name" + name + ",hash:" + this.hash);
	        }
	    }
	}
上述代码在多线程的情况下访问，可能会出现name与hash对应不上的情况。 
 
对于在访问和更新多个相关变量时出现的竞争条件问题（入上述name和hash两个相关变量），可以通过将这些变量全部保存在一个不可变对象中来消除，当线程获取了该对象的引用后，就不用担心另一个线程修改它，如果要更新这些变量，可以创建一个新的对象赋值给引用变量，引用变量需要设置为volatile来保持一致性。  
	
	public class CachedFactorizer implements Servlet {
	    private voiatile User user = new User(null, 0);
	    public void service(ServletRequest req, ServletResponse resp) {
	        String name = req.getAttribute("name");
	        int hash = 0;
	        if (hash = user.getHash(name) == 0) {
	            hash = name.hashCode();
	            user = new User(name, hash);
	        }
	        System.out.println("name" + name + ",hash:" + hash);
	    }
	    
	    class User {
	        private final String name; //姓名
	        private final int hash; //年龄
	        
	        public User(String name, int hash) {
	            this.name = name;
	            this.hash = hash;
	        }
	        public int getHash(String name) {
	            if (this.name == null || !this.name.equals(name)) {
	                return null;
	            }
	            return this.hash;
	        }
	    }
	}