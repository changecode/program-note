ReentrantReadWriteLock
----

ReadWriteLock:维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有writer，读取锁可以由多个reader线程同时保持。写入锁是独占的。对于ReadWriteLock而言，一个资源能够被多个读线程访问，或者被一个写线程访问，但是不能同时存在读写线程。
		
		public interface ReadWriteLock {
			Lock readLock();
			Lock writeLock();
		}

ReentrantReadWriteLock作为ReadWriteLock的实现类，特性如下:

1、公平性
	
	1. 非公平锁(默认)和独占锁的非公平性一样，由于读线程之间没有锁竞争，所以读操作没有公平性和非公平性之分，写操作时，由于写操作可能理解获取到锁，所以会推迟一个或多个读操作或者写操作。

	2. 公平说利用AQS的CLH队列，释放当前保存的锁时，优先为等待时间最长的那个写线程分配写入锁，当前前提是写线程的等待时间要比所有读线程的等待时间要长。同样一个线程持有写入锁或者有一个写线程已经在等待了，那么试图获取公平锁的(非重入)所有线程都将被阻塞，直到最先的写线程释放锁。如果读线程的等待时间比写线程的等待时间还要长，那么一旦上一个写线程释放锁，这一个线程将获取锁

2、重入性

	1. 读写锁运行读线程和谐线程按照请求锁的顺序重新获取读取锁或者写入锁

	2. 写线程获取写入锁后可以再次获取读取锁，但是读线程获取读取锁却不能获取写入锁

	3. 读写锁最多支持65535个递归写入锁和65535个递归读取锁	

3、锁降级

	1. 写线程获取写入锁后可以获取读取锁，然后师傅写入锁，写入锁就变成了读取锁，从而实现锁的降级

4、锁升级

	1. 读取锁是不能直接升级为写入锁的。因为获取一个写入锁需要释放所有的读取锁，如果有二个读取锁试图获取写入锁而都不释放读取锁时会发生死锁

5、锁获取中断
	1. 读取锁和写入锁都支持获取锁期间被中断

6、条件变量

	1. 写入锁提供了条件变量的支持，这个和独占锁一致，但是读取锁却不运行获取条件变量

7、重入数
	1. 读取锁和写入锁的数量最大分部只能是65535			  		


WriteLock-lock
----

		public void lock() {
        	sync.acquire(1);
    	}
    	====
    	public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    	}
    	====
    	protected final boolean tryAcquire(int acquires) {
	        //当前线程
	        Thread current = Thread.currentThread();
	        //当前锁个数
	        int c = getState();
	        //写锁个数
	        int w = exclusiveCount(c);
	        
	        //当前锁个数 != 0（是否已经有线程持有锁），线程重入
	        if (c != 0) {
	            //w == 0,表示写线程数为0
	            //或者独占锁不是当前线程，返回false
	            if (w == 0 || current != getExclusiveOwnerThread())
	                return false;
	            
	            //超出最大范围（65535）
	            if (w + exclusiveCount(acquires) > MAX_COUNT)
	                throw new Error("Maximum lock count exceeded");
	            //设置锁的线程数量
	            setState(c + acquires);
	            return true;
	        }
	        
	        //是否阻塞
	        if (writerShouldBlock() ||
	            !compareAndSetState(c, c + acquires))
	            return false;
	        
	        //设置锁为当前线程所有
	        setExclusiveOwnerThread(current);
	        return true;
    	}
独占锁ReentrantLock中有个state，共享锁也有一个state，其中独占锁中的state为0或者1，如果有重入，则表示重入的次数，共享锁中表示的持有锁的数量。而ReadWriteLock中则不同，由于其存在二个锁，之间还有联系，所以需要二个state分别表示，于是将state一分为二，高16位表示共享锁的数量，低16位表示独占锁的数量2^16-1=65535

WriteLock-unlock
----

		public void unlock() {
            sync.release(1);
        }
        ====
        public final boolean release(int arg) {
	        if (tryRelease(arg)) {
	            Node h = head;
	            if (h != null && h.waitStatus != 0)
	                unparkSuccessor(h);
	            return true;
	        }
	        return false;
	    }
	    ====
	    protected final boolean tryRelease(int releases) {
	        //若锁的持有者不是当前线程，抛出异常
	        if (!isHeldExclusively())
	            throw new IllegalMonitorStateException();
	        //写锁的新线程数
	        int nextc = getState() - releases;
	        //若写锁的新线程数为0，则将锁的持有者设置为null
	        boolean free = exclusiveCount(nextc) == 0;
	        if (free)
	            setExclusiveOwnerThread(null);
	        //设置写锁的新线程数
	        setState(nextc);
	        return free;
	    }

写锁的释放过程还是相对而言比较简单的：首先查看当前线程是否为写锁的持有者，如果不是抛出异常。然后检查释放后写锁的线程数是否为0，如果为0则表示写锁空闲了，释放锁资源将锁的持有线程设置为null，否则释放仅仅只是一次重入锁而已，并不能将写锁的线程清空

ReadLock-lock
----

		public void lock() {
            sync.acquireShared(1);
        }
        ====
        public final void acquireShared(int arg) {
	        if (tryAcquireShared(arg) < 0)
	            doAcquireShared(arg);
	    }
	    ====
	    protected final int tryAcquireShared(int unused) {
	        Thread current = Thread.currentThread();
	        //锁的持有线程数
	        int c = getState();
	        /*
	         * 如果写锁线程数 != 0 ，且独占锁不是当前线程则返回失败，因为存在锁降级
	         */
	        if (exclusiveCount(c) != 0 &&
	            getExclusiveOwnerThread() != current)
	            return -1;
	        //读锁线程数
	        int r = sharedCount(c);
	        /*
	         * readerShouldBlock():读锁是否需要等待（公平锁原则）
	         * r < MAX_COUNT：持有线程小于最大数（65535）
	         * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
	         */
	        if (!readerShouldBlock() && r < MAX_COUNT &&
	            compareAndSetState(c, c + SHARED_UNIT)) {
	            /*
	             * holdCount部分后面讲解
	             */
	            if (r == 0) {
	                firstReader = current;
	                firstReaderHoldCount = 1;
	            } else if (firstReader == current) {
	                firstReaderHoldCount++;        //
	            } else {
	                HoldCounter rh = cachedHoldCounter;
	                if (rh == null || rh.tid != current.getId())
	                    cachedHoldCounter = rh = readHolds.get();
	                else if (rh.count == 0)
	                    readHolds.set(rh);
	                rh.count++;
	            }
	            return 1;
	        }
	        return fullTryAcquireShared(current);
	    }

1、写锁的线程数C！=0且写锁的线程持有者不是当前线程，返回-1。因为存在锁降级，写线程获取写入锁后可以获取读取锁。

2、依据公平性原则，判断读锁是否需要阻塞，读锁持有线程数小于最大值（65535），且设置锁状态成功，执行以下代码（对于HoldCounter下面再阐述），并返回1。如果不满足改条件，执行fullTryAcquireShared()：（HoldCounter部分后面讲解）	  

		final int fullTryAcquireShared(Thread current) {
         	HoldCounter rh = null;
         	for (;;) {
             	//锁的线程持有数
             	int c = getState();
             	//如果写锁的线程持有数 != 0 且锁的持有者不是当前线程，返回-1
             	if (exclusiveCount(c) != 0) {
                 	if (getExclusiveOwnerThread() != current)
                     	return -1;
             	} 
             	//若读锁需要阻塞
             	else if (readerShouldBlock()) {
                 	//若队列的头部是当前线程
                 	if (firstReader == current) {
                 	} 
                 	else {   //下面讲解
                     	if (rh == null) {
                         	rh = cachedHoldCounter;
                         	if (rh == null || rh.tid != current.getId()) {
                             	rh = readHolds.get();
                             	if (rh.count == 0)
                                 	readHolds.remove();
                         	}
                     	}
                     	if (rh.count == 0)
                         	return -1;
                 	}
             	}
             	//读锁的线程数到达最大值：65536，抛出异常
             	if (sharedCount(c) == MAX_COUNT)
                 	throw new Error("Maximum lock count exceeded");
             	//设置锁的状态成功
             	if (compareAndSetState(c, c + SHARED_UNIT)) {
                 	if (sharedCount(c) == 0) {
                     	firstReader = current;
                     	firstReaderHoldCount = 1;
                 	}
                 	else if (firstReader == current) {
                     	firstReaderHoldCount++;
                 	}
                 	else {
                     	if (rh == null)
                         rh = cachedHoldCounter;
                     	if (rh == null || rh.tid != current.getId())
                         	rh = readHolds.get();
                     	else if (rh.count == 0)
                         	readHolds.set(rh);
                     	rh.count++;
                     	cachedHoldCounter = rh;
                 	}
                 	return 1;
             	}
         	}
     	}  


ReadLock-unlock
----

		public  void unlock() {
            sync.releaseShared(1);
        }
        ====
        public final boolean releaseShared(int arg) {
	        if (tryReleaseShared(arg)) {
	            doReleaseShared();
	            return true;
	        }
	        return false;
	    }
	    ====
	    protected final boolean tryReleaseShared(int unused) {
	        //当前线程
	        Thread current = Thread.currentThread();
	        /*
	         * HoldCounter部分后面阐述
	         */
	        if (firstReader == current) {
	            if (firstReaderHoldCount == 1)
	                firstReader = null;
	            else
	                firstReaderHoldCount--;
	        } else {
	            HoldCounter rh = cachedHoldCounter;
	            if (rh == null || rh.tid != current.getId())
	                rh = readHolds.get();
	            int count = rh.count;
	            if (count <= 1) {
	                readHolds.remove();
	                if (count <= 0)
	                    throw unmatchedUnlockException();
	            }
	            --rh.count;
	        }
	        //不断循环，不断尝试CAS操作
	        for (;;) {
	            int c = getState();
	            int nextc = c - SHARED_UNIT;
	            if (compareAndSetState(c, nextc))
	                return nextc == 0;
	        }
	    }

HoldCounter
----

对于共享锁其实我们可以稍微的认为它不是一个锁的概念，它更加像一个计数器的概念。一次共享锁操作就相当于一次计数器的操作，获取共享锁计数器+1，释放共享锁计数器-1。只有当线程获取共享锁后才能对共享锁进行释放、重入操作。所以HoldCounter的作用就是当前线程持有共享锁的数量，这个数量必须要与线程绑定在一起，否则操作其他线程锁就会抛出异常
		
		if (r == 0) {        //r == 0，表示第一个读锁线程，第一个读锁firstRead是不会加入到readHolds中
	        firstReader = current;
	        firstReaderHoldCount = 1;
	    } else if (firstReader == current) {    //第一个读锁线程重入
	        firstReaderHoldCount++;    
	    } else {    //非firstReader计数
	        HoldCounter rh = cachedHoldCounter;        //readHoldCounter缓存
	        //rh == null 或者 rh.tid != current.getId()，需要获取rh
	        if (rh == null || rh.tid != current.getId())    
	            cachedHoldCounter = rh = readHolds.get();
	        else if (rh.count == 0)
	            readHolds.set(rh);        //加入到readHolds中
	        rh.count++;        //计数+1
	    }	 

HoldCounter应该就是绑定线程上的一个计数器，而ThradLocalHoldCounter则是线程绑定的ThreadLocal。从上面我们可以看到ThreadLocal将HoldCounter绑定到当前线程上，同时HoldCounter也持有线程Id，这样在释放锁的时候才能知道ReadWriteLock里面缓存的上一个读取线程（cachedHoldCounter）是否是当前线程。这样做的好处是可以减少ThreadLocal.get()的次数，因为这也是一个耗时操作。需要说明的是这样HoldCounter绑定线程id而不绑定线程对象的原因是避免HoldCounter和ThreadLocal互相绑定而GC难以释放它们（尽管GC能够智能的发现这种引用而回收它们，但是这需要一定的代价），所以其实这样做只是为了帮助GC快速回收对象而已	       	