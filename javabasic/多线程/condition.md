Condition
----

Condition将Object监视器方法(wait nofity notifyAll)分解成不同的对象，以便通过将这些对象与任意lock实现组合使用，为每个对象提供了多个等待set(wait-set)
其中，lock替代了synchronized方法和语句的使用，Condition提的了Object监视器方法的使用。

条件为线程提供了一个含义，以便在某个状态条件可能为true的另一个线程通知它之前，一直挂起该线程(即让其等待)。因为访问此工序状态信息发生在不同的线程中，所以必须受保护，因此要将某种形式的锁与该条件相关联。等待提供一个条件的主要属性是：以原子方式释放相关的锁，并挂起当前线程，就像object.wait。Condition实质上被绑定到一个锁上。

生产者-消费者模型

		//生产者
		public class Product {
			private Repertory repertory;

			public Product(Repertory _repertory) {
				this.repertory = _repertory;
			}
			//生产商品
			public void product(final int num) {
				new Thread() {
					@Override
					public void run() {
						repertory.put(num);
					}
				}.start;
			}
		}
		=====
		//消费者
		public class Consumer {
			private Repertory repertory;

			public Consumer(Repertory _repertory) {
				this.repertory = _repertory;
			}

			public void consumer(final int num) {
				new Thread() {
					@Override
					public void run() {
						repertory.get(num);
					}
				}.start();
			}
		}
		=====
		//仓库类
		public class Repertory {
			private int totalSize;//仓库目前剩余商品数量
			private Lock lock;
			private int capaity; //仓库容量

			private Condition fullCondition;
			private Condition emptyCondition;

			public Repertory() {
				this.lock = new ReentrantLock();
				this.fullCondition = lock.newCondition();
				this.emptyCondition = lock.newCondition();
				this.capaity = 15;
				this.totalSize = 0;
			}

			//商品入库
			public void put(int num) {
				lock.lock();
				try{
					int left = value;
					while(left > 0) {
						while(totalSize >= capaity) {
							fullCondition.await();
						}

						int inc = totalSize + left > capaity ? capaity - totalSize : left;
						totalSize += inc;
						left -= inc;
						System.out.println(Thread.currentThread().getName() + "----要入库数量: " + value +";;实际入库数量：" + inc + ";;仓库货物数量：" + depotSize + ";没有入库数量：" + left);
            			//通知消费者消费
            			emptyCondition.signal();
					}
					}finally{
					lock.unlock();
				}
			}

			//商品出库
			public void get(int num) {
				lock.lock;
				try{
					int left = num;
					while(left > 0) {
						while(totalSize <= 0) {
							emptyCondition.await();
						}
						int dec = totalSize < left ? totalSize : left;
						totalSize -= dec;
						left -= dec;
						System.out.println(Thread.currentThread().getName() + "----要消费的数量：" + value +";;实际消费的数量: " + dec + ";;仓库现存数量：" + depotSize + ";;有多少件商品没有消费：" + left);
            			//通知生产者生产
            			fullCondition.signal();
					}
					}finally{
					lock.unlock;
				}
			}
		}
		====
		public class Test {
		    public static void main(String[] args) {
		        Depot depot = new Depot();
		        
		        Producer producer = new Producer(depot);
		        Customer customer = new Customer(depot);
		        
		        producer.produce(10);
		        customer.consume(5);
		        producer.produce(15);
		        customer.consume(10);
		        customer.consume(15);
		        producer.produce(10);
		    }
		}

