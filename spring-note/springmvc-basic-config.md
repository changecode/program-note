Springmvc(4.0.6) 简单环境搭建步骤

1、所需的jar包
	jackson-core-asl-1.8.8
	jackson-mapper-asl-1.8.8
	spring-aop/spring-beans/spring-context/spring-core/spring-expression/spring-web/spring-webmvc

2、web.xml
	<servlet>
		<servlet-name>dispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>	
	<servlet-mapping>
		<servlet-name>dispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

3、springmvc.xml
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">	
	    <mvc:annotation-driven/>
	    <context:component-scan base-package="com.xxx"/>

	    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	    	<property name="prefix" value="/WEB-INF/views/"></property>
	    	<property name="suffix" value=".jsp"></property>
	    </bean>    
	</beans>    

4、controller
	@Controller
	@RequestMapping("/test")
	public class Test {

		@RequestMapping("/mvc")
		public String testMvc() {
			return "springmvc";
		}

		@RequestMapping("/json")
	    @ResponseBody
	    public Map<String,String> show(){
	        Map<String,String> map = new HashMap<String,String>();
	        map.put("1", "2");
	        map.put("2", "2");
	        map.put("3", "2");
	        return map;
	    }
	}





