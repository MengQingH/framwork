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
3. 此时在应用中已经可以使用redis缓存，使用官方提供的类和方法来进行缓存，也可以自行重写redis的配置，自己决定缓存的方式。redis中的缓存方式：
* GenericToStringSerializer：可以将任何对象泛化为字符创并序列化
* Jackson2JsonRedisSerializer：序列化Object对象为json字符创（与JacksonJsonRedisSerializer相同），被序列化对象不需要实现Serializable接口，被序列化的结果清晰，容易阅读，而且存储字节少，速度快
* JdkSerializationRedisSerializer：序列化java对象，被序列化对象必须实现Serializable接口，被序列化除属性内容还有其他内容，长度长且不易阅读
* StringRedisSerializer：简单的字符串序列化