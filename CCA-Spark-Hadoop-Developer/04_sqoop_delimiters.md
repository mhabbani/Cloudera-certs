# 04 Sqoop delimiters

This file covers the conent of the videos about Sqoop delimiters available in [this playlist](https://www.youtube.com/watch?v=S4gt5lO8W70&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE&index=18&spfreload=1). 
All exercices and examples are based on the Cloudera Quickstart Virtual Machine provided
by Cloudera.

## Table of contents

* Sqoop import delimiters
  * Enclosing fields
  * Fields delimiters
  * Line delimiters
  * MySQL delimiters
  * Dealing with NULL values

## Sqoop import delimiters

### Enclosing fields

Let's assume we want to import to the HDFS the table `departments` and enclosed
its fields using the character ". Sqoop allows this type of import using the argument
`--enclosed-by`:

```
sqoop import \
-m 2 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/sqoop_import/departments_enclosed \
--enclosed-by \"
```

The resulting imported records look like this:

```
"2","Fitness"
"3","Footwear"
"4","Apparel"
"5","Golf"
"6","Outdoors"
"7","Fan Shop"
```

### Fields delimiters

So far we've been using the default deilimiters when importing with Sqoop:

* `\01` for Hive.
* `,` when not using Hive.

Let's assume now, that we want our imported fields to be separeted by a different character 
(`|` for example). Sqoop allows this feature throught the argument `--fields-terminated-by`:

```
sqoop import \
-m 2 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/sqoop_import/departments_enclosed \
--fields-terminated-by \|
```

The resulting imported files look like this:

```
2|Fitness
3|Footwear
4|Apparel
5|Golf
6|Outdoors
7|Fan Shop
```

### Lines delimiters 

To end this first section, we will see how can we changhe the line delimiter (so 
far we've been using the default value `\n`) using the argument `--line-terminated-by`:

```
sqoop import \
-m 2 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/sqoop_import/departments_enclosed \
--line-terminated-by \:
```

The resulting import lookg like this:

```
2,Fitness:3,Footwear:4,Apparel:5,Golf:6,Outdoors:7,Fan Shop
```

### MySQL delimiters

We've seen that's possible to specify the characters used to enclosed values, and
separte fields, and lines when importing to the HDFS. The arguments described above provide
a lot of flexibility when defining how our import would look like. However, 
Sqoop provides as well the argument `--mysql-delimiters` which tells Sqoop to use 
MySQL delimiters when importing to the HDFS. These delimiters are:

* `,` To separate fields.
* `\n` To separate lines.
* `\` To escape characters.
* `'` To enclose values.

### Dealing with NULL values

So far we've been importing tables without NULL values, so one may be wondering how
are NULL values imported into HDFS. 

First let's see what are the default options in Sqoop. Let's assume we have the following
table to be imported to HDFS:

```
+---------------+-----------------+
| department_id | department_name |
+---------------+-----------------+
|             2 | Fitness         |
|             3 | Footwear        |
|             4 | NULL            |
|          NULL | test_department |
+---------------+-----------------+
```

And we import it to HDFS using the following command: 

```
sqoop import \
-m 1 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments_test \
--target-dir /user/cloudera/sqoop_import/departments_null 
```

The imported records look like this, note the default value for NULL values is `null`:

```
2,Fitness
3,Footwear
4,null
null,test_department
```

Now let's assume we want the NULL values in string columns to be represented as NULL-STRING
and as NULL-VALUE in non-string columns. To do so Sqoop provides the following commands:

* `--null-string`: The string to be written for a null value for string columns.
* `--null-non-string`: The string to be written for a null value for non-string columns.

```
sqoop import \
-m 1 \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments_test \
--target-dir /user/cloudera/sqoop_import/departments_null \
--null-string NULL-STRING
--null-non-string NULL-VALUE
```

The resulting imported data looks like this:

```
2,Fitness
3,Footwear
4,NULL-STRING
NULL-VALUE,test_department
```
