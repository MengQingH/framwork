<!--
 * @Author: QingHui Meng
 * @Date: 2021-06-30 11:03:40
-->
## 在Spring中使用Junit进行单元测试
1. maven配置
    ```xml
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
        <version>4.3.12</version>
        <scope>test</scope>
    </dependency>
    ```
2. 创建基类，后面创建的测试类都可以继承基类
    ```java
    import org.junit.runner.RunWith;
    import org.springframework.test.context.ContextConfiguration;
    import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
    import org.springframework.test.context.web.WebAppConfiguration;

    // 使用junit进行测试
    @RunWith(SpringJUnit4ClassRunner.class) 
    // 设置web项目的环境，必须配置该属性，否则无法获取web容器相关的信息
    @WebAppConfiguration
    // 加载spring相关的配置文件
    @ContextConfiguration(locations = {
            "classpath:spring/*.xml",
            "classpath:mybatis/sqlMapConfig.xml"}) 
    //------------下面的注解，所有继承该类的测试类都会遵循该配置，也可以选择加在测试类的方法上
    // 对类中的所有方法都添加事务控制
    @Transactional
    // 把事务关联到配置文件中的事务控制器（transactionManager = "transactionManager"），同时指定自动回滚（defaultRollback = true），避免测试的数据污染数据库
    @TransactionConfiguration(transactionManager = "transactionManager", defaultRollback = true)
    //------------
    public class BaseTest {
    }

    ```
3. 创建测试类，继承基类。
    ```java
    public class Kc6wTest extends BaseTest {

        // 用于测试接口
        private MockMvc mockMvc;
        @Autowired
        private TaxRecordService taxRecordService;

        @Before
        public void before(){
            mockMvc = MockMvcBuilders.standaloneSetup(new TaxRecordController()).build();
        }

        @After
        public void after(){
        }

        @Test
        // 表示此方法需使用事务
        @Transactional
        // 表示使用完此方法后事务不回滚,true时为回滚
        @Rollback(false)
        public void test(){
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
相关注解含义(Junit4)：
* @BeforeClass – 表示在类中的任意public static void方法执行之前执行
* @AfterClass – 表示在类中的任意public static void方法执行之后执行
* @Before – 表示在任意使用@Test注解标注的public void方法执行之前执行
* @After – 表示在任意使用@Test注解标注的public void方法执行之后执行
Junit5注解：
* @BeforeAll  @BeforeClass注释的替代。
* @AfterAll  @AfterClass注释的替代。
* @BeforEach  @Before注释的替代。
* @AfterEach  @After注释的替代。

## 使用MockMvc来测试接口请求
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