# hadoop-hbase-basics

## Installing Hadoop 3.1.2 and HDFS, and HBase 1.4.10

Largely follows [this](https://phoenixnap.com/kb/install-hadoop-ubuntu), but deviates at certain points.

- Open bash in ```home```
- Install OpenJDK:
    Check the Java version:
    ```
    java -version; javac -version
    ```
    and [make sure it is Java 8](https://cwiki.apache.org/confluence/display/HADOOP/Hadoop+Java+Versions) (any minor version) such that
    ```
    openjdk version "1.8.0_292"
    OpenJDK Runtime Environment (build 1.8.0_292-8u292-b10-0ubuntu1~20.04-b10)
    OpenJDK 64-Bit Server VM (build 25.292-b10, mixed mode)
    javac 1.8.0_292
    ```
    If not 8, then install 8 alongside other Java versions and config such that:
    ```
    sudo update-alternatives --config java
    There are 2 choices for the alternative java (providing /usr/bin/java).

    Selection    Path                                            Priority   Status
    ------------------------------------------------------------
    * 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      auto mode
    1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java      1111      manual mode
    2            /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java   1081      manual mode

    Press <enter> to keep the current choice[*], or type selection number: 2
    update-alternatives: using /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java to provide /usr/bin/java (java) in manual mod
    ```
    and 
    ```
    sudo update-alternatives --config javac
    There are 2 choices for the alternative javac (providing /usr/bin/javac).

    Selection    Path                                          Priority   Status
    ------------------------------------------------------------
    * 0            /usr/lib/jvm/java-11-openjdk-amd64/bin/javac   1111      auto mode
    1            /usr/lib/jvm/java-11-openjdk-amd64/bin/javac   1111      manual mode
    2            /usr/lib/jvm/java-8-openjdk-amd64/bin/javac    1081      manual mode

    Press <enter> to keep the current choice[*], or type selection number: 2
    update-alternatives: using /usr/lib/jvm/java-8-openjdk-amd64/bin/javac to provide /usr/bin/javac (javac) in manual mode

    ```
- Enable passwordless SSH (if not yet done):
    Install OpenSSH on Ubuntu:
    ```
    sudo apt install openssh-server openssh-client -y
    ```
    Generate an SSH key pair and define the location is is to be stored in:
    ```
    ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
    ```
    Use the cat command to store the public key as authorized_keys in the ssh directory:
    ```
    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    ```
    Make sure SSH to localhost works:
    ```
    ssh localhost
    ```
- Download and Install Hadoop 2.10.0:
    Search the archive at the [Haddop Apache Download site](https://hadoop.apache.org/releases.html) and download ```hadoop-3.1.2.tar.gz```
    Once the download is complete, extract the files to initiate the Hadoop installation:
    ```
    tar xzf hadoop-3.1.2.tar.gz
    ```
    The Hadoop binary files are now located within the ```/home/USER_NAME/hadoop-3.1.2/``` directory where ```USER_NAME``` is your username on your local machine.
- Single Node Hadoop Deployment (Pseudo-Distributed Mode)

    Hadoop excels when deployed in a fully distributed mode on a large cluster of networked servers. However, if you are new to Hadoop and want to explore basic commands or test applications, you can configure Hadoop on a single node.

    This setup, also called pseudo-distributed mode, allows each Hadoop daemon to run as a single Java process. A Hadoop environment is configured by editing a set of configuration files:

    - bashrc
    - hadoop-env.sh
    - core-site.xml
    - hdfs-site.xml
    - mapred-site-xml
    - yarn-site.xml

    Edit the .bashrc shell configuration file using a text editor of your choice (we will be using nano):
    ```
    sudo nano .bashrc
    ```
    and insert:
    ```
    #Hadoop Related Options
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export HADOOP_HOME=/home/mark/hadoop-3.1.2
    export HADOOP_CONFIG=/$HADOOP_HOME/etc/hadoop
    export PATH=$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH
    ```
    Then source it:
    ```
    source .bashrc
    ```
    Set ```JAVA_HOME``` in the ```$HADOOP CONFIG/hadoop-env.sh``` shell script such that:
    ```
    # The java implementation to use. By default, this environment
    # variable is REQUIRED on ALL platforms except OS X!
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    ```
    Make two folders on local file system, where HDFS namenode and datanode store their data:
    ```
    mkdir -p $HADOOP_HOME/hdfs/namenode
    mkdir -p $HADOOP_HOME/hdfs/datanode
    ```
    Edit the ```$HADOOP_CONFIG/core-site.xml``` shell script such that:
    ```
    <configuration>
        <property>
            <name>fs.defaultFS</name>
            <value>hdfs://127.0.0.1:9000</value>
            <description>NameNode URI</description>
        </property>
    </configuration>
    ```
    Edit the ```$HADOOP_CONFIG/hdfs-site.xml``` file such that:
    ```
    <configuration>
        <property>
            <name>dfs.replication</name >
            <value>1</value>
        </property>
        <property>
            <name>dfs.namenode.name.dir</name>
            <value>/home/USER_NAME/hadoop-3.1.2/hdfs/namenode</value>
            <description>Path on the local filesystem ...</description>
        </property>
        <property>
            <name>dfs.datanode.data.dir</name>
            <value>/home/USER_NAME/hadoop-3.1.2/hdfs/datanode</value>
            <description>Comma separated list of paths on the local filesystem ...</description>
        </property>
    </configuration>
    ```
    where ```USER_NAME``` is your username on your local machine.

    Format the namenode directory (DO THIS ONLY ONCE, THE FIRST TIME):
    ```
    $HADOOP_HOME/bin/hdfs namenode -format
    ```
    Finally, start the namenode and datanode daemons:
    ```
    $HADOOP_HOME/sbin/hadoop-daemon.sh start namenode
    $HADOOP_HOME/sbin/hadoop-daemon.sh start datanode
    ```
    If successful, ```jps``` yields something similar to:
    ```
    14708 NameNode
    14890 Jps
    14799 DataNode
    ```
    Also, use your preferred browser and navigate to your localhost URL or IP. The default port number 50070 (http://localhost:50070/) gives you access to the Hadoop NameNode UI. The default port 50075 (http://localhost:50075/) is used to access individual DataNodes directly from your browser.
- install HBase 1.4.10
    - download ```hbase-1.4.10-bin.tar.gz ``` from the [HBase archive](https://hbase.apache.org/downloads.html)
    - extract:
    ```
    tar xzf hbase-1.4.10-bin.tar.gz
    ```
    - add the following environment variables to ```.bashrc``` and source it:
    ```
    #HBase Related Options
    export HBASE_HOME=/home/mark/hbase-1.4.10
    export HBASE_CONF=/$HBASE_HOME/conf
    export PATH=$HBASE_HOME/bin:$PATH
    ```
    - Edit JAVA HOME in ```$HBASE CONF/hbase-env.sh```:
    ```
    # The java implementation to use. By default, this environment
    # variable is REQUIRED on ALL platforms except OS X!
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    ```
    - Make a folder on local file system, where zookeeper stores its data:
    ```
    mkdir -p $HBASE_HOME/zookeeper
    ```
    - Edit ```$HBASE_CONF/hbase-site.xml``` by adding the following lines:
    ```
    <configuration>
        <property>
            <name>hbase.rootdir</name>
            <value>hdfs://localhost:9000/hbase</value>
        </property>
        <property>
            <name>hbase.zookeeper.property.dataDir</name>
            <value>/home/USER_NAME/hbase-1.4.10/zookeeper</value>
        </property>
        <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
        </property>
    </configuration>
    ```
    - Start HBase:
    ```
    $HBASE_HOME/bin/start-hbase.sh
    ```
    - ```jps``` should give something like:
    ```
    6768 DataNode
    6658 NameNode
    7380 HQuorumPeer
    7446 HMaster
    18377 Jps
    7578 HRegionServer
    ```

## Wordcount with Hadoop Mapreduce

The script at ```src/wordcount/WordCount.java``` counts the occurence of words in the files ```src/wordcount/data/file0``` and ```src/wordcount/data/file1```. The ouput is at ```src/wordcount/output/```.

First, put the input files to HDFS:

```
$HADOOP_HOME/bin/hdfs dfs -mkdir -p input
$HADOOP_HOME/bin/hdfs dfs -put src/wordcount/data/file0 input/file0
$HADOOP_HOME/bin/hdfs dfs -put src/wordcount/data/file1 input/file1
$HADOOP_HOME/bin/hdfs dfs -ls input
```

Then add the envrionment variable for the classpath in ```.bashrc``` and source it:
```
export HADOOP_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
```

Compile the script:
```
cd src
mkdir wordcount_classes
javac -cp $HADOOP_CLASSPATH -d wordcount_classes wordcount/WordCount.java
jar -cvf wordcount.jar -C wordcount_classes/ .
```

Run the application:
```
$HADOOP_HOME/bin/hadoop jar wordcount.jar wordcount.WordCount input output
```

Check the output on HDFS:
```
$HADOOP_HOME/bin/hdfs dfs -ls output
$HADOOP_HOME/bin/hdfs dfs -cat output/part-r-00000
```

## MapReduce and HBase for working with HBase tables

Start the HBase shell and create a table called test1 with some rows:
```
create 'test1', 'cf'
put 'test1', '20130101#1', 'cf:sales', '100'
put 'test1', '20130101#2', 'cf:sales', '110'
put 'test1', '20130102#1', 'cf:sales', '200'
put 'test1', '20130102#2', 'cf:sales', '210'

create 'test2', 'cf'

scan 'test1'
```

Then add the envrionment variables for the classpath in ```.bashrc``` and source it:
```
export HADOOP_CLASSPATH=$($HADOOP_HOME/bin/hadoop classpath)
export HBASE_CLASSPATH=$($HBASE_HOME/bin/hbase classpath)
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:$HBASE_CLASSPATH
```

```
cd src
mkdir hbase_classes
javac -cp $HADOOP_CLASSPATH -d hbase_classes hbase/HBaseMapReduce.java
jar -cvf hbaseMapreduce.jar -C hbase_classes/ .
```

Run the application:
```
$HADOOP_HOME/bin/hadoop jar hbaseMapreduce.jar hbase.HBaseMapReduce
```

Start the HBase shell and check the output (HBase stores everything as bytes),
we can convert it to readable integer format in HBase shell:
```
scan 'test2'
org.apache.hadoop.hbase.util.Bytes.toInt("\x00\x00\x00\xD2".to_java_bytes)
org.apache.hadoop.hbase.util.Bytes.toInt("\x00\x00\x01\x9A".to_java_bytes)
```