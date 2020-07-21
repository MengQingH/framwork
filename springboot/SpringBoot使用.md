## 依赖  spring-boot-starter
SpringBoot提供了很多开箱即用的**依赖模块**，都是以spring-boot-starter-xx来命名的，如果要使用这些模块，只需要在pom.xml文件中声明即可。

在pom.xml文件中，默认包含两个模块，spring-boot-starter和spring-boot-starter-test：
* spring-boot-starter：**核心模块**，包括自动配置支持、日志和 YAML，如果引入了 spring-boot-starter-web web 模块可以去掉此配置，因为 spring-boot-starter-web 自动依赖了 spring-boot-starter。
* spring-boot-starter-test：**测试模块**，包括 JUnit、Hamcrest、Mockito。


## Springboot建议的目录结构
```
com
  +- example
    +- myproject
      +- Application.java
      |
      +- model
      |  +- Customer.java
      |  +- CustomerRepository.java
      |
      +- service
      |  +- CustomerService.java
      |
      +- controller
      |  +- CustomerController.java
      |
```
其中：
1. Application.java 建议放到根目录下面,主要用于做一些框架配置
2. model 目录主要用于实体与数据访问层（Repository）
3. service 层主要是业务类代码
4. controller 负责页面访问控制

## Json接口开发
Springboot中使用json接口开发比较简单，只需要在controller上配置@RestController注解，该controller中的方法默认都会以json的格式返回。

## 自定义Filter
1. 实现 Filter 接口，实现 Filter 方法
2. 添加@Configuration 注解，将自定义Filter加入过滤链
```java
@Configuration
public class WebConfiguration {

    public class MyFilter implements Filter {
        @Override
        public void init(FilterConfig filterConfig) throws ServletException {}

        @Override
        public void destroy() {}

        @Override
        public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            System.out.println("this is MyFilter,url :" + request.getRequestURI());
            filterChain.doFilter(servletRequest, servletResponse);
        }
    }
}
```

## 自定义Property
1. 在application.properties中配置属性名和属性值
```
com.mqh.name = mengqh
com.mqh.age = 22
```
2. 使用@Value注解为属性设置值。
```java
@Component
public class NeoProperties {
	@Value("${com.mqh.name}")
	private String name;
	@Value("${com.mqh.age}")
	private int age;
}
```

## log配置
配置输出的位置和输出级别：
```
logging.path=/user/local/log
logging.level.com.favorites=DEBUG
logging.level.org.springframework.web=INFO
logging.level.org.hibernate=ERROR
```
path 为本机的日志位置，logging.level 后面可以根据包路径配置不同资源的 log 级别
