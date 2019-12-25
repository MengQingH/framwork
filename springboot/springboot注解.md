## @Primary
当有多个cacheManager的时候，使用该注解在一个管理器上，表示是默认的管理器

## @PostContruct
spring框架的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在spring容器初始化的时候执行该方法。

## @Date注解
@Date注解的主要作用就是使代码更简洁，使用这个注解可以省去代码中大量的get(), set(), toString()方法。
* 引入lombok，lombok是一个工具类库，可以用简单的注解形式简化代码
    ```xml
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.4</version>
        <scope>provided</scope>
    </dependency>
    ```
常用的注解：
* @Data ： 注在类上，提供类的get、set、equals、hashCode、canEqual、toString方法
* @AllArgsConstructor ： 注在类上，提供类的全参构造
* @NoArgsConstructor ： 注在类上，提供类的无参构造
* @Setter ： 注在属性上，提供 set 方法
* @Getter ： 注在属性上，提供 get 方法
* @EqualsAndHashCode ： 注在类上，提供对应的 equals 和 hashCode 方法
* @Log4j/@Slf4j ： 注在类上，提供对应的 Logger 对象，变量名为 log

## @Accessors注解
用来配置lombok如何getters和setters方法的格式，可以添加前缀，改变方法的格式。有以下三个属性：
* fluent：为一个布尔值，如果为true则生成的get/set方法没有set/get前缀，默认为false。
* chain：布尔值，如果为true则生成的set方法返回this，为false生成的set方法是void类型。默认为false，fluent为true时，默认为true。
* prefix：为一个String数组，可以指定前缀，生成get/set方法时会在属性上去掉指定的前缀。

## @Valid注解
@valid注解时用来在验证方法中的参数，在类中需要校验的字段上添加上校验注解，然后在方法中的参数中使用该注解，就可以进行字段的检验。可以用在controller中验证请求参数。
1. 首先添加依赖
```xml
<dependency>
    <groupId>javax.validation</groupId>
    <artifactId>validation-api</artifactId>
    <version>1.1.0.Final</version>
</dependency>
```
2. 在类中的字段上加上需要的校验注解，有以下几种：
    * @NotNull：字段不能为null，否则验证无法通过
    * @NotEmpty：不能为null，且长度不能为0，可以用在String类、Collection、Map、数组上。
    * @NotBlank：用在String上面，和NotEmpty不同的是尾部空格将会被忽略。
    * @Null：字段为null验证才能通过
    * @Size(min=,max=)：修饰的字段长度在min和max之间，min默认为0，max默认为2147483647
    * @Max(value=)：该值小于等于value，才能通过验证
    * @Min(value=)：大于等于value，才能通过验证
    * @AssertFalse：该属性的值只能为false才能通过
    * @AssertTrue：该属性的值只能为true才能通过
    * @DecimalMax(value=)：小数的最大值
    * @DecimalMin(value=)：小数的最小值
    * @Digits(integer=,fraction=)：验证数字的整数位和小数位的位数是否超过指定的长度
    * @Future：时间是否在当前时间之后
    * @Past：时间是否在当前时间之前
    * @Pattern(regexp = "")：字符串是否和该正则相匹配

上面的注解都可以设置message属性来设置提示未通过验证的提示信息。
3. 在Controller中的参数类前加上@Valid注解，表示进行属性验证。
    ```java
    public String test(@Valid Person person, BindingResult result){};
    ```

## @GetMapping、@PostMapping、@PutMapping、@DeleteMapping、@PatchMapping
设置相应HTTP请求方式的映射。以@GetMapping为例，该注解为@ReuqestMapping(method=RequestMethod.GET)的缩写，请求方式和uri都对应的时候，才能使用该映射。