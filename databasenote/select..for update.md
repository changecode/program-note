select ... for update
----

select * from t for update 会等待行锁释放之后，返回查询结果

select * from t for update nowait 不等待行锁释放，提示锁冲突，不返回结果

select * from t for update wait 5 等待5秒，若行锁没释放，则提示锁冲突，不返回结果

select * from t for update skip locked 查询返回查询结果，但忽略有行锁的记录


语法
----

select ... for update [of column_list] [wait n] [nowait] [skip locked] 其中of子句用于指定即将更新的列，即锁定行上的特定列
wait子句指定等待其他用户释放锁的秒数，防止无限期的等待
使用for update wait子句：
	1、防止无限期等待被锁定的行
	2、允许应用程序对锁的等待时间进行更多的控制
	3、对于交互应用程序非常有用，因为这些用户不能等待不确定
	4、若使用了skip locked则可以越过锁定的行，不会报告由wait n 引发的资源忙异常报告


select ... from t lock in share mode
----

example(from mysql document):

		create table child_codes(counter_field integer);
		insert into child_codes set counter_field = 1;
		start transaction;
		select counter_field from child_codes lock in share mode;
		start transaction;
		select counter_field from child_codes lock in share mode;
		update child_codes set counter_field = 2;
		error 1205: lock timeout exceeeded try restarting transaction




