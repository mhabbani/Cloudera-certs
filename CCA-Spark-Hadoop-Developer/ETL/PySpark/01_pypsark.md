# 01 PySpark

This file covers an introduction to PySpark based on [this playlist](https://www.youtube.com/watch?v=EGQjqcOIjIM&index=31&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE).

## Table of contents

* Connecting to a database
* Read and save from text file
* Read and save from sequence file
* Read and save from from json files and Hive
* Word Count program
* Joining datasets
  * Using RDDs
  * Using Dataframes
  * Using SQL
* Aggregating datasets
  * Using RDDS
  * Using Dataframes
* More complicated aggregations
  * Using RDDs
  * Using Dataframes
* Getting max values
  * Using RDDs
  * Using Dataframes
* Filtering data
  * Using RDDs
  * Using Dataframes
* Ordering and ranking data
  * Using RDDs
  * Using Dataframes
  
## Connecting to a database

In thi section we will cover how to connect Spark to a MySQL database. To do so
we have to tell Spark where the jdbc connectors are located:

```
# Import Spark SQLContext
from pypsark.sql import SQLContext

# Tell spark where the connector is located
os.environ['SPARK_CLASSPATH'] = "/usr/share/java/mysql-connector-java.jar"

# Create the connection URL
jdbcUrl = "jdbc:mysql://quickstart.cloudera:3306/retail_db?user=retail_dba&password=cloudera"

# Create a SQLContext
sqlContext = SQLContext(sc)

# Finally load the departments table into a Spark dataframe
df = sqlContext.read.jdbc(jdbcUrl, table="departments")
```

**NOTE**: In the playlist spark version used is 1.3, however for the certification
Cloudera provides Spark 1.6 where there are some changes from previous versions. 
I would cover this course using Spark 1.6. 



## Read and save from text file

### Read from HDFS 

Read and save from a text file. In this exercise we
are going to read a text file located in HDFS and 
save it again in a different HDFS location.

```
# Read the file from HDFS
dataRDD = sc.textFile("/user/cloudera/sqoop_import/departments")

# Print the file content
for line in dataRDD.collect():
	print(line)

# Save the file into HDFS
dataRDD.saveAsTextFile("/user/cloudera/pyspark/departmentsTesting")
```

If you want to run the previously task as an application you have
to create the Spark context.

```
# Import Spark context, create configuration and spark context
from pyspark import SparkContext, SparkConf
conf = SparkConf().setAppName("pyspark")
sc = SparkContext(conf=conf)
# Read the file from HDFS
dataRDD = sc.textFile("/user/cloudera/sqoop_import/departments")

# Print the file content
for line in dataRDD.collect():
	print(line)

# Save the file into HDFS
dataRDD.saveAsTextFile("/user/cloudera/pyspark/departmentsTesting")
```

You can save the previous code in a file and execute it using
`spark-submit`:

```
spark-submit file_name.py
```

After the job finish you should be able to see the saved files
into HDFS:

```
hdfs dfs -ls /user/cloudera/pyspark
Found 1 items
drwxr-xr-x   - cloudera cloudera          0 2017-08-13 01:58 /user/cloudera/pyspark/departmentsTesting
```

**NOTE**: Using fully qualified path:

```
data = sc.textFile("hdfs://quickstart.cloudera:8020/use/cloudera/sqoop_import/departments")
```

To get the fully qualified path of HDFS we can go to the file `/etc/hadoop/conf/core-site.xml`
and check the following part of the file:

```
<configuration>
  <property>
	<name>fs.defaultFS</name>
	<value>hdfs://quickstart.cloudera:8020</value>
  </property>
```

### Read from local file system

If we want to read from local file system we have to specify the path
to spark as follows:

```
data = sc.textFile("file:///home/cloudera/departments.json")
```

## Read and save from sequence file

To write into sequence files we need to have `(key, value)` RDDs:

```
# Read the file from HDFS
dataRDD = sc.textFile("/user/cloudera/sqoop_import/departments")

# Convert dataRDD into a key, value RDD
dataRDD = dataRDD.map(lambda x: tuple(x.split(",", 1)))

# Check map 
for d in dataRDD.collect():
	print(d)

# Save RDD into HDFS as a sequence file

dataRDD.saveAsSequenceFile("/user/cloudera/pyspark/departmentsSeq")
```

Now if want to read the previously saved sequence file
from Spark we could do the following:

```
data = sc.sequenceFile("/user/cloudera/pyspark/departmentsSeq")

# Check data has been read properly
for d in data.collect():
	print(d)
```

## Read and save from from json files and Hive

### Read and write into Hive

To read from Hive you have to create a `HiveContext` (it may be already created when
launching Hive):

```
# Create Hive Context
from pyspark.sql import HiveContext
sqlContext = HiveContext(sc)

# Read Hive table
df = sqlContext.sql("SELECT * FROM departments")
```

We used the function `sql` to execute queries in Hive. We can as well
use this function to create tables:

```
sqlContext.sql("CREATE TABLE departments_test AS SELECT * FROM departments")
```

### Read and write Json files

To read from JSON files located in HDFS we have to create a SQLContext
and then read it using `read.json()`

```
from pyspark.sql import SQLContext
sqlContext = SQLContext(sc)

json_df = sqlContext.read.json("/user/cloudera/pyspark/departments.json")
```

The json file we are reading has the following content:

```
{"department_id":2, "department_name":"Fitness"}
{"department_id":3, "department_name":"Footwear"}
{"department_id":4, "department_name":"Apparel"}
{"department_id":5, "department_name":"Golf"}
{"department_id":6, "department_name":"Outdoors"}
{"department_id":7, "department_name":"Fan Shop"}
{"department_id":8, "department_name":"TESTING"}
```

We can now register the loaded dataframe as a temporary table in Spark:

```
json_df.registerTempTable("json_d")

# Now select from the registered table
sqlContext.sql("SELECT * FROM json_d")
```

Finally, if we want to save back the temporary table
as JSON in HDFS we can proceed as follows:

```
sqlContext.sql("SELECT * FROM json_d").toJSON().saveAsTextFile("/user/cloudera/pyspark/departments_saved/")
```

## Word Count program

In this exercise we are going to implement a word count program using pyspark.
To do so we will put the following file in HDFS:

```
Hello world how are you?
I am fine and how about you.
Let us write the word count program using pyspark - python on spark
```

We read the file in `pyspark`:

```
data = sc.textFile("/user/cloudera/spark_word_count.txt")
```

Each element in the RDD is a file line, to count the words we have to split
each line into words:

```
dataFlatMap = data.flatMap(lambda x: x.split(" ")).map(lambda x: (x, 1)).reduceByKey(lambda a,b: a+b)

for i in data.FlatMap.collect():
	print(i)
	
(u'and', 1)
(u'you?', 1)
(u'about', 1)
(u'word', 1)
(u'python', 1)
(u'am', 1)
(u'-', 1)
(u'count', 1)
(u'us', 1)
(u'write', 1)
...
```

## Joining datasets

In this section we will cover how to join datasets using Pypsark. 
We have previously imported the tables `orders` and `order_items`
into Hive using Sqoop.

The goal of the exercise is to get the revenue and number of orders
on a daily basis.

The tables `orders` and `order_items` have the following structure:

```
hive> describe orders;
OK
order_id            int
order_date          string
order_customer_id   int
order_status        string

hive> describe order_items;
OK
order_item_id       int
order_item_order_id int
order_item_product_idint
order_item_quantity tinyint
order_item_subtotal double
order_item_product_pricedouble
```

We will solve the problem using first RDDs and then dataframes, but first
let's read the Hive tables using pyspark:

```
orders = sqlCtx.sql("SELECT * FROM orders")
order_items = sqlCtx.sql("SELECT * FROM order_items")
```

#### Using RDDs

Using spark 1.6 the variables previously loaded are dataframes, so in 
order to work with RDDs we have to convert them:

```
orders_rdd = orders.rdd
order_items_rdd = order_items.rdd

# We now map orders and order_items as tuples
orders_rdd_mapped = orders_rdd.map(lambda o: (o.order_id , ",".join([str(e) for e in o])))
orders_items_rdd_mapped = order_items_rdd.map(lambda o: (o.order_item_order_id, ",".join([str(e) for e in o])))

# We can finally join the two RDDS as follows:
ordersJoined = orders_items_rdd.join(orders_items_rdd_mapped)
```

The resulting RDD would look something like this:

```
... 
(32768, ('81958,32768,1073,1,199.99,199.99', '32768,2014-02-12 00:00:00.0,1900,PENDING_PAYMENT'))
(32768, ('81959,32768,403,1,129.99,129.99', '32768,2014-02-12 00:00:00.0,1900,PENDING_PAYMENT'))
(32768, ('81960,32768,957,1,299.98,299.98', '32768,2014-02-12 00:00:00.0,1900,PENDING_PAYMENT'))
```

We have now to extract from the joined RDD the subtotal and the date of the order:

**ORDERS PER DAY**

```
# Firs extract unique orders per day
orders_per_day = ordersJoined.map(lambda t: (t[1][1].split(",")[1] + "," + str(t[0]))).distinct()
orders_per_day_parsed = orders_per_day.map(lambda t: (t.split(",")[0], 1))

# We finally count the number of orders per day:
total_orders_per_day = orders_per_day_parsed.reduceByKey(lambda a,b: a+b)

```

**REVENUE PER DAY**

```
# To calculate the revenue we can use reduceByKey
rev_per_day = ordersJoined.map(lambda t: (t[1][1].split(",")[1], float(t[1][0].split(",")[4])))
rev_per_day_tot = rev_per_day.reduceByKey(lambda tot1, tot2: tot1 + tot2)
```

**JOINING REVENUE AND NUMBER OF ORDERS**

We can finally join revenue and number of orders by day:

```
final_join = rev_per_day_tot.join(total_orders_per_day)
```

#### Using Dataframes

```
# We first load the dataframes from Hive
orders = sqlCtx.sql("SELECT * FROM orders")
order_items = sqlCtx.sql("SELECT * FROM order_items")

# We now join the dataframes
orders_joined = order_items.join(orders, orders.order_id==order_items.order_item_order_id)
```

**ORDERS PER DAY**

We now calculate based on the joined dataframe the number of orders per day:

```
orders_per_day = (orders_joined
	.select(orders_joined.order_id, orders_joined.order_date)
	.distinct()
	.groupBy("order_date")
	.count()
	.withColumnRenamed("count", "orders")
	)
```

**REVENUE PER DAY**

We now calculate revenue per day based on joined dataframes:

```
rev_per_day = (orders_joined
	.select(orders_joined.order_date, orders_joined.order_item_subtotal)
	.groupBy("order_date")
	.sum()
	.withColumnRenamed("sum(order_item_subtotal)", "revenue")
	.withColumnRenamed("order_date", "rev_date")
		)
```

**JOINING REVENUE AND NUMBER OF ORDERS**

Finally, we join both datafarmes by the order date:

```
final_df = rev_per_day.join(orders_per_day, rev_per_day.rev_date==orders_per_day.order_date)
```

#### Using SQL

In this section we will see how to perform the previous joins using
Spark SQL.

**USING Hive Context**

A first approach would be something like this:

```
sqlCtx = HiveContext(sc)
joinDF = sqlCtx.sql(
"SELECT o.order_date, sum(oi.order_item_subtotal) as rev, count(distinct o.order_id) FROM orders o JOIN order_items oi on o.order_id=oi.order_item_order_id GROUP BY o.order_date ORDER BY o.order_date"
)
```

**USING SQL Context**

This approach is a little bit more complicated since we have to
register temporary tables. Assuming we have imported the tables `orders`
and `order_items` into the HDFS location `/user/cloudera/sqoop_import/`.

We can load them in spark as follows:

```
from pyspark.sql import SQLContext, Row
sqlContext = SQLContext(sc)

# Load RDD from HDFS
ordersRDD = sc.textFile("/user/cloudera/sqoop_import/orders")
# Break fields
ordersMap = ordersRDD.map(lambda o: o.split(","))
# Convert each row of the RDD into an object Row
orders = ordersMap.map(lambda o: Row(order_id= int(o[0]), order_date=o[1], order_customer_id=int(o[2]), order_status=o[3]))

# Finally we register the table as a temporary table
orders_df = sqlContext.createDataFrame(orders)
orders_df.registerTempTable("orders_df")
```

We will do the same with the table `order_items` so we will end up
having a temporary table called `order_items_df`:

```
order_itemsRDD = sc.textFile("/user/cloudera/sqoop_import/order_items")
order_itemsMap = order_itemsRDD.map(lambda oi: oi.split(","))
order_items = order_itemsMap.map(lambda oi: Row(order_item_id=int(oi[0]), order_item_order_id=int(oi[1]), order_item_product_id=int(oi[2]), order_item_quantity=int(oi[3]), order_item_subtotal=float(oi[4]), order_item_product_price=float(oi[5])))
order_items_df = sqlContext.createDataFrame(order_items)
order_items_df.registerTempTable("order_items_df")
```

Finally we can run the same query we did with the Hive context but using the 
temporary tables:

```
joinDF_sql = sqlContext.sql(
"SELECT o.order_date, sum(oi.order_item_subtotal) as rev, count(distinct o.order_id) FROM orders_df o JOIN order_items_df oi on o.order_id=oi.order_item_order_id GROUP BY o.order_date ORDER BY o.order_date"
)
```

## Aggregating datasets

In this section I will cover briefly how to aggregate 
dataset to make, counts, sums...

I will split the section into two sub-sections: one
covering aggregation function for RDDs and other for
aggregation functions for Dataframes.

### Using RDDS

In this section I will cover aggregation functions used 
provided by Spark to work with RDDs. 

Let's assume we want to count the number of distinct orders,
the total revenue, and the number of orders by status.

#### Counting orders

```
# Let's load a RDD containing all orders
order_i_RDD = sc.textFile("/user/cloudera/sqoop_import/order_items")

# Get orders_id and count them 
total_orders = order_i_RDD.map(lambda rec: int(rec.split(",")[1])).distinct().count()
```

#### Calculating revenue

```
# Based on the previous RDD we can now calculate revenue as follows:
total_rev = order_i_RDD.map(lambda rec: float(rec.split(",")[4])).reduce(lambda acc, val: acc+val)

# We can use reduceByKey or GroupByKey as follows:
total_rev = order_i_RDD.map(lambda rec: (1, float(rec.split(",")[4]))).reduceByKey(lambda acc, val: acc+val)
total_rev = order_i_RDD.map(lambda rec: (1, float(rec.split(",")[4]))).groupByKey().map(lambda t: (t[0], sum(t[1])))

# Note have used the same key in the mapping so everything reduce to one value when summing.
```

#### Counting orders by status

```
# READ orders RDD
orders_RDD = sc.textFile("/use/cloudera/sqoop_import/order_items")

# Parse lines
orders_map = orders_RDD.map(lambda rec: (rec.split(",")[3], 1))

# Using reduce By Key
orders_by_status = orders_map.reduceByKey(lambda acc, val: acc+val)

# Using group By Key
orders_by_status = orders_map.groupByKey().map(lambda t: (t[0], sum(t[1])))

# Using countByKey
orders_by_status = orders_map.countByKey()
```

### Using Dataframes

In this section we will cover the same exercises but using Spark Dataframes instead of
RDDs:

#### Counting orders

```
order_items_df = sqlCtx.sql("SELECT * FROM order_items")

total_orders = order_items_df.select(order_items_df.order_item_order_id).distinct().count()
```

#### Calculating revenue
```
# Based on the previous RDD we can now calculate revenue as follows:
total_rev = order_items_df.agg({'order_item_subtotal': 'sum'})

# Another option is to import Pyspark SQL functions
from pyspark.sql import functions as F
total_rev = order_items_df.agg(F.sum(order_items_df.order_item_subtotal).alias('Total Revenue'))

# Show results:
total_rev.show()

+--------------------+
|       Total revenue|
+--------------------+
|3.4322619930019915E7|
+--------------------+
```
#### Counting orders by status

```
orders_df = sqlCtx.sql("SELECT * FROM orders")

orders_by_status = orders_df.groupBy(orders_df.order_status).agg(F.count(orders_df.order_status).alias("Order by status"))

# Show results:
orders_by_status.show()

+---------------+---------------+                                               
|   order_status|Order by Status|
+---------------+---------------+
|        PENDING|           7610|
|        ON_HOLD|           3798|
| PAYMENT_REVIEW|            729|
|PENDING_PAYMENT|          15030|
|     PROCESSING|           8275|
|         CLOSED|           7556|
|       COMPLETE|          22899|
|       CANCELED|           1428|
|SUSPECTED_FRAUD|           1558|
+---------------+---------------+
```

## More complicated aggregations

In this section we will continue calculating some aggregations
but using more complicated functions like `aggregateByKey`
or `combineByKey` when dealing with RDDs. The goal of this 
section is to calculate:

1. Revenue per day.
2. Average revenue per day.

### Using RDDs

#### Revenue per day

Let's assume we have loaded as RDDs the tables `orders` and `order_items`. 
We will map order_items so the value `order_item_order_id` is set as key in order
to join both tables using `order_id` as matching key.

```
# Load data from HDFS
ordersRDD = sc.textFile('/user/cloudera/sqoop_import/orders').map(lambda rec: (rec.split(",")[0], rec))
order_itemsRDD = sc.textFile('/user/cloudera/sqoop_import/order_items/').map(lambda rec: (rec.split(",")[1], rec))

# Join both datasets.
ordersJoined = order_itemsRDD.join(ordersRDD)

# Map resulting dataset to get (date, subtotal)
ordersJoinedMap = ordersJoined.map(lambda t: (t[1][1].split(",")[1], float(t[1][0].split(",")[4])))

# Calculate revenue using Reduce By Key
revenue_per_day = ordersJoinedMap.reduceByKey(lambda acc, val: acc+val)

# Print results
for i in revenue_per_day.sortByKey().collect():
	print(i)
	
...
(u'2014-07-19 00:00:00.0', 108439.13999999991)
(u'2014-07-20 00:00:00.0', 141499.79999999993)
(u'2014-07-21 00:00:00.0', 121102.76999999993)
(u'2014-07-22 00:00:00.0', 73134.359999999957)
(u'2014-07-23 00:00:00.0', 87990.379999999976)
(u'2014-07-24 00:00:00.0', 97076.339999999938)
```

#### Average revenue per order and day

In this part of the exercise we will try to calculate the average revenue per order and 
day. We will start using the same joined RDDs:

```
ordersRDD = sc.textFile('/user/cloudera/sqoop_import/orders').map(lambda rec: (rec.split(",")[0], rec))
order_itemsRDD = sc.textFile('/user/cloudera/sqoop_import/order_items/').map(lambda rec: (rec.split(",")[1], rec))

# Join both datasets.
ordersJoined = order_itemsRDD.join(ordersRDD)

# In this case the key is to be a tuple (date, order_id) and the value will be the sub total
ordersJoinedMap = ordersJoined.map(lambda t: ((t[1][1].split(",")[1], t[0]), float(t[1][0].split(",")[4])))

# If we want to calculate the average revenue per day and order we cannot use
# reduceByKey in this case. Let's try aggregateByKey and combineByKey
revenue_interim = ordersJoinedMap.aggregateByKey(
	(0, 1), 
	lambda acc, val: (acc[0] + val, acc[1] + 1),
	lambda acc, val: (acc[0] + val, acc[1] + 1)
	)

revenue_interim = ordersJoinedMap.combineByKey(
	lambda x: (x, 1),
	lambda acc, val: (acc[0] + val, acc[1] + 1),
	lambda acc, val: (acc[0] + val, acc[1] + 1)
	)

# We can finally use map to calculate the average revenue per day and order
revenue_day_order = revenue_interim.map(lambda r: (r[0], r[1][0]/r[1][1]))

```

### Using Dataframes

In this section we will cover the same exercise but using
Spark dataframes instead of RDDs


#### Revenue per day

We will load dataframes from Hive:

```
# Load dataframes from sql
orders_df = sqlCtx.sql("SELECT * FROM orders")
order_items_df = sqlCtx.sql("SELECT * FROM order_items")

# Now calculating the revenue per day may be accomplished in
# just one command.
revenue_per_day = (order_items_df
	.join(orders_df, orders_df.order_id==order_items_df.order_item_order_id)
	.groupBy(orders_df.order_date)
	.agg({'order_item_subtotal': 'sum'})
	.orderBy('order_date')
	)

for i in revenue_per_day.collect():
	print(i)
	
...

Row(order_date=u'2014-07-19 00:00:00.0', sum(order_item_subtotal)=108439.14000000035)
Row(order_date=u'2014-07-20 00:00:00.0', sum(order_item_subtotal)=141499.80000000034)
Row(order_date=u'2014-07-21 00:00:00.0', sum(order_item_subtotal)=121102.77000000041)
Row(order_date=u'2014-07-22 00:00:00.0', sum(order_item_subtotal)=73134.360000000102)
Row(order_date=u'2014-07-23 00:00:00.0', sum(order_item_subtotal)=87990.380000000208)
Row(order_date=u'2014-07-24 00:00:00.0', sum(order_item_subtotal)=97076.340000000157)
```

#### Revenue per day and order

This case is not that different from the previous one, let's see:

```
# Import functions
from pyspark.sql import functions as F

# Load dataframes from sql
orders_df = sqlCtx.sql("SELECT * FROM orders")
order_items_df = sqlCtx.sql("SELECT * FROM order_items")

# Now calculating the revenue per day may be accomplished in
# just one command.
revenue_per_day_order = (order_items_df
	.join(orders_df, orders_df.order_id==order_items_df.order_item_order_id)
	.groupBy([orders_df.order_date, orders_df.order_id])
	.agg((F.sum('order_item_subtotal')/F.count('order_id')).alias('rev_day_order'))
	.orderBy('order_date')
	)

for i in revenue_per_day_order.collect():
	print(i)
	
...
Row(order_date=u'2014-07-24 00:00:00.0', order_id=57693, rev_day_order=39.990000000000002)
Row(order_date=u'2014-07-24 00:00:00.0', order_id=57694, rev_day_order=199.97999999999999)
Row(order_date=u'2014-07-24 00:00:00.0', order_id=57695, rev_day_order=39.979999999999997)
Row(order_date=u'2014-07-24 00:00:00.0', order_id=57696, rev_day_order=157.465)
Row(order_date=u'2014-07-24 00:00:00.0', order_id=57697, rev_day_order=149.94)
```

## Getting max values

In this section we will cover the example in which we want
to find the customer with the maximum revenue per day. We 
will do this exercise using RDDs first, and then using Dataframes.


### Using RDDs

As in the previous exercises we will cover
load the `orders` and `order_items` RDDs from HDFS:

```
ordersRDD = sc.textFile('/user/cloudera/sqoop_import/orders').map(lambda rec: (rec.split(",")[0], rec))
order_itemsRDD = sc.textFile('/user/cloudera/sqoop_import/order_items/').map(lambda rec: (rec.split(",")[1], rec))

# Join both datasets.
ordersJoined = order_itemsRDD.join(ordersRDD)

# Map the joined data and sum revenue per customer and day
revenueDayCustomer = ordersJoined.map(lambda rec: ((rec[1][1].split(",")[1], rec[1][1].split(",")[2]), float(rec[1][0].split(",")[4]))).reduceByKey(lambda acc, val: acc+ val)

# We have now revenue per day and custoemr
for i in revenueDayCustomer.take(4):
	print(i)
... 
((u'2014-02-22 00:00:00.0', u'7155'), 399.98000000000002)
((u'2013-10-21 00:00:00.0', u'11284'), 1364.8900000000001)
((u'2013-10-27 00:00:00.0', u'11875'), 699.96000000000004)
((u'2014-05-12 00:00:00.0', u'10125'), 463.96000000000004)

# Now we can map again the RDDs as follows:
revenueDayCustomerMap = revenueDayCustomer.map(lambda rec: (rec[0][0], (rec[0][1], rec[1])))

# Finally, we will claculate the customer with the maximum 
# revenue using reduceByKey
topCustomerPerDay = revenueDayCustomerMap.reduceByKey(lambda acc, val: (acc if acc[1] >= val[1] else val))

for i in topCustomerPerDay.sortByKey().collect():
	print(i)
	
...
(u'2014-07-20 00:00:00.0', (u'4956', 1629.76))
(u'2014-07-21 00:00:00.0', (u'5864', 1689.8600000000001))
(u'2014-07-22 00:00:00.0', (u'7988', 1749.8900000000001))
(u'2014-07-23 00:00:00.0', (u'5533', 2149.9899999999998))
(u'2014-07-24 00:00:00.0', (u'3464', 1825.75))
```

### Using Dataframes

In this subsection we will try to find the customers with maximum revenues
per day using Spark Dataframes

```
# Import functions
from pyspark.sql import functions as F

# Load dataframes from sql
orders_df = sqlCtx.sql("SELECT * FROM orders")
order_items_df = sqlCtx.sql("SELECT * FROM order_items")

max_rev_customer = (order_items_df
	.join(orders_df, order_items_df.order_item_order_id==orders_df.order_id)
	.groupBy(['order_date', 'order_customer_id'])
	.agg(F.sum('order_item_subtotal').alias('Revenue_per_customer'))
	.groupBy('order_date')
	.agg(F.max('Revenue_per_customer').alias('Max_Revenue'))
	.orderBy('order_date')
	)

for i in max_rev_customer.collect():
	print(i)
...
Row(order_date=u'2014-07-20 00:00:00.0', Max_Revenue=1629.76)
Row(order_date=u'2014-07-21 00:00:00.0', Max_Revenue=1689.8600000000001)
Row(order_date=u'2014-07-22 00:00:00.0', Max_Revenue=1749.8900000000001)
Row(order_date=u'2014-07-23 00:00:00.0', Max_Revenue=2149.9899999999998)
Row(order_date=u'2014-07-24 00:00:00.0', Max_Revenue=1825.75)
```

## Filtering data

In this section I will cover briefly how to filter a RDD or dataframe
with Spark.

### Using RDDs

We will start working with the table `orders` already available in the HDFS

```
# Load data 
orders = sc.textFile("/user/cloudera/sqoop_import/orders/")

# Get complete orders
for i in orders.filter(lambda line: line.split(",")[3] == "COMPLETE").take(5): print(i)

# Get all pending orders
for i in orders.filter(lambda line: "PENDING" in line.split(",")[3]).take(50): print(i)
```

Now let's check cancelled orders with amount greater than 1000 $

```
# Load data 
orders = sc.textFile("/user/cloudera/sqoop_import/orders/")
order_items = sc.textFile("/user/cloudera/sqoop_import/order_items/")

# Filter CANCELED orders
orders_parsed = (orders
	.filter(lambda rec: rec.split(",")[3] in "CANCELED")
	.map(lambda rec: (int(rec.split(",")[0]), rec))
		)
# Map order items
order_items_parsed = (order_items
	.map(lambda rec: (int(rec.split(",")[1]), float(rec.split(",")[4])))
		)
# Map order items aggregated
order_items_agg = order_items_parsed.reduceByKey(lambda acc, val: acc + val)

# Join the data
joined = order_items_agg.join(orders_parsed)

# Finally filter order with amount greater than 1000 $
gt_orders = joined.filter(lambda rec: rec[1][0] > 1000)

for i in gt_orders.collect():
	print(i)

```

### Using Dataframes

We will try now to accomplish the same task using this time
Spark Dataframes.

```
# Load data 
orders_df = sqlCtx.sql("SELECT * FROM orders")

# Get complete orders
complete_orders = orders_df.filter(orders_df.order_status=="COMPLETE")
for i in complete_orders.take(100):
	print(i)

# Get all pending orders
pending_orders = orders_df.filter(orders_df.order_status.startswith("PENDING"))
for i in pending_orders:
	print(i)
	
```

Now let's check cancelled orders with amount greater than 1000 $

```
from pyspark.sql import functions as F

# Load data 
orders_df = sqlCtx.sql("SELECT * FROM orders")
order_i_df = sqlCtx.sql("SELECT * FROM order_items")

# Get Canceled orders
orders_can = orders_df.filter(orders_df.order_status=="CANCELED")
orders_i_agg = (order_i_df
	.groupBy('order_item_order_id')
	.agg(F.sum('order_item_subtotal').alias('revenue'))
	)
	
joined = (orders_can
	.join(orders_i_agg, orders_i_agg.order_item_order_id==orders_can.order_id)
	.filter(orders_i_agg.revenue > 1000)
	)
	
for i in joined.collect():
	print(i)
```

## Ordering and ranking data

In this section we will cover how to order and rank datasets using Spark
RDDs and Dataframes.

### Using RDDs
### Using Dataframes
