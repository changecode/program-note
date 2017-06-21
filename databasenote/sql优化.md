mysql常见优化方式
----
1、最左前缀匹配原则，MySQL会一直向右匹配直到遇到范围查询
		(>,<,between,like)就停止匹配，比如a =1 and b = 2 and c > 3
		and d = 4如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立
		(a,b,d,c)的索引则都可以用到，a b d的顺序可以任意调整

2、尽量选择区分度高的列作为索引，区分度的公式是count(distinct cok)/count(*)，表示字段不重复的比例，比例越大需要扫描的记录数就越小
唯一键的区分度就是1，而一些状态、性别字段可能在大数据面前区分度就是0
一般要求join的字段要求是0.1以上，平均1条扫描10条记录
	
3、尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符类型，这会降低查询和连接的性能，并会增加存储开销，引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了

4、索引列不能参与计算，保持列干净，比如from_unixtime(create_time) = '2017-09-09'就不能使用到索引，因为b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都运用函数才能比较，显得成本太大。应该写成 			create_time = from_unixtime('2017-09-09')尽量避免在where子句中对字段进行null值判断，否则会导致引擎放弃使用
	
5、索引而进全表扫描，
		select id from t where num is null
		可以在num上设置默认值为0，确保表中num列没有null值，然后这样查询
		select if from where num = 0;
		6、避免在where子句中使用or连接条件，否则引擎会放弃使用而进行全表扫描
		select id from t where num = 10 or num = 20
		--> select id from t where num = 10 union all select id from t where num = 20
		7、like查询不能前置百分号
		select id from t where name like '%xxx%' 可以考虑全文检索
		8、in 和 not in也要慎用，否则会导致全表扫描，对于连续的数字范围可以
		考虑慎用between and语句
