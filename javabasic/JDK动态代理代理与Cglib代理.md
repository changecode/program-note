前言
===

Java中常见的代理有jdk代码和cglib代理，无论是AOP
还是mybatis动态生成数据库操作类无一不是通过代理
来实现


JDK代理
===

试验测试
====

		public interface UserService {
			public abstract void add();
		}

		//impl class
		public class UserServiceImpl implements
		UserService {
			@Override
			public void add() {
				sys.out("---add---");
			}
		}

		//InvocationHandler class
		public class MyInvocationHandler 
		implements InvocationHandler {
			private Object target;

			public MyInvocationHandler(Object 
				target) {
					super();
					this.target = target;
				}

			@Override
			public Object invoke(Object proxy,
				Method method, Object[] args) 
				throws Throwable {
					PerformanceMonior.begin(
					target.getClass().getName()
					+ "." + method.getName());
					sys.out("begin--" + method.
					getName());
					Object result = method.invoke(target, args);
					sys.out("end" + method.getName());
					PerformaneMonir.end();
					return result;
			}

			public Object getProxy() {
				return Proxy.newProxyInstance(
					Thread.currentThread().get
					ContextClassLoader(),target
					.getClass().getInterfaces(),
					this);
			}	
		}

		//test class
		public static void main(String[] args) {
			//生成的代理类保存到磁盘
			System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles",
			"true");

			UserService service = new UserServiceImpl();
			MyInvocationHandler handler = new
			MyInvocationHandler(service);
			UserService proxy = (UserService)
			handler.getProxy();
			proxy.add();
		}
		
		UserServiceImpl被jdk代理后的类，在项目
		的com.sun.proxy下面生成#Proxy0.class类

		package com.sun.proxy;
		import java.lang.reflect.InvocationHandler;
		import java.lang.reflect.Method;
		import java.lang.reflect.Proxy;
		import java.lang.reflect.UndeclaredThrowableException;
		import proxy.JDK.UserService;

		public final class $Proxy0
		  extends Proxy
		  implements UserService
		{
		  private static Method m1;
		  private static Method m3;
		  private static Method m0;
		  private static Method m2;

		  public $Proxy0(InvocationHandler paramInvocationHandler)
		  {
		    super(paramInvocationHandler);
		  }

		  public final void add()
		  {
		    try
		    {
		    //第一个参数是代理类本身，第二个是实现类的方法，第三个是参数
		      this.h.invoke(this, m3, null);
		      return;
		    }
		    catch (Error|RuntimeException localError)
		    {
		      throw localError;
		    }
		    catch (Throwable localThrowable)
		    {
		      throw new UndeclaredThrowableException(localThrowable);
		    }
		  }

		 ...

		  static
		  {
		    try
		    {

		      m3 = Class.forName("proxy.JDK.UserService").getMethod("add", new Class[0]);
		    ...
		      return;
		    }
		    catch (NoSuchMethodException localNoSuchMethodException)
		    {
		      throw new NoSuchMethodError(localNoSuchMethodException.getMessage());
		    }
		    catch (ClassNotFoundException localClassNotFoundException)
		    {
		      throw new NoClassDefFoundError(localClassNotFoundException.getMessage());
		    }
		  }
		}
		也就是说main函数里面的proxy时间就是
		$Proxy0的一个实例对象，可知JDK动态代理是使用接口生成新的实现类，实现类里面则
		委托给InvocaHandler，InvocationHandler
		里面则调用被代理的类方法


cglib代理
===

试验测试
====
	
		public void testCglibProxy() {
			//生成代理类到本地
    		System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/Users/zhuizhumengxiang/Downloads");

    		UserServiceImpl service = new
    		UserServiceImpl();
    		CglibProxy cp = new CglibProxy();
    		UserService proxy = (UserService)
    		cp.getProxy(service.getClass());
    		proxy.add();
    		proxy.sub();
    		proxy.hello("xxx");
    		proxy.service("xxx");
    		proxy.toString();
		}		
		//methodInrerceptor类
		public class CglibProxy implements
		MethodIntercepor {
			Private Enhancer enhancer = 
			new Enhancer();

			public Object getProxy(Class clazz) {
				enhancer.setSuperclass(clazz);
				enhancer.setCallback(this);
				//enhancer.setCallbackType(clazz);
				return enhancer.create();
			}

			@Override
			public Object intercept(Object obj, Method method, Object[] args,
				MethodProxy proxy) throws Throwable {
					PerformanceMonior.begin(obj.getClass().getName()+"."+method.getName());
       				Object result = proxy.invokeSuper(obj, args);
      				//  Object result = method.invoke(obj, args);
        			PerformanceMonior.end();

        		return result;
			}
		}

		cglib是通过直接继承被代理类，并委托为
		回调函数来做具体的事情。从代理类里面可
		知道对于原来的add函数，代理类里面对应
		了二个函数分布式add和cglib$add其中后者
		是在方法拦截里面调用的。前者则是使用代
		理类时候调用的函数。当代码调用add时候，
		会具体调用到方法拦截器的intercept方法
		该方法内则通过proxy.invokeSuper调用
		cglib$add


总结
===

对应jdk动态代理机制是委托机制，具体来说动态
实现接口类，在动态生成的实现类里面委托为handler
去调用原始实现类方法。
比如接口类interfaceA 实现类interfaceImplA 
interfaceImplA的代理类为$ProxyInterfaceImplA
那么$ProxyInterfaceImplA是能赋值给interfaceA
因为前者间接实现了候着，但是$ProxyInterfaceImplA不能赋值给interfaceImplA因为他们没有继承或者
实现关系。所以在项目中autowired都是interfaceA
进行的，不是对interfaceImplA

对应cglib则使用的继承机制，具体说被代理类和代理
类是继承关系，所以代理类是可以赋值给被代理类的
如果被代理类有接口，那么代理类也可以赋值给接口。

JDK代理只能对接口进行代理，CGLIB则是对实现类进
行代理