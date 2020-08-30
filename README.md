# etl-spark-on-hdfs
We will read data from HDFS, apply transformation and write back to using Hive

https://dzone.com/articles/example-of-etl-application-using-apache-spark-and

https://datashark.academy/how-to-setup-apache-hadoop-cluster-on-a-mac-or-linux-computer/

## Introduction 

Apache Spark is an in-memory distributed data processing engine that is used for processing and analytics of large data-sets. Spark presents a simple interface for the user to perform distributed computing on the entire clusters.Spark does not have its own file systems, so it has to depend on the storage systems for data-processing. It can run on HDFS or cloud based file systems like Amazon S3 and Azure BLOB. 

## Pseudo distributed mode Hadoop installation on single machine

### Enabling SSH
Follow up instructions written [here](https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-18-04/) to enable ssh server on Ubuntu. On Mac, Open System Preferences -> Sharing and enable Remote Login option as shown below.

![Package Structure](https://lh3.googleusercontent.com/yRUMzLmiWSigBrS6iVVlPsi1kJ8xnkULmClybBVB-FlSZ_maPUuHV0Y55axJTF9QGgGjBpDMiR-BVYdFf926Iro0D_azEFcsLoTamwFkqyv0i_NsfJTBsu8HMtfxNS6r1Y5yQYxlqMje0-EQvwYe8LqSu3Jwd_b_-9AOlGXjT-xuEx24CvPzsKGwB7vs_we6XregJCoWUrWyNztxB3AR9R27oDK6pw6XwYLwSCY5xJH3PqJP7CW4Uaow_MwJXZprusN9AOmDcFPfXgT1ejRmbGM5vXCxezsGk6zNXRKg9zm0r7PSjvAmP0gWshYX00lf5SogdQDarSvxV-trzPksOD6loZ2539nAwyUc8mwiicTBMxQ9N2LUXvjQNcNInyNY-T35yehBYSHwq4sUMSMKN8paiRtTi9tYc9SRpLzPb9ZRZJJAzkh4VdcjaTvF1ynIcAi2YXuEMWQ13_AvdSbTsIY9RbcH8A4KSVQ6gh_RgfTFglzABsG0qBnkRDSNWA--7qPAmahsU-Dub9d1md5UjnlWwRRC3HWJwSpybrg4rvXY4nBI6t7L81VzAt5yCm2p3B6OwZ-OO9rA8Ri8IEwBRWOuqU60W0aCyUzaxSZIpM74LB7IySSRFGd_fil-GeQoh1QbX9wT0-tuDC6A89kK1Pcl-_vbDDPju0Db46ywVtfokIqsXM9XhHgrTRVTyQ=w667-h531-no?authuser=0)

### Generate SSH keys 
Generate secured public and private keys for remote logins in your terminal

```bash
$ ssh-keygen -t rsa -C "me@example.com"
```
Keys were generated under /home/<user>/.ssh folder. To let hadoop access remote nodes, need to add public keys. Since its same machine, so just copy public key to the list of authorized keys as

```bash  
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```  

Type following in the terminal to test the connectivity without entering credentials

```bash  
$ ssh localhost
```  

### Download Hadoop and set environment variables
Download from https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.10.0/hadoop-2.10.0.tar.gz, extract and set environment variables.

```bash  
$ tar xvzf hadoop-2.10.1.tar.gz
```  
Update enviroment variables, under your home directory open `.bash_profile` 
```bash
$ pwd
/Users/kemalatik
$ vi .bash_profile
```

update it with environment variables.

```bash
export JAVA_HOME=$(/usr/libexec/java_home)
export HADOOP_HOME=/Users/kemalatik/etc/hadoop-2.10.0
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export PATH=$HADOOP_HOME/bin:$PATH
export PATH=$HADOOP_HOME/sbin:$PATH
```
### Update configuration files

Configuration files are located under `/Users/kemalatik/etc/hadoop-2.10.0/etc/hadoop`.


#### core-site.xml

```bash
$ vi core-site.xml
```


```xml
<configuration>
	   <property>

       <name>hadoop.tmp.dir</name>

       <value>/Users/kemalatik/etc/hdfs/tmp</value>

       <description>base for other temp directories</description>

   </property>

   <property>

       <name>fs.default.name</name>

       <value>hdfs://0.0.0.0:9000</value>

   </property>
</configuration>
```

#### hdfs-site.xml
```bash
$ vi hdfs-site.xml
```


```xml
<configuration>
     <property>

             <name>dfs.replication</name>

             <value>1</value>

     </property>
</configuration>

```

#### yarn-site.xml

```bash
$ vi yarn-site.xml
```


```xml
<configuration>

<!-- Site specific YARN configuration properties -->
<property>

       <name>yarn.nodemanager.aux-services</name>

       <value>mapreduce_shuffle</value>

</property>

<property>

       <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>

       <value>org.apache.hadoop.mapred.ShuffleHandler</value>

</property>

</configuration>

```

#### mapred-site.xml

```bash
$ vi mapred-site.xml
```


```xml
<configuration>
	   <property>

       <name>mapred.job.tracker</name>

       <value>yarn</value>

   </property>

   <property>

       <name>mapreduce.framework.name</name>

       <value>yarn</value>

   </property>

</configuration>

```

### Start DFS and YARN daemons

Now we need to start the Hadoop’s daemons. A daemon is a process which runs in the background silently. Hadoop comes with NameNode, Data node and YARN management. Before starting the  daemon, format the NameNode 

```bash
$ hadoop namenode –format
$ start-dfs.sh   # starts NameNode, DataNode & SecondaryNameNode
$ start-yarn.sh # starts ResourceManager & NodeManager
```

## Simple ETL Application using Apache Spark and Hive

- Send the data from Linux file system to the data storage unit of the Hadoop(HDFS)
- Read a sample data set with Spark from HDFS 
- Apply simple transformation
- write query results a table which will be created in Hive.

Hive is a substructure that allows us to query the data in the hadoop ecosystem, which is stored in this environment. With this infrastructure, we can easily query the data in our big data environment using SQL language.

Use below sample data and save it as `dept.csv`
```
department,designation,costToCompany,state
Sales,Trainee,12000,UP
Sales,Lead,32000,AP
Sales,Lead,32000,LA
Sales,Lead,32000,TN
Sales,Lead,32000,AP
Sales,Lead,32000,TN
Sales,Lead,32000,LA
Sales,Lead,32000,LA
Marketing,Associate,18000,TN
Marketing,Associate,18000,TN
HR,Manager,58000,TN

```

Copy the downloaded sample data to HDFS. Create a directory in HDFS and copy the sample data in from Linux file system to hdfs.

```
$ hdfs dfs -mkdir -p deptdata
$ hadoop fs -copyFromLocal ~kemalatik/IdeaProjects/spark-etl/dept.csv deptdata
$ hadoop dfs -ls deptdata/
-rw-r--r--   1 kemalatik supergroup     527958 2020-08-29 16:45 /user/kemalatik/deptdata/dept.csv
```
Generate `DataSet` from CSV file and perform aggregation with object chaining

```java
public class Cvs2DataSet {

  static Logger logger = LogManager.getLogger(Cvs2DataSet.class);
  final static String hdfsConfPath = "/Users/kemalatik/etc/hadoop-2.10.0/etc/hadoop";

  public static void main(String[] args) {

    SparkSession spark = SparkSession
        .builder().master("local[*]")
        .appName("Java Spark SQL Example")
        .getOrCreate();

    StructType schema = new StructType()
        .add("department", "string")
        .add("designation", "string")
        .add("costToCompany", "long")
        .add("state", "string");


    Dataset<Row> df = spark.read()
        .option("mode", "DROPMALFORMED").option("header","true")
        .schema(schema)
        .csv("hdfs://0.0.0.0:9000//user/kemalatik/deptdata/dept.csv");

    df.printSchema();
    df.show();


    df.createOrReplaceTempView("employee");

    Dataset<Row> dfResult = df.groupBy("department", "designation", "state")
        .agg(sum("costToCompany"), count("department"));

    dfResult.show();//for testing


  }

}

```


