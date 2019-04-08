# bigdata_interview
大数据面试总结

## 一、大数据相关岗位总结

- 数据爬虫
- 大数据开发（平台）
- 数据开发
- 数据挖掘

## 二、知识点总结

### 2.1 Java相关

   #### 2.1.1 JVM

   #### 2.1.2 GC

   #### 2.1.3 多线程

   #### 2.1.4 集合

   ​	ArrayList扩容

   ​	ConcurrentHashMap并发

### 2.2 算法相关

   前k大（堆排）

   前k大（大数据量）（MR归并？赛道比马模型？）

   > 剑指offer
   >
   > 

### 2.3 数据库相关

   左前缀匹配

   存储引擎（ISAM、InnoDB）

   HBASE（LSM树）

   索引的优化与失效

   SQL语句

   > 《高性能MySQL》

### 2.4 网络相关

### 2.5 大数据相关
#### 2.5.1 Hdfs
#### 2.5.2 Hadoop



##### 2.5.2.1 MapReduce
>[Hadoop官网](http://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)

应用程序通常实现`Mapper`和`Reducer`接口提供 `map`和`reduce`方法。


1. mapper

   Mapper将 输入key/value 映射为一组中间 key/value对。

   Hadoop MapReduce框架为作业的`InputFormat`生成的每个`InputSplit`生成一个map任务。 

   对`Mapper`输出进行排序，然后根据`Reducer`进行分区。分区总数与作业的reduce任务数相同。用户可以通过实现自定义`分区程序`来控制哪些键（以及记录）转到哪个`Reducer`。 

2. Reducer

   Reduer 将 同一个键的中间值归约成更小的一组值。

   Reducer包括三个基本阶段:`shuffle`,`sort`and`reduce`。

   - Shuffle

     将Mapper的输出排序后作为Reduer的输入。

   - Sort

     框架通过key来进行分组（不同的Mapper输出可能会有相同的key）

   - Reduce

     Reducer的输出是不排序的。

3. shuffle概念&流程

   shuffle（官网原话）

   Input to the `Reducer` is the sorted output of the mappers. In this phase the framework fetches the relevant partition of the output of all the mappers, via HTTP.

   定义：MR确保每个reduce的输入都是**按键排序**的。系统执行排序的过程（从map出到reduce输入的这段过程）称为shuffle。

   

#### 2.5.4 Spark 

1. RDD懒加载，更新
2. spark是怎么工作的
3. stage如何划分
4. groupbykey，reducebykey有什么区别
5. 混洗是干啥的，如何解决数据倾斜
6. 分区的算法，池塘抽样
7. 优化参数

> 大数据 spark 企业级实战

#### 2.5.5 HBase

1. 设计rowkey，region，定位。
2. wal，cap理论
3. 

#### 2.5.6 Kafka
#### 2.5.7 Hive




### 2.6 其他

   阿姆达尔法则

   安迪比尔定律

   