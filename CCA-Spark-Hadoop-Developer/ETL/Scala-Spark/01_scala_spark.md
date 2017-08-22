# 01 Scala Spark

This file covers an introduction to Scala Spark based on [this playlist](https://www.youtube.com/watch?v=iclGhV3s98o&index=53&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Table of contents

* Submitting tasks
* Read and save from text file

## Submitting tasks

First install `sbt`, you can go to [its website](http://www.scala-sbt.org/download.html) 
for installing instructions.

## Read and save from text file

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

# Saving as object file
departments.saveAsObjectFile("/user/cloudera/scala/departments_object/")
```
