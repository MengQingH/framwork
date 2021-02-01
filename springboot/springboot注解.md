## @Primary
当有多个cacheManager的时候，使用该注解在一个管理器上，表示是默认的管理器

## @RestController
controller类的注解，表示Controller里面的方法都以json格式输出

## @PostContruct
spring框架的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在spring容器初始化的时候执行该方法。

## @Value
设置在方法的属性上，表示把application.property文件中的值设置给该属性。@Value("${com.mqh.name}")

## @Bean注解
用于方法上，表示**产生一个对象，并把这个java对象交给Spring管理**，产生这个java对象的方法Spring只会调用一次，随后Spring会把这个对象放在自己的IOC容器中。Spring中对bean的处理默认是单例的，实例化的过程只有一次，即多次获取的bean对象都是同一个bean对象。
SpringIOC 容器管理一个或者多个bean，这些bean都需要在@Configuration注解下进行创建
```java
public class MyBean {
    public MyBean(){
        System.out.println("MyBean Initializing");
    }
    public void init(){
        System.out.println("Bean 初始化方法被调用");
    }
    public void destroy(){
        System.out.println("Bean 销毁方法被调用");
    }
}

@Configuration
public class AppConfig {
    // 使用@Bean 注解表明myBean需要交给Spring进行管理
    // 未指定bean 的名称，默认采用的是 "方法名" + "首字母小写"的配置方式
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public MyBean myBean(){
        return new MyBean();
    }

}
```
属性：
* name：此bean的名称，如果未指定，那么bean的名称是带注解方法的名称。
* initMethod：传入类中的一个方法名，在bean实例化的时候调用这个方法。
* destroyMethod：传入方法名，在bean被销毁的时候调用这个方法。

## @Data注解
@Data注解的主要作用就是使代码更简洁，使用这个注解可以省去代码中大量的get(), set(), toString()方法。
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
* @Data：注在类上，提供类的get、set、equals、hashCode、canEqual、toString方法
* @AllArgsConstructor：注在类上，提供类的全参构造
* @RequiredArgsConstructor：会生成一个包含常量和标识NotNull的变量的构造方法。生成的构造方法是private，如何想要对外提供使用可以使用staticName属性生成一个static方法。
* @NoArgsConstructor：注在类上，提供类的无参构造
* @Setter：注在属性上，提供 set 方法
* @Getter：注在属性上，提供 get 方法
* @EqualsAndHashCode：注在类上，提供对应的 equals 和 hashCode 方法
* @Log4j/@Slf4j：注在类上，提供对应的 Logger 对象，变量名为 log

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

## @JsonIgnoreProperties(ignoreUnknown = true)
添加在实体类上，json字符串转换为实体类时，如果json中有类中不包含的属性，就忽略这些属性。