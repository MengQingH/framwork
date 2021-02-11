## 简介
RestTemplate是Spring提供的用于**访问Rest服务的客户端**，提供了多种便捷访问远程Http服务的方法。RestTemplate 采用同步方式执行 HTTP 请求的类，底层使用 JDK 原生 HttpURLConnection API ，或者 HttpComponents等其他 HTTP 客户端请求类库。还有一处强调的就是 RestTemplate 提供模板化的方法让开发者能更简单地发送 HTTP 请求。

RestTemplate 类是在 Spring Framework 3.0 开始引入的。而在 5.0 以上，官方标注了更推荐使用**非阻塞的响应式 HTTP 请求处理类** org.springframework.web.reactive.client.**WebClient** 来替代 RestTemplate，尤其是对应异步请求处理的场景上 。

## 基本接口
1. **GET请求**：
    * public \<T> T **getForObject**()：通过 GET 请求**获得响应结果**。
    * public \<T> ResponseEntity\<T> **getForEntity**()：通过 GET 请求**获取 ResponseEntity 对象，包含状态码，响应头和响应数据**
    * 一个直接获取响应结果，一个将对象包装到ResponseEntity封装类中。如果只关心结果，使用getForObject即可，如果关心除结果之外的其他内容，如返回的Header等信息，可以使用getForEntity。

2. **Post请求**：
    * public \<T> T **postForObject**()：通过 POST 请求**创建资源，获得响应结果**
    * public \<T> ResponseEntity\<T> **postForEntity**()：通过 POST 请求**获取 ResponseEntity 对象，包含状态码，响应头和响应数据**
    * public URI **postForLocation**()：**POST 数据到一个URL，返回新创建资源的URL**

3. public HttpHeaders **headForHeaders**()：以 **HEAD** 请求资源**返回所有响应头信息**

4. public void **put**()：通过 PUT 方式请求来**创建或者更新资源**

5. public void **delete**()：通过 DELETE 方式**删除资源**

6. public\<T> T **patchForObject**()：通过 PATH 方式请求来**更新资源，并获得响应结果**。(JDK HttpURLConnection 不支持 PATH 方式请求，其他 HTTP 客户端库支持)


7. public Set\<HttpMethod> **optionsForAllow**()：通过 ALLOW 方式请求来**获得资源所允许访问的所有 HTTP 方法**，可用看某个请求支持哪些请求方式

8. public \<T> ResponseEntity\<T> **exchange**()：更通用版本的请求处理方法，接受一个 RequestEntity 对象，可以设置路径，请求头，请求信息等，最后返回一个 ResponseEntity 实体

9. **execute()**：最通用的执行 HTTP 请求的方法，上面所有方法都是基于 execute 的封装，全面控制请求信息，并通过回调接口获得响应数据


## GET请求的使用
下面GET请求getForObject的几种实现，getForEntity方法的实现也类似。
```java
//第三个参数为Object
@Nullable
public <T> T getForObject(String url, Class<T> responseType, Object... uriVariables) throws RestClientException ;

//第三个参数为Map
@Nullable
public <T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException ;

@Nullable
public <T> T getForObject(URI url, Class<T> responseType) throws RestClientException;

```
使用方法一时，传参的方式是**通过占位符{?}来传递参数**，按照实际的传参顺序填充。
使用方法二时，模板中**使用 {key}**, 而这个key，对应的就是map中的key。
使用示例：
```java
 @Test
public void testGet() {
    // 使用方法一，不带参数
    String url = "https://story.hhui.top/detail?id=666106231640";
    InnerRes res = restTemplate.getForObject(url, InnerRes.class);
    System.out.println(res);


    // 使用方法一，传参替换
    url = "https://story.hhui.top/detail?id={?}";
    res = restTemplate.getForObject(url, InnerRes.class, "666106231640");
    System.out.println(res);

    // 使用方法二，map传参
    url = "https://story.hhui.top/detail?id={id}";
    Map<String, Object> params = new HashMap<>();
    params.put("id", 666106231640L);
    res = restTemplate.getForObject(url, InnerRes.class, params);
    System.out.println(res);

    // 使用方法三，URI访问
    URI uri = URI.create("https://story.hhui.top/detail?id=666106231640");
    res = restTemplate.getForObject(uri, InnerRes.class);
    System.out.println(res);
}

```

## Post请求的使用
Post请求postForObject()方法的实现，**实现方式和Get方法类似，都有两种传参方式，一种直接传参，一种通过Map传参**。只不过相比Get请求多了一个request参数，而**request参数就是表单中提交的内容，用 MultiValueMap 对象保存表单中的数据**，post请求可以同时使用传参和表单提交两种方式。
```java
//最后的参数为Object
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Object... uriVariables) throws RestClientException ;

//最后的参数为Map
public <T> T postForObject(String url, @Nullable Object request, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException;

public <T> T postForObject(URI url, @Nullable Object request, Class<T> responseType) throws RestClientException ;
```

使用示例：
```java
@Test
public void testPost() {
    String url = "http://localhost:8080/post";
    String email = "test@hhui.top";
    String nick = "一灰灰Blog";

    //使用MultiValueMap对象保存表单中的信息，创建方式如下：
    MultiValueMap<String, String> request = new LinkedMultiValueMap<>();
    request.add("email", email);
    request.add("nick", nick);

    // 使用方法一，只提交表单，不使用uri传参
    ans = restTemplate.postForObject(url, request, String.class);
    System.out.println(ans);

    // 使用方法一，但是结合表单参数和uri参数的方式，其中uri参数的用的占位符填充
    request.clear();
    request.add("email", email);
    ans = restTemplate.postForObject(url + "?nick={?}", request, String.class, nick);
    System.out.println(ans);

    // 使用方法二，结合表单参数和uri参数的方式，其中uri参数的用的Map传参
    Map<String, String> params = new HashMap<>();
    params.put("nick", nick);
    ans = restTemplate.postForObject(url + "?nick={nick}", request, String.class, params);
    System.out.println(ans);

    // 使用方法三
    URI uri = URI.create(url);
    String ans = restTemplate.postForObject(uri, request, String.class);
    System.out.println(ans);
}

```
postForEntity和postForObject类似，只不过返回的内容为ResponseEntity 对象，包含状态码，响应头和响应数据。

postForLocation和这两种的区别，主要是POST 数据到一个URL，返回新创建资源的URL，**方法的返回值为URI对象**
```java
public URI postForLocation(String url, @Nullable Object request, Object... uriVariables)
		throws RestClientException ;

public URI postForLocation(String url, @Nullable Object request, Map<String, ?> uriVariables)
		throws RestClientException ;

public URI postForLocation(URI url, @Nullable Object request) throws RestClientException ;

```

## exchange方法
exchange方法可以使用多种方式发送请求，比如put和delete方法返回值为空，如果要获取响应内容，就可以使用exchange方法发送请求。
参数说明：
1. url: 请求地址；
2. method: 请求类型(如：POST,PUT,DELETE,GET)；
3. requestEntity: 请求实体，封装请求头，请求内容
4. responseType: 响应类型，根据服务接口的返回类型决定
5. uriVariables/Map: url中参数变量值
```java
public <T> ResponseEntity<T> exchange
//最后一个参数也可以是Map<String, ?> uriVariables
//Class<T> responseType也可以为ParameterizedTypeReference<T> responseType
(String url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType, Object... uriVariables) 

public <T> ResponseEntity<T> exchange(URI url, HttpMethod method, HttpEntity<?> requestEntity, Class<T> responseType)

```