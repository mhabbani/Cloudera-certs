# 01 Sqoop Import

This files covers the content of this [tutorial](https://www.youtube.com/watch?v=hY9nnU4PTFw&index=9&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Table of contents

* Initial Hadoop comands
* Sqoop exploratory commands
* Sqoop evaluation commands

## Initial Hadoop commands 

* List what's in the HDFS folder `/user/cloudera`

```
hadoop fs -ls /user/cloudera
```

* Create a folder where imported files will be stored

```
hadoop fs -mkdir /user/cloudera/sqoop_import
```

## Initial exploratory commands

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

* Hiding database password. Up to now we've been harcoding the password in every command
  which may lead to security issues. We can hide the password using the command `-P`:

```
sqoop list-tables \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
-P
```


## Sqoop evaluation commands

* Execute a query in a MySQL database:

```
sqoop eval \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
--password cloudera \
--query "SELECT * FROM departments"
```

Not only `SELECT` statements are allowed, you can execute `UPDATE, DELETE, INSERT` commands as well:

* INSERT STATEMENT:

```
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
--password cloudera \
--query "INSERT INTO departments VALUES (8, 'New Department')"
```

* UPDATE STATEMENT:

```
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
--password cloudera \
--query "UPDATE departments SET department_name = 'Updated Department' WHERE deparment_id=8"
```

* DELETE STATEMENT:
```
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
--username retail_dba \
--password cloudera \
--query "DELETE FROM departments WHERE department_id=8"
```

