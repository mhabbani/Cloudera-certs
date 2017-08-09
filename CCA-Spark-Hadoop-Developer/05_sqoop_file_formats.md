# 05 Sqoop file formats

This file covers the different file formats under which Sqoop allows data to be imported
into HDFS. The content of this guide comes from 
[this playlist](https://www.youtube.com/watch?v=rZP3xkR_0sU&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE&index=20)

## Table of contents

* Text files
* Sequence files
  * Import into table into HDFS
  * Import into table into Hive
* Avro files
* Comments on exporting

## Text files


## Sequence files

To import to HDFS under sequence format (a binary format) Sqoop provides the argument `--as-sequencefile`. 

### Import table into HDFS

Let's assume we want to import the MySQL table `departments` into HDFS using
formatting files as sequence.

```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/sqoop_import/departments_seq \
--as-sequencefile
```

Since resulting files are in binary format, it makes no sense to show here how they look like.

### Import table into Hive

Let's assume now that we want to import the same table into Hive. 

So far commands `--as-sequencefile` and `--as-avrodatafile` are note compatible
with `--hive-import` which means it is not possible to import tables directly
to Hive using sequence or avro format. However there is a workaround which
consists in importing data to HDFS and then create an external Hive table:

1. We have already imported into HDFS the table departments from MySQL under sequence format.
2. Let's create in Hive an external table using this imported data.

**NOTE**: The following command does not work properly, it is to be corrected.

```
CREATE EXTERNAL TABLE departments_seq (department_id INT, department_name STRING) 
ROW FORMAT SERDE 'com.cloudera.sqoop.contrib.FieldMappableSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.SequenceFileInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat'
LOCATION '/user/cloudera/sqoop_import/departments_seq/';
```

## Avro files

Although I will not go into details describing Avro format, I will just
describe them as quite similar to json files. More information 
on Avro format may be found [here](https://avro.apache.org/docs/current/)


## Comments on exporting

