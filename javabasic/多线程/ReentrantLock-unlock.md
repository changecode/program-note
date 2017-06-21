ReentrantLock-unlock
----

unlock是不分公平锁和非公平锁

		public void unlock() {
			sync.release(1);
		}

		public final boolean release(int arg) {
			if(tryRelease(arg)) {
				Node h = head;
				if(h != null && h.waitStatus != 0) {
					unparkSuccessor(h);
				}
				return true;
			}
			return false;
		}
		=====
		//release(1) 尝试在当前锁的锁定计数state值上减1，成功返回true，否则为false。如果-1之后新的state=0，则表示当前锁已经被线程释放了，同时会唤醒线程等待队列中的下一个线程，当然该锁不一定就一定会把所有权交给下一个线程。
		protected final boolean tryRelease(int releases) {
			int c = getState() - release;
			if(Thread.currentThread() != getExclusiveOwnerThread()) {
				throw new IllegalMonitorStateException();
			} 
			boolean free = false;
			if(c == 0) {
				free = true;
				setExclusiveOwnerThread(null);
			}
			setState(c);
			return free;
		}
		======
		private void unparkSuccessor(Node node) {
			int ws = node.waitStatus;
			if(ws < 0) {
				compareAndSetWaitStatus(node, ws, 0);
			}
			Node s = node.next;
			if(s == null || s.waitStatus > 0) {
				s = null;
				for(Node t = tail; t != null && t != node; t = t.prev) {
					if(t.waitStatus <= 0) {
						s = t;
					}
				}
			}
			if(s != null) {
				LockSupport.unpark(s.thread);
			}
		}

unlock最好放在finally中
		