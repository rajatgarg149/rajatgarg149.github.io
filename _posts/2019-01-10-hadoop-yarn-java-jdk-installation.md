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

{% highlight css %}
sudo apt-get install default-java
{% endhighlight %}

### Add a dedicated hadoop user

It'll create a new user to install/run hadoop keeping it separated from other user accounts.

{% highlight css %}
sudo -S addgroup hadoop
sudo adduser --ingroup hadoop hduser
{% endhighlight %}

### Configuring SSH

The below command will transfer the terminal access to newly created hduser

{% highlight css %}
su - hduser
{% endhighlight %}
Hadoop requires an SSH access to manage nodes present all over the cluster. This command will generate an SSH key with empty(string) password. In general, it's not recommended to use empty(string) password, but since we don't want to enter the passphrase each time Hadoop connects to its nodes therefore, leave it empty.

{% highlight css %}
ssh-keygen -t rsa -P ""
{% endhighlight %}
The command creates a new file and appends generated key to it.
{% highlight css %}
cat .ssh/id_rsa.pub >> .ssh/authorized_keys
{% endhighlight %}

Now we'll need root access through hduser, thus we'll add hduser to the list of sudoers. `sudo visudo` and then append `hduser ALL=(ALL:ALL) ALL` at the EOF.

Now we need to disable IPv6. Hence, `sudo gedit /etc/sysctl.conf` and then append following at the EOF.
{% highlight css %}
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1 
net.ipv6.conf.lo.disable_ipv6 = 1
{% endhighlight %}

Now, **reboot** system. On boot, check whether the ipv6 has been disabled.

## Install hadoop

### Download hadoop

{% highlight css %}
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
{% endhighlight %}

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



