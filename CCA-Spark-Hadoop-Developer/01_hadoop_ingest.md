# 01 Hadoop commands for ingestion

In this file we will cover how to move data from and to HDFS using Hadoop commands. 
The content of the file is based on [this playlist](https://www.youtube.com/watch?v=NbrDO5Z8IT0&list=PLf0swTFhTI8rJvGpOp-LujOcpk-Rlz-yE&index=28).

## Table of contents

* Introduction
* Import data into HDFS
* Export data from HDFS

## Introduction

We will not cover all Hadoop commands in this section, only those related to 
moving data from local to HDFS, and viceversa. However quick documentation of
about Hadoop commands is available using:

```
$ hdfs dfs -help ls
-ls [-d] [-h] [-R] [<path> ...] :
  List the contents that match the specified file pattern. If path is not
  specified, the contents of /user/<currentUser> will be listed. Directory entries
  are of the form:
  	permissions - userId groupId sizeOfDirectory(in bytes)
  modificationDate(yyyy-MM-dd HH:mm) directoryName
  
  and file entries are of the form:
  	permissions numberOfReplicas userId groupId sizeOfFile(in bytes)
  modificationDate(yyyy-MM-dd HH:mm) fileName
                                                                                 
  -d  Directories are listed as plain files.                                     
  -h  Formats the sizes of files in a human-readable fashion rather than a number
      of bytes.                                                                  
  -R  Recursively list the contents of directories.  
```

In the example above we asked help about the `ls` command. 
Finally, other option is to check [Hadoop Documentation](https://hadoop.apache.org/docs/r2.7.1/hadoop-project-dist/hadoop-common/FileSystemShell.html)

## Import data into HDFS

We've seen so far how to import data into HDFS using Hadoop and Flume. We will
cover in this section how to move local files and datasets to HDFS using
Hadoop commands. In that sense Hadoop provides four commands:

* `put`: This command moves data from local to HDFS. Let's assume we want to move the file `data.txt`
  to the directory `/user/cloudera/data/`:
	```
	hdfs dfs -put ./data.txt /user/cloudera/data/
	```
  The `put` command will copy data to HDFS, which means a copy of the file will remain in the local machine. 
  Let's discuss as well some of the options of this command:
    * `-p` Preserves access and modification times, ownership and the mode.
	* `-f` Overwrites the destination if it already exists.
* `copyFromLocal`: This command is identical to `put`.
* `moveFromLocal`: This command is identical to `put`, except that the source is deleted after it's copied.
* `cp`: This command is not used to copy data from local to HDFS but rather to copy data among HDFS directories.
  Let's assume we want to copy our previously imported file `data.txt`:
  ```
  hdfs dfs -cp /user/cloudera/data/data.txt /user/cloudera/data/data_copied.txt
  ```
* `mv`: This command is identical to `cp`, except that the source is deleted after it's copied.

## Export data from HDFS

We've seen previously how to export data from HDFS using Sqoop. We will cover in 
this section how to move files and datasets from HDFS to our local machine.