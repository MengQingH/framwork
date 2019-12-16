## 存储字符串
* **set key value**：向redis中**存储键值对**，如果key存在，就覆盖原来的值。总是返回OK。
* **get key**：**获取key的值，只能用来获取String类型的value**，如果不存在，返回null。如果key对应的value不是String类型，redis将返回错误信息。
* **getset key value**：**返回key对应的值，并重新设置**该key的值为value。
* **append key value**：在key对应的值后面拼接value

* 数值操作：**如果下面操作的value不能转为整型，返回错误信息**
    * **incr/decr key**：把对应的value**原子性的加/减1，并返回**
    * **incrby/decrby key increment**：把对应的value**原子性的减increment，并返回**

## 存储list类型
* **lpush/rpush key value1 [value2 ...]**：在key对应的list**头部/尾部**插入所有的values，如果key不存在，则创建一个空list再插入。插入成功返回元素的个数。
* **lpushx/rpushx key value**：当key对应的list存在时，在list的**头部/尾部**插入值，返回list中元素的个数。不存在返回0。
* **lpop/rpop key**：返回并弹出key对应的list中的**头部/尾部**元素
* **lindex key index**：获取list中索引为index的元素，不存在返回null
* **lset key index value**：通过索引设置list中的值
* **llen key**：返回key对应的list中元素的个数

## 存储set类型
* **sadd key value1 [value2 ...]**：向set中添加元素，如果有这个值不会重复添加，返回添加的个数
* **srem key value1 [value2 ...]**：删除set中的成员，返回删除的元素个数
* **smembers key**：返回set中所有的元素
* **sismember key member**：判断该值是否存在于set中，1表示存在，0表示不存在或set不存在。
* **sdiff/sinter/sunion key1 key2**：返回两个set的差集/交集/并集
* **scart key**：返回set的成员数量

## 存储hash
hash时一个string类型的field和value的映射表。
* hset key field value：为指定的hash设置field/value对
* hmset key field1 value1 [field2 value2...]：为指定的hash设置多个field/value对，插入成功返回OK
* hget key field：返回hash中field的值
* hmget key field1 [field2 ...]：获取key中的多个field值
* hkeys key：获取所有的key
* hvals key：获取所有的value
* hgetall key：获取key中所有的键值对
* hdel key field1 [field2 ...]：删除一个或多个字段，返回删除的个数
* hlen key：获取key中包含的filed的数量