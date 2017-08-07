# 02 Sqoop Import

This file covers the content of the videos about Sqoop import available in [this playlist](https://www.youtube.com/playlist?list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE). All exercices and examples are
based on the Cloudera Quickstart Virtual Machine provided by Cloudera.

## Table of contents

* Import all tables
  * Exercise
* Import all tables into Hive
  * Understanding Hive directories and databases structure
  * Import database and compress data
* Import tables using `sqoop-import` 
  * Import your first table
  * How is data distributed among files
  * Using `--boundary-query` to limit import
  * What if there is no primary key?
  * Using `--columns` to specify columns to be imported
  * Using `--where` to limit rows to be imported
  * Using `--query` to more complex imports
* Import tables in Hive using `sqoop-import`:
* Incremental import

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
* `--hive-database`: Specify destination database. If not used the `default` database is to be used.
* `--compress`: Enable compression.
* `--compression-codec`: Hadoop compression codec (default gzip).
* `--outdir`: Output directory for generated code.

Now let's review the output of that command: 

* The first thing we may check is that in the directory where the command was executed there is a 
  new directory `java_files` containing all the code autogenerated by Sqoop to import the tables.
* Then we can check that the `sqoop_import` database has now the tables imported. We can verify this
  with Hive, or exploring the HDFS:
  
  * Connect to Hive, select sqoop_import as database, and show its tables:

	```
	use sqoop_import;
	show tables;
		
	categories
	customers
	departments
	order_items
	orders
	products
	```

  * List all tables in the directory `/user/hive/warehouse/sqoop_import.db`:

	```
	hdfs dfs -ls /user/hive/warehouse/sqoop_import.db
  
    drwxrwxrwx   - cloudera hive          0 2017-08-01 15:31 /user/hive/warehouse/sqoop_import.db/categories
	drwxrwxrwx   - cloudera hive          0 2017-08-01 15:31 /user/hive/warehouse/sqoop_import.db/customers
	drwxrwxrwx   - cloudera hive          0 2017-08-01 15:32 /user/hive/warehouse/sqoop_import.db/departments
	drwxrwxrwx   - cloudera hive          0 2017-08-01 15:32 /user/hive/warehouse/sqoop_import.db/order_items
	drwxrwxrwx   - cloudera hive          0 2017-08-01 15:33 /user/hive/warehouse/sqoop_import.db/orders
	drwxrwxrwx   - cloudera hive          0 2017-08-01 15:33 /user/hive/warehouse/sqoop_import.db/products
	```

* Let's check now how compression has reduced the size of the tables in hive. 
  * From first import, get the size of the uncompressed table `order_items`:
	```
	hdfs dfs -du -s -h /user/cloudera/sqoop_import/order_items
	
	5.2 M  5.2 M  /user/cloudera/sqoop_import/order_items
	```
  * From the compressed Hive import, get the size of the uncompressed table `order_items`:
	```
	hdfs dfs -du -s -h /user/hive/warehouse/sqoop_import.db/order_items
	
	1.8 M  1.8 M  /user/hive/warehouse/sqoop_import.db/order_items
	```
  We verify that the compressed table is almost 3 times smaller that the uncompressed one. 
  
* Let's finally check that we have the same number of rows in the imported Hive table. Since
  we no longer have textfile to be processed with `wc`, we need to this calculation using Hive:
	```
	use sqoop_import;
	SELECT COUNT(*) FROM order_items;
	
	OK
	172198
	```
  We have the same number rows!
		
		
## Import using `sqoop-import`


### Import your first table

Let's start by importing a table into the HDFS:

```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments
```

In the command above we are telling Sqoop to import table `departments` from the database 
`retail_db` into the target directory (`--target-dir`) `/user/cloudera/departments/`. 

Since we are not specifying the number of mapper, Sqoop by default assigns 4 mappers to the processs, 
that's why there are 4 files with data in the target directory: 

```
hdfs dfs -ls /user/cloudera/departments/ 

Found 5 items
-rw-r--r--   1 cloudera cloudera          0 2017-08-06 05:59 /user/cloudera/departments/_SUCCESS
-rw-r--r--   1 cloudera cloudera         21 2017-08-06 05:59 /user/cloudera/departments/part-m-00000
-rw-r--r--   1 cloudera cloudera         10 2017-08-06 05:59 /user/cloudera/departments/part-m-00001
-rw-r--r--   1 cloudera cloudera          7 2017-08-06 05:59 /user/cloudera/departments/part-m-00002
-rw-r--r--   1 cloudera cloudera         22 2017-08-06 05:59 /user/cloudera/departments/part-m-00003
```

### How is data distributed among files

The way Sqoop distributes data among files is based on the primary key it finds in the original 
table. It takes the minimum and the maximum of the primary key and splits that 
range equally according to the number of mappers indicated by the user (4 by default). This 
approach may lead to skewed distribution of data among files if there is a skewed distribution of 
primary keys in the original table.

To ilustrate this let's insert the following row in our table `departments`:

```
sqoop eval \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--query "INSERT INTO departments VALUES (8000, 'TESTING')"
```

Add import the table `departments` again into the HDFS, but this time using 2 mappers. The resulting
folder would look like this:

```
hdfs dfs -ls /user/cloudera/departments/

Found 3 items
-rw-r--r--   1 cloudera cloudera          0 2017-08-06 06:56 /user/cloudera/departments/_SUCCESS
-rw-r--r--   1 cloudera cloudera         60 2017-08-06 06:56 /user/cloudera/departments/part-m-00000
-rw-r--r--   1 cloudera cloudera         13 2017-08-06 06:56 /user/cloudera/departments/part-m-00001
```

Note that the first file (`part-m-00000`) is almost 6 times heavier than the second file. That's
because data has been distributed according to their primary keys: 

* Sqoop takes the minium and maximum primary keys in the original table (1 and 8000 in our example).
* It splits that range equally according to the number of mappers to be used (2 in our example):
  * Which means a first range that goes from 2 to 4001, and a second range that goes from 4002 to 8000.
* It distributes rows among files depending on in which range each row primary key falls. 

The resulting files are the following:

* File `part-m-00000`:
  ```
  2,Fitness
  3,Footwear
  4,Apparel
  5,Golf
  6,Outdoors
  7,Fan Shop
  ```
  
* File `part-m-00001`:
  ```
  8000,TESTING
  ```

### Using `--boundary-query` to limit import

One of the Sqoop options to limit the rows to be imported to HDFS is the
argument `--boundary-query`, which allows the user to especify the range of 
primary keys to be imported. 

Going back to our previous example, if we want to exclude the row `(8000, TESTING)` from
the import process, use the `--boundary-query` as follows:

```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments \
--boundary-query "SELECT 2, 8" \
-m 2
```

As a result we have two files of similar size, with a similar number of rows, and without 
the outlier case `(8000, TESTING)`:

```
hdfs dfs -ls /user/cloudera/departments/

Found 3 items
-rw-r--r--   1 cloudera cloudera          0 2017-08-06 07:39 /user/cloudera/departments/_SUCCESS
-rw-r--r--   1 cloudera cloudera         31 2017-08-06 07:39 /user/cloudera/departments/part-m-00000
-rw-r--r--   1 cloudera cloudera         29 2017-08-06 07:39 /user/cloudera/departments/part-m-00001
```

* File `part-m-00000`:

	```
	2,Fitness
	3,Footwear
	4,Apparel	
	```
	
* File `part-m-00001`:

	```
	5,Golf
	6,Outdoors
	7,Fan Shop
	```

### What if there is no primary key?

So far we've been importing tables with a primary key that Sqoop has used to split the data
among the mappers and files. But what happens when we are to import a table that has no primary key?
Well here we have two options:

* To set the number of mappers to 1 (`-m 1`), so there is no need to split the table.
* To use the argument `--split-by` which allows the user to specify which column is to be used
  by Sqoop to split the data among mappers and files.


### Using `--columns` to specify columns to be imported

Sqoop provides the argument `--columns` allowing the user
to specify the columns to be imported to the HDFS:


```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments \
--boundary-query "SELECT 2, 8" \
--columns department_name \ 
-m 2 
```

The resulting files would look like this:

```
Fitness
Footwear
Apparel
```

### Using `--where` to limit rows to be imported

This argument allows user limit the rows to be imported to the HDFS imposing conditions on 
the rows to be imported:

Let's import all rows of the table department with a `department_id` less than 9:

```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments \
--where "department_id < 9" \
-m 2
```

#### Note

* The `WHERE` condition is to be imposed as well to the calculation of the range defining 
  the primary keys to be imported. In this case such a range is calculated as follows:
  ```
  SELECT MIN(`department_id`), MAX(`department_id`) FROM `departments` WHERE ( department_id <9 )
  ```

### Using `--query` to more complex imports

So far we have imported columns or tables combining the argments `--columns` and `--table`.
The problem is that these arguments fall short soon if we want to perform more complicated
imports, such as joined, grouped or filtered tables. To accomplish such complex imports
Sqoop provides the argument `--query`.


Let's import a join between the tables `orders` and `order_items`:

```
sqoop import \
-m 4
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--target-dir /user/cloudera/order_join \
--split-by order_id \
--query "SELECT * FROM orders JOIN order_items ON orders.order_id=order_items.order_item_order_id WHERE \$CONDITIONS" 
```

#### Notes

* `--query` argument is incompatible with arguments `--table` and `--columns`.
* `--query` argument needs to be used along with the argument `--split-by`.
* `--query` argument requires to have the `WHERE $CONDITIONS` clause in the query statement to be 
  used by Sqoop to split data among mappers. The user may add more conditions if needed.


## Import tables in Hive using `sqoop-import`

In this section I will cover how to import tables from MySQL to Hive:

Let's assume we want to import the table `departments` from MySQL to Hive:

```
sqoop import \
-m 4
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--hive-import \ 
--hive-table sqoop_import.departments \
--create-hive-table
```

Notes on the previous command:

* `--hive-import` Indicates Sqoop that this tabla is to be imported to Hive.
* `--hive-table` Tells Sqoop the name of the table where data is to be imported. If the table belongs
  to a database then specify the database as `database_name.table_name`.
* `--create-hive-table` Teels Sqoop to create the table while importing. If the table already exists the
  command will fail.
  
What if we have already created a table in Hive and want to add data to that table?

```
sqoop import \
-m 4
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--hive-import \ 
--hive-overwrite \
--hive-table sqoop_import.departments
```

Notes:

* Argument `--create-hive-table` has been removed from the command. Sqoop expects the table 
  `departments` to be already created in Hive.
* `--hive-overwrite` Tells Sqoop to overwrite any data already in that table in Hive. If not set,
  Sqoop will append imported data to existing data.

#### Important Note

While importing to Hive, Sqoop creates a staging temporary table in our user directory in the HDFS 
(`/user/cloudera/` in our case), if a directory with the name of the table to be imported
already exists in that directory Sqoop will fail. So check before importing to Hive, whether
there are directories in the user directory in HDFS with the same name of the tables to
be imported.

## Incremental import

### Using `--append` and `--where` arguments

One option to perform incremental import is:

1. To limit the imports from the original table to the new records using the argument `--where`.
2. To append these new records to the exisint table in the HDFS using the argument `--append`.

Let's see an example. Imagine we have in the HDFS the following records: 

```
2,Fitness
3,Footwear
4,Apparel
5,Golf
6,Outdoors
```

But we have the following new records in our original MySQL table that we would like to import to
the HDFS:

```
--------------------------------------
| department_id | department_name      | 
--------------------------------------
| 7           | Fan Shop             | 
| 8000        | TESTING              | 
--------------------------------------
```

One option would be to use the following command:

```
sqoop import \
-m 1 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments \
--append \
--where "department_id >= 7"
```

* The `--append` argument prevents Sqoop to fail because the destination directory already exists.
* The `--where` argument limit the rows to be imported to those not imported in the HDFS.

### Using `--incremetal`

A more sophisticated way of performing incremental imports is to use the following arguments:

* `--check-column`: Specifies the column to be examined when determining which rows to import.
* `--incremental`: Speicifies how Sqoop determines what rows are new:
  * `append`: To be used when in the original table only inserts are performed 
	(data is not updated)
  * `lastmodified`: To be used when the original table suffers inserts and updates.
* `--last-value`: Specifies the maximum value of the check column from the previous import. 

A good strategy when performing this kind of imports is to have a column in our table where
a timestamp is recorded every time any update or insert is performed on the table, and to 
use this timestamp as check column. This will allow as to compare easily the imported table in the
HDFS and the original table in MySQl.

Let's repeat now the previous exercise but using this time the `--incremental` argument:

```
sqoop import \ 
-m 1 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments \
--check-column department_id \
--incremental append \
--last-value 6
```

* We have set the column `department_id` as check column using the argument `-check-column`.
* Since we only want to append records, and not update them, we are using the option `append` 
  for the argument `--incremental`.
* We tell Sqoop using the argument `--last-value` 
  that the last record imported had a `department_id` equals to 6.
