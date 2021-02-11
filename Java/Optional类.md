Optional类是一个**容器类**，它可以**保存某个类型的值或对象**，或者只保存null。Optional类提供了很多方法，这样就不用进行显式的空值检测，避免了空指针异常的问题。

1. **创建Optional对象**，Optional类的构造方法私有，只能通过提供的静态方法来创建对象：
    * of(T t)：t不能为空，会抛出空指针异常
    * ofNullable(T t)：t可以为空，t为空的话创建一个包含空的Optional对象。

2. get()  **获取Optional对象中的值**，如果值为空，那么也会出现空指针异常。

3. **检查是否为空**：
    * boolean isPresent()  存在返回true，不存在返回false
    * void isPresent(Consumer<? super T> consumer)    存在则执行传入的lambda表达式

4. **返回默认值**
    * orElse(T t)  如果对象中的值不为空，返回该值，如果为空，返回t
    * orElseGet(Supplier<? extends T> other)  如果不为空返回，如果为空执行参数中的lambda表达式，并返回表达式的结果。
    * orElseThrow(Supplier<? extends X> exceptionSupplier)  如果不为空直接返回，如果为空抛出Supplier继承的异常。

    orElse和orElseGet的区别，如果Optional对象不为空，那么两种方法都会将对象中的值作为返回值。但是，**orElse() 方法仍然会创建默认的 User 对象，orElseGet()  方法将不再创建 User 对象**。

5. **对值进行转换**
    * map(Function<? super T,Optional<U>> mapper)  根据lambda表达式对对象中的值进行转换。返回转换后的Optional对象
    * flatMap(Function<? super T,Optional<U>> mapper)  lambda表达式的返回值需要为Optional对象，该方法会把返回的Optional对象中的值设置到当前的对象中。

6. **对值进行过滤**
    * filter(Predicate<? super T> predicate)   predicate为真时，返回实际值，如果为假，返回一个空的Optional对象。

```java
User user = new User();
user.setName("mqh");
user.setAge(21);
user.setId(1);
user.setOptional(Optional.ofNullable("test"));

//创建Optional对象
Optional.ofNullable(user);

//取值
Optional.ofNullable(uesr).get();

//返回默认值
Optional.ofNullable(user).orElse(new User());//直接创建对象
Optional.ofNullable(user).orElseGet(() -> new User());//使用lambda表达式创建对象并返回

//转换
Optional.ofNullable(user).map(u -> u.getName()).get();//map可以直接进行转换
Optional.ofNullable(user).flatMap(u -> u.getOptional()).get();//flatMap需要lambda表达式返回一个Optional对象，把对象中的值赋给当前的Optional对象

//过滤
Optional.ofNullable(user).filter(u -> u.getName() != null);
```