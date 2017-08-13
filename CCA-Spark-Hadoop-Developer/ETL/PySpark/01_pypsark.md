# 01 PySpark

This file covers an introduction to PySpark based on [this playlist](https://www.youtube.com/watch?v=EGQjqcOIjIM&index=31&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE).

## Table of contents

* Connecting to a database
* Read and save from text file
* Read and save from sequence file
* Read and save from from json files and Hive
  
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
