# Enable History Server with HDFS

- [Install HDFS with Hadoop 2.6.x](#install-hdfs-with-hadoop-26x)
- [Configure Hadoop and Yarn](#configure-hadoop-and-yarn)
    - [Configure /etc/profile](#configure-etcprofile)
    - [Configure JAVA_HOME in hadoop-env.sh and yarn-env.sh](#configure-javahome-in-hadoop-envsh-and-yarn-envsh)
    - [Configure ./hadoop-2.6.5/etc/hadoop/**core-site.xml**](#configure-hadoop-265etchadoopcore-sitexml)
    - [Configure ./hadoop-2.6.5/etc/hadoop/**hdfs-site.xml**](#configure-hadoop-265etchadoophdfs-sitexml)
    - [Configure ./hadoop-2.6.5/etc/hadoop/**mapred-site.xml**](#configure-hadoop-265etchadoopmapred-sitexml)
    - [Configure ./hadoop-2.6.5/etc/hadoop/**yarn-site.xml**](#configure-hadoop-265etchadoopyarn-sitexml)
    - [Configure the ./hadoop-2.6.5/etc/hadoop/slaves](#configure-the-hadoop-265etchadoopslaves)
    - [Distribute Hadoop dir into nodes](#distribute-hadoop-dir-into-nodes)
- [Start HDFS](#start-hdfs)
    - [Format and start Namenode](#format-and-start-namenode)
    - [Verify the HDFS daemons](#verify-the-hdfs-daemons)
    - [Verify health by CLI](#verify-health-by-cli)
    - [Verify health by Hadoop GUI](#verify-health-by-hadoop-gui)
    - [Hadoop fs commands](#hadoop-fs-commands)
- [Enable Spark History Server](#enable-spark-history-server)

## Install HDFS with Hadoop 2.6.x

Download HDFS package and decompress

http://hadoop.apache.org/releases.html

```shell
wget http://apache.fayea.com/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz
tar -zxvf hadoop-2.6.5.tar.gz
chown -R hadoop:hadoop hadoop-2.6.5
```
Create directories

```shell
cd hadoop-2.6.5/
mkdir tmp
mkdir name
mkdir data
```
## Configure Hadoop and Yarn
Refer to http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/ClusterSetup.html

Edit the configuration files of Hadoop under `./hadoop-2.6.5/etc/hadoop`

### Configure /etc/profile

```shell
export HADOOP_HOME=/myhome/hadoop/hadoop-2.6.5
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
```

### Configure JAVA_HOME in hadoop-env.sh and yarn-env.sh

```shell
vim /myhome/hadoop/hadoop-2.6.5/etc/hadoop/hadoop-env.sh
vim /myhome/hadoop/hadoop-2.6.5/etc/hadoop/yarn-env.sh
export JAVA_HOME=/pcc/app/Linux_jdk1.7.0_x86_64
```

### Configure ./hadoop-2.6.5/etc/hadoop/**core-site.xml**

```xml
<configuration>
  <property>
    <name>fs.defaultFS</name>
    <value>hdfs://9.111.159.156:9000</value>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>file:/myhome/hadoop/hadoop-2.6.5/tmp</value>
    <description>Abaseforothertemporarydirectories.</description>
  </property>
    <property>
    <name>hadoop.proxyuser.spark.hosts</name>
  <value>*</value>
  </property>
  <property>
    <name>hadoop.proxyuser.spark.groups</name>
    <value>*</value>
  </property>
</configuration>
```
### Configure ./hadoop-2.6.5/etc/hadoop/**hdfs-site.xml** 
```xml
<configuration>
  <property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>9.111.159.156:9001</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/myhome/hadoop/hadoop-2.6.5/name</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/myhome/hadoop/hadoop-2.6.5/data</value>
  </property>
  <property>
    <name>dfs.replication</name>
    <value>2</value>
  </property>
  <property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
  </property>
</configuration>
```
### Configure ./hadoop-2.6.5/etc/hadoop/**mapred-site.xml** 
```xml
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.address</name>
    <value>9.111.159.156:10020</value>
  </property>
  <property>
    <name>mapreduce.jobhistory.webapp.address</name>
    <value>9.111.159.156:19888</value>
  </property>
</configuration>
```
### Configure ./hadoop-2.6.5/etc/hadoop/**yarn-site.xml** 
```xml
<configuration>
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>9.111.159.156:8032</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>9.111.159.156:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>9.111.159.156:8031</value>
  </property>
  <property>
    <name>yarn.resourcemanager.admin.address</name>
    <value>9.111.159.156:8033</value>
  </property>
  <property>
    <name>yarn.resourcemanager.webapp.address</name>
    <value>9.111.159.156:8088</value>
  </property>
</configuration>
```
### Configure the ./hadoop-2.6.5/etc/hadoop/slaves
```
9.111.159.156
9.111.159.163
9.111.159.164
```
### Distribute Hadoop dir into nodes
```shell
cd /myhome/hadoop
scp -r hadoop-2.6.5/ hadoop@9.111.159.163://myhome/hadoop/
scp -r hadoop-2.6.5/ hadoop@9.111.159.164://myhome/hadoop/
```

## Start HDFS

### Format and start Namenode

```shell
cd /myhome/hadoop/hadoop-2.6.5
./bin/hdfs namenode -format
cd /myhome/hadoop/hadoop-2.6.5/sbin
./start-dfs.sh
```

### Verify the HDFS daemons

```shell
[hadoop@bjqilitst1 sbin]$ jps
532 NameNode
838 SecondaryNameNode
963 Jps
656 DataNode
[hadoop@bjqilitst2 hadoop]$ jps
1854 Jps
1782 DataNode
[hadoop@bjqilitst3 hadoop]$ jps
1169 DataNode
1248 Jps
```

### Verify health by CLI
```shell
[hadoop@bjqilitst1 hadoop-2.6.5]$  ./bin/hdfs dfsadmin -report
Configured Capacity: 91005493248 (84.76 GB)
Present Capacity: 81165398016 (75.59 GB)
DFS Remaining: 81165385728 (75.59 GB)
DFS Used: 12288 (12 KB)
DFS Used%: 0.00%
Under replicated blocks: 0
Blocks with corrupt replicas: 0
Missing blocks: 0

-------------------------------------------------
Live datanodes (3):

Name: 9.111.159.164:50010 (bjqilitst3)
Hostname: bjqilitst3
Decommission Status : Normal
Configured Capacity: 30335164416 (28.25 GB)
DFS Used: 4096 (4 KB)
Non DFS Used: 3317608448 (3.09 GB)
DFS Remaining: 27017551872 (25.16 GB)
DFS Used%: 0.00%
DFS Remaining%: 89.06%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Sat Oct 29 06:34:44 EDT 2016


Name: 9.111.159.156:50010 (bjqilitst1)
Hostname: bjqilitst1
Decommission Status : Normal
Configured Capacity: 30335164416 (28.25 GB)
DFS Used: 4096 (4 KB)
Non DFS Used: 3205844992 (2.99 GB)
DFS Remaining: 27129315328 (25.27 GB)
DFS Used%: 0.00%
DFS Remaining%: 89.43%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Sat Oct 29 06:34:44 EDT 2016


Name: 9.111.159.163:50010 (bjqilitst2)
Hostname: bjqilitst2
Decommission Status : Normal
Configured Capacity: 30335164416 (28.25 GB)
DFS Used: 4096 (4 KB)
Non DFS Used: 3316641792 (3.09 GB)
DFS Remaining: 27018518528 (25.16 GB)
DFS Used%: 0.00%
DFS Remaining%: 89.07%
Configured Cache Capacity: 0 (0 B)
Cache Used: 0 (0 B)
Cache Remaining: 0 (0 B)
Cache Used%: 100.00%
Cache Remaining%: 0.00%
Xceivers: 1
Last contact: Sat Oct 29 06:34:44 EDT 2016
```

### Verify health by Hadoop GUI

http://9.111.159.156:50070/dfshealth.html#tab-overview

Once the Hadoop cluster is up and running check the web-ui of the components as described below:

| Daemon                      | Web Interface           | Notes                       |
| --------------------------- | ----------------------- | --------------------------- |
| NameNode                    | http://*nn_host:port*/  | Default HTTP port is 50070. |
| ResourceManager             | http://*rm_host:port*/  | Default HTTP port is 8088.  |
| MapReduce JobHistory Server | http://*jhs_host:port*/ | Default HTTP port is 19888. |

### Hadoop fs commands

http://hadoop.apache.org/docs/r2.6.5/hadoop-project-dist/hadoop-common/FileSystemShell.html

```shell
./bin/hadoop fs -help
hadoop fs -ls    / 
hadoop fs -ls -R   / 
hadoop fs -mkdir /dir
hadoop fs -put  <local file path>  <hdfs file path>
hadoop fs -get  <hdfs file path>   <local file path>
hadoop fs -text <HDFS file> 
hadoop fs -rm   <HDFS file>
hadoop fs -rmr  <HDFS directory>
```

## Enable Spark History Server

Refer to doc http://spark.apache.org/docs/latest/monitoring.html

Enable Spark History Server on HDFS by edit `/myhome/hadoop/spark-1.6.2-bin-hadoop2.6/conf/spark-defaults.conf`
```properties
spark.eventLog.enabled           true
spark.eventLog.dir               hdfs://9.111.159.156:9000/hadoop/logdir
spark.history.fs.logDirectory    hdfs://9.111.159.156:9000/hadoop/logdir
```
Start the Spark history server
```shell
./sbin/start-history-server.sh
```

Open the Historty Server Web GUI

http://9.111.159.156:18080




