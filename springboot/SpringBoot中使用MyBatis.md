SpringBoot整合MyBatis主要有两种方案，一种是使用注解解决一切问题，另一种是简化后的配置文件。

## 无配置，纯注解版
1. 添加相关的maven文件
```xml
<dependencies>
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.0.0</version>
	</dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```
2. 在application中添加配置
```xml
mybatis.type-aliases-package=com.neo.model

spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```
SpringBoot 会自动加载 spring.datasource.* 相关配置，数据源就会自动注入到 sqlSessionFactory 中，sqlSessionFactory 会自动注入到 Mapper 中，一切都不用管。
3. 在启动类中对mapper包进行扫描
```java
@SpringBootApplication
@MapperScan("com.mqh.mapper")
public class MybatisAnnotationApplication {

	public static void main(String[] args) {
		SpringApplication.run(MybatisAnnotationApplication.class, args);
	}
}
```
或者直接在 Mapper 类上面添加注解@Mapper
4. 开发mapper，需要用到MyBatis中的增删改查注解和@Result注解。
```java
public interface UserMapper(){

    @Select("select * from user where id = #{id}")
    User getUserById();
}
```


## XML版本
1. 配置，pom文件配置和上一种相同，在application.properties文件中增加下面的配置：
```xml
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
```
指定了MyBatis配置文件和实体类映射文件的地址。

2. 创建mapper文件的映射

3. 创建mapper接口。