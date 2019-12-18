1. 导入依赖包：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```
2. application.yml中添加redis的配置
```yml
# properties文件
# Redis数据库索引（默认为0）
spring.redis.database=0  
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379  
# Redis服务器连接密码（默认为空）
spring.redis.password=
# 连接池最大连接数（使用负值表示没有限制） 默认 8
spring.redis.lettuce.pool.max-active=8
# 连接池最大阻塞等待时间（使用负值表示没有限制） 默认 -1
spring.redis.lettuce.pool.max-wait=-1
# 连接池中的最大空闲连接 默认 8
spring.redis.lettuce.pool.max-idle=8
# 连接池中的最小空闲连接 默认 0
spring.redis.lettuce.pool.min-idle=0

# yml文件
spring:
    redis:
        # redis数据库索引
        database: 0
        # 数据库地址
        host: localhost
        # redis连接端口
        port: 6379
        # redis连接密码
        password：
        # 超时时间
        timeout: 500

        lettuce:
            pool:
                # 连接池最大连接数（使用负值表示没有限制） 默认 8
                max-active: 8
                # 连接池最大阻塞等待时间（负值表示没有限制），默认-1
                max-wait: -1
                # 连接池中的最大空闲连接 默认 8
                max-idle: 8
                # 连接池中的最小空闲连接 默认 0
                min-idle: 0
```
3. 此时在应用中已经可以使用redis缓存，使用官方提供的方式来进行缓存。也可以自己重写以下配置：
    * 配置序列化方式（用于手动使用缓存）：当把数据存储到redis数据库时，key和value都是通过spring提供的Serializer序列化到数据库的，RedisTemplate默认使用的是JdkSerializationRedisSerializer，StringRedisTemplate默认使用的是StringRedisSerializer。可以重写redis的配置，自己决定序列化的方式。在RedisConfig类中重写redisTemplate方法。
    * 配置缓存管理器（用于使用注解）：如果要使用注解，那么就需要配置keyGenerator和cacheManager，让springboot自动管理缓存。其中，keyGenerator用来配置key的生成策略，非必须；cacheManager用来管理生成的缓存的过期时间和key前缀等，必须配置。
```java

/**
 * Redis配置类
 *
 * @Configure 表示当前类属于配置类
 * @EnableCaching 表示支持缓存
 */
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {


    /**
     * redis key的生成策略
     * o：类
     * method：方法
     * params：参数
     * 注意: 该方法只是声明了key的生成策略,还未被使用,需在@Cacheable注解中指定keyGenerator
     * 如: @Cacheable(value = "key", keyGenerator = "cacheKeyGenerator")
     *
     * @return
     */
    @Bean
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object o, Method method, Object... objects) {
                StringBuffer sb = new StringBuffer();
                sb.append(o.getClass().getName());
                sb.append(method.getName());
                for (Object obj : objects) {
                    sb.append(obj.toString());
                }
                return sb.toString();
            }
        };
    }


    /**
     * Redis缓存配置，在redis2.x中CacheManager有了较大的改变，创建方式发生变化
     *
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory redisConnectionFactory) {

        //配置value序列化的方式
//        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        //无效，一直出错
        //为序列化对象配置objectMapper，从redis中取出的value为map，无法直接转为对象，使用ObjectMapper类把map转为json再转为对象
//        ObjectMapper objectMapper = new ObjectMapper();
//        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
//        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);


        //使用Jackson2JsonRedisSerializer方式反序列化时出错，所以使用GenericJackson2JsonRedisSerializer方式
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //SerializationPair的创建方式，RedisCacheConfiguration中配置序列化只能使用该方式
        RedisSerializationContext.SerializationPair value = RedisSerializationContext.SerializationPair.fromSerializer(genericJackson2JsonRedisSerializer);

        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        RedisSerializationContext.SerializationPair key = RedisSerializationContext.SerializationPair.fromSerializer(stringRedisSerializer);

        //创建一个缓存配置对象，可以进行一些缓存的配置
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .prefixKeysWith("prefix.")
                .serializeKeysWith(key)
                .serializeValuesWith(value)
                .entryTtl(Duration.ofHours(1));
        //通过传入缓存配置对象来创建一个cacheManager对象，SpringBoot2之后RedisCacheManager的创建方式发生了改变
        RedisCacheManager cacheManager = RedisCacheManager.builder(redisConnectionFactory).cacheDefaults(config).build();
        return cacheManager;
    }


    /**
     * 自己配置RedisTemplate，重写手动使用redis时序列化的方式
     *
     * @param connectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<Object, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);

        // 创建一个Jackson2JsonRedisSerializer序列化器
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        //把从redis中取出的json字符串转为对象，用于反序列化时的对象转换
//        ObjectMapper objectMapper = new ObjectMapper();
//        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
//        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
//        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        // 设置key和value的序列化方式
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);

        // 设置hash中key和value的序列化方式
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);

        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```
4. 可以在类中使用redis缓存。有两种使用方法：使用代码和注解：
    * 代码：RedisTemplate是spring中提供的redis操作模板，其中提供了很多操作redis的方法。可以创建一个RedisTemplate引用并进行注入，然后使用springboot提供的方法来操作redis。
    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class TestRedis {

        @Autowired
        private StringRedisTemplate stringRedisTemplate;
        @Autowired
        private RedisTemplate redisTemplate;

        @Test
        public void test() throws Exception {
            stringRedisTemplate.opsForValue().set("aaa", "111");
            Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
        }

        @Test
        public void testObj() throws Exception {
            Student student = new Student(1, "name1", 21, "man");
            //获取ValueOperations来进行redis的value操作
            ValueOperations<String, Student> operations = redisTemplate.opsForValue();
            // 向redis中添加一个键值对
            operations.set("student1", student);
            // 向redis中添加一个键值对并设置过期时间
            operations.set("student2", student, 1, TimeUnit.MINUTES);

            
            boolean exists = redisTemplate.hasKey("student1");
            if (exists)
                System.out.println("exists is true");
            else
                System.out.println("exists is false");
            System.out.println(operations.get("student1"));
        }
    }
    ```
    * 注解：返回值需要放入缓存的方法上添加注解，使用后会自动把返回值放入缓存中
    ```java
    @Cacheable(value = "redis", keyGenerator = "keyGenerator")
    @Override
    public Student getStuById(int id) {
        return studentMapper.selStuById(id);
    }
    ```


## Redis中缓存的方式
redis中有下面几种序列化的方式：
* **StringRedisSerializer**：简单的字符串序列化，如果key、value都是string，就用这种类型
* GenericToStringSerializer：可以将任何对象泛化为字符创并序列化
* **JdkSerializationRedisSerializer**：使用JDK提供的序列化功能，默认使用的方式。被序列化对象**必须实现Serializable接口**，被序列化除属性内容还有其他内容，长度长且不易阅读。存储到redis后key前会加上类的类型信息：
```java
//序列化的结果
//key
\xAC\xED\x00\x05t\x00\x08student1
//value
\xAC\xED\x00\x05sr\x00\x1Ecom.mh.springboot.pojo.Student\x00\xDD\xF9\x85\x1D\xA8F\x99\x02\x00\x04I\x00\x03ageL\x00\x02idt\x00\x13Ljava/lang/Integer;L\x00\x04namet\x00\x12Ljava/lang/String;L\x00\x03sexq\x00~\x00\x02xp\x00\x00\x00\x15sr\x00\x11java.lang.Integer\x12\xE2\xA0\xA4\xF7\x81\x878\x02\x00\x01I\x00\x05valuexr\x00\x10java.lang.Number\x86\xAC\x95\x1D\x0B\x94\xE0\x8B\x02\x00\x00xp\x00\x00\x00\x01t\x00\x05name1t\x00\x03man
```
* **Jackson2JsonRedisSerializer**：使用Jackson库**序列化对象为json字符串**。被序列化对象**不需要实现Serializable接口**，序列化的结果清晰，容易阅读，而且存储字节少，速度快；对象中**必须有set方法**，并且**序列化时必须提供对象的类型信息**，好处是因为不需要class信息，所以只要字段名相同就可以反序列化成功（因为和包名无关），但是无法使用全局统一的序列化方式
```java
//Jackson2JsonRedisSerializer的构造方法：
public Jackson2JsonRedisSerializer(Class<T> type){...};
public Jackson2JsonRedisSerializer(JavaType javaType){...};
//所以初始化时需要传入对象的类型信息

//示例：
//使用之前需要传入对象的信息。（测试时传入Object.class也可以？？？？？）
redisTemplate.setValueSerializer(new Jackson2JsonRedisSerializer<>(Student.class));
Student student = new Student(1, "name1", 21, "man");
redisTemplate.opsForValue().set("student1", student);

//序列化的结果如下:
key：student1
{
  "id": 1,
  "name": "name1",
  "age": 21,
  "sex": "man"
}
```
* **GenericJackson2JsonRedisSerializer**：和Jackson2JsonRedisSerializer基本相同，但是**不需要在序列化时提供对象的类型信息**，更加通用。但是会在序列化信息中添加类的class信息，所以无法反序列化不同路径的对象。
```java
//构造方法如下，所以初始化时不需要向Jackson2JsonRedisSerializer那样传入对象的类型信息
public GenericJackson2JsonRedisSerializer(){...}

//示例：
redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
Student student = new Student(1, "name1", 21, "man");
redisTemplate.opsForValue().set("student1", student);

//序列化结果如下：
key：student1
{
  "@class": "com.mh.springboot.pojo.Student",
  "id": 1,
  "name": "name1",
  "age": 21,
  "sex": "man"
}
//序列化结果中包含了对象的类型信息，所以无法反序列化不同路径的对象
```

第三方序列化器：很多第三方组件都提供了对应的序列化器，如ali的FastJsonRedisSerializer：
* FastJsonRedisSerializer：也需要提供对象的类型信息
* GenericFastJsonRedisSerializer：不需要提供对象的类型信息，更加通用


## SpringBoot中的redis操作
RedisTemplate、StringRedisTemplate是spring中操作redis的中心类，提供了很多和redis交互的方法。二者的区别：
* StringRedisTemplate继承自RedisTemplate
* StringRedisTemplate默认使用String序列化方式，RedisTemplate默认使用jdk自带的序列化方式。
* 两者数据不互通，只能各自管理各自处理过的数据。

RedisTemplate中根据不同的redis数据类型（String, List, ZSet, Hash），封装了相应的相应的类opsForXXX。封装的接口，以及RedisTemplate中的相应方法：
* ListOperations：Redis列表操作
* SetOperations：Redis集合操作
* ValueOperations：Redis字符串操作
* ZSetOperations：Redis有序集合操作
* HashOperations：Redis散列类型操作
* GeoOperations：Redis的地理空间操作，如GEOADD，GEORADIUS..
* HyperLogLogOperations：Redis的HyperLogLog操作，如PFADD，PFCOUNT..
<br>获取方法：<br><img src=img/方法.png><br>

## redis缓存注解
* @Cacheable: 缓存入口，配置了该注解的方法返回值会加入缓存，也可以对类使用，对类中所有的方法都有效。
    * cacheNames value：指定缓存存储的集合名
    * cacheManager：用于指定使用哪个缓存管理器，非必需。只有当有多个时才需要使用
    * cacheResolver：用于指定使用那个缓存解析器，非必需
    * keyGenerator：用于指定key生成器，非必需。若需要指定一个自定义的key生成器
    * condition：缓存对象的条件，非必需，也需使用SpEL表达式，只有满足表达式条件的内容才会被缓存
    * key：缓存的key值，需要使用SpEL表达式配置，非必需，缺省按照函数的所有参数组合作为key值。
* @CacheConfig: 类级别缓存，可以设置本类中所有方法的cacheNames, cacheManager, cacheResolver, keyGenerator
* @CacheEvict: 删除缓存，配置在函数上，用来从缓存中移除相应数据。除了同@Cacheable一样的参数之外，它还有下面两个参数：
    * allEntries：非必需，默认为false。当为true时，会移除所有数据；
    * beforeInvocation：非必需，默认为false，会在调用方法之后移除数据。当为true时，会在调用方法之前移除数据。
* @CachePut: 配置于函数上，能够根据参数定义条件来进行缓存，其缓存的是方法的返回值，它与@Cacheable不同的是，它每次都会真实调用函数，所以主要用于数据新增和修改操作上。它的参数与@Cacheable类似