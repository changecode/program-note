Java优化建议
----

1、尽量指定类、方法的final修饰符

带有final修饰符的类是不可派生的。为类指定final修饰符可以让方法不可被重写。如果指定了一个类为final，则该类所有的方法都是final。Java编译器会寻找内联所有的final方法，内联对于提升Java运行效率作用重大

2、尽量重用对象

特别是string对象的使用，出现是非常连接时应该使用stringBuilder/StringBuffer代替。由于虚拟机不仅花时间生成对象，以后可能还需要花时间对这些对象进行垃圾回收和处理。因此，生成过多的对象会给程序的性能带来很大的影响

3、尽可能使用局部变量

调用方法传递的参数以及在调用中创建的临时变量都保存在栈中速度较快，其他变量，如静态变量、实例变量等，都在堆中创建，速度较慢。另外，栈中创建的变量随着方法的运行结束，这些内容就没了，不需要额外的垃圾回收

4、及时关闭流

进行数据库连接、I/O流操作时，在使用完毕之后，及时关闭以释放资源

5、减少对变量的重复计算

对方法的调用，即使方法中只有一句语句，也是有消耗的，包括创建栈帧、调用方法时保护现场、调用方法完毕时恢复现场
		
		for(int i=0; i< list.size(); i++)
		====
		for(int i=0, len = list.size(); i < len; i++)

6、懒加载
		String str = "xxx";
		if(i == 1) {
			list.add(str);
		}		
		====
		if(i == 1) {
			String str = "xxx";
			list.add(str);
		}

7、慎用异常

异常对性能不利，抛出异常首先要创建一个新的对象，Throwable接口的构造函数调用名为fillStackTrace的本地同步方法，该方法检查堆栈，收集调用跟踪信息。只要有异常被抛出，Java啊虚拟机就必须调整调用堆栈，因为在处理过程中创建了一个新的对象。异常只能用于错误处理，不应该用来控制程序流程

8、不要在循环中使用try...catch...

9、如果能估计到待添加的内容长度，为底层以数组方式实现的集合、工具类指定初始长度

10、当复制大量数据时，使用System.arraycopy

11、乘法和除法使用移位操作
		a = val * 8;
		b = val / 2;
		====
		a = val << 3;
		b = val >> 1;

12、循环内不要不断创建对象引用
		for(int i=1; i < count; i++) {
			Object obj = new Object();
		}				
		====
		Object obj = null;
		for(int i=1; i < count; i++) {
			obj = new Object();
		}

13、基于效率和类型检查的考虑，应该尽可能使用array，无法确定数组大小时才使用ArrayList

14、尽量使用HashMap、ArrayList、StringBuilder，除非线程安全需要，否则不推荐使用HashTable、Vector、StringBuffer，后三者使用了同步机制导致性能开销

15、不要将数组声明为public static final。因为毫无意义，只是定义了引用为static final 数组的内容还是可以随意改变的，将数组声明为public意味着这个数组可以被外部类所改变

16、在合适的场合使用单例

	1、控制资源的使用，通过线程同步来控制资源的并发访问
	2、控制实例的产生，以达到节约资源的目的
	3、控制数据的共享，在不建立直接关联的条件下，让多个不相关的进程或者线程之间实现通信

17、避免随意使用静态变量

当某个对象被定义为static的变量所引用，那么gc通常是不会回收这个对象所占有的堆内存的

18、及时清除不再需要的会话

为了清除不再活动的会话，许多应用服务器都有默认的会话超时时间，一般为30分钟。当应用服务器需要保存更多的会话时，如果内存不足，那么操作系统会把部分数据转移到磁盘，应用服务器也可能根据最近最频繁使用算法把部分不活跃的会话转存到磁盘，甚至可能抛出内存不足的异常。如果会话要被转存到磁盘。那么要先序列化，在大规模集群中，对对象进行序列化的代价是很昂贵的。当会话不在需要时，应该及时调用HttpSession的invalidate方法清除

19、实现RandomAccess接口的集合比如ArrayList，应当使用普通for循环而不是使用foreach来遍历

实现RandomAccess接口用来表明其支持快速的随机访问，此接口的主要目的是允许一般的算法更改其行为，从而将其应用到随机或连续访问列表时能提供良好的性能。如果是顺序访问，则使用Iterator效率会更高。 foreach循环的底层实现原理就是迭代器iterator

20、使用同步块代替同步方法

在多线程模块中的synchronized锁，除非鞥呢确定整个方法都需要进行同步，否则尽量使用同步代码块

21、将常量声明static final，并以大写命名

22、不要创建一些不使用的对象，不要导入一些不使用的类

23、程序运行过程中避免使用反射

不建议在程序运行中使用反射机制，特别是Method的invoke方法，如果确实有必要，一种建议性的做法是将那些需要通过反射加载的类在项目启动的时候通过发送实例化一个对象并存入内存

24、使用数据库连接池和线程池

25、使用带缓冲的输入输出流进行IO操作：BufferedReader、BufferedWriter BufferedInputStream BufferdOutputStream

26、顺序插入和随机访问比较多的场景使用ArrayList，元素删除和中间插入比较多的常用使用LinkedList

27、public方法中不应有过多的形参

28、字符串变量和字符串常量equals将字符串常量写前面

29、公用的集合类中不使用的数据一定要及时remove

如果一个集合类是公用的，那么这个集合里面的元素是不会自动释放的，因为始终有引用指向它们。

30、把一个基本数据类型转为字符串 toString最快，String.valueOf次之，+最慢

31、map遍历的正确方式
		HashMap<String, String> hm = new HashMap<String, String>();
		hm.put("1","a");
		Set<Map.Entry<String, String>> entrySet = hm.entrySet();
		Iterator<Map.Entry<String, String>> iter = entrySet.iterator();
		while(iter.hasNext()) {
			Map.Entry<String, String> entry = iter.next();
			entry.getKey() + " : " + entry.getValue(); 
		}	

32、对资源的close操作，分开操作

				