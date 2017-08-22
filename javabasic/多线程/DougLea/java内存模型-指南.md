Java内存模型_指南
===

单处理器
====

如果能保证正在生成的代码运行在单个处理器上，那就
可以跳过本节的其余部分。因为单处理器保持这明显的
顺序一致性，除非对象内存以某种方式与可异步访问的
IO内存共享，否则永远都不需要插入屏障指令。采用了
特殊映射的java.nio.buffers可能护出现这种情况，但
也许只会影响内部的JVM支持代码，而不影响Java代码
而且，可以想象，如果上下文切换时不要求充分的同步
那就需要使用一些特殊的屏障了


插入屏障
====

当程序执行时碰到了不同类型的存取，就需要屏障指令
几乎无法找到一个最理性位置，能将屏障执行总次数
降到最下。编译器不知道指定的load或store指令时先
于海上后于需要一个配置操作的另一个load或store指
令：当volatile store后面是一个return时，最简单
保守的策略是为了任一给定的load store lock或unloc
k生成代码时，都假设该类型的存取需要最重量级的屏
障：

1、在每条volatile store指令之前插入一个storestor
屏障

2、如果一个类包含final字段，在该类每个构造器的全
部store指令之后，return指令之前插入一个storestor
e屏障。

3、在每条volatile store指令之后插入一套storeload
屏障。注意，虽然也可以在每条volatile load指令之
前插入一个storeload屏障，但对于使用volatile的
典型程序来说则会更慢，因为读操作会大大超过写操作
或者可以的话，将volatile store实现成一条原子指令
就可以省略这个屏障操作。如果原子指令比storeload
屏障成本低，这个方式就更高效

4、在每条volatile load指令之后插入loadload和
loadstore屏障。在持有数据依赖顺序的处理器上，如
果吓一跳存取指令依赖于volatile load处理的值，就
不需要插入屏障。特别是，在load一个volatile引用之
后，如果后续指令时null检查或load此引用所指对象中
的某个字段，此时就无需屏障

5、在每条monitorenter指令之前或在每条monitorexit
指令之后插入一个exitenter

6、在每条monitorenter指令之后插入enterload和
enterstore屏障

7、在每条monitorexit指令之前插入storeexit和load
exit屏障

8、如果在未内置支持间接load顺序的处理器上，可在
final字段的每条load指令之前插入一个loadload屏障

这些屏障中的有一些通常简化成空操作，实际上，大部
分都会简化成空操作，只不过在不同的处理器和锁模式
下使用了不同的方式。


移除屏障
====

上面的保守策略对有些程序来说也许还能接受。volati
le的主要性能问题出在于store指令相关的storeLoad
屏障上。将volatile主要用于避免并发程序里读操作中
锁的使用，仅当读操作大大超过写操作才会有问题

1、重排代码以更进一步移除loadload和loadstore屏障
这些屏障因处理器维持着数据依赖而不再需要

2、移动指令流中屏障的为准以提高调度频率，只有在
该屏障被需要的时间内最终会在某处执行即可

3、移除那些没有多线程依赖而不需要的屏障：例如
某个volatile变量被证实只会对单个线程可见。而且
如果能证明线程仅能对某些特定字段执行store指令或
仅能执行load指令，则可以移除这里面使用的屏障


杂记
====

1、Thread.start需要屏障来确保该已启动的线程能看
到调用的时刻对调用者的所有store的内存。相反，
Thread.join需要屏障来确保调用者能看到正在终止的
线程所store内容。实现Thread.start和thread.join
时需要同步，这些屏障通常是通过这些同步来产生的

2、static final初始化需要storestore屏障，遵守
Java类加载和初始化规则的那些机制需要这些屏障

3、在构造器之外或静态初始化器之外神秘设置system
.in system.out和system.err的JVM私有例程需要特别
注意，因为他们是JMM final字段规则的遗留的例外
情况

4、类似的，JVM内部反序列化设置final字段的代码
通常需要一个storestore屏障

5、终结方法可能需要屏障(垃圾收集器里)来确保
object.finalize中的代码能看到某个对象不再被引用
之前store到该对象所有字段的值。这通常是通过同步
来确保的，这些同步用于在reference队列中添加和
删除reference

6、调用JNI例程以及从JNI例程中返回可能需要屏障
尽管这看起来是实现方面的一些问题

7、多数处理器都设计有其他专用于IO和OS操作的同步
指令。它们不会直接影响JMM的这些问题。但有可能
与IO，类加载以及动态代码的生成紧密相关