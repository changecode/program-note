java8新特性概述
----

函数式接口
----

所谓函数式接口就是只有一个抽象方法的接口，Java8加入了默认方法的特性，但是函数式接口是不关系接口中有没有默认方法的。一般函数式接口可以使用@FunctionInterface注解的形式来标注表示这是一个函数式接口，该注解标注与否对函数式接口没有实际的影响，不过一般还是推荐使用该注解
Predicate<T> Consumer<T> Function<T,R>  Supplier<T> UnaryOperator<T>
BinaryOperator<T> BiPreficate<L,R> BiConsumer<T,U> BiFunction<T,U,R>

Lambda表达式和方法引用
----

有了函数式接口，可以使用Lambda表达式。在函数式接口中Lambda表达式相当于匿名内部类的效果
		
		public class Lambda {
			public static void execute(Runnable runnable) {
				runnable.run();
			}
			public static main(String[] args) {
				//before java8
				execute(new Runnbale() {
					public void run() {
						System.out.println("run");
					}
				});
				//java8
				execute(() -> System.out.println("run"));
			}
		}
Lambda表达式还可以复合，把几个Lambda表达式串起来使用
Predicate<Apple> redAndHeavyApple = redApple.and(a -> a.getWeight() > 150).or(a -> "red".equals(a.getColor()));

方法引用是Lambda表达式更简洁形式，使用::操作符，方法引用主要有三类
1、指向静态方法的方法引用(Integer::parseInt)
2、指向任意类型实例方法的方法引用(String::length)
3、指向现有对象的实例方法的方法引用(localVariable::getValue
		Function<String, Integer> stringToInt = (String s) -> Integer.parseInt(s);
		Function<String, Integer> stringToInt = Integer::parseInt;
方法引用中还有一种特殊的形式，构造函数引用
		Supplier<SomeClass> c1 = SomeClass::new;
		SomeClass s1 = c1.get();
		====
		Supplier<SomeClass> c1 = () -> new SomeClass();
		SomeClass s1 = c1.get();
构造函数带参
		Supplier<SomeClass> c1 = SomeClass::new;
		SomeClass s1 = c1.apply(100);
		====
		Supplier<SomeClass> c1 = i -> new SomeClass(i);
		SomeClass s1 = c1.apply(100);

Stream
----

Stream可分为串行流和并行流。Stream提供了筛选、切片、映射、查找、匹配、归约等，这些操作又可以分为中间操作和中断操作，中间操作会返回一个流，因此可以使用多个中间操作来链式的调用，当使用了终端操作之后，那么这个流就被认为是消费了，每个流只能有一个终端操作
		
		List<Apple> vegetarianMenu = apples.stream().filter(App::isRed).collect(Collectors.toList());
		List<Integer> numbers = Arrays.asList<1,2,1,3,3,2,4);
		numbers.stream().filter(i -> i % 2 == 0).distinct().forEach(System.out::println)		

默认方法
----

默认方法出现的原因是为了对原有接口的扩展，有了默认方法之后改动原有的接口而对已经使用这些接口的程序造成的代码不兼容的影响。在Java8中也对一些接口增加了一些默认方法，比如Map接口等待。使用默认方法的场景有二个：可选方法和行为的多继承。

		public class C implements B, A {
			public void hello() {
				B.super().hello();
			}
		}

Optional
----

如果一个方法返回一个Object，那么在使用的时候总是要判断一下返回的结果是否为空，这样每个处理起来很麻烦。Optional一般使用在方法的返回值中，如果使用Optional来包装方法的返回值，使用Optional来检查还可以提供一个默认值
		
		Optional<String> opt = Optional.empty();
		Optional<String> opt = Optional.of("hello");
		Optional<String> opt = Optional.ofNullable(null);

CompletableFuture
----

CompletableFuture实现了Future接口，并且基于ForkJoinPool来执行任务，可以将二个Future组合起来。
	
		public Future<Double> getPriceAsync(final String product) {
			final CompletableFuture<Double> futurePrice = new CompletableFuture<>();
			new Thread(() -> {
				double price = calculatePrice(product);
				futurePrice.complete(price);
			}).start();
			return futurePrice;
		}				
		====
		Future<Double> price = CompletableFuture.supplyAsync(() ->
		calculatePrice(product));
		//如果第二个请求依赖于第一个请求的结果，可以使用thenCompose来组合二个Future
		public List<String> findPriceAsync(String product) {
			List<CompletableFuture<String>> priceFutures = tasks.stream()
			.map(task -> CompletableFuture.supplyAsync(() -> task.getPrice(product), executor)
			.map(future -> future.thenApply(Work::parse))
			.map<future -> future.thenCompose(work -> CompletableFuture.supplyAsync(() -> Count.applyCount(work), execuotr)))
			.collect(Collectors.toList());

			return priceFutures.stream().map(CompletableFuture::join).collect(Collectors.toList());
		}
上面这段代码使用了thenCompose来组合二个CompletableFuture。supplyAsync方法第二个参数接受一个自定义的Executor。首先使用CompletableFuture执行一个任务，调用getPrice方法，得到一个Future，之后使用thenApply方法，将Future结果运用parse方法，之后再使用完parse之后的结果作为参数再执行一个applyCount方法，然后收集策划和那个一个CompletableFuture<String>的list，最后再使用一个流，调用CompletableFuture的join方法，等待所有的异步任务执行完毕，获得最后的结果。

二个Future没有依赖关系示例
		Future<String> futurePrice = CompletableFuture.supplyAsync(() -> shop.getPricet("price1"))
		.thenCombine(CompletableFuture.supplyAsync(() -> shop.getPrice("price2")),(s1+s2) -> s1 + s2);

不需要等待所有的异步任务结束，只需要其中的一个完成就可以了，CompletableFuture也提供了这样的方法
		//假设getStream方法返回一个Stream<CompletableFuture<String>>
		CompletableFuture[] futures = getStream(“listen”).map(f -> f.thenAccept(System.out::println)).toArray(CompletableFuture[]::new);
		//等待其中的一个执行完毕
		CompletableFuture.anyOf(futures).join();	

时间和日期API
----		

LocalDateTime, LocalDate, LocalTime, Duration, Period, Instant, DateTimeFormatter
		//创建日期
		LocalDate date = LocalDate.of(2017,1,21); //2017-01-21
		int year = date.getYear() //2017
		Month month = date.getMonth(); //JANUARY
		int day = date.getDayOfMonth(); //21
		DayOfWeek dow = date.getDayOfWeek(); //SATURDAY
		int len = date.lengthOfMonth(); //31(days in January)
		boolean leap = date.isLeapYear(); //false(not a leap year)
		 
		//时间的解析和格式化
		LocalDate date = LocalDate.parse(“2017-01-21”);
		LocalTime time = LocalTime.parse(“13:45:20”);
		 
		LocalDateTime now = LocalDateTime.now();
		now.format(DateTimeFormatter.BASIC_ISO_DATE);
		 
		//合并日期和时间
		LocalDateTime dt1 = LocalDateTime.of(2017, Month.JANUARY, 21, 18, 7);
		LocalDateTime dt2 = LocalDateTime.of(localDate, time);
		LocalDateTime dt3 = localDate.atTime(13,45,20);
		LocalDateTime dt4 = localDate.atTime(time);
		LocalDateTime dt5 = time.atDate(localDate);
		 
		//操作日期
		LocalDate date1 = LocalDate.of(2014,3,18); //2014-3-18
		LocalDate date2 = date1.plusWeeks(1); //2014-3-25
		LocalDate date3 = date2.minusYears(3); //2011-3-25
		LocalDate date4 = date3.plus(6, ChronoUnit.MONTHS); //2011-09-25	