redis-order-strike
===

摘录于https://tech.imdada.cn/2017/06/30/daojia-redis/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io


订单列表在Redis中的存储结构
====

# 订单列表数据在缓存中，是以用户的唯一标识作为键，以按下单时间倒序的有序集合
进行存储的。 Redis的sorted set中每个元素都有一个分数，Redis就是根据这个分数
排序的。订单有序集合中的每个元素是将时间毫秒数+订单号最后三位作为分数进行排序
的。至于不单只用毫秒数作为分数，因为下单时间只精确到秒，如果不加订单号最后3位
若同一秒有两个或两个以上订单时，排序分数就会一样，从而导致根据分数从缓存查询
时不能保证唯一性。而订单号的生成规则可以保证同一秒内的订单号最后三位是不一样的

# 没有必要将一个用户的所有订单都放入缓存中，因为很少用户去看很久以前的历史订单
真正的热点数据其实也就是最近下过的一些订单。所以为了节省内存空间，只需要存放一
个用户最近下过的N条订单就可以了，这个N，相当于一个阈值，超过了这个阈值，再从
数据库中查询订单数据


redis和db数据一致性
====

订单数据先更新数据库，数据库更新成功后，再更新缓存，若数据库操作成功，缓存
操作失败了，就会出现数据不一致的情况，两种方案记录。

# 方式一
  1、 循环五次更新缓存操作，直到更新成功推出循环，主要能减少由于网络瞬间抖动
  导致的更新缓存失败的概率，对于缓存借款长时间不可用，靠循环调用更新接口是不
  能补救接口调用失败的

  2、 如果循环五次还没有更新成功，就通过worker去定时扫描数据库的数据，去和缓存
  中的数据进行比较，对缓存中的状态不正确的数据进行纠正

# 方式二
  1、 和方式一的第一步一样
  2、 若循环更新五次仍不成功，则发一个缓存更新失败的mq，通过消费mq去更新缓存
  会比通过定时任务扫描更及时，也不会有扫库的耗时操作

  	for(int i = 0; i < 5; i++) {
  		try{
  			//加入缓存
  			addOrderlistRedis(key, score, orderlist);
  		}catch(Exception e) {
  			//如果五次还失败了，发送到mq处理
  			if(4 == i) sendOrderCacheMQ(orderlist, logSid);
  		}
  	}


Redis分布式锁
====

Redis分布式锁在2.6.12版本之后的实现方式比较简单，只需要使用一个命令
SET key value [EX seconds] [NX]
其中，可选参数EX seconds 设置键的过期时间为seconds秒；NX只在键不存在时，才对
键进行设置操作。这个命令相当于2.6.12之前的setNx和expire二个命令的原子操作命令

		public boolean getLock(String lockKey, String lockvalue) {
			if(shardedXCommands.set(key, lockValue, 10, TimeUnit.SECOND, false)) {
				return true;
			}
			return false;
		}
2.6.12版本之前不能简单的使用setNx expire两个命令，因为前一个成功，后一个失败的
话，恰好执行删除lockKey的也执行失败，key就永远不会过期，就会出现死锁问题。改进
版的代码如下

		public booelan getLock(String lockKey) {
		    boolean lock = false;
		    while (!lock) {
		        String expireTime = String.valueOf(System.currentTimeMillis() + 5000);
		        // (1)第一个获得锁的线程，将lockKey的值设置为当前时间+5000毫秒，后面会判断，如果5秒之后，获得锁的线程还没有执行完，会忽略之前获得锁的线程，而直接获取锁，所以这个时间需要根据自己业务的执行时间来设置长短。
		        lock = shardedXCommands.setNX(lockKey, expireTime);
		        if (lock) { // 已经获取了这个锁 直接返回已经获得锁的标识
		            return lock;
		        }
		         // 没获得锁的线程可以执行到这里：从Redis获取老的时间戳
		        String oldTimeStr = shardedXCommands.get(lockKey);
		        if (oldTimeStr != null && !"".equals(oldTimeStr.trim())) {
		            Long oldTimeLong = Long.valueOf(oldTimeStr);
		            // 当前的时间戳
		            Long currentTimeLong = System.currentTimeMillis();
		            // (2)如果oldTimeLong小于当前时间了，说明之前持有锁的线程执行时间大于5秒了，就强制忽略该线程所持有的锁，重新设置自己的锁
		            if (oldTimeLong < currentTimeLong) { 
		                // (3)调用getset方法获取之前的时间戳,注意这里会出现多个线程竞争，但肯定只会有一个线程会拿到第一次获取到锁时设置的expireTime
		                String oldTimeStr2 = shardedXCommands.getSet(lockKey, String.valueOf(System.currentTimeMillis() + 5000)); 
		                // (4)如果刚获取的时间戳和之前获取的时间戳一样的话,说明没有其他线程在占用这个锁,则此线程可以获取这个锁.
		                if (oldTimeStr2 != null && oldTimeStr.equals(oldTimeStr2)) { 
		                    lock = true; // 获取锁标记
		                    break;
		                }
		            }
		        }
		        // 暂停50ms,重新循环
		        try {
		            Thread.sleep(50);
		        } catch (InterruptedException e) {
		            log.error(e);
		        }
		    }
		    return lock;
		}


缓存防穿透
====

		// 锁的数量 锁的数量越少 每个用户对锁的竞争就越激烈，直接打到数据库的流量就越少，对数据库的保护就越好，如果太小，又会影响系统吞吐量，可根据实际情况调整锁的个数
		public static final String[] LOCKS = new String[128];
		// 在静态块中将128个锁先初始化出来
		static {
		    for (int i = 0; i < 128; i++) {
		        LOCKS[i] = "lock_" + i;
		    }
		}

		// 代码段2
		public List<OrderVOList> getOrderVOList(String userId) {
		    List<OrderVOList> list = null;
		    // 1.先判断缓存中是否有这个用户的数据，有就直接从缓存中查询并返回
		    if (orderRedisCache.isOrderListExist(userId)) {
		        return  getOrderListFromCache(userId); 
		    }
		    // 2.缓存中没有，就先上锁，锁的粒度是根据用户Id的hashcode和127取模
		    String[] locks = OrderRedisKey.LOCKS;
		    int index = userId.hashCode() & (locks.length - 1);
		    try {
		        // 3.此处加锁很有必要，加锁会保证获取同一个用户数据的所有线程中，只有一个线程可以访问数据库，从而起到减小数据库压力的作用
		        orderRedisCache.lock(locks[index]);
		        // 4.上锁之后再判断缓存是否存在，为了防止再获得锁之前，已经有别的线程将数据加载到缓存，就不允许再查询数据库了。
		        if (orderRedisCache.isOrderListExist(userId)) {
		            return getOrderListFromCache(userId); 
		        }
		        // 查询数据库
		        list = getOrderListFromDb(userId);
		        // 如果数据库没有查询出来数据，则在缓存中放入NULL，标识这个用户真的没有数据，等有新订单入库时，会删掉此标识，并放入订单数据
		        if(list == null || list.size() == 0) {
		            jdCacheCloud.zAdd(OrderRedisKey.getListKey(userId), 0, null);
		        } else {
		            jdCacheCloud.zAdd(OrderRedisKey.getListKey(userId), list);
		        }
		        return list;
		    } finally {
		        orderRedisCache.unlock(locks[index]);
		    }
		}

防止穿透的关键地方在于使用分布式锁和锁的粒度控制。上述代码首先初始化了128个
锁，然后让所有缓存没命中的用户去竞争这128个锁，得到锁后并且再一次判断缓存中
依然没有数据的，才去查询数据库，没有将锁粒度限制到用户级别，因为如果粒度太小
某一个时间点有太多的用户去请求，同样会有很多的请求连到数据库。		