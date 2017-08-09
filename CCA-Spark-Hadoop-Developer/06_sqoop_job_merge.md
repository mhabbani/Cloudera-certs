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
--connect "jdbc:mysql://quickstart.cloudera:3306/retail_db" \
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

Obtaining the following result:

```
Available jobs:
  sqoop_job
```

### Show job

If we want to check the details of a particular job we can proceed as follows:

```
sqoop job --show sqoop_job
```

It may ask for a password, which is `true` for the Cloudera Quickstart virtual machine. 
The output of the previouse command is a detailed view of the job.

### Run job

Finally if you want to execute the job simply execute the following command:

```
sqoop job --exec sqoop_job
```

It may ask for the MySQL password to proceed.

**TODO** Make Sqoop jobs paramtetric

## Sqoop merge

In this section we will cover how to merge two datasets in HDFS. Let's
assume we have imported the following table in HDFS (`/user/cloudera/merge/departments/`):

```
2,Fitness
3,Footwear
4,Apparel
5,Golf
6,Outdoors
7,Fan Shop
10,New record
12,New record
8000,TESTING
```

But in the original MySQL table we have inserted and updated the following values:

```
+---------------+---------------------+
| department_id | department_name     |
+---------------+---------------------+
|            12 | modified record     |
|         10000 | New Record to Merge |
+---------------+---------------------+
```

We then have imported these different files into HDFS in the directory 
(`/user/cloudera/merge/departments_delta/`), and want to merge 
this changes with the previously imported directory. So the final result
should contains:

* The row with `department_id` 12 updated.
* The row with `department_id` 10000 inserted.

To do so, we will run the following command:

```
sqoop merge --merge-key department_id \
--new-data /user/cloudera/merge/departments_delta \
--onto /user/cloudera/merge/departments \
--target-dir /user/cloudera/merge/departments_stage \
--class-name departments \
--jar-file /path/to/jar/file
```

* `--merge-key`: Specify the name of a column to be used as the merge key.
* `--new-data`: Where the new or delta data is located.
* `--target-dir`: Directory where the output is to be located.
* `--onto`: Directory where the original data is located.
* `--class-name`: Specify the name of the record-specific class to use during the merge job.
* `--jar-file`: Specify the name of the jar to load the record class from.

Finally check the results in the directory `/user/cloudera/merge/departments_stage`:

```
10,New record
10000,New Record to Merge
12,modified record
2,Fitness
3,Footwear
4,Apparel
5,Golf
6,Outdoors
7,Fan Shop
8000,TESTING
```

We can verify that Sqoop has successfully completed the merge!
