MySQL线程池
----

在么MySQL中，线程池指的是用来管理处理MySQL客户端连接任务的线程的一种机制。 如果把线程看做系统资源那么线程池本质上是对系统资源的管理，对应操作系统来说线程的创建和销毁是比较消耗系统资源的，频繁的创建与销毁线程必然给系统带来不必要的资源浪费，特别是在高负载的情况下。线程池技术一方面可以减少线程重复创建与销毁这部分开销，从而
更好地利用已经创建的线程资源，另一方面也可以控制线程的创建与系统的负载


核心参数
----

* thread_pool_size:指的是线程组大小，默认是CPU核心数，线程池初始化的时候会根据这个数字来生成线程组，每个线程组初始化一个poolfd句柄
* thread_pool_stall_limit:Timer Thread 迭代的时间间隔，默认是500ms
* thread_pool_oversubscribe:用于计算线程组是否过于活跃或者太过繁忙，即系统的负载程度，用于在一定场景决策新的工作线程是否被创建于和任务是否被除了，默认值为3
* thread_pool_idle_timeout:工作线程最大空闲时间，工作线程超过这个数还空闲的话就退出，这个值默认是60秒
* thread_pool_high_prio_mode: 这个参数可用于控制任务队列的使用，可取三个值： transactions\statements\none 当为statements的时候则线程组只使用优先队列，当值为none的时候，则只使用普通队列，当值为transactions的时候配合thread_pool_high_prio_tickets参数生效，用于控制任务被放入优先队列的最大次数。


MySQL线程池的大致架构
----

线程组
-----

MySQL线程池在初始化的时候根据宿主机的CPU核心数设置thread_pool_size，也就是线程池的线程组个数。每个线程组在初始化之后会通过底层的IO库分配一个网络特殊的句柄与之关联，IO库可以通过这个句柄监听与之绑定的socket句柄就绪的IO任务，线程组的结构体定义如下：
		struct thread_group_t {
			mysql_mutex_t mutex;
			connection_queue_t queue;
			connection_queue_t high_prio_queue;
			worker_list_t waiting_threads;
			worker_thread_t *listener;
			pthread_attr_t *pthread_attr;

			int pollfd;
			int thread_count;
			int active_thread_count;
			int connection_count;
			int waiting_thread_count;
			int io_event_count;
			int queue_event_count;
			ulonglong last_thread_creation_time;
			int shutdown_pipe[2];
			bool shutdown;
			bool stalled;
		} MY_ALIGNED(512);

线程池由多个线程组构成，线程池的细节基本都在线程组内


worker线程
-----

线程组内有0个或多个线程，和netty有些不同，netty中有固定的线程用于轮询IO事件，工作线程只负责处理IO任务，而在MySQL线程池中listener只是一种角色，每个线程的角色可以是listener或者是worker，工作线程为listener的时候负责从poolfd中读取就绪的IO任务，处于worker角色的时候负责处理这些IO任务，我们需要区分工作线程的以下几种状态：
	1、活跃状态：当工作线程处于正在处理任务且状态未被阻塞的状态。如果worker线程将自己设置为listener则不算进线程组的活跃线程状态
	2、空闲状态：没有任务处理而被处于空闲状态
	3、等待状态：如果工作线程在执行命令的过程中由于IO、锁、条件、sleep等需要等待则线程池将被通知将这些工作线程记作等待状态，

在正常工作的情况下，当工作线程检索任务的时候，如果线程组太活跃即使有任务工作线程也不会执行，如果不是太繁忙才会考虑高优先级的任务，对于低优先级的任务只有当线程组不是太繁忙的时候才会执行。


Check Stall机制
-----

如果后来的IO任务被前面的执行时间过长的任务影响会怎么样？这必然会导致一些无辜的任务(或是一个简单的insert操作)被影响，结果是有可能回被延迟处理。线程池中有一个timer thread类似我们很多系统里面的Timeout Thread线程，这个线程每隔一定时间间隔就会进行一次迭代，迭代中做的如下二部分事情
	1、检查线程组的负载情况进行工作线程的唤醒与创建
	2、检查与处理超时的客户端连接

这里主要介绍第一部分工作也就是check stall机制。timer thread周期性检查线程组内的线程是否被阻塞，所谓阻塞也就是新来的任务但是线程组内没有线程来处理。timer thread通过queue_event_count和IO任务队列释放为空来判断线程组释放为阻塞状态，每次工作线程检索任务的时候queue_event_count都会累加，累加意味着任务呗正常处理，工作线程正常工作，在每一次check_stall之后queue_event_count会被清零，因此如果在一定时间间隔后的下一次迭代中，IO任务对垒不为空并且queue_event_count为空，则说明这段时间间隔内都没有工作线程来处理IO任务，那么check stall机制会尝试唤醒或者创建一个工作线程，唤醒线程的逻辑很简单，如果waiting_threads中有空闲线程则唤醒一个空闲线程，否则需要尝试创建一个工作线程，创建线程不一定会创建成果，创建的条件如下
	1、如果没有空闲线程并且没有货源线程则立马创建，这个时候可能是因为没有任务工作或者工作线程都被阻塞了，或者存在潜在的死锁
	2、否则如果距离上次创建的时间大于一定阈值才创建线程，这个阈值由线程组内的线程决定


任务队列
-----

任务队列也就是listener每次从poolfd轮询出来的就绪任务，分为优先队列和普通任务队列，优先队列中的IO任务会先被除了，然后普通队列中的任务才能被处理。满足以下二个条件为优先执行
	1、连接处于事务中
	2、连接关联的priority tickets值大于0
参数priority tickets的设计是为了防止高优先级的任务总是被处理，而一些非高优先级的任务处于较长时间的饥饿状态，当高优先级的任务每一次被放入高优先级队列之后都会对priority tickets的值进行减1，因此达到一定次数priority tickets的值必然会小于等于0，因此避免了永久高优先级的问题。另外队列的使用受参数thread_pool_high_prio_mode影响。当就绪IO任务呗轮询出来放入队列之后会对io_event_count进行累加，当IO任务从队列取出处理的时候会对queue_event_count进行计数

listener线程
-----

listener做的事情主要是从poolfd中轮询与其绑定的socket句柄的就绪IO事件，事件以任务的形式被放入任务队列并做相应处理.

如果任务队列不为空，也不一定会有线程来及时处理任务，这就导致了耗时任务影响了后来任务的执行，未来可能通过摒弃每个线程只保持一个活跃线程的规则来避免网络任务长时间得不到处理

