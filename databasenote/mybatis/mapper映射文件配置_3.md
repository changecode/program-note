mapper映射文件配置
===

在mapper文件中，以mapper作为根节点。其
下面可以配置的元素节点有 select insert
update delete cache cache-ref resultMap
sql


insert update delete 
====

		<?xml version="1.0" encoding="UTF-8" ?>   
		<!DOCTYPE mapper   
		PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
		"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd"> 

		<!-- mapper 为根元素节点， 一个namespace对应一个dao -->
		<!-- 
		Mapper元素只有一个属性namespace，它有两个作用：`一是用于区分不同的mapper`（在不同的mapper文件里，子元素的id可以相同，mybatis通过namespace和子元素的id联合区分），`二是与接口关联`（应用程序通过接口访问mybatis时，mybatis通过接口的完整名称查找对应的mapper配置，因此namespace的命名务必小心一定要某接口同名）。
		-->
		<mapper namespace="com.dy.dao.UserDao">
		    
		    <!-- 
		    cache- 配置本定命名空间的缓存。
		        type- cache实现类，默认为PERPETUAL，可以使用自定义的cache实现类（别名或完整类名皆可）
		        eviction- 回收算法，默认为LRU，可选的算法有：
		            LRU– 最近最少使用的：移除最长时间不被使用的对象。
		            FIFO– 先进先出：按对象进入缓存的顺序来移除它们。
		            SOFT– 软引用：移除基于垃圾回收器状态和软引用规则的对象。
		            WEAK– 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。
		        flushInterval- 刷新间隔，默认为1个小时，单位毫秒
		        size- 缓存大小，默认大小1024，单位为引用数
		        readOnly- 只读
		    -->
		    <cache type="PERPETUAL" eviction="LRU" flushInterval="60000"  
		        size="512" readOnly="true" />
		    
		    <!-- 
		    cache-ref–从其他命名空间引用缓存配置。
		        如果你不想定义自己的cache，可以使用cache-ref引用别的cache。因为每个cache都以namespace为id，所以cache-ref只需要配置一个namespace属性就可以了。需要注意的是，如果cache-ref和cache都配置了，以cache为准。
		    -->
		    <cache-ref namespace="com.someone.application.data.SomeMapper"/>
		    
		    <insert
		      <!-- 1. id （必须配置）
		        id是命名空间中的唯一标识符，可被用来代表这条语句。 
		        一个命名空间（namespace） 对应一个dao接口, 
		        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致 -->
		      
		      id="insertUser"
		      
		      <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
		        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
		        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
		      
		      parameterType="com.demo.User"
		      
		      <!-- 3. flushCache （可选配置，默认配置为true）
		        将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句） -->
		      
		      flushCache="true"
		      
		      <!-- 4. statementType （可选配置，默认配置为PREPARED）
		        STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 -->
		      
		      statementType="PREPARED"
		      
		      <!-- 5. keyProperty (可选配置， 默认为unset)
		        （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
		      
		      keyProperty=""
		      
		      <!-- 6. keyColumn     (可选配置)
		        （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 -->
		      
		      keyColumn=""
		      
		      <!-- 7. useGeneratedKeys (可选配置， 默认为false)
		        （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。  -->
		      
		      useGeneratedKeys="false"
		      
		      <!-- 8. timeout  (可选配置， 默认为unset, 依赖驱动)
		        这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 -->
		      timeout="20">

		    <update
		      id="updateUser"
		      parameterType="com.demo.User"
		      flushCache="true"
		      statementType="PREPARED"
		      timeout="20">

		    <delete
		      id="deleteUser"
		      parameterType="com.demo.User"
		      flushCache="true"
		      statementType="PREPARED"
		      timeout="20">
		</mapper>

select配置
====

		<select
		     <!--  1. id （必须配置）
		        id是命名空间中的唯一标识符，可被用来代表这条语句。 
		        一个命名空间（namespace） 对应一个dao接口, 
		        这个id也应该对应dao里面的某个方法（相当于方法的实现），因此id 应该与方法名一致
		     -->
		     
		     id="selectPerson"
		     
		     <!-- 2. parameterType （可选配置, 默认为mybatis自动选择处理）
		        将要传入语句的参数的完全限定类名或别名， 如果不配置，mybatis会通过ParameterHandler 根据参数类型默认选择合适的typeHandler进行处理
		        parameterType 主要指定参数类型，可以是int, short, long, string等类型，也可以是复杂类型（如对象） -->
		     parameterType="int"
		     
		     <!-- 3. resultType (resultType 与 resultMap 二选一配置)
		         resultType用以指定返回类型，指定的类型可以是基本类型，可以是java容器，也可以是javabean -->
		     resultType="hashmap"
		     
		     <!-- 4. resultMap (resultType 与 resultMap 二选一配置)
		         resultMap用于引用我们通过 resultMap标签定义的映射类型，这也是mybatis组件高级复杂映射的关键 -->
		     resultMap="personResultMap"
		     
		     <!-- 5. flushCache (可选配置)
		         将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false -->
		     flushCache="false"
		     
		     <!-- 6. useCache (可选配置)
		         将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true -->
		     useCache="true"
		     
		     <!-- 7. timeout (可选配置) 
		         这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）-->
		     timeout="10000"
		     
		     <!-- 8. fetchSize (可选配置) 
		         这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动)-->
		     fetchSize="256"
		     
		     <!-- 9. statementType (可选配置) 
		         STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED-->
		     statementType="PREPARED"
		     
		     <!-- 10. resultSetType (可选配置) 
		         FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）-->
		     resultSetType="FORWARD_ONLY">

resultMap解析
====

esultMapElement方法负责解析resultMap元
素，它通过调用ResultMapResolver的相应
方法完成resultMap的解析。创建好的resul
tMap存入configuration的resultMaps缓存
中（该缓存以namespace+resultMap的id为k
ey，这里再次体现了mybatis的namespace的
强大用处）

		private ResultMap resultMapElement(XNode resultMapNode) throws Exception {
		    return resultMapElement(resultMapNode, Collections.<ResultMapping> emptyList());
		  }

		  private ResultMap resultMapElement(XNode resultMapNode, List<ResultMapping> additionalResultMappings) throws Exception {
		    ErrorContext.instance().activity("processing " + resultMapNode.getValueBasedIdentifier());
		    String id = resultMapNode.getStringAttribute("id",
		        resultMapNode.getValueBasedIdentifier());
		    String type = resultMapNode.getStringAttribute("type",
		        resultMapNode.getStringAttribute("ofType",
		            resultMapNode.getStringAttribute("resultType",
		                resultMapNode.getStringAttribute("javaType"))));
		    String extend = resultMapNode.getStringAttribute("extends");
		    Boolean autoMapping = resultMapNode.getBooleanAttribute("autoMapping");
		    Class<?> typeClass = resolveClass(type);
		    Discriminator discriminator = null;
		    List<ResultMapping> resultMappings = new ArrayList<ResultMapping>();
		    resultMappings.addAll(additionalResultMappings);
		    List<XNode> resultChildren = resultMapNode.getChildren();
		    for (XNode resultChild : resultChildren) {
		      if ("constructor".equals(resultChild.getName())) {
		        processConstructorElement(resultChild, typeClass, resultMappings);
		      } else if ("discriminator".equals(resultChild.getName())) {
		        discriminator = processDiscriminatorElement(resultChild, typeClass, resultMappings);
		      } else {
		        ArrayList<ResultFlag> flags = new ArrayList<ResultFlag>();
		        if ("id".equals(resultChild.getName())) {
		          flags.add(ResultFlag.ID);
		        }
		        resultMappings.add(buildResultMappingFromContext(resultChild, typeClass, flags));
		      }
		    }
		    ResultMapResolver resultMapResolver = new ResultMapResolver(builderAssistant, id, typeClass, extend, discriminator, resultMappings, autoMapping);
		    try {
		      return resultMapResolver.resolve();
		    } catch (IncompleteElementException  e) {
		      configuration.addIncompleteResultMap(resultMapResolver);
		      throw e;
		    }
		  }

总结
====

每个mapper文件的namespace属性对应于某
个接口，应用程序通过接口访问mybatis，
mybatis会为这个接口生成一个代理对象，
这个对象叫mapper对象，在生成代理对象
钱mybatis会校验接口是否已注册，未注册
的接口会产生一个异常。为了避免这种异常
就需要注册mapper类型。这个步骤是在
XMLMapperBuilder的bindMapperForNamespa
ce方法中完成de.tg调用Configuration对象
的addMapper方法完成，而Configuration
对象的addMapper方法是通过MapperRegistr
y的addMapper方法完成的，它只是简单的
将namespace属性对应的接口类型存入本地
缓存中

Configuration对象提供了一个重载的add
Mappers(String packagename)方法，该
方法以包路径名为参数，它的功能是自动
扫描包路径下的接口并注册到MapperRegis
try的缓存中，同时扫描包路径下的mapper
配置文件并解析。解析配置文件是在Mapper
AnnotationBuilder类的parse方法里完成的
该方法先解析配置文件，然后在解析接口
里的注解配置，且注解里的配置会覆盖配置
文件里的配置。

		private void bindMapperForNamespace() {
		    String namespace = builderAssistant.getCurrentNamespace();
		    if (namespace != null) {
		      Class<?> boundType = null;
		      try {
		        boundType = Resources.classForName(namespace);
		      } catch (ClassNotFoundException e) {
		        //ignore, bound type is not required
		      }
		      if (boundType != null) {
		        if (!configuration.hasMapper(boundType)) {
		          // Spring may not know the real resource name so we set a flag
		          // to prevent loading again this resource from the mapper interface
		          // look at MapperAnnotationBuilder#loadXmlResource
		          configuration.addLoadedResource("namespace:" + namespace);
		          configuration.addMapper(boundType);
		        }
		      }
		    }
		  }		  