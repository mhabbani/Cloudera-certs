# 01 Scala Spark

This file covers an introduction to Scala Spark based on [this playlist](https://www.youtube.com/watch?v=iclGhV3s98o&index=53&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Table of contents

* Submitting tasks
* Read and save files
  * Read and save text files
  * Read and save sequence files
  * Read and save from/into Hive
  * Read and save JSON Files

## Submitting tasks

First install `sbt`, you can go to [its website](http://www.scala-sbt.org/download.html) 
for installing instructions.

## Read and save files
### Read and save text files

Let's read some data from HDFS

```
# Departments
val departments = sc.textFile("/user/cloudera/sqoop_import/departments/")

# If the HDFS is in another machine
val departments = sc.textFile("hdfs://quickstart.cloudera:8020/user/cloudera/sqoop_import/departments")

# We can print the loaded data:
departments.collect().foreach(println)
2,Fitness                                                                       
3,Footwear
4,Apparel
5,Golf
6,Outdoors
7,Fan Shop
```

We can now save back departments RDDs as text and objects files:

```
# Saving as text file
departments.saveAsTextFile("/user/cloudera/scala/departments_text/")
```

### Read and save sequence files

When saving sequence file a key has to be defined. If there is no key to be saved
then you can use `NullWritable.get()` function to create a null key.

Let's assume we have the table `departments` loaded as an RDD:

```
# Import packages
import org.apache.hadoop.io._
departments_RDD.map(x => (NullWritable.get(), x)).saveAsSequenceFile("/user/cloudera/spark/scala/departmens_sequence/")
```

Now if we want to read from a sequence file:

```
# Read from sequence file
# You have to specify the Key class as well as the Value class
val dep_seq = sc.sequenceFile("/user/cloudera/scala/departments_sequence/", classOf[NullWritable], classOf[Text]).map(rec => rec._2.toString())

# You can now print the results
dep_seq.collect().foreach(println)
```

### Read and save from/into Hive

Let's start by reading from a table stored in Hive:

```
# You need to define the sqlContext 
import org.apache.spark.sql.hive.HiveContext

val sqlC = new HiveContext(sc)
val depts = sqlC.sql("SELECT * FROM departments") 

```

Let's create now a table in Hive:

```
sqlC.sql("CREATE TABLE departments_new AS SELECT * FROM departments")
```

Finally we are going to load a JSON file from HDFS into Spark. Assuming
we have the following json file in the directory `/user/cloudera/scala/departments.json`:

```
{"department_id":2, "department_name":"Fitness"}
{"department_id":3, "department_name":"Footwear"}
{"department_id":4, "department_name":"Apparel"}
{"department_id":5, "department_name":"Golf"}
{"department_id":6, "department_name":"Outdoors"}
{"department_id":7, "department_name":"Fan Shop"}
{"department_id":8, "department_name":"TESTING"}
```

If we want to load the file in Spark we can proceed as follows:

```
import org.apache.spark.sql.SQLContext
val sqlC = new SQLContext(sc)
val json_deps = sqlC.jsonFile("/user/cloudera/scala/departments.json")

# We can register the dataframe as a temporary table
json_deps.registerTempTable("deps_temp").show()
```

**NOTE**: `jsonFile` command is deprecated. Instead we should used `read.json()`:

```
import org.apache.spark.sql.SQLContext
val sqlC = new SQLContext(sc)
val json_deps = sqlC.read.json("/user/cloudera/scala/departments.json")

# We can register the dataframe as a temporary table
json_deps.registerTempTable("deps_temp").show()

```
