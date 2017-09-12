以下所有论述基于JDK1.7
===

HashMap是非现场安全的。而HashMap的线程
不安全体系在resize时的死循环及使用迭代
器时的fast-fail上

HashMap工作原理
===

数据结构
====

常用的底层数据结构主要有数组的链表。数
组存储区间连续，占用内存较多，寻址容易
插入和删除困难。链表存储区间离散，占用
内存较少，寻址困难，插入和删除容易。

HashMap要实现的是哈希表的效果，尽量实现
O(1)级别的增删改查。具体实现则是同时
使用了数组和链表，可以认为最外层是一个
数组，数组的每个元素是一个链表的表头


HashMap寻址方式
====

对于新插入的数据或者待读取的数据，HashM
ap将key的哈希值对数组长度取模，结果作为
该entry在数组中的index。在计算机中，取
模的代价远高于位操作的代价，因此HashMap
要求数组的长度必须是2的N次方。此时将key
的哈希值对2^N-1进行与运算，其效果即与
取模等效。HashMap并不要求用户在指定Hash
Map容量时必须传入一个2的N次方的整数，而
是会通过Integer.highestOneBit算出比指定
整数小的最大的2^N值

public static int highestOneBit(int i){
	i |= ( i >> 1);
	i |= ( i >> 2);
	i |= ( i >> 4);
	i |= ( i >> 8);
	i |= ( i >> 16);
	return i - (i >>> 1);
}

由于key的哈希值的分部直接决定了所有的
数据在哈希表上的分部或者说决定了哈希冲
突的可能性，因此为防止糟糕的key的hashco
de实现，JDK1.7的HashMap通过如下方法使得
最终的哈希值的二进制形式中的1尽量均匀分
布从而尽可能减少哈希冲突

int h = hashSeed;
h ^= k.hashCode();
h ^= (h >>> 20) ^ (h >>> 12);
return h ^ (h >>> 7) ^ (h >>> 4);


resize死循环
====

transfer方法
=====

当HashMap的size超过Capacity*loadFactor
时，需要对HashMap进行扩容。具体方法时，
创建一个新的长度为原来capacity两倍的
数组，保证新的capacity仍为2的N次方，从
而保证上述寻址方式仍适用。同时需要通过
如下transfer方法将原来的所有数据全部重
新插入rehash到新的数组中

void transfer(Entryp[] newTable, 
boolean rehash) {
	int newcapacity = newTable.length;
	for(Entry<K,V> e :  table) {
		while(null != e) {
			Entry<K,V> next = e.next;
			if(rehash){
				e.hash = null == e.key
				? 0 : hash(e.key);
			}
			int i = indexFor(e.hash,newCapacity);
			e.next = newTable[i];
			newTable[i] = e;
			e = next;
		}
	}	
}

该方法并不保证线程安全，而且在多线程
并发调用时，可能出现死循环。其执行过程
如下。从步骤2可见，转移时链表顺序反转

1、遍历原数组中的元素
2、对链表上的每个节点遍历；用next取得
要转移那个元素的下一个，将e转移到新
数组的头部，使用头插法插入节点
3、循环2，直到链表节点全部转移
4、循环1，直到所有元素全部转移

在单线程的rehash没有问题，多线程的并发
下就会出现问题

fast-fail
====

在使用迭代器的过程中，如果HashMap被修
改，那么ConcurrentModificationExceptio
n将被抛出。

当HashMap的iterator方法被调用时，会构
造并返回一个新的EntryIterator对象，并
将EntryIteator的expectedModCount设置
为HashMap的modCount

HashIterator() {
  expectedModCount = modCount;
  if (size > 0) { // advance to first entry
  Entry[] t = table;
  while (index < t.length && (next = t[index++]) == null)
    ;
  }
}

在通过该iterator的next方法访问下一个
Entry时，它会先检查自己的expectedMod
Count与HashMap的modCount是否相等，如果
不等，说明HashMap被修改，抛出移除


ConcurrentHashMap
===

ConcurrentHashMap的底层数据结构仍然是
数组和链表。与HashMap不同的是，Concurr
entHashMap最外层不是一个大的数组，而是
一个Segment的数组。每个Segment包含一个
与HashMap数据结构差不多的链表的链表
数组

寻址方式
====

在读写某个key时，先取该key的哈希值。
并将哈希值的高N位对Segment个数取模从
而得到该key应该属于哪个Segment，接着
如同操作HashMap一样操作这个Segment。
为了保证不同的值均匀分布到不同的segmen
t需要通过如下方法计算哈希值
private int hash(Object k) {
	int h = hashSeed;
	((0 != h) && (k instanceof String)) {
    return sun.misc.Hashing.stringHash32((String) k);
  }
  h ^= k.hashCode();
  h += (h <<  15) ^ 0xffffcd7d;
  h ^= (h >>> 10);
  h += (h <<   3);
  h ^= (h >>>  6);
  h += (h <<   2) + (h << 14);
  return h ^ (h >>> 16);
}

同样为了提高取模运算效率，通过如下计算
，ssize即为大于concurrencyLevel的最小
的2的N次方，同时segmentMask为2^N-1。这
一点跟上文中计算数组长度的方法一致。对
于某一个Key的哈希值，只需要向右移segme
ntShift位以取高sshift位，再与segmentMa
sk取与操作即可得到它在Segment数组上的
索引。

int sshift = 0;
int ssize = 1;
while (ssize < concurrencyLevel) {
  ++sshift;
  ssize <<= 1;
}
this.segmentShift = 32 - sshift;
this.segmentMask = ssize - 1;
Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];


同步方式
====

Segment继承自ReentrantLock，所以我们可
以很方便的对每一个Segment上锁

对于读操作，获取key所在的segment时，需
要保证可见性。具体实现上可以使用volati
le关键字，也可使用锁。但使用锁开销太
大，使用volatile时每次写操作都会让所有
CPU内缓存无效，也有一定开销，Concurren
tHashMap使用如下方法保证可见性，取得
最新的segment

Segment<K,V> s = (Segment<K,V>)
UNSAFE.getObjectVolatile(segments, u)

对于写操作，并不要求同时获取所有Segment的锁，因为那样相当于锁住了整个Map。它会先获取该Key-Value对所在的Segment的锁，获取成功后就可以像操作一个普通的HashMap一样操作该Segment，并保证该Segment的安全性。
同时由于其它Segment的锁并未被获取，因此理论上可支持concurrencyLevel（等于Segment的个数）个线程安全的并发读写。

获取锁时，并不直接使用lock来获取，因为该方法获取锁失败时会挂起（参考可重入锁）。事实上，它使用了自旋锁，如果tryLock获取锁失败，说明锁被其它线程占用，此时通过循环再次以tryLock的方式申请锁。如果在循环过程中该Key所对应的链表头被修改，则重置retry次数。如果retry次数超过一定值，则使用lock方法申请锁。

这里使用自旋锁是因为自旋锁的效率比较高，但是它消耗CPU资源比较多，因此在自旋次数超过阈值时切换为互斥锁


size操作
====

put remove和get操作只关心一个segment
而size操作需要遍历所有的segment。 计算
完后再解锁。但这样做的话，不利于map的
并行操作

为更好支持并发操作，ConcurrentHashMap会在不上锁的前提逐个Segment计算3次size，如果某相邻两次计算获取的所有Segment的更新次数（每个Segment都与HashMap一样通过modCount跟踪自己的修改次数，Segment每修改一次其modCount加一）相等，说明这两次计算过程中无更新操作，则这两次计算出的总size相等，可直接作为最终结果返回。如果这三次计算过程中Map有更新，则对所有Segment加锁重新计算Size，该计算方法代码如下

public int size() {
  final Segment<K,V>[] segments = this.segments;
  int size;
  boolean overflow; // true if size overflows 32 bits
  long sum;         // sum of modCounts
  long last = 0L;   // previous sum
  int retries = -1; // first iteration isn't retry
  try {
    for (;;) {
      if (retries++ == RETRIES_BEFORE_LOCK) {
        for (int j = 0; j < segments.length; ++j)
          ensureSegment(j).lock(); // force creation
      }
      sum = 0L;
      size = 0;
      overflow = false;
      for (int j = 0; j < segments.length; ++j) {
        Segment<K,V> seg = segmentAt(segments, j);
        if (seg != null) {
          sum += seg.modCount;
          int c = seg.count;
          if (c < 0 || (size += c) < 0)
            overflow = true;
        }
      }
      if (sum == last)
        break;
      last = sum;
    }
  } finally {
    if (retries > RETRIES_BEFORE_LOCK) {
      for (int j = 0; j < segments.length; ++j)
        segmentAt(segments, j).unlock();
    }
  }
  return overflow ? Integer.MAX_VALUE : size;
}


java8 基于CAS的ConcurrentHashMap
===

数据结构
====

Java 7为实现并行访问，引入了Segment这一结构，实现了分段锁，理论上最大并发度与Segment个数相等。Java 8为进一步提高并发性，摒弃了分段锁的方案，而是直接使用一个大的数组。同时为了提高哈希碰撞下的寻址性能，Java 8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(long(N))）。其数据结构如下图所示


寻址方式
====

Java 8的ConcurrentHashMap同样是通过Key的哈希值与数组长度取模确定该Key在数组中的索引。同样为了避免不太好的Key的hashCode设计，它通过如下方法计算得到Key的最终哈希值。不同的是，Java 8的ConcurrentHashMap作者认为引入红黑树后，即使哈希冲突比较严重，寻址效率也足够高，所以作者并未在哈希值的计算上做过多设计，只是将Key的hashCode值与其高16位作异或并保证最高位为0（从而保证最终结果为正整数）。

static final int spread(int h) {
  return (h ^ (h >>> 16)) & HASH_BITS;
}


同步方式
====

对于put操作，如果Key对应的数组元素为null，则通过CAS操作将其设置为当前值。如果Key对应的数组元素（也即链表表头或者树的根元素）不为null，则对该元素使用synchronized关键字申请锁，然后进行操作。如果该put操作使得当前链表长度超过一定阈值，则将该链表转换为树，从而提高寻址效率。

对于读操作，由于数组被volatile关键字修饰，因此不用担心数组的可见性问题。同时每个元素是一个Node实例（Java 7中每个元素是一个HashEntry），它的Key值和hash值都由final修饰，不可变更，无须关心它们被修改后的可见性问题。而其Value及对下一个元素的引用由volatile修饰，可见性也有保障。


static class Node<K,V> implements Map.Entry<K,V> {
  final int hash;
  final K key;
  volatile V val;
  volatile Node<K,V> next;
}

对于Key对应的数组元素的可见性，由Unsafe的getObjectVolatile方法保证。


static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
  return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

size操作
====

put方法和remove方法都会通过addCount方法维护Map的size。size方法通过sumCount获取由addCount方法维护的Map的size。