## 常用的json包：
* com.alibaba.fastjson: 
    * JSONObject: 可以直接进行put get操作，可以使用getXXX(key)取出指定类型的值。
    * JSONArray:可以直接get，可以使用getXXX(index)取出指定类型的值， JSONArray可以使用stream；可以直接进行add set操作，前者直接添加，后者添加并返回

* cn.hutool.json:
    * JSONObject: 可以直接进行put get操作，可以使用getXXX(key)取出指定类型的值。
    * JSONArray: 可以直接get，可以使用getXXX(index)取出指定类型的值，JSONArray可以使用stream；添加方法区别：
        * put(index, Obj)：在index上添加Obj，如果有index上有元素，则插入，之前的元素继续向后排；如果大于最大长度，那么之前的值设置为null。
        * add(index, Obj): 功能和put相同，区别是put返回当前的JSONArray，add返回void。
        * set(index, Obj)：覆盖index位置的值，如果index超出长度，则报错。

* com.fasterxml.jackson:
    * JsonNode ArrayNode: 只读，无法写入。如果要写入需要转为ObjectNode。
    * ObjectNode：可以写入，无法使用stream


## JSONUtil
JSONUtil是hutool包下的工具类。
```java
//创建JSONObject和JSONArray
JSONObject object = JSONUtil.createObj();
JSONArray array = JSONUtil.createArray();

//从json字符串转到JSONObject和JSONArray
JSONObject object = JSONUtil.parseObj(json);
JSONArray jsonArray = JSONUtil.parseArray(json);

//从jsonObject中取值
Integer intKey = jsonObject.getInt("intKey");
String str = jsonObject.getStr("intKey", "defaultValue");

//存值
jsonObject.put("key", "value");
jsonObject.putAll(map);

//对象转为json字符串
String s = JSONUtil.toJsonStr(object);
```


## JsonNode
jackson包中使用的Json类。和json字符串、实体类进行转化时，需要使用到ObjectMapper对象。JsonNode对象不可变，只能get，不能进行put操作。
```java
//创建ObjectMapper对象
ObjectMapper mapper = new ObjectMapper();

//Json字符串  转换  JsonNode对象
String json = "{\"id\":\"1\",\"name\":\"mqh\",\"age\":20}";
JsonNode jsonNode = mapper.readTree(json);
json = mapper.writeValueAsString(jsonNode);

//转换JsonNode对象为实体类，需要实体类中有set方法
//如果json中有额外的属性，可以在实体类上添加@JsonIgnoreProperties(ignoreUnknown = true)注解忽略其他的属性
User user = mapper.readValue(json, User.class);
User user = mapper.readValue(jsonNode.toString(), User.class);

//获取JsonNode的属性，返回值为一个JsonNode
JsonNode node1 = jsonNode.get("id");
JsonNode node2 = jsonNode.path("id");
JsonNode node3 = jsonNode.findPath("id");
JsonNode node4 = jsonNode.findValue("id");
//如果获取的对象为String int等基本类型，可以直接转换
int id = jsonNode.get("id").asInt();
String name = jsonNode.get("name").asText();

//获取所有的key值
Iterator<String> keys = jsonNode.fieldNames();  
  while(keys.hasNext()){  
     String key = keys.next();  
     System.out.println("key键是:"+key);  
}

//如果是一个JsonNode数组，使用jsonNode.elements()读取数组中每个node。如果不是JsonNode数组，使用jsonNode.elements()返回jsonNode的values
Iterator<JsonNode> elements = jsonNode.elements();  
 while(elements.hasNext()){  
     JsonNode node = elements.next();  
     System.out.println(node.toString());  
 }

//获取key为"id"的所有JsonNode
List<JsonNode> findKeys = jsonNode.findParents("id");  
for (JsonNode result:findKeys){  
    System.err.println(result.toString());  
}

//取出所有key为"id"对应的value(如果value中包含子jsonNode并且子jsonNode的key值也为number，是无法捕获到并加入list的)
List<JsonNode> findValues = jsonNode.findValues("id");  
for(JsonNode value: findValues){  
System.out.println( value.toString());  
}
```

## ObjectNode
ObjectNode是JsonNode的子类，JsonNode对象不可变，如果要更改JsonNode对象中的内容，可以使用ObjectNode。
```java
//创建ObjectNode对象也需要ObjectMapper对象
ObjectMapper objectMapper = new ObjectMapper();
ObjectNode objectNode = objectMapper.createObjectNode();

//JsonNode对象可以强转为ObjectNode对象，然后对属性进行操作
objectNode = (ObjectNode) jsonNode;
objectNode.put("key", value);
//如果要添加jsonNode对象，需要使用set
objectNode.set("jsonNode", jsonNode);

```