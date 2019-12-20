一个计时类，用于计算程序运行的时间。有两个StopWatch类，一个在org.springframework.util包中，一个在rg.apache.commons.lang3.time包中，具体方法也有所不同。commons包：
* start()：开始计时
* stop()：结束计时
* resume()：恢复计时
* suspend()：暂停计时
* getTime()：获取运行的时间
* split()：设置分隔点
* getSplitTime()：获取从分割点到现在运行的时间
* reset()：重设时间

springframework中：
* start()：开始计时
* stop()：结束计时
* getTotalTimeMillis()：获取运行的毫秒数
* getTotalTimeSeconds()：获取运行的秒数


