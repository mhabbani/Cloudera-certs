# 01 Flume

## Table of contents

* Introduction
* Creating your first agent
* Flowing data into HDFS
* Ingest data in real time with Flume

## Introduction

The content of this file is to provide a brief introduction
to [Apache Flume](https://flume.apache.org/FlumeUserGuide.html). 
Apache Flume is a designed to import data from real-time streaming into HDFS.

In order to import data from streaming sources into HDFS 
a Flume agent has to be designed. I won't go into details about
Flume architecture, but the most basic agent in Flume is to
be composed of:

* a source (e.g. logs).
* a channel.
* a sink (HDFS, logs, another Flume agent...)

Depending on the architecturen of the agents we may have multiple channels and sinks.
But for simplicity we will only work with agents containing one channel and one sink.

## Creating your first agent

Agents are defined through a configuration file where
source, channels and sinks are to be configured. The following 
configuration file may be found in the Apache Flume documentation.

```
# telnet_flume.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

* We have defined an agent called `a1`, for which we have defined:
  * A source called `r1`.
	* The source is of type `netcat`.
	* It's to be found in `localhost`.
	* Listening in the port `44444`.
  * A sink called `k1`.
	* The sink is of type `logger`.
  * A channel called `c1`.
	* The channel is to be run in memory.
	* It has a capacity of 1000 (maximum number of events stored in the channel).
	* It has a transaction capacity of 100 (maximum number of events the channel will take
	  from the source or give to a sink per transaction).
* We finally bind source `r1` with channel `c1`, and sink `k1` with channel `c1`.

To run this agent we can use the following command:

```
flume-ng agent 
--name a1 \
--conf ./ \
--conf-file telnet_agent.conf
```

* We define the agent name using `--name`, the agent name has to match the one defined
  in the configuration file.
* We have to specify the route to the directory where the configuration file is located 
  with `--conf`.
* We have to specify as well the route to the configuration file (`--conf-file`).

Once the agent is running we could test it using telnet:

```
telnet localhost 44444
```

If you type now in the telnet terminal you should see
that flume is catching the content in the launched 
agent trace:

```
17/08/10 10:26:13 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
17/08/10 10:28:14 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 57 6F 72 6C 64 0D             Hello World. }


```

I typed `Hello World` in the telnet terminal!

## Flowing data into HDFS

In this section we will cover how to ingest data into HDFS using Flume, which means
we have to change the previous sink (a logger) and use HDFS as sink. So if we redefine
our agent configuration file to use HDFS as sink it will look like the following:

```
# telnet_flume.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = hdfs
# Customizing sink
a1.sinks.k1.hdfs.path = /user/cloudera/flume/
a1.sinks.k1.hdfs.filePrefix = netcat

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

Note the three changes:

1. We have changed the sink type to `hdfs`.
2. We have defined the path of the sink `/user/cloudera/flume/`.
3. We have defined as well the prefix all files imported in HDFS will have `netcat` 
   in this case using the option `hdfs.filePrefix`.

If we run now the agent and type something in telnet, we should see how
flume creates files in HDFS:

```
[cloudera@quickstart flume]$ hdfs dfs -ls /user/cloudera/flume
Found 1 items
-rw-r--r--   1 cloudera cloudera        127 2017-08-10 10:45 /user/cloudera/flume/netcat.1502387110114.tmp
```

Note the file is terminated with the suffix `.tmp`. However after a time (30 seconds to be precise),
we get the following results:

```
[cloudera@quickstart flume]$ hdfs dfs -ls /user/cloudera/flume
Found 1 items
-rw-r--r--   1 cloudera cloudera        127 2017-08-10 10:45 /user/cloudera/flume/netcat.1502387110114
```

Once temporary files reach their roll interval (30 seconds by default), roll size or roll count
they are committed into HDFS, and the `tmp` extension removed. If we try to display the content
of the files we will discover that the content is binary (it's a sequence file). We
can change the format under which data is imported into HDFS using the following option
in the configuration file.

```
a1.sinks.k1.hdfs.fileType = DataStream
```

In this case we are telling Flume to import data as text. So if we now rerun the agent
with this modification we could be able to read the content of the imported files.

## Ingest data in real time with Flume

In this section we will redefine our agent in order to source data from running logs
and use a file as channel.

**NOTE**: We will generate the logs to be ingested using a command provided
by Cloudera in the Quickstart Cloudera Sandbox. These logs simulate the requests
that an e-commerce web receives. To start generating logs we have the command `start_logs`,
and `stop_logs` to stop the process.

The configuration file of our agent looks like this now:

```
# logs_flume.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe source
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /opt/gen_logs/logs/access.log
a1.sources.r1.channels = c1

# Describe Sink
a1.channels.c1.type = FILE
# Maximum size of transaction supported by the channel
a1.channels.c1.capacity = 20000
a1.channels.c1.transactionCapacity = 1000

# Amount of time (in milis) between checkpoints
a1.channels.c1.checkpointInterval 300000

# Describe the sink
a1.sinks.k1.type = hdfs
a1.sinks.k1.channel = c1
a1.sinks.k1.hdfs.path = /user/cloudera/flume/%y-%m-%d
a1.sinks.k1.hdfs.filePrefix = flume-%y-%m-%d
a1.sinks.k1.hdfs.rollSize = 1048576
a1.sinks.k1.hdfs.rollCount = 100
a1.sinks.k1.hdfs.rollInterval = 120
a1.sinks.k1.hdfs.fileType = DataStream
a1.sinks.k1.hdfs.idleTimeout = 10
a1.sinks.k1.hdfs.useLocalTimeStamp = true
```

* Comments on the source:
  * We have change the type of the source to `exec` which means 
	we need to provide a command to be executed.
  * The command to be executed by the source is `tail -F /opt/gen_logs/logs/access.log`
	which shows the last logs in the `access.log` file as they generate.
* Comments on the channel:
  * We keep the type of the channel as in the previous agent (a file).
  * We have defined the maximum capacity of events to be hold by the file
	and the transaction capacity.
* Comments on sink:
  * We keep HDFS as our sink.
  * We have added the option `useLocalTimeStamp` that allows us
	to use regular expression in the folders and file prefix to be cretad 
	in HDFS.
  * We have as well delimited the size of the files before commit them 
	into HDFS.

We can start the agent using:

```
flume-ng agent 
--name a1 \
--conf ./ \
--conf-file logs_flume.conf
```

And then start generating logs:

```
start_logs
```

To verify Flume is working properly let's check the HDFS directory 
`/user/cloudera/flume/`:

```
drwxr-xr-x   - cloudera cloudera          0 2017-08-10 23:41 /user/cloudera/flume/17-08-10
```

We see flume has created a directory, lets check the content
of that directory:

```
Found 2 items
-rw-r--r--   1 cloudera cloudera       1981 2017-08-10 23:38 /user/cloudera/flume/17-08-10/flume-17-08-10.1502433520132
-rw-r--r--   1 cloudera cloudera        198 2017-08-10 23:41 /user/cloudera/flume/17-08-10/flume-17-08-10.1502433703444.tmp
```

And finally show part of the content of the files imported:

```
192.87.175.186 - - [01/Aug/2014:11:51:44 -0400] "GET /departments HTTP/1.1" 503 1572 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
195.231.2.207 - - [01/Aug/2014:11:51:45 -0400] "GET /department/fitness/products HTTP/1.1" 200 515 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:30.0) Gecko/20100101 Firefox/30.0"
65.62.183.244 - - [01/Aug/2014:11:51:46 -0400] "GET /departments HTTP/1.1" 200 756 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.153 Safari/537.36"
89.92.128.155 - - [01/Aug/2014:11:51:47 -0400] "GET /departments HTTP/1.1" 200 1226 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
30.100.199.8 - - [01/Aug/2014:11:51:48 -0400] "GET /departments HTTP/1.1" 200 768 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_4) AppleWebKit/537.77.4 (KHTML, like Gecko) Version/7.0.5 Safari/537.77.4"
```

We can see how Flume imported correctly the logs generated by our faked web server!
