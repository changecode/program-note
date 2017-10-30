插件
===

默认情况下，mybatis允许使用插件来拦截
的方法如下

# Executor(update query flushStatements commit rollback getTra
nsaction close isClosed)拦截执行器方法

# ParameterHandler(getParameterObject
setParameters)拦截参数的处理

# ResultSetHandler(handleResultSets
handleOutputParameters)拦截结果集处理

# StatementHandler(prepare parameteri
ze batch update query)拦截sql语法构建

mybatis采用责任链模式，通过动态代理组织
多个拦截器，通过这些拦截器可以改变mybat
is的默认行为


Interceptor  
====

public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  Object plugin(Object target);

  void setProperties(Properties properties);

}

# Object intercept(Invocation invocation)
是实现拦截逻辑的地方，内部通过invocatio
n.proceed显式地推进责任链前进，即调用
下一个拦截器目标方法

#Object plugin(Object Target)就是用当前
这个拦截器生成对目标target的代理，实际
是通过Plugin.wrap(target, this)来完成
把目标target和拦截器this传给了包装函数

# setProperties(Properties p)用于设置
额外的参数，参数配置在拦截器的Propertie
s节点中


Plugin
====

根据@Interceptors注解，得到这个注解的
属性@Signature数组，然后跟根据每个
@Signature注解的type method arts属性，
使用反射找到对应的method，最终根据调用
的target对象实现的接口决定是否范湖一个
代理对象替代原先的target对象


自定义插件注意点
====

# 所有可能被拦截的处理类都会生成一个
代理

# 处理类代理在执行对应方法时，判断要不
药执行插件中的拦截方法

# 执行插件中的拦截方法后，推进目标的
执行

如果有N个插件，就有N个代理，每个代理都
要执行上面的逻辑。这里面的层层代理要多
此生成动态代理，是比较影响性能的。虽然
能指定插件拦截的位置，但这个是在执行
方法时动态判断，初始化的时候就是简单的
吧插件包装到了所有可以拦截的地方

实现plugin方法时判断一下目标类型，是本
插件要拦截的对象才执行Plugin.wrap方法
否则直接返回目标本身，这样可以减少目标
被代理的次数