1、 synchronized(this)
----
1、当二个并发线程访问同一个对象object中的这个synchronized(this)同步代码块时，一个时间内只能有一个线程得到执行。另一个线程必须等待当前线程执行完这个代码块以后才能执行该代码块

2、当一个线程访问object的一个synchronized(this)同步代码块时，另一个线程仍然可以访问object中的非synchronized同步代码块

3、当一个线程访问object的一个synchronized同步代码块时，其他线程对object中所有其他synchronized(this)同步代码块的访问将被阻塞

2、 锁的升级
----
java中锁一共有四种状态，无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，它会随着竞争情况逐级升级。锁可以升级但不能降级。

1、锁自旋：当某个线程在进入同步方法/代码块时若发现该同步方法/代码块被其他线程所占，则它就要等待，进入阻塞状态，这个过程性能是地下的。在遇到锁的争用或等待时，线程可以不那么着急进入阻塞状态，而是等一等，看看锁是不是马上就释放了，这就是锁自旋

2、偏向锁:在大多数情况下锁锁不仅不存在多线程竞争，而且总是有同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。当某个线程获得锁的情况，该线程是可以多次锁住该对象，但是每次只需这样的操作都会因为CAS操作而造成一些开销而消耗性能，为了减少这种开销，这个锁会偏向于第一个获得它的线程，如果在接下来的执行过程中，该锁没有被其他的线程获取，则持有偏向锁的线程将永远不需要再进行同步。 当有其他线程在尝试着竞争偏向锁时，持有偏向锁的线程就会释放锁

3、 轻量级锁:提升程序同步性能的依据是对于绝大部分的锁，在整个同步周期内都是不存在竞争的，这是一个经验数据。轻量级锁在当前线程的栈帧中建立一个名为锁记录的空间，用于存储锁对象目前的指向和状态。如果没有竞争，轻量级锁使用CAS操作避免了使用互斥量的开销，但如果存在锁竞争，除了互斥量的开销外，还额外发生了CAS操作，因此在有竞争的情况下，轻量级锁比传统的重量级锁更慢。

3、锁的可重入性
----
		public class A {
			public synchronized void method() {}
		}	
		public class B extends A{
			public synchronized method() {
				super.method();
			}
		}

java多线程的课重入性的实现是通过每个锁关联一个请求计算和一个占用它的线程，当计数为0，认为该锁是没有被占用的，那么任何线程都可以获得该锁的专用权。当某一个线程请求成功后，JVM会记录该锁的持有线程，并且将计数设置为1，如果这时其他线程请求该锁时则必须等待。当该线程再次请求获得锁时，计数会加1；当占用线程退出同步代码块时，计数就会减1，直到为0，释放该锁。这时其他线程才有机会获得该锁的占用权。	
