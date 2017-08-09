# 06 Sqoop Job and Merge

This is the last file about Sqoop, and it will cover Sqoop jobs and merge options. 
The content of the file is based on [this playlist](https://www.youtube.com/watch?v=PXBbFz9Ty8I&index=21&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE)

## Table of content

* Sqoop job
  * Create job
  * List jobs
  * Show job
  * Run job
* Sqoop merge

## Sqoop job

Sqoop allows to save commands for further reuse. Let's assume you have to execute the
same Sqoop command on a daily basis, well, with Sqoop jobs you can save this command
and execute is whenever you want. Let's dive into it!

### Create job

To create a Sqoop job, proceed as follows:

```
sqoop job --create sqoop_job \
-- import \
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db \
--username=retail_dba \
--password=cloudera \
--table departments \
--target-dir /user/cloudera/departments/ \
--append 
```

* In the first line we are using `sqoop job` command along with its argument `--create`
  to create a job called `sqoop_job`.
* After this first line we write the usual command: in this case we are importing a table
  from MySQL into HDFS.
* Note that the main command the job has to execute (`import`) is passed as argument (preceeded by
  two dashes).

### List jobs

If we want to check already created jobs in Sqoop just use the following command:

```
sqoop job --list
```

### Show job

### Run job


## Sqoop merge
