spring-advanced
===

加载外部配置文件或者属性文件
====

spring在JavaEE工程中，是一个管理其他模块和组件
的轻量级容器。其他开源框架的配置文件就通过spring
的propertyPlaceholderConfigurer加载在spring中进行管理。 另外，数据库连接信息，JNDI连接信息属性文件
等也可以加载到spring中来管理，用法如下

1、添加如下配置,也可以使用<context:Propery-Placeholderlocation=”classpath:要加载的文件名”/>
<bean class="org.springframework.beans.factory.
config.PropertyPlaceholderConfigurer">
	<property name="locations">
			<value>classpath:xxx</value>
	</property>
</bean>

2、经过第一步加载的配置或属性文件就被加载到spring
如果还需要在运行时使用加载进来的配置或数据文件的
一些信息，就可以使用类型EL表达式进行引用
<bean id="dataSource" destroy-method="close"
	class="xxx">
	<propery name="dirverClassName" value="${xxx}"/>
	<propery name="url" value="${xxx}"/>
</bean>	


java动态代理
====

原理： 当调用一个目标对象或者其方法时，系统并不是
直接返回目标对象，而是返回一个代理对象，通过这个
代理对象去访问目标镀锡或者目标对象的方法
客户端调用者--> 代理对象-->被调用的目标对象

动态代码分为两种，针对接口的动态代理和针对普通类
的动态代码，Java中的动态代理是针对接口的动态代理
cglib是针对普通类的动态代理。

1、接口类动态代理，目标对象必须实现接口，代码对象
要实现目标对象的所有接口
动态代理必须实现InvocationHandler接口，同时实现以
下方法：
-- Object invoke(object代理实例，method代理
实例上调用的接口方法method实例，Object[]传入代理
实例上方法调用的参数值的对象数组)
--创建代理对象
Proxy.newProxyInstance(类加载器，Class<?>[]
接口数组，回调代理对象(一般是this))

当调用目标对象方法时，通过该方法创建目标对象的代
理对象，代理对象会自动调用其invoke方法调用目标对
象，并将调用结果返回


cglib针对普通类动态代理

1、Enhancer enhancer = new Enhancer();
enhancer.setSuperClass(目标对象.getClass());
enhancer.setCallback(this);

2、实现MethodInterceptor接口
Object intercept(object代理实例，method代理实例
上调用的接口方法的method实例，Object[]
传入代理实例上调用的参数值的对象数组，methodProxy
方法代理实例)

AOP
====

1、在spring中使用面向切面编程，需要在spring配置文件
引入aop命名空间
xmlns:aop=”http://www.springframework.org/schema/aop”  
“http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-2.5.xsd”  

注解方式aop需要添加aop支持
<aop:aspectj-autoProxy/>

2、定义切面，在类前面加上@Aspect注解，表名该类是
一个切面

3、在切面中加入切入点，切入点就是被拦截对象方法
的集合，通常切入点定义在切面中某个对切入点进行
处理的方法上使用@Pointcut注解
@Pointcut("execution(* xx.xxx.service..*.*(..))")
public void method(){ 切入点处理}

4、在切面中添加通知
@Before 前置通知
@AfterRuturning 后置通过
@After 最终通知
@AfterThrowing 例外通知
@Around  环绕通知


spring事务处理
====

1、基于注解方式的事务管理，在spring配置文件中
添加事务管理的命令空间
xmlns:ts=http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx  
http://www.springframework.org/schema/tx/spring-tx-2.5.xsd  

2、在spring配置文件中配置事务管理器
<bean id="txManager" class="org.springframework.
jdbc.datasource.DataSourceTransactionManager">
	<property name="dataSource" ref="dataSource"/>
</bean>

3、在配置文件中添加支持注解方式的事务配置项
<tx:annotation-driven transaction-manager="txManager"/>

在需要使用事务的业务逻辑地方加上@Transaction注解


xml配置事务
====

1、<bean id="txManger" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
	<propery name="dataSource" ref="dataSource"/>
</bean>

2、添加事务管理的切面

<aop:config>
	<aop:pointcut id="transactionPointcut"
		expression="execution(* com.xx.service..*.*(..))"/>
	<!--事务通知-->
	<aop:advisor advice-ref="txAdvice" pointcut-
	ref="transactionPointcut" />
</aop:config>

3、为事务添加事务处理特征
<tx:advice id="txAdvice" transactionManager="txManger">
	<tx:attributes>
		<tx:method name="get*" read-only="true"
		propagation="NOT_SUPPORTED"/>
		<tx:method name="*"/>
	</tx:attributes>
</tx:advice>