# Stream实战指南

## 流的创建

- 通过Stream接口的静态工厂方法
- 通过Arrays方法
- 通过Collection接口的默认方法

### 1. 通过Stream接口的静态工厂方法

Stream

- public static<T> Builder<T> builder().add().build()

    ```Stream.builder().add("a1").add("a2").build();```

- public static<T> Stream<T> empty()

    ```Stream.empty()```

- public static<T> Stream<T> of(T t)

    ```Stream.of(1)```

- public static<T> Stream<T> of(T... values)

    ```Stream.of(1, 2, 3)```

- public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)

    ```Stream.iterate(1, integer -> integer + 1).limit(10)```

- public static<T> Stream<T> generate(Supplier<T> s)

    ```Stream.generate(() -> 1)```
- public static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)

    ```Stream.concat(Stream.of(1), Stream.of(2))```

IntStream

- public static Builder builder().build()
- public static IntStream empty()
- public static IntStream of(int t)
- public static IntStream of(int... values)
- public static IntStream iterate(final int seed, final IntUnaryOperator f)
- public static IntStream generate(IntSupplier s)
- public static IntStream range(int startInclusive, int endExclusive)

    ```IntStream.range(0, 10).forEach(System.out::println);// 0..9```

- public static IntStream rangeClosed(int startInclusive, int endInclusive)

    ```IntStream.rangeClosed(0, 10).forEach(System.out::println);// 0..10```

- public static IntStream concat(IntStream a, IntStream b)

LongStream

- public static Builder builder().build()
- public static IntStream empty()
- public static IntStream of(int t)
- public static IntStream of(int... values)
- public static IntStream iterate(final int seed, final IntUnaryOperator f)
- public static IntStream generate(IntSupplier s)
- public static IntStream range(int startInclusive, int endExclusive)
- public static IntStream rangeClosed(int startInclusive, int endInclusive)
- public static IntStream concat(IntStream a, IntStream b)

DoubleStream

- public static Builder builder().build()
- public static IntStream empty()
- public static IntStream of(int t)
- public static IntStream of(int... values)
- public static IntStream iterate(final int seed, final IntUnaryOperator f)
- public static IntStream generate(IntSupplier s)
- public static IntStream concat(IntStream a, IntStream b)

### 2. 通过Arrays方法

- public static <T> Stream<T> stream(T[] array)

    ```Arrays.stream(new String[]{"A", "B", "C"}).forEach(System.out::print);```

- public static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)

    ```Arrays.stream(new String[]{"A", "B", "C"}, 0, 2).forEach(System.out::print);```

- public static IntStream stream(int[] array)
- public static IntStream stream(int[] array, int startInclusive, int endExclusive)
- public static LongStream stream(long[] array)
- public static LongStream stream(long[] array, int startInclusive, int endExclusive)
- public static DoubleStream stream(double[] array)
- public static DoubleStream stream(double[] array, int startInclusive, int endExclusive)

### 3. 通过Collection接口的默认方法

- default Stream<E> stream()

    ```Arrays.asList(1, 2, 3).stream().forEach(System.out::print);```

- default Stream<E> parallelStream()

    ```Arrays.asList(1, 2, 3).parallelStream().forEach(System.out::print);```

## 中间操作

略，比较简单

## 终止操作

- void forEach(Consumer<? super T> action)
- void forEachOrdered(Consumer<? super T> action)
- Object[] toArray()
- <A> A[] toArray(IntFunction<A[]> generator)

    ```Stream.of(1, 2, 3).toArray(size -> new Integer[size])```
    
    generator的作用是根据指定的元素size创建元素数组。简化为Lamda，可得到如下：

    ```Stream.of(1, 2, 3).toArray(Integer[]::new)```

- T reduce(T identity, BinaryOperator<T> accumulator)

    ```123```

- Optional<T> reduce(BinaryOperator<T> accumulator)

    ```Optional<Integer> sum = Stream.of(1, 2, 3).reduce(Integer::sum)```

- <U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner)

    ```Integer sum = Stream.of(1, 2, 3).reduce(0, Integer::sum)```

- <R> R collect(Supplier<R> supplier, BiConsumer<R, ? super T> accumulator, BiConsumer<R, R> combiner)

    ```
    观察系统源码 Collectors.toList() 来学习该方法的使用

    Collector<T, ?, List<T>> toList() {
        return new CollectorImpl<>(
            (Supplier<List<T>>) ArrayList::new, 
            List::add,
            (left, right) -> { left.addAll(right); return left; },
            CH_ID
        );
    }

    CollectorImpl(
                    Supplier<A> supplier,
                    BiConsumer<A, T> accumulator,
                    BinaryOperator<A> combiner,
                    Set<Characteristics> characteristics) {
            this(supplier, accumulator, combiner, castingIdentity(), characteristics);
    }

    // supplier 提供者，提供一个列表
    // accumulator 累加器，将每一个元素累加到列表中
    // combiner 合并器，Stream支持并行parallel，合并多个线程的数据
    ```

- <R, A> R collect(Collector<? super T, A, R> collector)

    ```关于Collectors的静态方法相当多，放在下面单独讲解```

- Optional<T> min(Comparator<? super T> comparator)
- Optional<T> max(Comparator<? super T> comparator)
- long count()
- boolean anyMatch(Predicate<? super T> predicate)
- boolean allMatch(Predicate<? super T> predicate)
- boolean noneMatch(Predicate<? super T> predicate)
- Optional<T> findFirst()
- Optional<T> findAny()

## Collectors

- public static <T, C extends Collection<T>>
    Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)

    ```Stream.of(1, 2, 3).collect(Collectors.toCollection(ArrayList::new));```
    
- public static <T>
    Collector<T, ?, List<T>> toList()    
- Collector<T, ?, Set<T>> toSet()
- public static Collector<CharSequence, ?, String> joining()

    ```Stream.of("A", "B", "C").collect(Collectors.joining());// ABC```

- public static Collector<CharSequence, ?, String> joining(CharSequence delimiter)
- public static Collector<CharSequence, ?, String> joining(CharSequence delimiter,
                                                             CharSequence prefix,
                                                             CharSequence suffix)

    ```Stream.of(1, 2, 3).collect(Collectors.joining(",", "[", "]"));// [A,B,C]```

- public static <T, U, A, R>
    Collector<T, ?, R> mapping(Function<? super T, ? extends U> mapper,
                               Collector<? super U, A, R> downstream)

    ```Stream.of("1", "2", "3").collect(Collectors.mapping(Integer::valueOf, Collectors.toList()));// [1, 2, 3]```

- public static<T,A,R,RR> Collector<T,A,RR> collectingAndThen(Collector<T,A,R> downstream,
                                                                Function<R,RR> finisher)

    ```
    Stream.of("1", "2", "3").collect(collectingAndThen(
                toList(), 
                strings -> strings.stream().map(s -> s + "S").collect(toList())
        ));
    // [1S, 2S, 3S]
    ```
- public static <T> Collector<T, ?, Long>
    counting()
- public static <T> Collector<T, ?, Optional<T>>
    minBy(Comparator<? super T> comparator)

    ```Optional<String> min = Stream.of("1", "2", "3").collect(minBy(Comparator.<String>naturalOrder().reversed()));// 3```
- public static <T> Collector<T, ?, Optional<T>>
    maxBy(Comparator<? super T> comparator)
- public static <T> Collector<T, ?, Integer>
    summingInt(ToIntFunction<? super T> mapper)

    ```Integer sum = Stream.of("1", "2", "3").collect(summingInt(Integer::valueOf));// 6```

- public static <T> Collector<T, ?, Long>
    summingLong(ToLongFunction<? super T> mapper)
- public static <T> Collector<T, ?, Double>
    summingDouble(ToDoubleFunction<? super T> mapper)
- public static <T> Collector<T, ?, Double>
    averagingInt(ToIntFunction<? super T> mapper)

    ```Double average = Stream.of("1", "2", "3").collect(averagingInt(Integer::valueOf));// 2.0```

- public static <T> Collector<T, ?, Double>
    averagingLong(ToLongFunction<? super T> mapper)
- public static <T> Collector<T, ?, Double>
    averagingDouble(ToDoubleFunction<? super T> mapper)
- public static <T> Collector<T, ?, T>
    reducing(T identity, BinaryOperator<T> op)

    ```Stream.of(1, 2, 3).collect(reducing(0, Integer::sum));// 6```

- public static <T> Collector<T, ?, Optional<T>>
    reducing(BinaryOperator<T> op)
- public static <T, U>
    Collector<T, ?, U> reducing(U identity,
                                Function<? super T, ? extends U> mapper,
                                BinaryOperator<U> op)

    ```Stream.of("1", "2", "3").collect(reducing(0, Integer::valueOf, Integer::sum));// 6```

- public static <T, K> Collector<T, ?, Map<K, List<T>>>
    groupingBy(Function<? super T, ? extends K> classifier)

    ```Map<Integer, List<Integer>> result = Stream.of(1, 2, 3, 2, 3).collect(groupingBy(integer -> integer));// {1=[1], 2=[2, 2], 3=[3, 3]}```
- public static <T, K, A, D>
    Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier,
                                          Collector<? super T, A, D> downstream)

    ```Map<Integer, Integer> result = Stream.of(1, 2, 3, 2, 3).collect(groupingBy(integer -> integer, reducing(0, Integer::sum)));// {1=1, 2=4, 3=6}```

- public static <T, K, D, A, M extends Map<K, D>>
    Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                  Supplier<M> mapFactory,
                                  Collector<? super T, A, D> downstream)

    ```Map<Integer, Integer> result = Stream.of(1, 2, 3, 2, 3).collect(groupingBy(integer -> integer, ConcurrentHashMap::new, reducing(0, Integer::sum)));// {1=1, 2=4, 3=6}```
- public static <T, K>
    Collector<T, ?, ConcurrentMap<K, List<T>>>
    groupingByConcurrent(Function<? super T, ? extends K> classifier)
- public static <T, K, A, D>
    Collector<T, ?, ConcurrentMap<K, D>> groupingByConcurrent(Function<? super T, ? extends K> classifier,
                                                              Collector<? super T, A, D> downstream)
- public static <T, K, A, D, M extends ConcurrentMap<K, D>>
    Collector<T, ?, M> groupingByConcurrent(Function<? super T, ? extends K> classifier,
                                            Supplier<M> mapFactory,
                                            Collector<? super T, A, D> downstream)
- public static <T>
    Collector<T, ?, Map<Boolean, List<T>>> partitioningBy(Predicate<? super T> predicate)

    ```Stream.of(1, 2, 3, 2, 3).collect(partitioningBy(integer -> 0 == (integer & 1)));// {false=[1, 3, 3], true=[2, 2]}```

- public static <T, D, A>
    Collector<T, ?, Map<Boolean, D>> partitioningBy(Predicate<? super T> predicate,
                                                    Collector<? super T, A, D> downstream)

    ```Stream.of(1, 2, 3, 2, 3).collect(partitioningBy(integer -> 0 == (integer & 1), reducing(0, Integer::sum)));// {false=7, true=4}```

- public static <T, K, U>
    Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper)

    ```
    // java.lang.IllegalStateException: Duplicate key 2
    Stream.of(1, 2, 3, 2, 3).collect(toMap(integer -> integer, integer -> integer));
    
    // {1=1, 2=2, 3=3}
    Stream.of(1, 2, 3,).collect(toMap(integer -> integer, integer -> integer));

    // 利用Functions提供的静态方法 {1=1, 2=2, 3=3}
    Stream.of(1, 2, 3).collect(toMap(Functions.identity(), Functions.identity()));
    ```
- public static <T, K, U>
    Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper,
                                    BinaryOperator<U> mergeFunction)

    ```Map<Integer, Integer> result = Stream.of(1, 2, 3, 2, 3).collect(toMap(integer -> integer, integer -> integer, Integer::sum));// {1=1, 2=4, 3=6}```

- public static <T, K, U, M extends Map<K, U>>
    Collector<T, ?, M> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper,
                                BinaryOperator<U> mergeFunction,
                                Supplier<M> mapSupplier)

    ```Map<Integer, Integer> result = Stream.of(1, 2, 3, 2, 3).collect(toMap(integer -> integer, integer -> integer, Integer::sum, ConcurrentHashMap::new));// {1=1, 2=4, 3=6}```

- public static <T, K, U>
    Collector<T, ?, ConcurrentMap<K,U>> toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                                                        Function<? super T, ? extends U> valueMapper)
- public static <T, K, U>
    Collector<T, ?, ConcurrentMap<K,U>>
    toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                    Function<? super T, ? extends U> valueMapper,
                    BinaryOperator<U> mergeFunction)
- public static <T, K, U, M extends ConcurrentMap<K, U>>
    Collector<T, ?, M> toConcurrentMap(Function<? super T, ? extends K> keyMapper,
                                       Function<? super T, ? extends U> valueMapper,
                                       BinaryOperator<U> mergeFunction,
                                       Supplier<M> mapSupplier)
- public static <T>
    Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)

    ```
    // IntSummaryStatistics{count=3, sum=6, min=1, average=2.000000, max=3}

    IntSummaryStatistics statistics = Stream.of("1", "2", "3").collect(summarizingInt(Integer::valueOf));
    ```

- public static <T>
    Collector<T, ?, LongSummaryStatistics> summarizingLong(ToLongFunction<? super T> mapper)
- public static <T>
    Collector<T, ?, DoubleSummaryStatistics> summarizingDouble(ToDoubleFunction<? super T> mapper)