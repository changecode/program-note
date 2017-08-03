spring-quartz 2.2
----

		<bean id="quartzJob" class="web.Quartz"></bean>
		<!-- 定义调用对象和调用对象的方法 -->
		<bean id="jobtask" class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
			<!-- 调用的类 -->
			<property name="targetObject">
				<ref bean="quartzJob"/>
			</property>
			<!-- 调用类中的方法 -->
			<property name="targetMethod">
			 	<value>work</value>
			</property>
		</bean>
		<!-- 定义触发时间 -->
		<bean id="doTime" class="org.springframework.scheduling.quartz.CronTriggerBean">
		 	<property name="jobDetail">
				<ref bean="jobtask"/>
		 	</property>
			<!-- cron表达式 -->
			<property name="cronExpression">
					<!-- 每隔10秒执行一次-->
		            <value>0/10 * * * * ?</value>
		       </property>
		</bean>
		<!-- 总管理类 如果将lazy-init='false'那么容器启动就会执行调度程序 -->
		<bean id="startQuertz" lazy-init="false" autowire="no" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		 	<property name="triggers">
		 		<list>
		 			<ref bean="doTime"/>
				</list>
			</property>
		</bean>


web.xml
		<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
		 /WEB-INF/classes/applicationContext*.xml
		</param-value>
		</context-param>
		<servlet>
		<servlet-name>context</servlet-name>
		<servlet-class>
		org.springframework.web.context.ContextLoaderServlet
		</servlet-class>
		<load-on-startup>1</load-on-startup>
		</servlet>


		package web;
		public class Quartz {
		public void work(){
		 	System.out.println("Quartz的任务调度！！！");
		 }
		}



cronExpression表达式
-----

0 0 12 ?———————-在每天中午12：00触发
0 15 10 ? ———————-每天上午10:15 触发
0 15 10 ?———————-每天上午10:15 触发
0 15 10 ? ———————-每天上午10:15 触发
0 15 10 ? 2005———————-在2005年中的每天上午10:15 触发
0 14 ?———————-每天在下午2：00至2：59之间每分钟触发一次
0 0/5 14 ?———————-每天在下午2：00至2：59之间每5分钟触发一次
0 0/5 14,18 ?———————-每天在下午2：00至2：59和6：00至6：59之间的每5分钟触发一次
0 0-5 14 * ?———————-每天在下午2：00至2：05之间每分钟触发一次
0 10,44 14? 3 WED———————-每三月份的星期三在下午2：00和2：44时触发
0 15 10 ? MON-FRI———————-从星期一至星期五的每天上午10：15触发
0 15 10 15 ?———————-在每个月的每15天的上午10：15触发
0 15 10 L ?———————-在每个月的最后一天的上午10：15触发
0 15 10 ? 6L———————-在每个月的最后一个星期五的上午10：15触发
0 15 10 ? 6L 2002-2005———————-在2002, 2003, 2004 and 2005年的每个月的最后一个星期五的上午10：15触发
0 15 10 ? 6#3———————-在每个月的第三个星期五的上午10：15触发
0 0 12 1/5 ?———————-从每月的第一天起每过5天的中午12：00时触发
0 11 11 1111 ?———————-在每个11月11日的上午11：11时触发.