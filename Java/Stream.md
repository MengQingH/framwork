Stream 是一个 **来自数据源** 的 **元素队列** 并 **支持聚合操作**。
* 元素是一系列元素的集合，可以对这些元素进行不同的操作。
* 数据流的来源，可以是集合，数组，IO等。
* Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
* 内部迭代： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 创建流
* 通过集合的stream()方法或者parallelStream()，比如Arrays.asList(1,2,3).stream()。
* 通过Arrays.stream(Object[])方法, 比如Arrays.stream(new int[]{1,2,3})。
* 使用流的静态方法，比如Stream.of(Object[]), IntStream.range(int, int) 或者 Stream.iterate(Object, UnaryOperator)，如Stream.iterate(0, n -> n * 2)，或者generate(Supplier<T> s)如Stream.generate(Math::random)。

## 流操作
这些方法的返回值为操作后的流，可以连续调用
1. distinct()  去除流中的重复元素，是通过equals方法来检测是否相同的。

2. filter(Predicate<? super T> predicate)  过滤，传入一个断言类型的lambda表达式，过滤出满足条件的元素。

3. map(Function<? super T, ? extends R> mapper)  将元素映射成另外的值

4. flatMap(Function<? super T, ? extends Stream<? extends R>> mapper)：将每一个元素转换成一个流对象，而flatMap方法返回的流包含的元素为mapper生成的所有流中的元素

5. limit(long maxSize)  返回前n个元素

6. peek(Consumer<? super T> action)  使用lambda表达式消费流中的元素，但是返回的流中还是包含这些元素

7. sorted()  对流中的元素进行排序，流中的元素需要实现Comparable接口，也可以使用sorted(Comparator<? super T> comparator)指定排序的方式

8. skip(long n)  抛弃前n个元素，如果流中的元素个数小于n个，返回一个空的流


## 终点操作
终点操作的方法不返回流，为流的最终操作方法。
1. boolean allMatch(Predicate<? super T> predicate)：全部满足断言时返回true，否则false，流为空返回true

2. boolean anyMatch(Predicate<? super T> predicate)：任意一个满足断言返回true，否则false

3. boolean noneMatch(Predicate<? super T> predicate)：所有元素都不满足断言返回true，否则false

4. long count()：返回流中元素的数量

5. collect(Collector<? super T,A,R> collector)：使用一个collector对流中的数据进行操作，辅助类Collectors提供了很多collector，如聚合类averagingInt、最大最小值maxBy minBy、计数counting、分组groupingBy、字符串连接joining、分区partitioningBy、汇总summarizingInt、化简reducing、转换toXXX等。

6. Optional<T> findAny()：返回任意一个元素，如果流为空，返回一个包含空的Optional。

7. Optional<T> findFirst()：返回第一个元素，如果流为空，返回一个包含空的Optional。

8. void forEach(Consumer<? super T> action)：遍历每一个元素，并执行指定的操作，和peek不同，这是一个终点操作。这个方法不担保按照流的encounter order顺序执行，如果对于有序流按照它的encounter order顺序执行，你可以使用forEachOrdered方法。

9. void forEachOrdered(Consumer<? super T> action)：

10. Optional<T> max(Comparator<? super T> comparator)：max  min返回流中的最大值最小值

11. reduce：reduce方法接受BinaryOperator积累函数，该函数有两个参数，执行后将返回值和下一个元素传入再次进行操作，返回最终的结果。
    * Optional<T> reduce(BinaryOperator<T> accumulator)
    * T reduce(T identity, BinaryOperator<T> accumulator)
    * <U> U reduce(U identity,
                 BiFunction<U, ? super T, U> accumulator,
                 BinaryOperator<U> combiner)