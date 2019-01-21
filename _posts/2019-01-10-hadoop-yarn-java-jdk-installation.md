---
layout: post
title:  "Hadoop Yarn installation (java-jdk)"
date:   2019-01-08
excerpt: ""
tag:
- hadoop
- yarn
- installation
- mapreduce
---


## Installing Prerequisites

Start with installing prerequisites:

### JAVA-JDK

```bash
sudo apt-get install default-java
```

### Add a dedicated hadoop user

It'll create a new user to install/run hadoop keeping it separated from other user accounts.

```bash
sudo -S addgroup hadoop
sudo adduser --ingroup hadoop hduser
```

### Configuring SSH

The below command will transfer the terminal access to newly created hduser

```bash
su - hduser
```

Hadoop requires an SSH access to manage nodes present all over the cluster. This command will generate an SSH key with empty(string) password. In general, it's not recommended to use empty(string) password, but since we don't want to enter the passphrase each time Hadoop connects to its nodes therefore, leave it empty.

```bash
ssh-keygen -t rsa -P ""
```

The command creates a new file and appends generated key to it.

```bash
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
```

Now we'll need root access through hduser, thus we'll add hduser to the list of sudoers. `sudo visudo` and then append `hduser ALL=(ALL:ALL) ALL` at the EOF.

Now we need to disable IPv6. Hence, `sudo gedit /etc/sysctl.conf` and then append following at the EOF.

```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1 
net.ipv6.conf.lo.disable_ipv6 = 1
```

Now, **reboot** system. On boot, check whether the ipv6 has been disabled.

## Install hadoop

### Download hadoop

```bash
#change directory
cd /usr/local

#download hadoop 3.1 in this directory
#to download other/newer version check the link http://www-eu.apache.org/dist/hadoop/core/
wget http://www-eu.apache.org/dist/hadoop/core/hadoop-3.1.0/hadoop-3.1.0.tar.gz

#extract the tar file
sudo tar xzf hadoop-3.1.0.tar.gz

#rename it to hadoop
sudo mv hadoop-3.1.0 hadoop

#change the owner of files to hduser
sudo chown -R hduser:hadoop hadoop
```

### Set environment variables

Set hadoop and java home environment variables.

```bash
#Open the editor
sudo gedit ~/.bashrc

#and append the following lines at the EOF

export HADOOP_HOME=/usr/local/hadoop
export JAVA_HOME=/usr/lib/jvm/default-java

# Some convenient aliases and functions for running Hadoop-related commands  
unalias fs &> /dev/null 
alias fs="hadoop fs" 
unalias hls &> /dev/null 
alias hls="fs -ls" 

# Add Hadoop bin/ directory to PATH  
export PATH=$PATH:$HADOOP_HOME/bin
# Add Hadoop sbin/ directory to PATH  
export PATH=$PATH:$HADOOP_HOME/sbin
```

Now edit the hadoop-env.sh and update JAVA_HOME. *You know the drill*

```bash
sudo gedit $HADOOP_HOME/etc/hadoop/hadoop-env.sh

#update JAVA_HOME (don't append, instead search for likewise line of code, it might be in the comments!)
export JAVA_HOME=/usr/lib/jvm/default-java

#you can also update HADOOP_HOME (not necessary)
export HADOOP_HOME=/usr/local/hadoop
```


## Start hadoop cluster

### Standalone Mode

In this mode, hadoop will be set to run in a non-distributed mode, as a single java process. Using this mode we can check whether the installation is upto-mark.

```bash
#create a directory to store input files
mkdir $HADOOP_HOME/input

#now to verify no-errors in the installation, we will run a sample using example jar file
#copy all xml files to the newly created directory
cp $HADOOP_HOME/etc/hadoop/*.xml $HADOOP_HOME/input

#1st argument is the /path/to/hadoop command (required to run MapReduce)
#2nd argument is jar, specifying MapReduce is in JAVA archive
#3rd argument is come along MapReduce example, it name of jar can be version specific (check your file/version)
#4th argument is grep, to execute regular expression example
#5th argument is input directory, containing all the .xml files
#6th argument is output directory, which will be created and will contain output files
#7th argument is 'dfs[a-z.]+', basically the string to be searched
$HADOOP_HOME/bin/hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.0.jar grep input output 'dfs[a-z.]+'
```

### Pseudo Distributed Mode

In this mode, hadoop runs on a single node in a pseudo distributed mode where each hadoop daemon run as separate java process.

#### Configuring site xml(s)

Create the tmp directory

```bash
sudo mkdir -p /app/hadoop/tmp
sudo chown hduser:hadoop /app/hadoop/tmp
```



Now edit core-site.xml and hdfs-site.xml. You'll find these files in `$HADOOP_HOME/etc/hadoop` directory.

Start with **core-site.xml**

`sudo gedit $HADOOP_HOME/etc/hadoop/core-site.xml`

```python
#paste these lines between <configuration> </configuration> tags
<property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
</property>

<property>
  <name>fs.defaultFS</name>
  <value>hdfs://localhost:9000</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
</property>
```

**hdfs-site.xml**

`sudo gedit $HADOOP_HOME/etc/hadoop/hdfs-site.xml`

```python
#paste these lines between <configuration> </configuration> tags
<property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
</property>
```

Format namenode (you'll need to do this only the first time you set up hadoop cluster i.e, the time of installation)

```bash
#On running this command, you'll get the o/p with SHUTDOWN_MSG at the end.
#Don't worry it's not an error
$HADOOP_HOME/bin/hdfs namenode -format
```

Now it's time to *start the HADOOP CLUSTER!!!*

```bash
#start the namenode and datanode daemon
$HADOOP_HOME/sbin/start-dfs.sh
```

**Yeah, it's done!**

You can check the cluster nodes. Run this command `jps` and also you can browse the namenode web-interface on this link `http://localhost:9870`
Now create the HDFS directories which we'll need to execute MapReduce jobs. `$HADOOP_HOME/bin/hdfs`. To stop the network `$HADOOP_HOME/sbin/stop-dfs.sh`

**YARN on a single node**

Open the `.bashrc` and append other environment variables `sudo gedit ~/.bashrc`.

```bash
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_YARN_HOME=${HADOOP_HOME}
```

**mapred-site.xml**

`sudo gedit $HADOOP_HOME/etc/hadoop/mapred-site.xml`

```python
#paste these lines between <configuration> </configuration> tags
<property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
</property>
```

**yarn-site.xml**

`sudo gedit $HADOOP_HOME/etc/hadoop/yarn-site.xml`

```python
#paste these lines between <configuration> </configuration> tags
<property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
</property>
<property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME</value>
</property>
```

**To check the distributed filesystem working properly**

```bash
#Type and check the results for the following commands
#list of files and directories on your distributed filesystem
hdfs dfs -ls

#now create a relative path
hdfs dfs -mkdir /user
hdfs dfs -mkdir /user/<username>

#relative and absolute path
hdfs dfs -mkdir /cluster   #----> This directory will be created in your dfs home i.e., where directory user is!

hdfs dfs -mkdir cluster    #----> This will be created inside /user/<username>

#You can view the added directories in the WebUI too
#browse localhost:9870

#and check the option Utilities -> Browse filesystem
#it displays some webhdfs Server Error [This error is shown for the java versions >=9]
#open and edit hadoop-env.sh
export HADOOP_OPTS="--add-modules java.activation"
```

**I hope your hadoop distributed file system is working fine!**
**In case of trouble or mistake in the code, notify me! - rajatgarg149@gmail.com**


