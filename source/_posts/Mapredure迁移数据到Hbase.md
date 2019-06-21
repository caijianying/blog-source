---
title: Mapredure迁移数据到Hbase
date: 2016-10-24 21:11:23
tags: 大数据
categories:
- hadoop
---
## 目标
1. 将txt数据通过mapredure写到hbase中
2. 将sqlserver数据写入hive表中，从hive表中写入hbase
3. 将sqlserver数据写入hbase
## 实现
### 1.第一个目标

一、上传原数据文件

将data.txt文件上传到到hdfs上，内容如下：
```
key1	col1	value1  
key2	col2	value2  
key3	col3	value3  
key4	col4	value4  
```
数据以制表符（\t）分割。

二、将数据写成HFile

通过mapredure将data.txt按hbase表格式写成hfile

pom.xml文件中依赖如下
```
<!-- hadoop -->
<dependency>
	<groupId>org.apache.hadoop</groupId>
	<artifactId>hadoop-client</artifactId>
	<version>2.6.0-cdh5.4.0</version>
</dependency>
<!-- hbase -->
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-client</artifactId>
	<version>1.0.0-cdh5.4.0</version>
</dependency>
<dependency>
	<groupId>org.apache.hbase</groupId>
	<artifactId>hbase-server</artifactId>
	<version>1.0.0-cdh5.4.0</version>
</dependency>
```

编写BulkLoadMapper
```
public class BulkLoadMapper extends Mapper<LongWritable, Text, ImmutableBytesWritable, Put> {
	private static final Logger logger = LoggerFactory.getLogger(BulkLoadMapper.class);
    private String dataSeperator;
    private String columnFamily1;
    private String columnFamily2;

    public void setup(Context context) {
        Configuration configuration = context.getConfiguration();//获取作业参数
        dataSeperator = configuration.get("data.seperator");
        columnFamily1 = configuration.get("COLUMN_FAMILY_1");
        columnFamily2 = configuration.get("COLUMN_FAMILY_2");
    }

    public void map(LongWritable key, Text value, Context context){
        try {
            String[] values = value.toString().split(dataSeperator);
            ImmutableBytesWritable rowKey = new ImmutableBytesWritable(Bytes.toBytes(values[0]));
            Put put = new Put(Bytes.toBytes(values[0]));
            put.addColumn(Bytes.toBytes(columnFamily1), Bytes.toBytes("words"), Bytes.toBytes(values[1]));
            put.addColumn(Bytes.toBytes(columnFamily2), Bytes.toBytes("sum"), Bytes.toBytes(values[2]));
            
            context.write(rowKey, put);
        } catch(Exception exception) {
            exception.printStackTrace();
        }

    }

}
```
编写BulkLoadDriver
```
public class BulkLoadDriver extends Configured implements Tool {
	private static final Logger logger = LoggerFactory.getLogger(BulkLoadDriver.class);
	private static final String DATA_SEPERATOR = "\t";
    private static final String TABLE_NAME = "truman";//表名
    private static final String COLUMN_FAMILY_1="personal";//列组1
    private static final String COLUMN_FAMILY_2="professional";//列组2

    public static void main(String[] args) {
    	System.setProperty("hadoop.home.dir", "D:/hadoop");
    	System.setProperty("HADOOP_USER_NAME", "root");
    	logger.info("---------------------------------------------");
        try {
            int response = ToolRunner.run(HBaseConfiguration.create(), new BulkLoadDriver(), args);
            if(response == 0) {
                System.out.println("Job is successfully completed...");
            } else {
                System.out.println("Job failed...");
            }
        } catch(Exception exception) {
            exception.printStackTrace();
        }
    }

    public int run(String[] args) throws Exception {
        String inputPath = "/user/truman/data.txt";
        String outputPath = "/user/truman/hfile";
        /**
         * 设置作业参数
         */
        Configuration configuration = getConf();
       
        configuration.set("mapreduce.framework.name", "yarn");
        configuration.set("yarn.resourcemanager.address", "192.168.1.2:8032");
        configuration.set("yarn.resourcemanager.scheduler.address", "192.168.1.2:8030");
        configuration.set("fs.defaultFS", "hdfs://192.168.1.2:8020");
        configuration.set("mapred.jar", "D://workspace//SqlDataToHbase//target//SqlDataToHbase-0.0.1-SNAPSHOT-jar-with-dependencies.jar");
        
        configuration.set("data.seperator", DATA_SEPERATOR);
        configuration.set("hbase.table.name", TABLE_NAME);
        configuration.set("COLUMN_FAMILY_1", COLUMN_FAMILY_1);
        configuration.set("COLUMN_FAMILY_2", COLUMN_FAMILY_2);
        
       /* configuration.set("hbase.zookeeper.quorum", "192.168.1.2,192.168.1.3,192.168.1.4");
        configuration.set("hbase.zookeeper.property.clientPort", "2181");*/
        
        Job job = Job.getInstance(configuration, "Bulk Loading HBase Table::" + TABLE_NAME);
        job.setJarByClass(BulkLoadDriver.class);
        job.setInputFormatClass(TextInputFormat.class);
        job.setMapOutputKeyClass(ImmutableBytesWritable.class);//指定输出键类
        job.setMapOutputValueClass(Put.class);//指定输出值类
        job.setMapperClass(BulkLoadMapper.class);//指定Map函数
        FileInputFormat.addInputPaths(job, inputPath);//输入路径
        FileSystem fs = FileSystem.get(configuration);
        Path output = new Path(outputPath);
        if (fs.exists(output)) {
            fs.delete(output, true);//如果输出路径存在，就将其删除
        }
        FileOutputFormat.setOutputPath(job, output);//输出路径
        

        Connection connection = ConnectionFactory.createConnection(configuration);
        TableName tableName = TableName.valueOf(TABLE_NAME);
        HFileOutputFormat2.configureIncrementalLoad(job, connection.getTable(tableName), connection.getRegionLocator(tableName));
        job.waitForCompletion(true);
        if (job.isSuccessful()){
            HFileLoader.doBulkLoad(outputPath, TABLE_NAME,configuration);//导入数据
            return 0;
        } else {
            return 1;
        }
    }

}
```
整个项目需要将hbase-site.xml、yarn-site.xml、mapred-site.xml放入resources下。本地运行出错的话，再加入org.apache.hadoop.io.nativeio.NativeIO到当前工程中

三、数据加载

1. 命令方式

首先修改hadoop-env.sh配置，加入以下：
```
export HBASE_HOME=/data/bigdata/hbase-1.0.0-cdh5.4.0
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HBASE_HOME/lib/hbase-server-1.0.0-cdh5.4.0.jar:$HBASE_HOME/lib/hbase-server-1.0.0-cdh5.4.0-tests.jar:$HBASE_HOME/conf:$HBASE_HOME/lib/zookeeper-3.4.5-cdh5.4.0.jar:$HBASE_HOME/lib/guava-12.0.1.jar:$HBASE_HOME/lib/hbase-client-1.0.0-cdh5.4.0.jar:$HADOOP_CLASSPATH:$HBASE_HOME/lib/*
```

将数据按HFile写入到hdfs中，然后进入$HBASE_HOME/bin中执行以下命令
```
/data/bigdata/hadoop-2.6.0-cdh5.4.0/bin/hadoop jar ../lib/hbase-server-1.0.0-cdh5.4.0.jar completebulkload /user/truman/hfile  truman
```

2. java方式
```
public class HFileLoader {
    public static void doBulkLoad(String pathToHFile, String tableName,Configuration configuration){
        try {
            
            HBaseConfiguration.addHbaseResources(configuration);
            LoadIncrementalHFiles loadFfiles = new LoadIncrementalHFiles(configuration);
            HTable hTable = new HTable(configuration, tableName);//指定表名
            loadFfiles.doBulkLoad(new Path(pathToHFile), hTable);//导入数据
            System.out.println("Bulk Load Completed..");
        } catch(Exception exception) {
            exception.printStackTrace();
        }

    }

}
```
四、结果查询
```
[root@SXLAB30 bin]# ./hbase shell
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.0.0-cdh5.4.0, rUnknown, Tue Apr 21 12:19:34 PDT 2015

hbase(main):001:0> scan 'truman'
ROW                          COLUMN+CELL
 key1                        column=personal:words, timestamp=1476179060738, value=col1
 key1                        column=professional:sum, timestamp=1476179060738, value=value1
 key2                        column=personal:words, timestamp=1476179060738, value=col2
 key2                        column=professional:sum, timestamp=1476179060738, value=value2
 key3                        column=personal:words, timestamp=1476179060738, value=col3
 key3                        column=professional:sum, timestamp=1476179060738, value=value3
 key4                        column=personal:words, timestamp=1476179060738, value=col4
 key4                        column=professional:sum, timestamp=1476179060738, value=value4
4 row(s) in 0.4300 seconds

```