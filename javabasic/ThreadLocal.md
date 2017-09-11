ThreadLocal
===

其实它是一个容器，用于存放线程的局部
变量。 它是为了解决多线程并发问题而设计
的。

例子
====

一个序列化生成器的程序，可能同时会有多
个线程并发访问它，要保证每个线程得到的
序列化都是自增的，而不能相互干扰

public interface Sequence {
	int getNumber();
}

public class SequenceImpl implements
Sequence {
	private static ThreadLocal<Integer>
	numberContainer = new ThreadLocal<I
	nteget>() {
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};

	public int getNumber(){
		numberContainer.set(numberContainer.get() + 1);
		return numberContainer.get();
	}
}

public class ClientThread extend Thread{
	private Sequence sequence;

	public ClientThread(Sequence sequence) {
		this.sequence = sequence;
	}

	@Override
	public void run() {
		for(int i = 0; i < 3; i++) {
			Thread.currentThread().getName() + "===" + sequence.getNumber();
		}
	}
}

每个线程相互独立，同样是static变量，对
于不同的线程而言，没有被共享，而是每个
线程各一份，这样也就保证了线程安全问题


ThreadLocal API
====

将值放入线程局部变量中
public void set(T value);

从线程局部变量中过去值
public T get()

从线程局部变量中移除值
public void remove()

返回线程局部变量中的初始值(自行实现)
protected T initialValue()