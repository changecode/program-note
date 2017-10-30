动态SQL
===

通过if choose when otherwise trim where
set foreach标签，可组合成非常灵活的sql

if
====

<select id="findUserById" resultType="user">
    select * from user where 
        <if test="id != null">
               id=#{id}
        </if>
    and deleteFlag=0;
</select>


where
====

<select id="findUserById" resultType="user">
    select * from user 
        <where>
            <if test="id != null">
                id=#{id}
            </if>
            and deleteFlag=0;
        </where>
</select>


trim
====

<trim prefix="WHERE" prefixOverrides="AND |OR ">
    ... 
</trim>

当WHERE后紧随AND或则OR的时候，就去除AND或者OR


set
====

<update id="updateUser" parameterType="com.dy.entity.User">
    update user set 
        <if test="name != null">
            name = #{name},
        </if> 
        <if test="password != null">
            password = #{password},
        </if> 
        <if test="age != null">
            age = #{age}
        </if> 
        <where>
            <if test="id != null">
                id = #{id}
            </if>
            and deleteFlag = 0;
        </where>
</update>


set
====

<update id="updateUser" parameterType="com.dy.entity.User">
    update user
        <set>
            <if test="name != null">name = #{name},</if> 
            <if test="password != null">password = #{password},</if> 
            <if test="age != null">age = #{age},</if> 
        </set>
        <where>
            <if test="id != null">
                id = #{id}
            </if>
            and deleteFlag = 0;
        </where>
</update>

<trim prefix="SET" suffixOverrides=",">
  ...
</trim>

WHERE是使用的 prefixOverrides（前缀）， SET是使用的 suffixOverrides （后缀）


foreach
====

<select id="selectPostIn" resultType="domain.blog.Post">
    SELECT *
    FROM POST P
    WHERE ID in
    <foreach item="item" index="index" collection="list"
        open="(" separator="," close=")">
        #{item}
    </foreach>
</select>


choose
====

<select id="findActiveBlogLike"
     resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <choose>
        <when test="title != null">
            AND title like #{title}
        </when>
        <when test="author != null and author.name != null">
            AND author_name like #{author.name}
        </when>
        <otherwise>
            AND featured = 1
        </otherwise>
    </choose>
</select>

当title和author都不为null的时候， 那么选择二选一（前者优先）， 如果都为null, 那么就选择 otherwise中的， 如果tilte和author只有一个不为null, 那么就选择不为null的那个


栗子
====

<update id="update" parameterType="org.format.dynamicproxy.mybatis.bean.User">
    UPDATE users
    <trim prefix="SET" prefixOverrides=",">
        <if test="name != null and name != ''">
            name = #{name}
        </if>
        <if test="age != null and age != ''">
            , age = #{age}
        </if>
        <if test="birthday != null and birthday != ''">
            , birthday = #{birthday}
        </if>
    </trim>
    where id = ${id}
</update>

源码分析
在spring与mybatis整合的时候需要配置
SqlSessionFactoryBean，该配置会加入
数据源和mybatis xml配置文件路径等信息

<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:mybatisConfig.xml"/>
    <property name="mapperLocations" value="classpath*:org/format/dao/*.xml"/>
</bean>

SqlSessionFactoryBean实现了spring的
InitializingBean接口，InitializingBean
接口的afterPropertiesSet方法中会调用
buildSqlSessionFactory方法，该方法内部
会使用XMLConfigBUilder解析属性Configur
ation中配置的路径，还会使用XMLMapper
属性解析mapperLocations属性中各个xml

分析上述sql
=====

1、得到所有子节点
# 文本节点 update users
# trim子节点
# 文本节点 where id = #{id}

2、遍历各个子节点
# 如果节点类型是文本或者CDATA,构造一个
TextSqlNode或StaticTextSqlNode
# 如果节点类型是元素，说明该update节点
是动态sql，然后会使用NodeHandler处理
各个类型的自己诶单。这里的NodeHandler
是XMLScriptBuilder的一个内部接口，其
实现类包括TrimHandler WhereHandler
SetHandler  IfHandler  ChooseHandler

3、遇到子节点是元素的话，重复以上步骤
