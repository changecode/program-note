Semaphore
----

信号量Semaphore是一个控制访问多个共享资源的计数器，本质上是一个共享锁

Java并发提供了二种加锁模式：共享锁和独占锁。ReentrantLock是独占锁，每次只能有一个线程持有，共享锁运行多个线程并行持有锁，并发访问共享资源

独占锁采用的是一种悲观的加锁策略，对应写而言为了避免冲突独占是可以的，但是对于读就没必要了。 如果某个只读线程获取独占锁，则其他读线程都只能等待了，这种情况下就现在了不必要的并发性。共享锁采用了乐观锁机制，允许多个读线程同时访问同一个共享资源

信号量Semaphore是一个非负整数(>= 1)。当一个线程想要访问某个共享资源时，它必须要先获取Semaphore，当其大于0时，获取该资源并信号量-1。如果Semaphore=0，则表示全部的共享资源已经被其他线程全部占用，线程必须等待其他线程释放资源。当线程释放资源时，Semaphore+1； 当信号量=0时，可以当做互斥锁使用。

		public class PrintQueue {
			private final Semaphore semaphore;

			public PrintQueue() {
				semaphore = new Semaphore(1);
			}

			public void printJob(Object document) {
				try{
					semaphore.acquire();
					long duration = (long) (Math.random() * 10);
					System.out.println(Thread.currentThread().getName() + 
					"printqueue printing a job during " + duration);
					Thread.sleep(duation);
					}finally{
						semaphore.release();
				}
			}
		}
		=====
		public class Job implements Runnbale {
			private PrintQueue printQueue;

			public Job(PrintQueue queue) {
				this.printQueue = printQueue;
			}

			@Override
		    public void run() {
		        System.out.println(Thread.currentThread().getName() + " Going to print a job");
		        printQueue.printJob(new Object());
		        System.out.println(Thread.currentThread().getName() + " the document has bean printed");
		    }
		}
		=====
		public class Test {
		    public static void main(String[] args) {
		        Thread[] threads = new Thread[10];
		        
		        PrintQueue printQueue = new PrintQueue();
		        
		        for(int i = 0 ; i < 10 ; i++){
		            threads[i] = new Thread(new Job(printQueue),"Thread_" + i);
		        }
		        
		        for(int i = 0 ; i < 10 ; i++){
		            threads[i].start();
		        }
		    }
		}