---
title: 利用外部表读取orc文件
date: 2016-08-21 20:45:56
tags: 大数据
categories:
- hive
---
# 利用外部表读取orc文件

## 前言
因为orc文件压缩，并且可以快速加载到hive中，因为这种应用于hadoop平台的上文件获得许多开发者的注意。

ORC : Optimized Row Columnar (ORC) file
据官方文档介绍，这种文件格式可以提供一种高效的方法来存储Hive数据。它的设计目标是来克服Hive其他格式的缺陷。运用ORC File可以提高Hive的读、写以及处理数据的性能。 
## 方案
可以利用构建hive中外部表来读取orc文件。
### 1.建表
其中location即为orc文件所在的目录，该目录下新增的文件，都可以被hive查出
```
CREATE EXTERNAL TABLE test_orc_v2(
status string,
type string
)
stored as orc location "/user/test";
```
### 2.orc文件生成
```
import java.io.IOException;
import java.util.List;

import org.apache.crunch.types.orc.OrcUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.hive.ql.io.orc.OrcFile;
import org.apache.hadoop.hive.ql.io.orc.OrcFile.ReaderOptions;
import org.apache.hadoop.hive.ql.io.orc.OrcStruct;
import org.apache.hadoop.hive.ql.io.orc.Reader;
import org.apache.hadoop.hive.ql.io.orc.RecordReader;
import org.apache.hadoop.hive.ql.io.orc.Writer;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorUtils;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfo;
import org.apache.hadoop.hive.serde2.typeinfo.TypeInfoUtils;
import org.apache.hadoop.io.Text;
public class HdfsFileTest {

	public static void main(String[] args) throws IOException {
		System.setProperty("hadoop.home.dir", "D:/hadoop");
		HdfsFileTest test = new HdfsFileTest();
		test.createOrcFile();
	}

	public void createOrcFile() throws IOException {
		String typeStr = "struct<status:string,type:string>";
		TypeInfo typeInfo = TypeInfoUtils.getTypeInfoFromTypeString(typeStr);
		ObjectInspector inspector = OrcStruct.createObjectInspector(typeInfo);


		Configuration conf = new Configuration();
		Path tempPath = new Path("/user/test/test1.orc");

		Writer writer = OrcFile.createWriter(tempPath, OrcFile.writerOptions(conf).inspector(inspector).stripeSize(100000).bufferSize(10000));
        
		OrcStruct struct = OrcUtils.createOrcStruct(typeInfo,new Text("OK1"),new Text("http2"));
		
		writer.addRow(struct);
		writer.close();
		
		ReaderOptions options = OrcFile.readerOptions(conf); 
	    Reader reader = OrcFile.createReader(tempPath, options); 
	    RecordReader rows = reader.rows(); 
	 
	    List  next =   (List) ObjectInspectorUtils.copyToStandardJavaObject(rows.next(null), 
	        reader.getObjectInspector()); 
	    System.out.println(next.toString());
	    rows.close(); 
	}

}
```
其中OrcUtils使用如下依赖，在pom中注意添加
```
<dependency>
	<groupId>org.apache.crunch</groupId>
	<artifactId>crunch-hive</artifactId>
	<version>0.14.0</version>
</dependency>
```
