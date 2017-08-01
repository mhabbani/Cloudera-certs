# 02 Sqoop Import

This file covers the content of the videos about Sqoop import available in [this playlist](https://www.youtube.com/playlist?list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE). All exercices and examples are
based on the Cloudera Quickstart Virtual Machine provided by Cloudera.

## Table of contents

* Import all tables
  * Exercise
* Import all tables into Hive


## Import all tables

Import all table from a MySQL database to Hadoop.

```
sqoop import-all-tables \
-m 12
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba
--as-avrodatafile
--warehouse-dir=/user/hive/warehouse/retail_stage.db
```

* `-m` Indicates the number of threads to be used by table, default is 4.
* `--as-avrodatatile` Implies data will be stored in HDFS under [Avro](https://avro.apache.org/) format. If no format is specified then textfile format is to be used.
* `--warehouse-dir` Identifies the place where data is to be located in HDFS (this should point to an existing HDFS location).

### Exercise:

Let's import all tables from MySQL to HDFS, as text files, using comma as field separators, and verify
the all rows were correctly imported: 

#### 1. Import all tables 

Default separator and file format are comma and text file, so there is no need for specification.

```
sqoop import-all-tables \
-m 12
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba
--warehouse-dir=/user/cloudera/sqoop_import
```

If the folder `/user/cloudera/sqoop_import` is not created in the HDFS, you can create it with:

```
hdf dfs -mkdir /user/cloudera/sqoop_import
```

#### 2. Verify import process

Check all tables have been imported:

```
hdfs dfs -ls /user/cloudera/sqoop_import/

drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:44 /user/cloudera/sqoop_import/categories
drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:46 /user/cloudera/sqoop_import/customers
drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:47 /user/cloudera/sqoop_import/departments
drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:49 /user/cloudera/sqoop_import/order_items
drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:51 /user/cloudera/sqoop_import/orders
drwxr-xr-x   - cloudera cloudera          0 2017-08-01 09:53 /user/cloudera/sqoop_import/products
```

For simplicity we will focus only in the table `order_items`. If we list the files in the folder 
`/user/cloudera/sqoop_imports/order_items` we will see that the table `order_items`
has been broken down into 12 files since we used 12 mappers during the import process. We
can have a look at one of these files by:

```
hdfs dfs -tail /user/cloudera/sqoop_import/order_items/part-m-00000

14343,5760,403,1,129.99,129.99
14344,5760,191,2,199.98,99.99
14345,5761,1073,1,199.99,199.99
14346,5761,957,1,299.98,299.98
14347,5762,403,1,129.99,129.99
14348,5763,403,1,129.99,129.99
14349,5764,627,1,39.99,39.99
14350,5765,502,4,200.0,50.0
```

Note that the resulting file is basically a comma separated value (CSV) file.

To check that all rows have been correctly imported let's count the number of rows in the original
MySQL database, and in the imported files.

Count rows in a HDFS table:

```
hdfs dfs -cat /user/cloudera/sqoop_import/order_items/part-m-* | wc -l

172198
```

Count rows in a MySQL table using Sqoop:

```
sqoop eval \
--connect "jdbc:mysql://quickstart.cloudera/retail_db" \
--username retail_dba \ 
--password cloudera \ 
--query "SELECT COUNT(*) FROM order_items"

------------------------
| COUNT(*)             | 
------------------------
| 172198               | 
------------------------
```

We get the same results!


## Import all tables into Hive

### Understanding Hive directories and databases structure

* Hive data is to be found in the directory `/user/hive/warehouse/`
* All folders inside `/user/hive/warehouse/` with extension `.db` are databases.
* Folders without the `.db` extension are tables.

If we start a Hive terminal (just type `hive`) we can see all available databases in Hive:

```
show databases;

OK
default
sqoop_import
Time taken: 0.724 seconds, Fetched: 2 row(s)
```

Let's check now what's the folder architecure: 

```
hdfs dfs -ls /user/hive/warehouse

drwxrwxrwx   - cloudera hive          0 2017-08-01 10:38 /user/hive/warehouse/sqoop_import.db
```

We see the database `sqoop_import`, but we don't see the `default` database, and that's because
in Hive the `default` database is `/user/hive/warehouse/`.

### Import database and compress data

Now we are going to import the MySQL `retail_db` database to Hive and compress the files in the HDFS 
using Sqoop:

```
sqoop import-all-tables \
--m 1 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--hive-import \
--hive-overwrite \
--hive-database "sqoop_import"
--create-hive-table \
--compress \
--compression-codec org.apache.hadoop.io.compress.SnappyCodec \
--outdir java_files
```

Let's review that command step by step:

* We are using just one mapper `-m 1`
* `--hive-import`: this argument indicates Sqoop that we are importing the tables to Hive, so there
  is no need for a `--warehouse-dir`
* `--hive-overwrite`: this argument makes Sqoop overwrite existing data in Hive tables.
* `--create-hive-table`: If set, then the job will fail if the target hive
  table exits. By default this property is false.
* `--hive-database`: Specify destination database.
* `--compress`: Enable compression.
* `--compression-codec`: Hadoop compression codec (default gzip).
* `--outdir`: Output directory for generated code.

Now let's review the output of that command: 

* The first thing may check is that in the directory where the command was executed there is a 
  new directory `java_files` containing all the code autogenerated by Sqoop to import the tables.

