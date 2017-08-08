# 03 Sqoop Export

This file covers the content of the videos about Sqoop export available in [this playlist](https://www.youtube.com/watch?v=GyA6lhBIe9g&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE&index=16&spfreload=1).

The aim of this file is to cover basic export from the HDFS to RDBM (MySQL in this case).

## Table of contents

* Export table without updating content
* Export table updating content

## Export table without updating content

In this first part we will cover the case in which we want to export data from a HDFS table
to a MySQL table, considering there are no updates to be made in the destination table, which means
only inserts will be performed.

Let's begin with an example. Our goal is to export the Hive table `departments` into the MySQL database 
`retail_db`:

```
sqoop export \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--export-dir /user/hive/warehouse/sqoop_import.db/departments \
--m 2 \
```
