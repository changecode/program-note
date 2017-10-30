简介
===

fork/join并行方式是获取良好的并行计算性能的一种最
最简单同时也是最有效的设计计算。fork/join并行算法是
我们所熟悉的分治算法的并行版本。典型用法如下：
		result solve(problem problem) {
			if(problem is small) {
				directly solve problem
			}else {
				split problem into independent parts
				fork new subtasks to solve each part
				join all subtasks
				compose result from subresults
			}
		}
fork操作将会启动一个新的并行fork/join子任务。join
操作会一直等到直到所有的子任务都结束。fork/join算法
如其他的分治算法一样，总是会递归的、反复的划分子任
务，直到这些子任务可以用足够简单的、短小的顺序方法
来执行


设计
===

fork/join程序可以在任何执行以下特性的框架之上运行：
架构能够让构建的子任务并行执行，并且拥有一种等待子
任务运行结束的机制。

Java标准的线程框架对fork/join而言太笨重了。fork/joi
n设计：

1 一组工作这线程池是准备好的。每个工作线程都是标准
的处理存放在队列中任务的线程。通常情况下，工作线程
应该与系统的处理器数量一致。对于一些原生的框架例如
cilk，他们首先将映射成内核线程或者是轻量级的进程，
然后再在处理器上面运行。在Java中，虚拟机和操作系统
需要相互结合来完成线程到处理器的映射。然后对应计算
密集型的运算来说，这种映射对应操作系统来说是一种相对简单的任务。任何合理的映射策略都会导致线程映射到
不同的处理器

2 所有的fork/join任务都是轻量级执行类的实例。而不是
线程实例。在Java中，独立的可执行任务必须要实现Runna
ble接口并重写run方法。

3 采用一个特殊的队列和调度原则来管理任务并管理任务
并通过工作线程来执行任务。这些机制是由任务类中提供
的相关方式实现的：主要是由fork join isDone
forkjointask.java
示例：
		class Fib extends FJTask {
			static final int threshold = 13;
			volatile int number;

			Fib(int n) {
				number = n;
			}

			int getAnswer() {
				if(!isDone()) {
					throw new IllegalStateException();
					return number;
				} 
			}

			public void run() {
				int n = number;
				if(n <= threshold) {
					number = seqFib(n);
				}else {
					Fib f1 = new Fib(n ? 1);
					Fib f2 = new Fib(n ? 2);
					coInvoke(f1, f2);
					number = f1.number + f2.number;
				}
			}

			public static void main(String[] args) {
				try{
					int groupSize = 2;
					FJTaskRunnerGroup group = 
					new FJTaskRunnerGroup(groupSize);
					Fib f = new Fib(35);
					group.invoke(f);
					int result = f.getAnswer();
					System.out.println(result);
				}
			}

			int seqFib(int n) {
				if (n <= 1) return n;
				else return seqFib(n ? 1) + seqFib(n ? 2);
			}
		}	

1 对应工作线程的创建数量，通常情况下可以与平台所
拥有的处理器数量保持一致

2 一个粒度参数代表了创建任务的代价会大于并行化所
带来的潜在的性能提升的临界点


work-stealing
====

fork/join框架的核心在于轻量级调度机制：

1 每一个工作线程维护自己的调度队列中的可运行任务
2 队列以双端队列的形式被维护，不仅支持后进先出LIFO
的push 和pop操作，还支持先进先出FILE的take操作
3 对于一个给定的工作线程来说，任务所产生的自认为将
会被放入到工作者自己的双端队列中
4 工作线程使用后进先出，通过弹出任务来处理队列中的
任务
5 当一个工作线程的本地没有任务去运行的时候，它将
使用先进先出的规则尝试随机从别的工作线程中拿一个
任务去运行
6 当一个工作线程触及了join操作，如果可能的话它将
处理其他任务，直到目标任务呗告知已经结束。所有的
任务都会无阻塞的完成
7 当一个工作线程无法再从其他线程中获取任务和失败
处理的时候，就会推出并经过一段时间之后再度尝试直到
所有的工作线程都被告知他们都处于空闲的状态。在这种
情况下，他们都阻塞直到其他的任务再度被上层调用

使用后进先出-LIFO用来处理每个工作线程的自己任务，
但是使用先进先出FIFO规则用于获取别的任务，这是一种
呗广泛使用的进行递归fork/join设计的一种调优手段。

让偷取任务的线程动队列拥有者相反的方向进行操作会减
减少线程竞争。同样体现了递归分治算法的打任务优先
策略


加速比
====

虽然我们无法保证Java虚拟机是否总是能够将每一个线程
映射到不同的空闲的CPU上，有可能映射一个新的线程到
CPU的延迟会随着线程数目的增加而变大，也可能会随不同的系统以及不同的测试程序而变化。但是，增加线程的
数目确实能够增加使用的CPU的数目

垃圾回收
====

现在的垃圾回收机制的性能是能够与fork/join框架所匹
配的：fork/join程序在运行时会产生巨大数量的任务单
元，然而这些任务再被执行之后又会很快转变为内存垃圾
相比较顺序执行的但线程程序，在任何时候，其对应的
fork/join程序需要最多P倍的内存空间(p为线程数量)基
于分代的半空间拷贝垃圾回收器能够很好的处理这种情况
因为这种垃圾回收机制在进行内存回收的时候仅仅拷贝
非垃圾内存单元。这样做，避免了在手工并发内存管理上
的一个复杂的问题，即跟踪那些被一个线程分配却在另一
个线程中使用的内存单元。 
然而，只有内存使用率达到一个很高的情况下，垃圾回收
机制才会成为影响扩展性的一个因素，因为这种情况下，
虚拟机必须经常停止其他线程来进行垃圾回收

任务同步
====

任务窃取模型经常会在处理任务的同步上遇到问题，如果
工作线程获取任务的时候，但相应的队列已经没有任务可
供获取，这样就会产生竞争。在forkjointask框架中，这
种情况有时会导致线程强制睡眠。

任务局部性
====

ForkJoinTast在任务分配上都是做了优化的，尽可能多的
使工作线程处理自己分解产生的任务。因为如果不这样做
，程序的性能就会受到影响：
1、从其他队列窃取任务的开销要 比自己队列执行pop
操作的开销大
2、在大多数程序中，任务操作是一个共享的数据单元，
如果只运行自己部分的任务可以获得更好的局部数据访问