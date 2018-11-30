---
layout:     post
title:      FLink DataStream
subtitle:   在Java中使用flink流处理
date:       2018-11-20
author:     zhanghaichao
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - Java
    - flink
    - 流处理
---

#### 数据源（Data Sources）
- StreamExecutionEnvironment.addSource(sourceFunction)来创建数据源。
- 通过实现SourceFunction来编写自定义的非并行数据源
- 通过实现ParallelSourceFunction接口或继承RichParallelSourceFunction来编写自定义并行数据源。

1. 基于文件
readTextFile(path) // TextInputFormat - 以行读取方式读文件并返回字符串
readFile(path) // 任意输入格式 - 按用输入格式的描述读取文件

2. 基于Socket
socketTextStream // 从socket读取，element可以通过分割符来分开

3. 基于Collection
fromCollection(Collection) - 从Java.util.Collection创建一个数据流。collection中所有的element都必须是同一类型的。
fromCollection(Iterator, Class) - 从一个迭代器中创建一个数据流。class参数明确了迭代器返回的element的类型。
fromElement(T …) - 从一个给定的对象序列创建一个数据流。所有对象都必须是同一类型的。
fromParallelCollection(SplittableIterator, Class) - 从一个迭代器中创建一个并行数据流。class参数明确了迭代器返回的element的类型。
generateSequence(from, to) - 从一个给定区间中生成一个并行数字序列。   

```
// Create a DataStream from a list of elements
DataStream<Integer> myInts = env.fromElements(1, 2, 3, 4, 5);

// Create a DataStream from any Java collection
List<Tuple2<String, Integer>> data = ...
DataStream<Tuple2<String, Integer>> myTuples = env.fromCollection(data);

// Create a DataStream from an Iterator
Iterator<Long> longIt = ...
DataStream<Long> myLongs = env.fromCollection(longIt, Long.class);
```
注意：当前Collection数据源需要实现Serializable接口的数据类型和迭代器。此外，Collection数据源无法并行执行（并行度=1）

4. 自定义：
addSource - 附上一个新的source方法。例如，通过调用addSource(new FlinkKafkaConsumer08<>(…))来从Apache Kafka读取数据

#### Data Sink
Data Sink消耗DataStream并将它们转发到文件、socket、外部系统或打印它们。
Flink自带了许多内置的输出格式，封装为DataStream的operation中：

1. writeAsText() / TextOutputFormat 
以行字符串的方式写文件，字符串通过调用每个element的toString()方法获得。

2. writeAsCsv(…) / CsvOutputFormat 
以逗号分隔的值来把Tuple写入文件，行和域的分隔符是可以配置的。每个域的值是通过调用object的toString()方法获得的。

3. print() / printToErr() 
将每个element的toString()值打印在标准输出 / 标准错误流中。可以提供一个前缀（msg）作为输出的前缀，使得在不同print的调用可以互相区分。如果并行度大于1，输出也会以task的标识符（identifier）为产生的输出的前缀。

4. writeUsingOutputFormat() / FileOutputFormat 
自定义文件输出所用的方法和基类，支持自定义object到byte的转换。

5. writeToSocket 
依据SerializationSchema将element写到socket中。

6. addSink 
调用自定义sink方法，Flink自带连接到其他系统的connector（如Apache Kafka），这些connector都以sink方法的形式实现。
 
7.迭代器 data sink
```
Iterator< Integer> myOutput = DataStreamUtils.collect(stream);
while (myOutput.hasNext()) {
    System.out.println(myOutput.next()); 
}
```
####  DataStream的Transformation操作
1. Map:
DataStream → DataStream: 输入一个参数产生一个参数
数据翻倍等操作: `dataStream.map { x => x * 2 }`

2. FlatMap
DataStream → DataStream: 输入一个参数，产生0个、1个或者多个输出. 
这个 flatmap 的功能是将句子中的单词拆分出来: `dataStream.flatMap { str => str.split(" ") }`

3. Filter
DataStream → DataStream: 结算每个元素的布尔值，并返回布尔值为true的元素. 
过滤出非0的元素: `dataStream.filter { _ != 0 }`

4. KeyBy
DataStream → KeyedStream: 逻辑地将一个流拆分成不相交的分区，每个分区包含具有相同key的元素. 在内部是以hash的形式实现的.
数组和POJO不可以作为key
```
dataStream.keyBy("someKey") // 通过 "someKey"进行分组
dataStream.keyBy(0) // 通过Tuple的第一个元素进行分组
dataStream.keyBy(new KeySelector())
```

5.  Reduce
KeyedStream → DataStream: 一个分组数据流的滚动规约操作. 合并当前的元素和上次规约的结果，产生一个新的值.
创建部分流的和: `keyedStream.reduce { _ + _ }`

6. Fold
KeyedStream → DataStream: 一个有初始值的分组数据流的滚动折叠操作. 合并当前元素和前一次折叠操作的结果，并产生一个新的值.
下面的fold函数就是当我们输入一个 (1,2,3,4,5)的序列, 将会产生一下面的句子:"start-1", "start-1-2", "start-1-2-3", ...
```
 val result: DataStream[String] =
 keyedStream.fold("start")((str, i) => { str + "-" + i })
```

7. Aggregations
KeyedStream → DataStream: 分组数据流上的滚动聚合操作. min和minBy的区别是min返回的是一个最小值，而minBy返回的是其字段中包含最小值的元素(同样原理适用于max和maxBy)
```
keyedStream.sum(0)
keyedStream.sum("key")
keyedStream.min(0)
keyedStream.min("key")
keyedStream.max(0)
keyedStream.max("key")
keyedStream.minBy(0)
keyedStream.minBy("key")
keyedStream.maxBy(0)
keyedStream.maxBy("key")
```

8. Window
KeyedStream → WindowedStream: Windows 是在一个分区的 KeyedStreams中定义的. Windows 根据某些特性将每个key的数据进行分组 
一段时间内到达的数据，并且按照tupple的第一个元素聚合：`dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5))) `
`dataStream.keyBy(0)..timeWindow(Time.minutes(2L))`

9. WindowAll
DataStream → AllWindowedStream:这个操作在许多情况下并非并行操作. 所有的记录都会聚集到一个windowAll操作的任务中。
一段时间内到达的所有数据： `dataStream.windowAll(TumblingEventTimeWindows.of(Time.seconds(5)))`

10. Window Apply
WindowedStream → DataStream、AllWindowedStream → DataStream 将一个通用函数作为一个整体传给window. 
`windowedStream.apply { WindowFunction }`

11. Window Reduce
WindowedStream → DataStream 给window赋一个reduce功能的函数，并返回一个规约的结果.参考5

12. Window Fold
WindowedStream → DataStream 给窗口赋一个fold功能的函数，并返回一个fold后的结果. 参考6

13. Aggregations on windows
WindowedStream → DataStream 对window的元素做聚合操作.参考7

14. Union
DataStream → DataStream 对两个或者两个以上的DataStream进行union操作，产生一个包含所有DataStream元素的新DataStream.***在2个DataStream中指定连接的key以及window下来运算。***
注意:如果你将一个DataStream跟它自己做union操作，在新的DataStream中，你将看到每一个元素都出现两次.
`dataStream.union(otherStream1, otherStream2, ...)`

15. Window Join
DataStream,DataStream → DataStream **根据一个给定的key和window**对两个DataStream做join操作.筛选两个stream中符合指定条件等数据。
```
dataStream.join(otherStream)
    .where(<key selector>).equalTo(<key selector>)
    .window(TumblingEventTimeWindows.of(Time.seconds(3)))
    .apply { ... }
```

16. Window CoGroup
DataStream,DataStream → DataStream 根据一个给定的key和window对两个DataStream做Cogroups操作.
```
dataStream.coGroup(otherStream)
.where(0).equalTo(1)
.window(TumblingEventTimeWindows.of(Time.seconds(3)))
.apply (new CoGroupFunction () {...});
```

[join可coGroup对比](https://blog.csdn.net/lmalds/article/details/51743038)
apply方法中的参数类型不一样，join中提供的apply方法，参数是T1与T2这2种数据类型，对应到SQL中就是T1.* 与 T2.*的一行数据。而coGroup中提供的apply方法，参数是Iterator[T1]与Iterator[2]这2种集合，对应SQL中类似于Table[T1]与Table[T2]。

基于这2种方式，如果我们的逻辑不仅仅是对一条record做处理，而是要与上一record或更复杂的判断与比较，甚至是对结果排序，那么join中的apply方法显得比较困难。 

17. Connect
DataStream,DataStream → ConnectedStreams: 连接两个保持他们类型的数据流.  

```
DataStream<Integer> someStream = //...
DataStream<String> otherStream = //...

ConnectedStreams<Integer, String> connectedStreams = someStream.connect(otherStream);
```

CoMap, CoFlatMap
ConnectedStreams → DataStream 作用于connected 数据流上，功能与map和flatMap一样  

```
connectedStreams.map(new CoMapFunction<Integer, String, Boolean>() {
　　@Override
　　public Boolean map1(Integer value) {
　　　　return true;
　　}

　　@Override
　　public Boolean map2(String value) {
　　　　return false;
　　}
});

 

connectedStreams.flatMap(new CoFlatMapFunction<Integer, String, String>() {

　　@Override
　　public void flatMap1(Integer value, Collector<String> out) {
　　　　out.collect(value.toString());
　　}

　　@Override
　　public void flatMap2(String value, Collector<String> out) {
　　　　for (String word: value.split(" ")) {
　　　　　　out.collect(word);
　　　　}
　　}
});
```

18. Split
DataStream → SplitStream: 根据某些特征把一个DataStream拆分成两个或者多个DataStream.  

```
SplitStream<Integer> split = someDataStream.split(new OutputSelector<Integer>() {
　　@Override
　　public Iterable<String> select(Integer value) {
　　　　List<String> output = new ArrayList<String>();
　　　　if (value % 2 == 0) {
　　　　　　output.add("even");
　　　　}
　　　　else {
　　　　　　output.add("odd");
　　　　}
　　　　return output;
　　}
});

``` 

Select
SplitStream -> DataStream
从SplitStream中选择1个或多个stream  

``` 
SplitStream<Integer> split;
DataStream<Integer> even = split.select("even");
DataStream<Integer> odd = split.select("odd");
DataStream<Integer> all = split.select("even","odd");
```

19. Iterate
DataStream -> IterativeStream -> DataStream
通过将一个Operator的输出重定向到前面的某个Operator的方法，在数据流图中创建一个“反馈”循环
大于0的element被送回到反馈通道，而其他的element则被转发到下游  

```
IterativeStream<Long> iteration = initialStream.iterate();
DataStream<Long> iterationBody = iteration.map (/*do something*/);
DataStream<Long> feedback = iterationBody.filter(new FilterFunction<Long>(){
　　@Override
　　public boolean filter(Integer value) throws Exception {
　　　　return value > 0;
　　}
});
iteration.closeWith(feedback);
DataStream<Long> output = iterationBody.filter(new FilterFunction<Long>(){
　　@Override
　　public boolean filter(Integer value) throws Exception {
　　　　return value <= 0;
　　}
});
```

一个持续将整数序列中的数字减1知道它们变为0的程序：
```
DataStream<Long> someIntegers = env.generateSequence(0, 1000);

IterativeStream<Long> iteration = someIntegers.iterate();

DataStream<Long> minusOne = iteration.map(new MapFunction<Long, Long>() {
　　@Override
　　public Long map(Long value) throws Exception {
　　　　return value - 1 ;
　　}
});

DataStream<Long> stillGreaterThanZero = minusOne.filter(new FilterFunction<Long>() {
　　@Override
　　public boolean filter(Long value) throws Exception {
　　　　return (value > 0);
　　}
});

iteration.closeWith(stillGreaterThanZero);

DataStream<Long> lessThanZero = minusOne.filter(new FilterFunction<Long>() {
　　@Override
　　public boolean filter(Long value) throws Exception {
　　　　return (value <= 0);
　　}
});
```

20. Extract Timestamps
DataStream → DataStream 提取记录中的时间戳来跟需要事件时间的window一起发挥作用.

```
stream.assignTimestamps (new TimeStampExtractor() {...});
```

21. Project
DataStream -> DataStream  从tuple中选择出域的子集而产生新的DataStream

```
DataStream<Tuple3<Integer, Double, String>> in = // [...]
DataStream<Tuple2<String, Integer>> out = in.project(2,0);
```
#### 执行参数
StreamExecutionEnvironment包含ExecutionConfig，它可以使用户设置job的确切运行时配置值。
请参考https://ci.apache.org/projects/flink/flink-docs-release-1.0/apis/common/index.html#execution-configuration

以下这些参数仅适用于DataStream API：
1. enableTimestamps() / disableTimestamps()：在每一个source发出的事件上附加上一个时间戳。函数areTimestampsEnabled()可以返回该状态的当前值。

2. setAutoWatermarkInterval(long milliseconds)：设置自动水印发布（watermark emission）区间。你可以通过调用函数getAutoWatermarkInterval()来获取当前值。

#### 容错
https://doc.flink-china.org/doc/ops/state/checkpoints.html

 
#### 物理级别分割
#### 链接任务以及资源组（Task chaining & resource groups）
https://www.cnblogs.com/lanyun0520/p/5730403.html
