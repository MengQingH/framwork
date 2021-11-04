<!--
 * @Author: QingHui Meng
 * @Date: 2020-10-12 16:58:08
-->
Stream 是一个 **来自数据源** 的 **元素队列** 并 **支持聚合操作**。
* 元素是一系列元素的集合，可以对这些元素进行不同的操作。
* 数据流的来源，可以是集合，数组，IO等。
* Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
* 内部迭代： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代，这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

## 特点：
* 只能遍历一次：数据流的从一头获取数据源，在流水线上依次对元素进行操作，当元素通过流水线，便无法再对其进行操作，可以重新在数据源获取一个新的数据流进行操作；

* 采用内部迭代：对Collection进行处理，一般会使用 Iterator 遍历器的遍历方式，这是一种外部迭代；
而对于处理Stream，只要申明处理方式，处理过程由流对象自行完成，这是一种内部迭代，对于大量数据的迭代处理中，内部迭代比外部迭代要更加高效；

## 相比iterator的优点
* 无存储：流并不存储值；流的元素源自数据源（可能是某个数据结构、生成函数或I/O通道等等），通过一系列计算步骤得到；
* 函数式风格：对流的操作会产生一个结果，但流的数据源不会被修改；
* 惰性求值：多数流操作（包括过滤、映射、排序以及去重）都可以以惰性方式实现。这使得我们可以用一遍遍历完成整个流水线操作，并可以用短路操作提供更高效的实现；
* 无需上界：不少问题都可以被表达为无限流（infinite stream）：用户不停地读取流直到满意的结果出现为止（比如说，枚举 完美数 这个操作可以被表达为在所有整数上进行过滤）；集合是有限的，但流可以表达为无线流；
* 代码简练：对于一些collection的迭代处理操作，使用 stream 编写可以十分简洁，如果使用传统的 collection 迭代操作，代码可能十分啰嗦，可读性也会比较糟糕；

## 创建流
* 通过集合的**stream()方法**或者parallelStream()，比如Arrays.asList(1,2,3).stream()。
* 通过**Arrays.stream(Object[])方法**, 比如Arrays.stream(new int[]{1,2,3})。
    * 如果使用这种方法传入基本数据类型int, double, long，那么创建的就是对应的流IntStream，DoubleStream，LongStream，可以使用boxed()方法转换为普通的流Stream<Integer>。
* 使用流的静态方法，比如**Stream.of(object)**, **IntStream.range(int, int)** 或者 **Stream.iterate(Object, UnaryOperator)**，如Stream.iterate(0, n -> n * 2)，或者**generate(Supplier<T> s)**如Stream.generate(Math::random)。
    * Stream.of(object)：创建的是object类型的流，如Stream.of(new int[]{1,2})，那么创建的就是int []类型的流
* 可以使用StreamSupport工具类，用Spliterator来创建流。如**StreamSupport.stream(Spliterator<T> spliterator, boolean parallel)**来创建一个stream<T>类型的流。集合类的stream()方法来创建流都是用的此方法。

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

## Collectors
### toMap()
把流转为map，需要执行key和value
1. 指定key-value，value是对象中的某个值 ``Collectors.toMap(User::getId, User::getName);``
2. value是对象本身 
    ```java
    Collectors.toMap(User::getId, User->User);
    Collectors.toMap(User::getId, Function.identity());
    ```
3. value是对象本身，key冲突的解决办法
    ```java
    // 如果key产生冲突，用第二个key覆盖第一个
    Collectors.toMap(User::getId, Function.identity(),(key1,key2)->key2)
    ```

### partitioningBy()
将Stream中的元素依据某个二值逻辑（满足条件、不满足条件）分成互补的两部分，比如男女性别，成绩是否及格等
```java
Map<Boolean, List<Student>> passingFailing = students
                    .stream().collect(Collectors.partitioningBy(s -> s.getGrade() >= PASS_THRESHOLD));
```

### groupingBy()
1. 按照某个属性对数据进行分组，属性相同的元素会被分到map中的同一个key上。
    ```java
    Map<Department, List<Employee>> byDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
    ```
2. 分组之后进行某种运算，如求和，计数，平均值，类型转换等。将元素分组的收集器叫做上游收集器，之后执行其他运算的收集器叫做下游收集器(downstream Collector)。
    ```java
    // 使用下游收集器统计部门人数
    Map<Department, Integer> totalByDept = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,
                                                   Collectors.counting()));// 下游收集器
    ```
3. 下游收集器还可以包含更下游的收集器
    ```java
    // 按照部门对员工分布组，并只保留员工的名字
    Map<Department, List<String>> byDept = employees.stream()
                    .collect(Collectors.groupingBy(Employee::getDepartment,
                            Collectors.mapping(Employee::getName,// 下游收集器
                                    Collectors.toList())));// 更下游的收集器
    ```

### joining()
字符串拼接。
```java
Stream<String> stream = Stream.of("I", "love", "you");
String joined1 = stream.collect(Collectors.joining());// "Iloveyou"
String joined2 = stream.collect(Collectors.joining(","));// "I,love,you"
String joined3 = stream.collect(Collectors.joining(",", "{", "}"));// "{I,love,you}"
```

### collectingAndThen(Collector<T, A, R> var0, Function<R, RR> var1)
分组之后再进行其他操作，先collector，再function
```java
// 将servers joining 然后转成大写
 servers.stream.collect(Collectors.collectingAndThen(Collectors.joining(","), String::toUpperCase));
```