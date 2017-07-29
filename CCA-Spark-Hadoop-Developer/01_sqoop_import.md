# 01 Sqoop Import

This files covers the content of this [tutorial](https://www.youtube.com/watch?v=hY9nnU4PTFw&index=9&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Import data from MySQL to HDFS using Sqoop

* List what's in the HDFS folder `/user/cloudera`

```
hadoop fs -ls /user/cloudera
```

* Create a folder where imported files will be stored

```
hadoop fs -mkdir /user/cloudera/sqoop_import
```

* To get some help on Sqoop commands type:

```
sqoop help
```

* List databases in MySQL (a jdbc driver should be available to connect to MySQL):

```
sqoop list-databases \
--connect "jdbc:mysql://quickstart.cloudera:3306" \
--username retail_dba \
--password cloudera
```

* List tables in a MySQL database:

```
sqoop list-tables \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
--password cloudera
```
Note that the name of the database hass to be included in the connection url.

