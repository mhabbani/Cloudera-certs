# 01 Scala Spark

This file covers an introduction to Scala Spark based on [this playlist](https://www.youtube.com/watch?v=iclGhV3s98o&index=53&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Table of contents

* Submitting tasks
* Read and save files
  * Read and save text files
  * Read and save sequence files
  * Read and save from/into Hive
  * Read and save JSON Files
* Word count
* Joining datasets
  * Using RDDs
  * Using Dataframes


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

## Word count

In this section we will implemenet a word count using Scala. Let's assume
we have the following file in HDFS:

```
Hello world how are you?
I am fine and how about you.
Let us write the word count program using pyspark - python on spark
```

We could count how many time a word appears as follows:

```
# First read the file from HDFS
val dataRDD = sc.textFile("/user/cloudera/spark_word_count.txt")

# Split data and assing an integer
val data_mapped = dataRDD.flatMap(x => (x.split(" "))).map(x => (x, 1))

# Finally count words using reduceByKey
val reduced = data_mapped.reduceByKey(_+_)

# Print results
reduced.collect().foreach(println)

(us,1)
(are,1)
(fine,1)
(pyspark,1)
(Hello,1)
(am,1)
(how,2)
(python,1)
(you?,1)
(using,1)
(word,1)
(program,1)
(world,1)
(Let,1)
(spark,1)
(about,1)
(on,1)
(I,1)
(you.,1)
(-,1)
(count,1)
(and,1)
(write,1)
(the,1)
```

## Joining datasets

In this section we will cover how to join datasets using Scala spark.
Given the MySQL database imported into HDFS we would like to
calculate the revenue and number of orders from the `order_items` table
on daily basis.

### Using RDDs

We will use RDDs in this first approach

```
# Load files from HDFS.
val orders = sc.textFile("/user/cloudera/sqoop_import/orders/")
val order_items = sc.textFile("/user/cloudera/sqoop_import/order_items/")

# Let's map the RDDs putting the order id as key.
val orders_parsed = orders.map(rec => (rec.split(",")(0).toInt, rec))
val order_items_parsed = order_items.map(rec => (rec.split(",")(1).toInt, rec))

# We can join now the two RDDs
val joined_rdds = order_items_parsed.join(orders_parsed)

# Once the data is joined, we could start mapping and grouping
# We will get first the revenue by day.
val revenuePerDay = joined_rdds.map(t => (t._2._2.split(",")(1), t._2._1.split(",")(4).toFloat)).reduceByKey(_+_)

# We can check the results as follows:
revenuePerDay.take(5).foreach(println)
(2013-10-05 00:00:00.0,105913.14)                                               
(2014-05-17 00:00:00.0,81789.07)
(2014-06-29 00:00:00.0,60257.92)
(2014-04-23 00:00:00.0,113409.17)
(2013-10-27 00:00:00.0,106982.91)

# Let's calculate now the number of orders per day.
# We will create a key with the dates and orders id, and will remove duplicates.
val ordersPerDay = joined_rdds.map(rec => rec._2._2.split(",")(1) + "," + rec._1).distinct()

# Now let's calculate the number of orders per day
val orderPD = ordersPerDay.map(rec => (rec.split(",")(0), 1)).reduceByKey(_+_)

# We can finally join the revenue and orders per day joins as follows:
val rev_order_date = orderPD.join(revenuePerDay)

rev_order_date.take(5).foreach(println)
(2013-10-05 00:00:00.0,(184,105913.14))
(2014-05-17 00:00:00.0,(138,81789.07))
(2014-05-06 00:00:00.0,(231,137078.08))
(2014-04-23 00:00:00.0,(185,113409.17))
(2013-10-27 00:00:00.0,(184,106982.91))
```

### Using Dataframes

In this subsection we will try to solve the same problem but using 
Spark Dataframes instead of RDDs:

```
# Import functions
import org.apache.spark.sql.functions._


# Let's load the Dataframes from Hive
val orders = sqlContext.sql("SELECT * FROM orders")
val order_items = sqlContext.sql("SELECT * FROM order_items")

# We will start calculating the revenue per day.
val revenuePerDay = (order_items
	.join(orders, $"order_item_order_id"===$"order_id")
	.groupBy($"order_date")
	.agg(sum($"order_item_subtotal").alias("Revenue"))
	)

# We will calculate now the number of orders per day
val ordersPerDay = (order_items
	.join(orders, $"order_item_order_id"===$"order_id")
	.select($"order_date", $"order_item_order_id")
	.distinct()
	.groupBy($"order_date")
	.agg(count($"order_item_order_id").alias("Orders"))
	)
	
	
# Finally we just need to join both dataframes
val rev_orders_per_day = ordersPerDay.join(revenuePerDay, "order_date")

rev_orders_per_day.show()
+--------------------+------+------------------+                                
|          order_date|Orders|           Revenue|
+--------------------+------+------------------+
|2014-07-11 00:00:...|   119| 71334.14000000007|
|2013-09-02 00:00:...|   162| 100127.4900000002|
|2014-01-09 00:00:...|   168|103455.13000000018|
|2014-02-24 00:00:...|   150|  93628.8300000003|
|2013-07-29 00:00:...|   216|137287.09000000026|
|2014-02-19 00:00:...|   229|141857.25000000038|
|2014-01-30 00:00:...|   215|126157.02000000018|
|2014-03-29 00:00:...|    82| 55485.87000000003|
|2014-01-25 00:00:...|    89|          56422.01|
|2014-07-01 00:00:...|   161| 99060.76000000011|
|2014-02-14 00:00:...|   141| 79936.26000000001|
|2014-05-28 00:00:...|   197|106615.78000000007|
|2014-02-09 00:00:...|   197|115172.03000000013|
|2014-03-24 00:00:...|   116| 65613.72000000003|
|2013-08-29 00:00:...|   178| 99960.57000000017|
|2013-10-27 00:00:...|   184|106982.81000000023|
+--------------------+------+------------------+
```
