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

{% highlight css %}
sudo -S addgroup hadoop
sudo adduser --ingroup hadoop hduser
{% endhighlight %}

### Configuring SSH

{% highlight css %}
su - hduser
ssh-keygen -t rsa -P ""
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

