# 03 Sqoop Export

This file covers the content of the videos about Sqoop export available in [this playlist](https://www.youtube.com/watch?v=GyA6lhBIe9g&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE&index=16&spfreload=1).

The aim of this file is to cover basic export from the HDFS to RDBM (MySQL in this case).

## Table of contents

* Export without updating records
* Export considering updates

## Export without updating records

In this first part we will cover the case in which we want to export data from a HDFS directory
to a MySQL table, considering there are no updates to be made in the destination table, which means
only inserts will be performed.

Let's begin with an example. Our goal is to export the Hive table `departments` into the MySQL database 
`retail_db`:

```
sqoop export \ 
-m 2 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments_export \
--export-dir /user/hive/warehouse/sqoop_import.db/departments \
--fields-terminated-by "\01"
```

#### Comments

* `--table` argument tells Sqoop the destination table in the MySQL database.
* `--export-dir` argument tells Sqoop the original directory from which data is to be exported.
* `--fields-terminated-by` argument tells Sqoop which delimiter has been used to store information in
  the HDFS. In general, it's a good practice to explicitly define which delimiter has been used, since
  Hive default delimiter ("\01") may be different from a HDFS default delimiter.

#### Notes on primary key

It's important to note the different outcomes of the previous command depending
on whether the destination table has a primary key or not.

* If the table is not empty, has primary key, and we are exporting records with
  a primary key already present in the table, Sqoop will fail to insert these new records comming from
  the HDFS since they will violate the primary key constraint. We will see in the following section
  how to update those values already present in the destination table.
* If the table is not empty, and has no primary key, records coming from the HDFS matching 
  existing records in the destination table will be added creating duplicates. In this case
  no constraint is violated in the MySQL table since there is no primary key.

## Export considering updates

In this section we will cover the cases where some records are present both in the destination table
and the HDFS directory. Our goal is to add to the MySQL table new records coming from the HDFS
directory and update those already present in the MySQL table. 

More precisely we have the following records in HDFS:

```
10,New record
7,Modified record
```

And the MySQL `departments` table has the followint content:

```
--------------------------------------
| department_id | department_name      | 
--------------------------------------
| 2           | Fitness              | 
| 3           | Footwear             | 
| 4           | Apparel              | 
| 5           | Golf                 | 
| 6           | Outdoors             | 
| 7           | Fan Shop             | 
| 8000        | TESTING              | 
--------------------------------------
```

If we want to add new records to the MySQL table and update those
modified we could proceed with the following command:

```
sqoop export \
-m 2 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--export-dir /user/cloudera/sqoop_export/departments/ \
--update-key department_id \
--update-mode allowinsert
```

* `--update-key` This is the column Sqoop will check to look for updates. If records comming from 
  the HDFS already exists in the MySQL table, they will be used to update existing values in MySQL. 
  **This column must be a primary key, otherwise values will not be updated**, but inserted
  if `--update-mode` is set to `allowinsert`.
* `--update-mode` Specify the mode to be used when exporting data from the HDFS. 
  Two options are allowed:
  * `allowinsert` Sqoop will update exising values and insert new values in the MySQL table.
  * `updateonly` This is the default value, if used then only updates will be performed. New values
	will not be inserted in the MySQL database.
	
After executing the previous command the content of the table `departments` in MySQL looks like this:

```
--------------------------------------
| department_id | department_name      | 
--------------------------------------
| 2           | Fitness              | 
| 3           | Footwear             | 
| 4           | Apparel              | 
| 5           | Golf                 | 
| 6           | Outdoors             | 
| 7           | Modified record      | 
| 10          | New record           |
| 8000        | TESTING              | 
--------------------------------------
```


