Annotation
===

Annotation是Java5开始引入的特性。提供
了一种安全的类似于注释和Java doc的机制


Meta Annotation
====

@Target
=====

@Target说明了Annotation所修饰的对象范围
Annotation可被用于packages types 类型
成员(方法、构造方法、成员变量、枚举值)
方法参数和本地变量。

@Target作用：用于描述注解的使用范围

@Target取值
1、CONSTRUCTOR  用于描述构造器
2、FIELD  用于描述域
3、LOCAL_VARIABLE  用于描述局部变量
4、METHOD  用于描述方法
5、PACKAGE  用于描述包
6、PARAMETER  用于描述参数
7、TYPE  用于描述类、接口或enum声明

@Retention
=====

该注解定义了Annotation的生命周期。某些
Annotation仅出现在源代码中，而被编译器
丢弃；而另一些却被编译在class文件中；
编译在class文件中的Annotation可能被虚拟
机忽略，而另一些在class被装载时将被读取
Annotation与class在使用上是被分离的。
@Retention有唯一的value作为成员

@Retention作用  表示需要在什么级别保存
该注释信息，用于描述注解的声明周期

@Retention取值来自java.lang.annotation.
RetentionPolicy的枚举类型值
1、SOURCE  源文件中有效
2、CLASS   class文件中有效
3、RUNTIME  运行时有效

@Document
=====

用户描述其类型的Annotation应该被标注的
程序成员的公共API，该注解只是一个标记注
解，没有成员

@Inherited
====


是一个标记注解，如果一个使用了@Inherite
d修饰的Annotation类型被用于一个class，
则这个Annotation将被用于该class的子类


自定义Annotation
====

在实际项目中，经常会碰到下面这种场景，
一个接口的实现类或者抽象类的子类很多，
经常需要根据不同的情况实例化并使用不同
的子类。典型的例子是结合工厂使用职责链
模式。 此时，可以为每个实现类加上特定的
Annotation，并在Annotation中给该类取一
个标识符，应用程序可通过该标识符来判断
一个实例化哪个子类

例子
=====

自定义注解
@Target(Elmentype.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Document
@Inherited
public @interface Component{
	String identifier() default "";
}

实现类加上@Component
@Component(identifier="upper")
public class UpperCaseComponent {
	public String doWork(String input){
		return input.toUpperCase();
	}
}

反射获取UpperCaseComponent对应的identif
ier
public class Client {
	public static void main(String[] args) {
		try{
			Class componentClass = 
			Class.forName("xx.xx.UpperCaseComponent");
			if(componentClass.isAnnotationPresent(Component.class)) {
				Component component = 
				(Component)componentClass.getAnnotation(Component.class);
				String identifier = 
				component.identifier();
			}
		}catch(ClassNotFoundException e) {
			e.printStackTrace();
		}
	}
}

Annotation与interface的异同
====

Annotation的方法定义是受限制的。其方法
必须声明为无参数、无异常抛出的。这些
方法同上也定义了Annotation的成员--方法
名即为成员名，而方法返回类型即为成员
类型。方法返回类型必须为Java基础类型、
class类型、枚举类型、Annotation类型或
者相应的一维数组。方法后面可以使用
default关键字和一个默认值来声明成员的
默认值，null不能作为成员默认值。成员
一般不能是泛型，只有其类型是class时可
使用泛型

Annotation和interface都可以定义常量、
静态成员类型也都可以被实现或者继承