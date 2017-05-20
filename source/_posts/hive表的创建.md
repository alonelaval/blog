---
title: hive表创建的几种方式
date: 2017-05-01 13:39:23
categories: 
	- 大数据
tags:
	- hive 
---

##   hive表的创建
### 一、语法
```sql
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [COMMENT col_comment], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
 
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name
  LIKE existing_table_or_view_name
  [LOCATION hdfs_path];
 
data_type
  : primitive_type
  | array_type
  | map_type
  | struct_type
  | union_type  -- (Note: Available in Hive 0.7.0 and later)
 
primitive_type
  : TINYINT
  | SMALLINT
  | INT
  | BIGINT
  | BOOLEAN
  | FLOAT
  | DOUBLE
  | STRING
  | BINARY      -- (Note: Available in Hive 0.8.0 and later)
  | TIMESTAMP   -- (Note: Available in Hive 0.8.0 and later)
  | DECIMAL     -- (Note: Available in Hive 0.11.0 and later)
  | DECIMAL(precision, scale)  -- (Note: Available in Hive 0.13.0 and later)
  | DATE        -- (Note: Available in Hive 0.12.0 and later)
  | VARCHAR     -- (Note: Available in Hive 0.12.0 and later)
  | CHAR        -- (Note: Available in Hive 0.13.0 and later)
 
array_type
  : ARRAY < data_type >
 
map_type
  : MAP < primitive_type, data_type >
 
struct_type
  : STRUCT < col_name : data_type [COMMENT col_comment], ...>
 
union_type
   : UNIONTYPE < data_type, data_type, ... >  -- (Note: Available in Hive 0.7.0 and later)
 
row_format
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
        [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
        [NULL DEFINED AS char]   -- (Note: Available in Hive 0.13 and later)
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, property_name=property_value, ...)]
 
file_format:
  : SEQUENCEFILE
  | TEXTFILE    -- (Default, depending on hive.default.fileformat configuration)
  | RCFILE      -- (Note: Available in Hive 0.6.0 and later)
  | ORC         -- (Note: Available in Hive 0.11.0 and later)
  | PARQUET     -- (Note: Available in Hive 0.13.0 and later)
  | AVRO        -- (Note: Available in Hive 0.14.0 and later)
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname

```
### 二、创建表时的4种模式
1. TEMPORARY模式，在该模式下创建的表，只在当前session中有效，当前session关闭时，该表会被hive删除，不支持分区，也不支持创建索引.  
2. EXTERNAL模式，俗称外部表，该模式的表，数据不由hive管理，删除表时，数据不会被删除，创建表时可以指定数据加载的路径如:  

		CREATE EXTERNAL TABLE [IF NOT EXISTS] [db_name.]table_name
 		LIKE existing_table_or_view_name
		[LOCATION hdfs_path];
3. 	默认模式，俗称管理表，该模式的表，数据由hive管理,删除表时数据也会被删除。
4. Create Table As Select (CTAS)模式，该模式的表是由查询的结果来创建，在该模式下，不能创建分区表，不能创建外部表，也不支持数据分桶。  

### 三、表中的字段的数据类型
##### 在hive中字段支持5种类型:  
1.基本类型（primitive_type）:
	 
	   TINYINT
	  | SMALLINT
	  | INT
	  | BIGINT
	  | BOOLEAN
	  | FLOAT
	  | DOUBLE
	  | STRING
	  | BINARY      
	  | TIMESTAMP   
	  | DECIMAL    
	  | DECIMAL(precision, scale)  
	  | DATE        
	  | VARCHAR     
	  | CHAR 
2.数组类型（array_type）  
3.字典类型(map_type)   
4.结构类型(struct_type)  
5.联合类型(union_type)  
如:  

		CREATE TABLE employees (  
		    name STRING,  
		    salary FLOAT,  
		    subordinates ARRAY<STRING>,  
		    deductions MAP<STRING, FLOAT>,  
		    address STRUCT<street:STRING, city:STRING, state:STRING, zip:INT>,
		    dddd:UNIONTYPE<data_type,data_type,....>
		) PARTITIONED BY (country STRING, state STRING);  


###  四、数据分区
数据分区为将数据按照指定字段的值来分别存放，这样在查询时，可以只查某一个分区的数据，以提高数据查询效率，也可以在数据的管理上更有效率。指定分区时，数据将按照分区字段先后的顺序来存放.
如：  
  
 	CREATE TABLE employees (.....)
	PARTITIONED BY (pt_dt string)
	ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t'
	STORED AS TEXTFILE;
### 五、数据分桶
#####把表（或者分区）组织成桶（Bucket）有两个理由：  
1.为获得更高的查询处理效率。桶为表加上了额外的结构，Hive 在处理有些查询时能利用这个结构。具体而言，连接两个在（包含连接列的）相同列上划分了桶的表，可以使用 Map 端连接 （Map-side join）高效的实现。比如JOIN操作。对于JOIN操作两个表有一个相同的列，如果对这两个表都进行了桶操作。那么将保存相同列值的桶进行JOIN操作就可以，可以大大较少JOIN的数据量。  

2.使取样（sampling）更高效。在处理大规模数据集时，在开发和修改查询的阶段，如果能在数据集的一小部分数据上试运行查询，会带来很多方便。

### 六、数据导入与数据导出
在创建表时，可以指定3个属性[INPUTFORMAT, OUTPUTFORMAT, SERDE],由这些属性来指定数据的导入格式和导出格式  
首先来理清这三者之间的关系，我们直接引用Hive官方说法：  

	SerDe is a short name for “Serializer and Deserializer.”
	Hive uses SerDe (and !FileFormat) to read and write table rows.
	HDFS files –> InputFileFormat –> <key, value> –> Deserializer –> Row object
	Row object –> Serializer –> <key, value> –> OutputFileFormat –> HDFS files
总结一下，当面临一个HDFS上的文件时，Hive将如下处理（以读为例）：
	
	1.调用InputFormat，将文件切成不同的文档。每篇文档即一行(Row)。  
	2.调用SerDe的Deserializer，将一行(Row)，切分为各个字段。
当HIVE执行INSERT操作，将Row写入文件时，主要调用OutputFormat、SerDe的Seriliazer，顺序与读取相反  
【INPUTFORMAT】属性的类必须实现org.apache.hadoop.mapred.InputFormat接口  
【OUTPUTFORMAT】属性的类必须实现org.apache.hadoop.mapreduce.OutputFormat接口  
【SERDE】属性的类必须实现org.apache.hadoop.hive.serde2接口












###注：参考文档
	https://cwiki.apache.org/confluence/display/Hive/DeveloperGuide
	https://hive.apache.org/javadocs/r1.2.1/api/
	http://hadoop.apache.org/docs/r2.7.1/api/
	https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL

