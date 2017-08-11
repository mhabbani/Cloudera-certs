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

**TODO**: Hive command to create external table from Sequence import


## Avro files

Although I will not go into details describing Avro format, I will just
describe them as quite similar to json files. More information 
on Avro format may be found [here](https://avro.apache.org/docs/current/).

### Import table into HDFS

Let's import now the MySQL table `departments` into HDFS as avro files using
the argument `--as-avrodatafile`:

```
sqoop import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/sqoop_import/departments_avro \
--as-avrodatafile
```

Once the import has terminated you should have a `.avsc` file 
in the directory you executed the command. These files contain
information about the structure of the imported data, they contain
the schema of the imported table:

```
{
  "type" : "record",
  "name" : "sqoop_import_departments",
  "doc" : "Sqoop import of departments",
  "fields" : [ {
	"name" : "department_id",
	"type" : [ "int", "null" ],
	"columnName" : "department_id",
	"sqlType": : "4"
  }, { 
	"name" : "department_name",
	"type" : [ "string", "null" ],
	"columnName" : "department_name",
	"sqlType" : "12"
  } ],
  "tableName" : "departments"
}
```

### Import table into Hive

Again, we cannot use the argument `--as-avrodatafile` along with
the `--hive-import` so we need to create an external table in Hive from
what we have already imported into HDFS. 

To do so we first need to move the avro schema (the file with extension `.avsc`) to 
HDFS and then create an external Hive table using the following command:

```
CREATE EXTERNAL TABLE departments 
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.avro.AvroSerDe'
STORED AS INPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.avro.AvroContainerOutputFormat'
LOCATION 'hdfs:///user/cloudera/sqoop_import/departments_avro'
TBLPROPERTIES ('avro.schema.url'='hdfs://quickstart.cloudera/user/cloudera/sqoop_import_departments.avsc');
```

## Comments on exporting

Finally I'd like to comment that there is no need to specify the 
delimiters when exporting squence and avro files. The delimiters have
to be specified when dealing with plain text files.
