从源码分析mybatis加载过程
===


SqlSessionFactoryBuilder
====

		public class SqlSessionFactoryBuilder {

		    //Reader读取mybatis配置文件，传入构造方法
		    //除了Reader外，其实还有对应的inputStream作为参数的构造方法，
		    //这也体现了mybatis配置的灵活性
		    public SqlSessionFactory build(Reader reader) {
		        return build(reader, null, null);
		    }

		    public SqlSessionFactory build(Reader reader, String environment) {
		        return build(reader, environment, null);
		    }
		  
		    //mybatis配置文件 + properties, 此时mybatis配置文件中可以不配置properties，也能使用${}形式
		    public SqlSessionFactory build(Reader reader, Properties properties) {
		        return build(reader, null, properties);
		    }
		  
		    //通过XMLConfigBuilder解析mybatis配置，然后创建SqlSessionFactory对象
		    public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
		        try {
		            XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
		            //下面看看这个方法的源码
		            return build(parser.parse());
		        } catch (Exception e) {
		            throw ExceptionFactory.wrapException("Error building SqlSession.", e);
		        } finally {
		            ErrorContext.instance().reset();
		            try {
		                reader.close();
		            } catch (IOException e) {
		                // Intentionally ignore. Prefer previous error.
		            }
		        }
		    }
		    
		    public SqlSessionFactory build(Configuration config) {
		        return new DefaultSqlSessionFactory(config);
		    }
		}

通过源码，可以看到SqlSessionFactory
Builder通过XMLConfiguration去解析传入
的mybatis的配置文件

		public class XMLConfigBuilder extends BaseBuilder {
		    public XMLConfigBuilder(InputStream inputStream, String environment, Properties props) {
		        this(new XPathParser(inputStream, true, props, new XMLMapperEntityResolver()), environment, props);
		    }

		    private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
		        super(new Configuration());
		        ErrorContext.instance().resource("SQL Mapper Configuration");
		        this.configuration.setVariables(props);
		        this.parsed = false;
		        this.environment = environment;
		        this.parser = parser;
		    }
		  
		    //外部调用此方法对mybatis配置文件进行解析
		    public Configuration parse() {
		        if (parsed) {
		            throw new BuilderException("Each XMLConfigBuilder can only be used once.");
		        }
		        parsed = true;
		        //从根节点configuration
		        parseConfiguration(parser.evalNode("/configuration"));
		        return configuration;
		    }

		    //此方法就是解析configuration节点下的子节点
		    //由此也可看出，我们在configuration下面能配置的节点为以下10个节点
		    private void parseConfiguration(XNode root) {
		        try {
		            propertiesElement(root.evalNode("properties")); //issue #117 read properties first
		            typeAliasesElement(root.evalNode("typeAliases"));
		            pluginElement(root.evalNode("plugins"));
		            objectFactoryElement(root.evalNode("objectFactory"));
		            objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
		            settingsElement(root.evalNode("settings"));
		            environmentsElement(root.evalNode("environments")); // read it after objectFactory and objectWrapperFactory issue #631
		            databaseIdProviderElement(root.evalNode("databaseIdProvider"));
		            typeHandlerElement(root.evalNode("typeHandlers"));
		            mapperElement(root.evalNode("mappers"));
		        } catch (Exception e) {
		            throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
		        }
		    }
		}		

# configuration节点为根节点。
# 在configuration节点之下，我们可以配置10个子节点， 分别为：properties、typeAliases、plugins、objectFactory、objectWrapperFactory、settings、environments、databaseIdProvider、typeHandlers、mappers

配置元素properties
=====

		<configuration>
		    <!-- 方法一： 从外部指定properties配置文件, 除了使用resource属性指定外，还可通过url属性指定url  
		        <properties resource="dbConfig.properties"></properties> 
		    -->
		    <!-- 方法二： 直接配置为xml -->
		    <properties>
		        <property name="driver" value="com.mysql.jdbc.Driver"/>
		        <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
		        <property name="username" value="root"/>
		        <property name="password" value="root"/>
		    </properties>

envirements元素
=====		   

		<environments default="development">
		    <environment id="development">
		        <!-- 
		        JDBC–这个配置直接简单使用了JDBC的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围。
		        MANAGED–这个配置几乎没做什么。它从来不提交或回滚一个连接。而它会让容器来管理事务的整个生命周期（比如Spring或JEE应用服务器的上下文）。
		        -->
		        <transactionManager type="JDBC"/>
		        <!--
		        UNPOOLED–这个数据源的实现是每次被请求时简单打开和关闭连接
		        POOLED–mybatis实现的简单的数据库连接池类型，它使得数据库连接可被复用，不必在每次请求时都去创建一个物理的连接。
		        JNDI – 通过jndi从tomcat之类的容器里获取数据源。
		        -->
		        <dataSource type="POOLED">
		            <!--
		            如果上面没有指定数据库配置的properties文件，那么此处可以这样直接配置 
		            <property name="driver" value="com.mysql.jdbc.Driver"/>
		            <property name="url" value="jdbc:mysql://localhost:3306/test1"/>
		            <property name="username" value="root"/>
		            <property name="password" value="root"/>
		            -->
		         
		            <!-- 上面指定了数据库配置文件， 配置文件里面也是对应的这四个属性 -->
		            <property name="driver" value="${driver}"/>
		            <property name="url" value="${url}"/>
		            <property name="username" value="${username}"/>
		            <property name="password" value="${password}"/>  
		        </dataSource>
		    </environment>
		    
		    <!-- 我再指定一个environment -->
		    <environment id="test">
		        <transactionManager type="JDBC"/>
		        <dataSource type="POOLED">
		            <property name="driver" value="com.mysql.jdbc.Driver"/>
		            <!-- 与上面的url不一样 -->
		            <property name="url" value="jdbc:mysql://localhost:3306/demo"/>
		            <property name="username" value="root"/>
		            <property name="password" value="root"/>
		        </dataSource>
		    </environment>
		</environments> 

environments元素节点可以配置多个environment子节点,通过设置default值就可以切
换不同的环境		

在配置dataSource的时候使用了${driver} 
这种表达式，是通过PropertyParser这个类
解析

		public class PropertyParser {

		    public static String parse(String string, Properties variables) {
		        VariableTokenHandler handler = new VariableTokenHandler(variables);
		        GenericTokenParser parser = new GenericTokenParser("${", "}", handler);
		        return parser.parse(string);
		    }

		    private static class VariableTokenHandler implements TokenHandler {
		        private Properties variables;

		        public VariableTokenHandler(Properties variables) {
		            this.variables = variables;
		        }

		        public String handleToken(String content) {
		            if (variables != null && variables.containsKey(content)) {
		                return variables.getProperty(content);
		            }
		            return "${" + content + "}";
		        }
		    }
		}


mappers元素
=====



mappers节点下，配置我们的mapper映射文
件，所谓的mapper映射文件，就是让mybati
s用来建立数据表和javabean映射的一个桥梁

		<configuration>
		    ......
		    <mappers>
		        <!-- 第一种方式：通过resource指定 -->
		        <mapper resource="com/dy/dao/userDao.xml"/>
		    
		        <!-- 第二种方式， 通过class指定接口，进而将接口与对应的xml文件形成映射关系
		             不过，使用这种方式必须保证 接口与mapper文件同名(不区分大小写)， 
		             我这儿接口是UserDao,那么意味着mapper文件为UserDao.xml 
		        <mapper class="com.dy.dao.UserDao"/>
		        -->
		      
		        <!-- 第三种方式，直接指定包，自动扫描，与方法二同理 
		        <package name="com.dy.dao"/>
		        -->
		        <!-- 第四种方式：通过url指定mapper文件位置
		        <mapper url="file://........"/>
		        -->
		    </mappers>
		    ......
		</configuration>