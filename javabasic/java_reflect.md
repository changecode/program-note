java反射机制的实现原理
----

Java的反射机制的实现要借助四个类：class constructor field method; class代表的是类对象， constructor类的构造器对象， Field类的属性对象， method类的方法对象，通过这四个对象可以看到一个类的组成部分。

class：程序运行时，Java运行时系统会对所有的对象进行运行时类型的处理，这项信息记录了每个对象所属的类，虚拟机通常使用运行时类型信息选择正确的方法来执行。 在object类中定义了getClass方法，可以通过这个方法获得制定对象的类对象。

第二步是调用诸如getDeclaredMethods方法，以取得该类中定义的所有方法的列表。一旦取得这个新消息，就可以进行第三部，使用reflection来操作这些信息

		Class c = Class.forName("java.lang.String");
		Method[] m = c.getDeclaredMethods();
		System.out.println(m[0].toString());
		Field[] f = c.getDeclaredFields();
		

java.lang.reflect.AccessibleObject类，定义一种setAccessible方法，使我们能够启动或关闭对这些类中其中一个类的实例的接入检测。







