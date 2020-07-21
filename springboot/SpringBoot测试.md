## 单元测试
1. 在pom文件中导入测试相关的包
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-test</artifactId>
	<scope>test</scope>
</dependency>

<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```
2. 在test/java/com/mqh/demo目录下创建测试类，在测试类的类头部添加：**@RunWith(SpringRunner.class)（需要在pom文件中导入junit包）**和**@SpringBootTest**注解，在测试方法的顶端添加@Test注解，最后点击方法上的run运行。如果需要对mapper或者service类中的内容进行测试，可以在类中创建对象并使用@Autowired注解，在方法中使用该对象。
```java
@SpringBootTest
public class TestDemo {

    @Autowired
    private TestMapper testMapper;

    @Test
    public void demo() {
        User user = testMapper.selUserById(1);
        System.out.println(user);
    }

}
```

3. 如果要**对Controller中的内容进行测试**，springboot也引入了**MockMvc**支持了对Controller层的测试。
```java
@SpringBootTest
public class ControllerTestDemo {

    private MockMvc mockMvc;

    @BeforeEach     //该注解表示每次执行方法前都执行这个方法。
    public void setUp() {
        //使用构造器模式创建MockMvc对象，传入要测试的Controller对象
        mockMvc = MockMvcBuilders.standaloneSetup(new TestController()).build();
    }

    @Test
    public void test() throws Exception {
        //使用perform方法测试请求，传入一个requestBuilder对象
        mockMvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
                //测试是否和期望的结果相匹配，andXXX()方法可以链式调用
                .andExpect(MockMvcResultMatchers.status().isOk())
                //测试完之后做什么
                .andDo(MockMvcResultHandlers.print())
                .andReturn();
    }
}
```
代码解析：
1. **perform()**：**执行一个RequestBuilder请求**，自动处理SpringMVC的流程并交给相应的控制器处理。
2. **get()**：**声明一个发送get请求的方法**，**可以链式调用，设置请求的内容**。源码如下，get方法是MockMvcRequestBuilders的一个静态方法，可以直接调用，返回一个MockHttpServletRequestBuilder对象。另外也有post、delete等其他请求。MockMvcRequestBuilders类是MockHttpServletRequestBuilder类的一个构建器类，类中中可以根据需要的不同创建不同类型的RequestBuilder对象。
3. **andExperct()**： 添加ResultMatcher验证规则,**验证perform执行完成后的结果是否正确**。可以在方法中判断响应码的值、跳转的链接、返回的json等内容。
4. **andDo()**: 添加ResultHandler结果处理器,比如调试打印结果到控制台print()
5. **andReturn()**: 最后返回相应的MvcResult,然后进行自定义验证/进行下一步的异步处理.
```java
    public static MockHttpServletRequestBuilder get(String urlTemplate, Object... uriVars) {
        return new MockHttpServletRequestBuilder(HttpMethod.GET, urlTemplate, uriVars);
    }

    public static MockHttpServletRequestBuilder get(URI uri) {
        return new MockHttpServletRequestBuilder(HttpMethod.GET, uri);
    }

    public static MockHttpServletRequestBuilder post(String urlTemplate, Object... uriVars) {
        return new MockHttpServletRequestBuilder(HttpMethod.POST, urlTemplate, uriVars);
    }

    public static MockHttpServletRequestBuilder post(URI uri) {
        return new MockHttpServletRequestBuilder(HttpMethod.POST, uri);
    }
```
测试返回的结果如下所示，可以很方便的查看请求和相应的内容。
```
MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /hello
       Parameters = {}
          Headers = [Accept:"application/json"]
             Body = <no character encoding set>
    Session Attrs = {}

Handler:
             Type = com.mqh.demo.controller.TestController
           Method = com.mqh.demo.controller.TestController#hello()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = [Content-Type:"application/json", Content-Length:"15"]
     Content type = application/json
             Body = HelloSpringBoot
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

2020-07-21 15:11:14.458  INFO 3976 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'

Process finished with exit code 0

```


## 集成测试
热部署，在pom文件中添加下面的内容：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
</plugins>
</build>
```