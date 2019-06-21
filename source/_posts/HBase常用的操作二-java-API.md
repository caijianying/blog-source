---
title: HBase常用的操作二(java API)
date: 2016-06-14 20:31:20
tags: 大数据
categories:
- hbase
---
本文接前篇[**HBase常用的操作一(shell)**](http://trumandu.github.io/2016/06/14/HBase%E5%B8%B8%E7%94%A8%E7%9A%84%E6%93%8D%E4%BD%9C%E4%B8%80-shell/)系列，继续探讨总结操作hbase的方法，重点在于java语言对hbase的操作。
此次选用最新api。

<!-- more -->
#### 前言
在进行已下操作之前，需要首先实例化 Configuration，Admin，Connection。具体代码如下
```
private static Configuration conf = null;
private static Admin admin = null;
private static Connection connection = null;
static {
	conf = HBaseConfiguration.create();
	conf.set("hbase.zookeeper.quorum", "*********");
	conf.set("hbase.zookeeper.property.clientPort", "2181");
	try {
		connection = ConnectionFactory.createConnection(conf);
		admin = connection.getAdmin();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}

}
```
#### 1. DDL操作
##### 1.1 创建表
```
/**
 * 创建表
 * @param tableName
 * @param columnsName
 * @throws IOException
 */
public static void createTable(String tableName, String[] columnsName) throws IOException {
	HTableDescriptor tableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));
	// Adding column families to table descriptor
	if (columnsName != null) {
		for (String name : columnsName) {
			tableDescriptor.addFamily(new HColumnDescriptor(name));
		}
	}
	admin.createTable(tableDescriptor);
	System.out.println(" Table created ");
}
```
##### 1.2 删除表
```
/**
 * 删除表
 * @param tableName
 * @throws IOException
 */
public static void deleteTable(String tableName) throws IOException {
	admin.disableTable(TableName.valueOf(tableName));
	admin.deleteTable(TableName.valueOf(tableName));

}
```
##### 1.3 删除 columns
```
/**
 * 删除 columns
 * @param tableName
 * @param columnsName
 * @throws IOException
 */
public static void deleteColumn(String tableName, String[] columnsName) throws IOException {
	if (columnsName != null) {
		for (String name : columnsName) {
			admin.deleteColumn(TableName.valueOf(tableName), name.getBytes());
		}
	}
}
```
##### 1.4 获取所有的表
```
/**
 * 获取所有的表
 * @param tableName
 * @param columnsName
 * @throws IOException
 */
public static HTableDescriptor[] listTable() throws IOException {
	HTableDescriptor[] tables = admin.listTables();
	return tables;
}
```
##### 1.5 查看表是否存在
```
/**
 * 查看表是否存在
 * @param tableName
 * @return
 * @throws IOException
 */
public static boolean tableIsExists(String tableName) throws IOException {
	return admin.tableExists(TableName.valueOf(tableName));
}
```

#### 2. DML操作
##### 2.1 读取数据
```
/**
 * 读取数据
 * @param tableName
 * @param row
 * @param family
 * @param column
 * @return
 */
public static String getValue(String tableName, byte[] row, byte[] family, byte[] column) {
	Table table = null;
	try {
		table = connection.getTable(TableName.valueOf(tableName));
		Get g = new Get(row);
		Result reasult;
		reasult = table.get(g);
		byte[] value = reasult.getValue(family, column);
		if (value != null) {
			return new String(value);
		}
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}finally {
		if (table != null)
			try {
				table.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	}

	return null;
}
```
##### 2.2 删除数据
```
/**
 * 删除数据
 * @param tableName
 * @param row
 * @param family
 * @param column
 */
public static void remove(String tableName, byte[] row, byte[] family, byte[] column) {
	Table table = null;
	try {
		table = connection.getTable(TableName.valueOf(tableName));
		Delete delete = new Delete(row);
		if (column != null) {
			delete.addColumn(family, column);
		}
		if (family != null) {
			delete.addFamily(family);
		}
		table.delete(delete);
		table.close();
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	}
}
```
##### 2.3 增加数据
```
/**
 * 增加数据
 * @param tableName
 * @return
 * @throws IOException
 */
public static void addData(String tableName, byte[] row, byte[] family, byte[] column, byte[] data) {
	Table table = null;
	try {
		table = connection.getTable(TableName.valueOf(tableName));
		Put p = new Put(row);
		p.addColumn(family, column, data);
		table.put(p);
	} catch (IOException e) {
		// TODO Auto-generated catch block
		e.printStackTrace();
	} finally {
		if (table != null)
			try {
				table.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	}

}
```
#### 3. 其他操作
##### 3.1 查询集群状态
```
/**
 * 获取hbase集群状态
 * @return
 * @throws IOException
 */
public static ClusterStatus getClusterStatus() throws IOException {
	ClusterStatus clusterStatus = admin.getClusterStatus();
	return clusterStatus;
}
```