spring-basic
===

spring主要核心
====

1、IOC 以前传统的额Java开发模式中，当需要一个对象
时，我们就new或者getInstance等直接或者间接调用
构造方法构建一个对象，而在spring中，spring容器使
用工厂模式为我们创建了所需要的对象，使用的时候
不需要去创建，直接调用spring提供的对象即可。这
就是IOC的思想。实例化一个Java对象有三种方式：
使用类构造器、使用静态工厂方法、使用实例工厂
方法。

2、DI spring使用Java bean对象的set方法或者带参数
的构造方法为我们在创建所需对象时将属性自动设置
所需要的值过程就是依赖注入的基本思想

3、AOP  在面向对象编程(OOP)思想中，将事物纵向
抽象成一个个的对象。而在面向切面编程中，将一个个
对象某些类似的方面横向抽象成一个切面，对这个切面
进行一些如权限验证、事务管理、记录日志等公用操作
处理的过程就是面向切面编程的思想

在spring中，所有管理的对象都是Javabean对象，而
beanfactory和applicationContext就是spring框架的
两个IOC日期，现在一般使用applicationContext,其
不但包含了beanfactory的作用，同时还进行更多的扩展

实例化spring IOC容器的简单方法：

spring配置
====

如果只有一个spring配置文件可以直接传递一个string
参数，不需要使用数组
1、ApplicationContext context = new 
	ClassPathXmlApplicationContext(new String[]{"spring config file path"});

2、Resource res = new FileSystemResource("config file");
BeanFactory factory = new XMLBeanFactory(res);

spring多个配置文件组合方法
====

1、方法一 在一个作为spring总配置文件中的<bean>
元素定义之前，通过<import>元素将要引入的spring
其他配置文件引入
<beans>
	<import resource="xxx.xml"/>
	<bean></bean>
</beans>

2、 方法二 对于Javase工程是，当使用下面方法获取
applicationContext对象时将所有的spring配置文件
通过数组传递进去，也可以使用通配符如spring-*.xml
方式； 对于JavaEE工程，在web.xml 文件中指定spring
配置文件可以指定多个，用逗号分割，也可以使用通配
符


spring配置文件bean的配置规则
====

1、一个bean可以通过一个ID属性唯一指定和引用，如果
spring配置文件中有两个以上相同的ID时，spring报错
2、一个bean也可以通过一个name属性来引用和指定，
如果spring配置文件中有两个以上相同name的bean，则
spring通过name引用时，运行时后面的会自动覆盖前面
相同的name的bean引用，而不会报错


spring依赖注入三种方式
====

1、使用构造器注入
<bean id="xxx" class="xxx">
	<constructor-arg>param_1</constructor-arg>
	<constructor-arg>param_2</constructor-arg>
</bean>

2、setter注入
<bean id="xxx" class="xxx">
	<property name="xxx" value="xxx"/>
	<property name="xxx" value="xxx"/>
</bean>

3、Field注入

字段注入
@Resource 
private UserDao dao;
属性注入
@Resource
public void setUserDao(UserDao dao) {
	this.dao = dao;
}

spring通过注入方式注入依赖bean
====

<beans>
	<bean id="xxx" class="xxx">
		<property name="xxx" value="xxx"/>
	</bean>
	<bean id="zzz" class="zzz">
		<property name="xxx" ref="xxx"/> 
	</bean>
</beans>

set集合注入
====
<bean id="xxx" class="xxx">
	<set>
		<value>1</value>
		<value>2</value>
	</set>
</bean>

list集合注入
====

<bean id="xxx" class="xxx">
	<list>
		<value>1</value>
		<value>2</value>
	</list>
</bean>

map集合注入
====

<bean id="xxx" class="xxx">
	<props>
		<prop key="xxx">1</prop>
		<prop key="yyy">2</prop>
	</props>
</bean>

java注解简单说明
====

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.CLASS)
public @interface TestAnnotation{
	String value() default "";
}
若要自定义的主要可以被继承，加上@Inherited注解

注意说明

1、Java的注解实际上自动继承了Annotation接口，
因此在自定义注解时不能继承其他的注解或者接口
2、Retention:告诉编译器如何处理注解，即注解运行
在什么时刻
	SOURCE 存储在源文件中，编译过后即废弃
	CLASS  存储在class文件中，是其缺省的值
	RUNTIME  存储在class文件中，并且有Java虚拟机
	读入解析，可以使用反射机制获取注解相关信息
3、@Target 指定注解的目标使用对象
ElementType
	TYPE  适用于class、interface、enum等类级别
	的目标对象
	FIELD  类中字段级别的目标对象
	METHOD  类中方法级别的目标对象
	PARAMETER  方法参数级别的目标对象
	CONSTRUCTOR  类构造方法
	LOCAL_VARIABLE  局部变量
	ANNOTATION_type  适用annotation对象
	PACKAGE  package对象

注解里面只能声明属性，不能声明方法，声明属性的
方式比较特殊；语法格式为：  数据类型 属性() 
default 默认值(默认值是可选的) 如stringvalue();
使用时，注解对象(属性=属性值)为注解指定属性值
通过注解对象.属性就可以得到注解属性的值

注解的解析 使用Java反射机制获得注解的目标对象
可以得到注解镀锡。 如field.getAnnotation(注解类
.class)就可以得到注解对象


基于注解的spring配置准备条件
====

spring2.5之后，开始全面支持注解式配置
1、使用spring注解方式加入common-annpotation.jar
2、使用注解方式时，必须在spring配置文件的schema
中添加注解的命名空间		
xmlns:context=”http://www.springframework.org/schema/context”  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-2.5.xsd  

spring使用四个注解按功能来进行对bean的配置
@Component  泛指组件，对于一般不好归类的Javabean
@Service 用于service层
@Controller  控制层
@Repository 数据访问层

注意： 使用spring注解方式配置的bean对象，bean引
用时默认名称为被注解名称的首字母小写形式，也可以
指定名称

spring自动装配
====

@Autowired 按对象的数据类型进行自动装配
@Resource 按名称或者ID进行自动装配，只有当找不
到匹配的名称或者ID时才按类型进行装配

spring的自动扫描,多个表名用逗号分隔.自动装配和
自动扫描是紧密联系在一起协同工作的，都需要引入
context的命名空间
<context:component-scan base-package="
xxx.xxx,yyy.yyy"/>