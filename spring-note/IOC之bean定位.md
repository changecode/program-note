IOC之bean定位
===

声明 本文来源http://blog.csdn.net/chjttony/article/details/6256709

IOC容器初始化
====

IOC容器初始化包括：bean定义资源文件、
载入和注册3个基本过程

1、定位
bean定义资源文件定位由ResourceLoader通
过统一的Resource接口来完成，Resource
接口将各种形式的bean定义资源文件封装
成统一的、IOC容器可进行载入操作的对象

2、载入
bean定义资源文件载入的过程是将bean定义
资源文件中配置的bean转换成IOC容器中所
管理的bean的数据结构形式。IOC中管理的
bean数据结构是BeanDefinition，BeanDefin
ition是POJO对象在IOC容器中的抽象

3、注册
通过调用BeanDefinitionRegistry接口把从
bean定义资源文件中解析的bean向IOC容器
进行注册，在IOC容器内部，是通过hashmap
存储bean对象数据
注意： IOC容器和上下文初始化一般不包含
bean依赖注入的实现，一般而言，依赖注入
发送在应用第一次通过getBean方法向容器
获取bean时。但有个特例，IOC容器预实例化
配置的lazyinit属性，如果某个bean设置了
lazyinit属性，则该bean的依赖注入在IOC
容器初始化时就预先完成了

Bean资源文件定位过程
====

ApplicationContext是一个在BeanFactory
基础上提供了扩展的接口，具体的IOC容器
实现常有：从文件系统中读取bean资源文件
classPathXmlApplicationContext从classp
ath类路径中读入bean定义资源文件，XmlWeb
ApplicationContext从web容器读入bean资源
文件


可以使用一个字符串配置多个springbean
，也可以使用字符串数组

ClasspathResource res = new ClasspathResource(“a.xml,b.xml,……”);

ClasspathResource res = new ClasspathResource(newString[]{“a.xml”,”b.xml”,……});