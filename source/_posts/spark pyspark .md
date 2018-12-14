---
title: Spark 运行Bug
date: 2018-06-26 13:57:23
categories: 
	- 大数据
	- spark
tags:
	- spark
	- pyspark
---
## Spark 运行Bug
spark-shell执行命令出现以下错误提示：

java.io.IOException: Cannot run program “/etc/hadoop/conf.cloudera.yarn/topology.py” (in directory “/home/108857”): error=2。

这个算是Cloudera 的bug ， 把datanode上的 /etc/hadoop/conf.cloudera.yarn/topology* 复制到执行spark-shell的机器上即可。



## spark pyspark 
```

from pyspark.sql import HiveContext
sqc = HiveContext(sc)

sqc.sql("use xcredit").show()
sqc.sql(select * from xxx.xxx).show()


```

## spark spark-shell 

```


import org.apache.spark.SparkConf
import org.apache.spark.SparkContext
import org.apache.spark.sql.hive.HiveContext

val sqlContext = new HiveContext(sc)
import sqlContext.implicits._

sqlContext.sql(" SELECT * FROM xxx.xxx limit 10").collect().foreach(println)






sqlContext.sql("insert overwrite local directory '/data/spark_test' row format delimited fields terminated by ',' select ln.index_digest,ln.begin_date,count(distinct lt.p_id) as c  from tb_distinct_analyze_loan ln join tb_distinct_analyze_loan lt
on ln.index_digest = lt.index_digest where ln.begin_date>=date_sub(current_date, 1095) and lt.begin_date>=date_sub(current_date, 1095) and
((ln.begin_date < lt.end_date and ln.begin_date >= lt.begin_date) or
(ln.end_date >= lt.begin_date and ln.end_date <= lt.end_date)) group by  ln.index_digest,ln.begin_date ")


```