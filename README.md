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

![MapReduce过程](https://github.com/Shengliannan/bigdata_interview/blob/master/img/MapReduce2.png?raw=true)

![shuffle](https://github.com/Shengliannan/bigdata_interview/blob/master/img/mapreduce%20%20shuffle.png?raw=true)

***Mapper -> Partitioner -> Combiner -> Reducer***

应用程序通常实现`Mapper`和`Reducer`接口提供 `map`和`reduce`方法。


1. mapper

   Mapper将 输入key/value 映射为一组中间 key/value对。

   Hadoop MapReduce框架为作业的`InputFormat`生成的每个`InputSplit`生成一个map任务。 

   对`Mapper`输出进行排序，然后根据`Reducer`进行分区。分区总数与作业的reduce任务数相同。用户可以通过实现自定义`分区程序`来控制哪些键（以及记录）转到哪个`Reducer`。 

   框架为该任务的`InputSplit`中的每个键/值对调用[map（WritableComparable，Writable，Context）](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapreduce/Mapper.html)。 

2. Reducer

   Reduer 将 同一个键的中间值归约成更小的一组值。

   框架为分组输入中的每个`<key，（list of values）>`对调用[reduce（WritableComparable，Iterable ，Context）](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapreduce/Reducer.html)方法。然后，应用程序可以覆盖`清理（Context）`方法以执行任何所需的清理。 

   Reducer包括三个基本阶段:`shuffle`,`sort`and`reduce`。

   - Shuffle

     将Mapper的输出排序后作为Reduer的输入。

   - Sort

     框架通过key来进行分组（不同的Mapper输出可能会有相同的key）

   - Reduce

     Reducer的输出是不排序的。

   **reduceTask的数量和reduce后的文件数相等**

3. shuffle概念&流程

   > [切片和分区](https://www.cnblogs.com/huqiaoblog/p/8064182.html )

   shuffle（官网原话）

   Input to the `Reducer` is the sorted output of the mappers. In this phase the framework fetches the relevant partition of the output of all the mappers, via HTTP.

   定义：MR确保每个reduce的输入都是**按键排序**的。系统执行排序的过程（从map出到reduce输入的这段过程）称为shuffle。

   **shuffle分为map端shuffle和reduce端的shuffle**

   > [MapReduce计算模型](https://blog.csdn.net/u014374284/article/details/49205885 )
   >
   > [MapReduce的shuffle过程详解（分片、分区、合并、归并。。。）](https://blog.csdn.net/ASN_forever/article/details/81233547 )  （写的最好）

   - Map端shuffle

     ①分区partition

     ②写入环形内存缓冲区

     ③执行溢出写

     ​        排序sort--->合并combiner--->生成溢出写文件

     ④归并merge
   - Reduce端shuffle
     ①复制copy
     
     ②归并merge
     
     ③reduce

   ```markdown
   因为频繁的磁盘I/O操作会严重的降低效率，因此“中间结果”不会立马写入磁盘，而是优先存储到map节点的“环形内存缓冲区”，在写入的过程中进行分区（partition），也就是对于每个键值对来说，都增加了一个partition属性值，然后连同键值对一起序列化成字节数组写入到缓冲区（缓冲区采用的就是字节数组，默认大小为100M）。当写入的数据量达到预先设置的阙值后（mapreduce.map.io.sort.spill.percent,默认0.80，或者80%）便会启动溢写出线程将缓冲区中的那部分数据溢出写（spill）到磁盘的临时文件中，并在写入前根据key进行排序（sort）和合并（combine，可选操作）。溢出写过程按轮询方式将缓冲区中的内容写到mapreduce.cluster.local.dir属性指定的目录中。当整个map任务完成溢出写后，会对磁盘中这个map任务产生的所有临时文件（spill文件）进行归并（merge）操作生成最终的正式输出文件，此时的归并是将所有spill文件中的相同partition合并到一起，并对各个partition中的数据再进行一次排序（sort），生成key和对应的value-list，文件归并时，如果溢写文件数量超过参数min.num.spills.for.combine的值（默认为3）时，可以再次进行合并。至此，map端shuffle过程结束，接下来等待reduce task来拉取数据。对于reduce端的shuffle过程来说，reduce task在执行之前的工作就是不断地拉取当前job里每个map task的最终结果，然后对从不同地方拉取过来的数据不断地做merge最后合并成一个分区相同的大文件，然后对这个文件中的键值对按照key进行sort排序，排好序之后紧接着进行分组，分组完成后才将整个文件交给reduce task处理。
   ```

4. Shuffle-Partitioner

    > [MapReduce中partition、shuffle、combiner的作用与关系介绍](https://blog.csdn.net/YYDU_666/article/details/79465073 )

    1) reducer数量与partiton分区数的关系

    ​	**reduceTask的数据量要和分区数相等**

       	reducer数量 > partition数量：和partition不对应的文件为空

       	reducer数量 < partition数量：和reducer不对应的partition产生的数据会没有reducer处理，会抛出异常。

      2）partition

    - 对partition的理解

      partition意思为分开，划分。它分割map每个节点的结果，按照key分别映射给不同的reduce，也是可以自定义的。其实可以理解归类。也可以理解为根据key或value及reduce的数量来决定当前的这对输出数据最终应该交由哪个reduce task处理；partition的作用就是把这些数据归类。每个map任务会针对输出进行分区，及对每一个reduce任务建立一个分区。划分分区由用户定义的partition函数控制，默认使用哈希函数来划分分区。HashPartitioner是mapreduce的默认partitioner。

      计算方法是which reducer=(key.hashCode() & Integer.MAX_VALUE) % numReduceTasks，得到当前的目的reducer。

    - partition过程

      1)，计算(key，value)所属与的分区。

      **当map输出的时候，写入缓存之前，会调用partition函数**，计算出数据所属的分区，并且把这个元数据存储起来。

      2)，把属与同一分区的数据合并在一起。

       当数据达到溢出的条件时(即达到溢出比例，启动线程准备写入文件前)，读取缓存中的数据和分区元数据，然后把属与同一分区的数据合并到一起。

5. Combiner

   数据量大，占用了过多的带宽资源，需要进行二次排序，Combiner

   **编写Combiner继承Reducer**

   Combiner是可选的，如果这个过程适合于你的作业，Combiner实例会在每一个运行map任务的节点上运行。Combiner会接收特定节点上的Mapper实例的输出作为输入，接着Combiner的输出会被发送到Reducer那里，而不是发送Mapper的输出。Combiner是一个“迷你reduce”过程，***它只处理单台机器生成的数据***（特别重要，作者在做一个矩阵乘法的时候，没有领会到这点，把它当成一个完全的reduce的输入数据来处理，结果出错。）

   ***combiner之后才落地到磁盘，再进行map端merger的时候才将map的输出合并。*** 

   - 适用场景

   		适用于累加，求平均值不能用combiner

6. 一个MapReduce的具体业务应用

   ```java
   /*
   * 功能描述：
   *   推荐系统需要根据用户的行为信息进行推荐
   *	1. 通过埋点记录用户的信息和行为保存到日志服务器中，数据组织形式为json，此时为bdm层
   *   2. 从日志服务器拉去日志文件到本地的HDFS上
   *   3. mr程序解析日志：
   *		（1）map
   *			将客户端标记（eg：0代表手机客户端，1代表PC端）、时间戳、ip地址作为key，业务数据（用户
   *			的数据信息）作为value
   *		（2）reduce
   *			可进一步分解析业务数据中的信息，然后将数据写到HDFS上。
   *   4. 解析后的数据为fdm层，此时进行落表处理
   *   5. 如果需要特殊的指标，再通过建模然后加工成gdm层供用户使用
   */
   public void map(LongWritable key, Text value, Context context)
               throws IOException, InterruptedException {
           try {
               /*{"c": "0","rtm":"13位时间戳","ip":"上报ip地址","t":"topic名称", "d":"业务数据"}*/
               JSONObject jsonObject = JSONObject.parseObject(value.toString());
               String c = util.nullToString(jsonObject.getString("c"));
               String rtm = util.nullToString(jsonObject.getString("rtm"));
               String ip = util.nullToString(jsonObject.getString("ip"));
               String t = util.nullToString(jsonObject.getString("t"));
               String d = util.nullToString(jsonObject.getString("d"));
               d = URLDecoder.decode(d);
               StringBuffer outKey = new StringBuffer().append(c).append(tab)
                       .append(rtm).append(tab)
                       .append(ip).append(tab)
                       .append(t);
               context.write(new Text(outKey.toString()), new Text(d));
           } catch (Exception e) {
           
           }
   }
   
   protected void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
       	//values是一个list
           Iterator<Text> iter = values.iterator();
           while(iter.hasNext()){
               Text value = iter.next();
               context.write(key, value);
   }
   ```

7. mapper+combiner+partition+reducer

   ```java
   // mapper
   
   // combiner
   public class WordCountCombiner extends Reducer<Text,LongWritable,Text,LongWritable>{
       @Override
       protected void reduce(Text key,Iterable<LongWritable> values,Context context throws IOException{
           long sum = 0;
           for(LongWritable value : values){
               sum += value;
           }
       }
      	context.write(key,new LongWritable(sum))
   }
   
   // partitioner
   
   // reducer
   ```

   

##### 2.5.2.2 数据倾斜问题



##### 2.5.2.3 优化



##### 2.5.2.4 MR实现join



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

##### 2.5.7.1 Hive语法

```
1. 执行顺序
FROM -> WHERE -> GROUP BY -> HAVING -> SELECT -> DISTINCT -> UNION -> ORDER BY
2. 示例
1)order by可以使用sleclt后的别名，因为先执行selelct后执行order by
求被评分次数最多的10部电影，并给出评分次数（电影名，评分次数）
select a.moviename as moviename,count(a.moviename) as total
from t_movie a join t_rating b on a.movieid=b.movieid
group by a.moviename
order by total desc
limit 10;
思路：先进行分组，count，然后根据count的值排序

2）having后也可以用别名，做了扩展（重要）
女性当中评分最高的10部电影（性别，电影名，影评分）评论次数大于等于50次
（这个语句有问题）
select "F" as sex, c.moviename as name, avg(a.rate) as avgrate, count(c.moviename) as total  
from t_rating a 
join t_user b on a.userid=b.userid 
join t_movie c on a.movieid=c.movieid 
where b.sex="F" 
group by c.moviename 
having total >= 50
order by avgrate desc 
limit 10;



3）where后不能使用select后的别名

gropy by后面没有出现的字段不能再select后出现？(待解决)
4）select后的字段必须在group by后出现，否则是聚合函数
计算emp每个部门中每个岗位的最高薪水
select deptno,job,avg(sal) avg_sal from emp groupy by deptno,job;

5）having后可以使用select后的别名
部门平均薪水大于2000的部门和平均薪水
select deptno,avg(sal) avg_sal from emp group by deptno having avg_sal >2000;



```

sort by

distributed by

order by 区别

***Order by：全局排序，一个Reduce***

distribute by:类似于mr中partition，进行分区，结合sort by使用

***sort by：每个Reduce内部排序，对全局来说不是有序的***

只用sortby没有指定根据哪个字段进行分区，数据会随机分区

distribute by可能根据指定字段进行分区

cluster by

***当distribute by和sort by字段相同时，可以使用cluster by***，

cluster by除了具有distribute by的功能外还兼具sort by的功能，但是排序只能是倒序排序，蹦年指定排序规则为ASC或者DESC。





问题总结一:
oracle、mysql、hive中的字段别名是否可以在where、group by、having、order by中直接使用

Mysql 版本5.7.20

where中不能直接使用字段的别名，group by、having、order by可以直接使用

Oracle 版本12c

 where、group by、having中不能直接使用字段的别名，order by可以直接使用

Hive 版本1.3.0

where、group by、having中不能直接使用字段的别名，order by可以直接使用


窗口函数

90分位实现

mapjoin实现

#### 2.5.8 数据仓库之数据建模

> [深入对比数据仓库模式:Kimball vs Inmon](https://segmentfault.com/a/1190000006255954 )

维度建模，范式建模

Inmon和Kimball建模

| Imon                                 | Kimball                              |
| ------------------------------------ | ------------------------------------ |
| 自顶向下：数据源->数据仓库->数据集市 | 自底向上：数据集市->数据仓库->数据源 |
|                                      |                                      |
|                                      |                                      |
|                                      |                                      |

企业数据仓库：3NF



Inmon

Kimball

Kimball模型将分散异构的数据源经ETL转化为事实表和维度表导入数据集市。数据集市由若干事实表和维度表组成




### 2.6 其他

   阿姆达尔法则

   安迪比尔定律

   