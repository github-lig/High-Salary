FactoryBean - 作用、用法。  
BeanFactory - 接口，为ioc容器的实现提供了规范。更多的是使用其子类ApplicationContext  


## BeanFactory
访问Spring bean容器的根接口，为Ioc容器的实现提供了规范。下面是其接口中定义的方法。
	
	boolean containsBean(String name); //返回Ioc容器中是否包含名称为name的Bean实例
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException; //返回bean是否是单例
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException; //判断bean的类型
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException; 
	Class<?> getType(String name) throws NoSuchBeanDefinitionException; //获取bean的类型
	Object getBean(String name) throws BeansException; //获取bean
	...
	
## FactoryBean
FactoryBean也是一种SpringBean，它与普通的Bean不同，是用来生成或修饰对象生成的工厂Bean。其返回的Bean不是对象的实例，而是其getObject方法返回的对象，是否单例是由isSingleton方法的返回值决定。  
其生成的Bean也是由BeanFactory管理的。  
