IOC容器体系结构
===
声明 本文内容摘录
http://blog.csdn.net/chjttony/article/category/1239946

BeanFactory类结构体系
====

BeanFactory接口及其子类定义了spring IOC
容器体系结构，由于BeanFactory体系非常
复杂，参考下图链接
http://hi.csdn.net/
attachment/201103/14/0_1300091585KhGe.gif

ApplicationContext结构体系
====

ApplicationContext接口是一个BeanFactory
基础上封装了更多功能的，最为常用的IOC
容器，包含两个子接口ConfigurableAppli
cationContext  WebApplicationContext
参考图片链接
http://hi.csdn.net/attachment/201103/14/0_1300091636JbBr.gif
http://hi.csdn.net/attachment/201103/14/0_1300091650ZB6m.gif
http://hi.csdn.net/attachment/201103/14/0_1300091656DZ1g.gif
http://hi.csdn.net/attachment/201103/14/0_1300091661eJVW.gif

UML类图
http://hi.csdn.net/attachment/201103/16/0_1300259400APII.gif

BeanFactory
====

BeanFactory接口定义了IOC容器的基本功能
规范，所定义的IOC容器的主要方法如下：

1、Object getBean(String name) throws 
BeansException;
通过该方法取得IOC容器中管理的bean，bean
的取得是通过指定名字来进行所引的

2、<T> T getBean(String name, Class<T>
 requiredType) throws BeansException
 根据指定的名字和类型取得IOC容器中管理
 的Bean，提供了一个类型安全验证机制

3、<T> T getBean(Class<T> requiredType)
throws BeansException 
根据制定类型取得IOC容器中管理的Bean，该
方法根据指定的类型调用ListableBeanFact
ory(BeanFactory)取得bean方法，但是也可
能根据给定的类型调用通过名字取得bean方
法

4、Object getBean(String name, Object
... args) throws BeansException
重载getBean(String name)方法，可变参数
主要指定是否显示使用静态工厂方法创建
一个原型bean

5、Boolean containsBean(String name)

6、boolean isSingleton(String name) throws NoSuchBeanDefinitionException

7、boolean isPrototype(String name) throws NoSuchBeanDefinitionException

8、boolean isTypeMatch(String name, Class targetType)throws NoSuchBeanDefinitionException

9、Class<?> getType(String name) throws NoSuchBeanDefinitionException

10、Class<?> getType(String name) throws NoSuchBeanDefinitionException


XmlBeanFactory只提供基本的IOC容器功能
主要以XML形式定义的BeanDefinition

		public class XmlBeanFactory extends DefaultListableBeanFactory {  
		//读取XML形式的Bean定义，然后解析XML生成IoC管理的Bean      
		private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);  
		    //Resouce是Spring中用来封装IO操作的接口  
		    public XmlBeanFactory(Resource resource) throws BeansException {  
		        this(resource, null);  
		    }  
		//调用父类的构造方法，同时调用loadBeanDefinitions方法从指定XML文件加载Bean定义  
		    public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {  
		        super(parentBeanFactory);  
		        this.reader.loadBeanDefinitions(resource);  
		    }  
		}   

XmlBeanFactory实现IoC容器的基本过程

		//创建IoC容器管理的Bean的xml配置文件，即定位资源  
		ClassPathResource resource = new ClassPathResource(“beans.xml”);  
		//创建BeanFactory  
		DefaultListableBeanFactory factory = new DefaultListableBeanFactory ();  
		//创键Bean定义读取器  
		XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);  
		//使用Bean定义读取器读入Bean配置信息，即载入配置  
		reader.loadBeanDefinitions(resource);  	

ApplicationContext
====

		public interface ApplicationContext extends ListableBeanFactory, HierarchicalBeanFactory,  
		        MessageSource, ApplicationEventPublisher, ResourcePatternResolver {  
		    //获取ApplicationContext的id  
		    String getId();  
		    //获取ApplicationContext的displayName  
		    String getDisplayName();  
		    //获取ApplicationContext第一次加载的时间戳  
		    long getStartupDate();  
		    //获取ApplicationContext容器的父容器  
		    ApplicationContext getParent();  
		    //获取自动装配功能的BeanFactory  
		    AutowireCapableBeanFactory getAutowireCapableBeanFactory() throws IllegalStateException;  
		}

ApplicationContext的特性
====

1、支持不同的信息源
其扩展了MessageSource接口，可以支持
国际化的实现

2、其继承了DefaultResourceLoader的子类
通过ResourceLoader和Resource的支持
ApplicationContext可以加载不同地方的
bean定义资源文件

3、支持运用事件
其继承了ApplicationEventPublisher接口
在程序上下文中引入了事件机制，这些
事件和bean生命周期的结合为bean提供了
便利