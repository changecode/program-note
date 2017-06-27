有时配置spring和springMVC时会出现一些奇怪的异常，比如bean被多次加载，多次实例化，或者依赖注入时，bean不能被自动注入
----

在spring整体框架的核心概念中，容器是核心思想，就是用来管理bean的整个生命周期，而在一个项目中，容器不一定只有一个，spring中科院包含多个容器，而且容器有上下层关系，spring和springMVC其实就是二个容器。 spring是根容器，springMVC是子容器，并且在spring根容器中的bean是可见的，也就是说子容器可以看见父容器中注册的bean，反之则不行。 

bean注册配置的功能是扫描默认包下的所有@Component注解，自动注册到容器中，同时也扫描@Controller @Service @Respository，它们都是继承@Component注解

springMVC必须要配置的<mvc:annotation-driven>参数，@RequestMapping必须结合此参数配置才能生效

spring配置文件applicationContext.xml  springMVC配置文件applicationContext-mvc.xml这样项目中就有二个容器了。

配置方式一，在spring配置文件中配置了负责所有需要注册的bean扫描工作，spring MVC中配置了springMVC相关注解的使用，启动项目发现，springMVC失效，无法进行跳转。

配置二，将扫描配置到springMVC配置文件中，重启后，验证成功

源码分析
----

springMVC初始化时，会寻找所有当前容器中的所有@Controller注解的bean，来确定其是否是一个handler,而当前容器spring mvc中注册的bean中并没有@Controller注解的，注意上面配置方法一，所有的@controller配置的bean都注册在spring这个父容器中

 在实际项目中，spring根容器负责所有非Controller的bean注册，而springMVC只负责controller相关的bean注册，第三种方案spring容器配置排除所有@Controller相关的bean注册：
 		<context:component-scan base-package="com.xxx.xxxx">
  		<context:exclude-filter expression="org.springframework.stereotype.Controller" type="annotation">
  		</context:exclude-filter></context:component-scan>