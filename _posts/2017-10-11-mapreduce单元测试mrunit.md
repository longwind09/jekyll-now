---
layout: post
title:  mrunit 使用遇到的问题
---

在使用mapreduce的过程中，一直希望能有个工具能够在本地单机测试，从而避免反复提交到服务器上跑结果。

也常常羡慕hadoop-streaming 的方式，通过命令行字符流的方式就能测试，但是hadoop-streaming也有一些不方便的地方，比如不是jar包，不方便在其他地方引用。复杂的一点的任务，代码难以管理。

如何在java mapreduce 里进行单元测试，之前的路子无非是分离mapreduce流程和业务逻辑代码，然后单元测试里只测试逻辑代码而不用关心mapreduce流程部分。当然，这个分离操作是需要一番设计的。

现在好了，我们有了mrunit，看名字就知道是mapreduce的单元测试，[官网地址](http://mrunit.apache.org/)。遗憾的是该软件与2016年4月30日退休了，官网红色背景鲜艳的大字“2016/04/30 - APACHE MRUNIT HAS BEEN RETIRED.
For more information, please explore the Attic.”, 不过，目前仍然可以使用。

最新版本
>
	<dependency>
            <groupId>org.apache.mrunit</groupId>
            <artifactId>mrunit</artifactId>
            <version>1.1.0</version>
            <classifier>hadoop2</classifier>
	</dependency>

需要用到其中的三种driver，MapDriver，ReduceDriver，MapReduceDriver，其实第三种不太好用
```java
    private MapDriver<LongWritable, Text, Text, Text> mapDriver;
    private ReduceDriver<Text, Text, NullWritable, Text> reduceDriver;
    private MultipleInputsMapReduceDriver<LongWritable, Text, NullWritable, Text> mapReduceDriver;

    @Before
    public void setUp() {
        //测试mapreduce
        SampleJoinner.LeftOutJoinMapper mapper = new SampleJoinner.LeftOutJoinMapper();
        SampleJoinner.LeftOutJoinReducer reducer = new SampleJoinner.LeftOutJoinReducer();
        mapDriver = MapDriver.newMapDriver(mapper);
        reduceDriver = ReduceDriver.newReduceDriver(reducer);
        // 这个要求参数是mapred下面的mapper和reducer，现在我们用的都是新的mapreduce下面的
//        mapReduceDriver = MultipleInputsMapReduceDriver.newMultipleInputMapReduceDriver(mapper, reducer);
    }

    @Test
    public void testMapper_shortsample() throws Exception {
        mapDriver.withMapInputPath(new Path(mapInput_path_shortsample));
        mapDriver.withInput(new LongWritable(), new Text(mapInput_shortsample));
        mapDriver.withOutput((new Pair<>(new Text(mapInput_key_1), new Text(mapOutput_shortsample))));
        mapDriver.runTest();
//
    }


    @Test
    public void testReducer() throws Exception {
        List<Pair<Text, List<Text>>> allInput = new ArrayList<>();

        List<Text> list = new ArrayList<>();
        list.add(new Text(mapOutput_shortsample));
        list.add(new Text(mapOutput_doc));
        allInput.add(new Pair<>(new Text(mapInput_key_1), list));

        List<Text> list2 = new ArrayList<Text>();
        list2.add(new Text(mapOutput_shortsample));
        allInput.add(new Pair<>(new Text(mapInput_key_2), list2));

        List<Pair<NullWritable, Text>> outputRecords = new ArrayList<>();
        outputRecords.add(new Pair<>(NullWritable.get(), new Text(reduceOutput_1)));
//        outputRecords.add(new Pair<>(NullWritable.get(), new Text(reduceOutput_1)));

        reduceDriver.withAll(allInput);
        reduceDriver.withAllOutput(outputRecords);
        reduceDriver.runTest();
    }
```

上面的代码，基本上可以测试一个简单的mapreduce程序了

