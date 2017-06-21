ReentrantLock
----
ReentrantLock实现lock接口，Sync与ReentrantLock是组合关系，且FairSync(公平锁)、NonfairySync(非公平锁)是Sync的子类。Sync基础AQS(**AbstractQueuedSynchronized**)

AbstractQueuedSynchronized ：Java中管理锁的抽象类。该类为实现依赖于先进先出等待队列的阻塞锁和相关同步器(信号量、事件等待)提供一个框架。 此类的设计目标是称为依靠单个原子int值来表示状态的大多数同步器的一个有用基础。子类必须定义更改此状态的受保护方法，并定义哪种状态对应此对象意味着被获取或被释放。嘉定这些条件之后，此类中的其他方法就可以实现所有排队和阻塞机制。子类可以维护其他状态字段，但至少为了获得同步而只追踪使用getState setState 和compareAndSetState方法来操作以原子方式更新int值。

CLH: AQS中等待锁的线程队列。在多线程环境中为了保护资源的安全性常使用锁将其保护起来，同一时刻只能有个线程能够访问，其余线程则需要等待，CLH就是管理哲学等待锁的队列

CAS:比较并交换函数，通过CAS操作的数据都是以原子方式进行的。

公平锁:lock
----
		final void lock(){
			//对应独占锁而言，如果所处可获取状态，其状态为0，当锁初次被线程获取时状态变成1
			acquire(1);
		}
		======
		public final void acquire(int arg) {
			if(!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) selfInterrupt();
		}
		======
		protected final boolean tryAcquire(int acquires) {
			final Thread current = Thread.currentThread();
			int c = getState();

			/*
         	* 当c==0表示锁没有被任何线程占用，在该代码块中主要做如下几个动作：
         	* 则判断“当前线程”是不是CLH队列中的第一个线程线程（hasQueuedPredecessors），
         	* 若是的话，则获取该锁，设置锁的状态（compareAndSetState），
         	* 并切设置锁的拥有者为“当前线程”（setExclusiveOwnerThread）。
         	*/
			if(c == 0) {
				if(!hasQueuedPredecessors() && 
				compareAndSetState(0, acquires)) {
					setExclusiveOwnerThread(current);
					return true;
				}
			}
			else if(current == getExclusiveOwnerThread()) {
				int nextc = c + acquires;
				if(nextc < 0) {
					throw new Error("maxnum lock count exceeded");
					setState(nextc);
					return true;
				}
				return false;
			}
		}
		=====
		public final boolean hasQueuedPredecessors() {
			Node t = tail;
			Node h = head;
			Node s;
			return h != t && ((s = h.next) == null ||
			s.thread != Thread.currentThread());
		}
		=====
		//设置当前线程为锁的拥有者
		protected final void setExclusiveOwnerThread(Thread t) {
			exclusiveOwnerThread = t;
		}
		=====
		//将当前线程加入CLH队列尾部
		private Node addWaiter(Node mode) {
			Node node = new Node(Thread.currentThread(), mode);
			//队列尾部
			Node pred = tail;
			if(pred != null) {
				node.prev = pred;
				if(compareAndSetTail(pred,node)) {
					pred.next = node;
					return node;
				}
			}
			enq(node);//队列为空时，新建CLH队列，再讲当前线程添加到CLH队列中
			return node;
		}
		====
		//新建队列
		private Node enq(final Node node) {
			for(;;) {
				Node t = tail;
				if(t == null) {
					if(compareAndSetHead(new Node())) {
						tail = head;
					}
				} else {
					node.prev = t;
					if(compareAndSetTail(t, node)) {
						t.next = node;
						return t;
					}
				}
			}
		}
		========
		final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            //线程中断标志位
            boolean interrupted = false;
            for (;;) {
                //上一个节点，因为node相当于当前线程，所以上一个节点表示“上一个等待锁的线程”
                final Node p = node.predecessor();
                /*
                 * 如果当前线程是head的直接后继则尝试获取锁
                 * 这里不会和等待队列中其它线程发生竞争，但会和尝试获取锁且尚未进入等待队列的线程发生竞争。这是非公平锁和公平锁的一个重要区别。
                 */
                if (p == head && tryAcquire(arg)) {
                    setHead(node);     //将当前节点设置设置为头结点
                    p.next = null; 
                    failed = false;
                    return interrupted;
                }
                /* 如果不是head直接后继或获取锁失败，则检查是否要阻塞当前线程,是则阻塞当前线程
                 * shouldParkAfterFailedAcquire:判断“当前线程”是否需要阻塞
                 * parkAndCheckInterrupt:阻塞当前线程
                 */
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);     
        }
        p == head && tryAcquire(arg) 
		首先，判断“前继节点”是不是CHL表头。如果是的话，则通过tryAcquire()尝试获取锁。 
		其实，这样做的目的是为了“让当前线程获取锁”，但是为什么需要先判断p==head呢？理解这个对理解“公平锁”的机制很重要，因为这么做的原因就是为了保证公平性！ 
      	(a) 前面，我们在shouldParkAfterFailedAcquire()我们判断“当前线程”是否需要阻塞； 
      	(b) 接着，“当前线程”阻塞的话，会调用parkAndCheckInterrupt()来阻塞线程。当线程被解除阻塞的时候，我们会返回线程的中断状态。而线程被解决阻塞，可能是由于“线程被中断”，也可能是由于“其它线程调用了该线程的unpark()函数”。 
      	(c) 再回到p==head这里。如果当前线程是因为其它线程调用了unpark()函数而被唤醒，那么唤醒它的线程，应该是它的前继节点所对应的线程(关于这一点，后面在“释放锁”的过程中会看到)。 OK，是前继节点调用unpark()唤醒了当前线程！ 
		此时，再来理解p==head就很简单了：当前继节点是CLH队列的头节点，并且它释放锁之后；就轮到当前节点获取锁了。然后，当前节点通过tryAcquire()获取锁；获取成功的话，通过setHead(node)设置当前节点为头节点，并返回。 
       	总之，如果“前继节点调用unpark()唤醒了当前线程”并且“前继节点是CLH表头”，此时就是满足p==head，也就是符合公平性原则的。否则，如果当前线程是因为“线程被中断”而唤醒，那么显然就不是公平了。这就是为什么说p==head就是保证公平性！
       	======
       	判断当前线程释放需要阻塞
       	private static boolean shouldParkAfterFialedAcquire(Node pred, Node node) {
       		//当前节点的状态
       		int ws = pred.waitStatus();
       		if(ws == Node.SIGNAL) {
       			return true;
       		}
       		if(ws > 0) {
       			do{
       				node.prev = pred = pred.prev;
       			}while(pred.waitStatus > 0);
       			pred.next = node;
       		}else {
       			compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
       		}
       		return false;
       	}

以上过程总结     
首先通过tryAcquire方法尝试获取锁，如果成功直接返回，否则通过acquireQueued再次获取，在acquireQueued中会先通过addWaiter将当前线程加入到CLH队列的队尾，在CLH队列中等待。在等待过程中线程处于睡眠状态，直到成功获取锁才会返回。

非公平锁NonfairSync
----

非公平锁采用非公平策略，无视等待队列，直接尝试获取
 		final void lock() {
 			if(compareAndSetState(0,1)) {
 				setExclusiveOwnerThread(Thread.currentThread());
 			}else {
 				acquire(1);
 			}
 		}  	
公平锁通过hasQueuedPredecessors判断该线程是否位于CLH队列中头部，是则获取锁；而非公平锁则不管你在哪个位置都直接获取锁