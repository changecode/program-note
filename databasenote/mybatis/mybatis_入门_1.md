mybatis
===

ORM思想
====

# 从配置文件(通常是xml配置文件)得到
sessionFactory

# 有sessionFactory产生session

# 在session中完成对数据的增删改查

# 用完之后关闭session

# 在Java对象和数据库之间有做mapping的
配置文件，通常也是xml文件


功能架构
====

# API接口层 提供给外部使用的接口API，
开发人员通过这些本地API来操纵数据库。
接口层一接受到调用请求就会调用数据处理
层来完成具体的数据处理

# 数据处理层 负责具体的sql查找、sql解析
sql执行和执行结果映射处理等

#基础支撑层 负责最基础的功能支撑，包括
连接管理、事务管理、配置加载和缓存处理
这些都是共用的，将它们抽取出来作为最
基础的组件。为上层的数据处理层提供支撑


mybatis流程
====

启动 -- SqlSessionFactoryBuilder(通过
parse创建Configuration对象) -- build --
SqlSessionFactory(openSession) -- 
SqlSession(query) -- Executor -- State
mentHandler -- ResultSetHandler -- 结束


流程分析
====

SqlSessionFactoryBuilder
=====

每个mybatis的应用程序入口是SqlSession
FactoryBuilder，通过xml配置文件创建
Configuration对象(也可在程序中创建)，然
后通过build方法创建SqlSessionFactory,没
有必要每次访问mybatis就创建一次，通常的
做法创建一个全局的对象就可以了

private static SqlSessionFactoryBuilder
	 builder;
priavte static SqlSessionFactory 
factory;
private static init() throws Exception{
	String resource= "mybatis-conf.xml";
	Reader reader = Resources.getResourceAsReader(resource);
	builder = new SqlSessionFactoryBuilder();
	factoty = builder.build(reader);
}


SqlSessionFactory
=====

SqlSessionFactory由sqlSessionFactoryBu
ilder创建。SqlsessionFactory对象一个
必要的属性是Configuration对象，它是保存
mybatis全局配置的一个配置对象，通常由
sqlSessionFactoryBuilder从xml配置文件
中创建

		<?xml version="1.0" encoding="UTF-8" ?>
		<!DOCTYPE configuration PUBLIC 
			"-//mybatis.org//DTD Config 3.0//EN"
			"http://mybatis.org/dtd/mybatis-3-config.dtd">
		<configuration>
			<!-- 配置别名 -->
			<typeAliases>
				<typeAlias type="org.iMybatis.abc.dao.UserDao" alias="UserDao" />
				<typeAlias type="org.iMybatis.abc.dto.UserDto" alias="UserDto" />
			</typeAliases>
			 
			<!-- 配置环境变量 -->
			<environments default="development">
				<environment id="development">
					<transactionManager type="JDBC" />
					<dataSource type="POOLED">
						<property name="driver" value="com.mysql.jdbc.Driver" />
						<property name="url" value="jdbc:mysql://127.0.0.1:3306/iMybatis?characterEncoding=GBK" />
						<property name="username" value="iMybatis" />
						<property name="password" value="iMybatis" />
					</dataSource>
				</environment>
			</environments>
			
			<!-- 配置mappers -->
			<mappers>
				<mapper resource="org/iMybatis/abc/dao/UserDao.xml" />
			</mappers>
			
		</configuration>


SqlSession
=====		

SqlSession对象的主要功能是完成一次数据
库的访问和结果的映射，类似于数据库的
session概念，由于不是线程安全的。所以
SqlSession对象的作用域需限制方法内。
SQLSession的默认实现类是DefaultSqlSes
sion,它有两个必须配置的属性：Configura
tion和Executor。 SqlSession对数据库的
操作都是通过Executor来完成

SqlSession有个方法getMapper。根据其官
方文档说明，应用程序除了要初始化并启动
mybatis之外，还需要定义一些接口，接口
定义访问数据库的方法，存放接口的包路径
下需要放置同名的xml配置文件。SqlSessio
n的getMapper方法时联系应用程序和mybati
s纽带，应用程序访问getMapper时，mybati
s会根据传入的接口类型和对应的xml配置
文件生成一个代理对象，这个代理对象就
是叫mapper对象。应用程序获得mapper对象
后，就应该通过mapper对象来访问SqlSessi
on对象

		SqlSession session = SqlSessionFactory.openSession();
		UserDao userDao = session.get
		Mapper(UserDao.class);
		User user = new User();
		user.setName("123");
		userDao.selectUser(user);
========================
		public interface UserDao {
		   public List<User> queryUsers(User user) throws Exception;
		}
========================
		<?xml version="1.0" encoding="UTF-8" ?>  
		<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
		<mapper namespace="org.iMybatis.abc.dao.UserDao">  
		    <select id="queryUsers" parameterType="UserDto" resultType="UserDto"  
		        useCache="false">  
		        <![CDATA[ 
		        select * from t_user t where t.username = #{username} 
		        ]]>  
		    </select>  
		</mapper>  		


Executor
=====

Executor对象在创建Configuration对象的
时候创建，并且缓存在Configuration对象
中。Executor对象的主要功能是调用State
mentHandler访问数据库，并将查询结果
存入缓存中(如果配置了缓存的话)


StatementHandler
=====

真正访问数据库的地方，并调用ResultSet
Handler处理查询结果


ResultSetHandler
=====

处理查询结果