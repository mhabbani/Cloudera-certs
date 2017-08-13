# 01 Spark introduction

The aim of this file is to provide a first introduction to Spark in 
the context of the Cloudera certification. During this guide both Scala
and Python would be used since both programming languages are required 
during the certification.


## Table of contents

* Using Hive Context


## Using Hive Context

The first thing to do in order to access Hive tables from
Spark is to make Spark aware of Hive configuration. To do
so we will create a link in the Spark configuration directory
(`/etc/spark/conf/`) to the file `hive-site.xml` located
in the directory `/etc/hive/conf/`. This file contains 
Hive configuration an allows spark to create a `sqlContext` based
on Hive.

If we now launch a spark-shell terminal we should see the following:

```
17/08/12 11:26:41 INFO repl.SparkILoop: Created sql context (with Hive support)..
```

We can now make queries on Hive table using spark:

```
sqlContext.sql("SELECT * FROM sqoop_import.departments").collect().foreach(println)


[2,Fitness]
[3,Footwear]
[4,Apparel]
[5,Golf]
[6,Outdoors]
[7,Fan Shop]
[10,New record]
[12,New record]
[8000,TESTING]
```


* We have used `sqlContext` to perform a SQL query on the sqoop_import Hive database.
* We then have collected the results using `collect`.
* Finally we have printed each row collected.

**Python version**:

```
sqlCtx.sql("SELECT * FROM sqoop_import.departments").collect()
```

When launching the pyspark shell it already comes with the
variable `sqlCtx` which is a SQL context for python. Another option 
to create a SQL context would be:

```
from pyspark.sql import HiveContext
sqlContext = HiveContext(sc)
```
