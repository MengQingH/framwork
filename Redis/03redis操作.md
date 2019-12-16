## 存储字符串
* set key value：向redis中存储键值对，如果key存在，就覆盖原来的值。总是返回OK。
* get key：获取key的值，只能用来获取String类型的value，如果不存在，返回value。如果key对应的value不是String类型，redis将返回错误信息。
* getset key value:返回key对应的值，并重新设置该key的值为value。
* incr key：
* decr key：
* incrby key increment：
* decrby key decrement：
* append key value：