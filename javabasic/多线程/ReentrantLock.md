ReentrantLock
----

定义：可重入的互斥锁，是一种递归无阻塞的同步机制。可以等同于synchronized的使用，但是比其更强大、灵活可减少死锁的发生的概率。ReentrantLock将由最近成功获得锁，并且还没有释放该锁的线程所拥有。当锁没有被另一个线程所拥有时，调用lock的线程将成功获取该锁并返回。如果当期线程已经拥有该锁，此方法立即返回，可以使用isHeldByCurrentThread和getHoldCount方法来检查此情况释放发生。

ReentrantLock提供公平锁机制，构造方法接受一个可选的公平参数。设置为true时，则为公平锁，这些锁将访问权授予等待时间最长的的线程。否则该锁将无法保证线程获取锁的访问顺序。
		/**
	     * 默认构造方法，非公平锁
	     */
	    public ReentrantLock() {
	        sync = new NonfairSync();
	    }

	    /**
	     * true公平锁，false非公平锁
	     * @param fair
	     */
	    public ReentrantLock(boolean fair) {
	        sync = fair ? new FairSync() : new NonfairSync();
	    }
	    //简单实用示例
	    public class PrintQueue {
    		private final Lock queueLock = new ReentrantLock();
    
		    public void printJob(Object document){
		        try {
		            queueLock.lock();
		            System.out.println(Thread.currentThread().getName() + ": Going to print a document");
		            Long duration = (long) (Math.random() * 10000);
		            System.out.println(Thread.currentThread().getName() + ":PrintQueue: Printing a Job during " + (duration / 1000) + " seconds");
		            Thread.sleep(duration);
		            System.out.printf(Thread.currentThread().getName() + ": The document has been printed\n");
		        } catch (InterruptedException e) {
		            e.printStackTrace();
		        }finally{
		            queueLock.unlock();
		        }
		    }
		}

		public class Job implements Runnable{
    
		    private PrintQueue printQueue;
		    
		    public Job(PrintQueue printQueue){
		        this.printQueue = printQueue;
		    }
		    
		    @Override
		    public void run() {
		        printQueue.printJob(new Object());
		    }
		}	

		public class Test {
		    public static void main(String[] args) {
		        PrintQueue printQueue = new PrintQueue();
		        
		        Thread thread[]=new Thread[10];
		        for (int i = 0; i < 10; i++) {
		            thread[i] = new Thread(new Job(printQueue), "Thread " + i);
		        }
		        
		        for(int i = 0 ; i < 10 ; i++){
		            thread[i].start();
		        }
		    }
		}

ReentrantLock与synchronized区别
----
1、ReentrantLock提供了更多具备更强的扩展性。例如：时间锁等候，可中断锁等待，锁投票

2、ReetrantLock提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更适合

3、ReentrantLock提供了可轮询的锁请求。它会尝试去获取锁，如果成功则继续，否则可以等到下次运行时除了，而synchronized则一旦进入锁请求要不成功要不阻塞

4、ReentrantLock支持更加灵活的同步代码块，使用synchronized时，只能在同一个synchronized块结构中获取和释放。ReentrantLock的锁释放一定要在finally中处理

5、ReentrantLock支持中断处理
