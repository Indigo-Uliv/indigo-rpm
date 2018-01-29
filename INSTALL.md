# Install RPM version of Indigo lib and Indigo Web Server


## Prerequisite

### Java 8 (Needed by Cassandra/DSE)

Install Oracle Java SE Runtime Environment 8 (JDK) (1.8.0_40 minimum) or 
OpenJDK 8. Earlier or later versions are not supported by DataStax Enterprise.

See https://docs.datastax.com/en/dse/5.1/dse-dev/datastax_enterprise/install/installJdkRHEL.html

```
$ sudo rpm -ivh jdk-8u121-linux-x64.rpm
$ sudo alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_121/bin/java 200000
$ sudo alternatives --config java
```

### DataStax Enterprise 5.1

See https://docs.datastax.com/en/dse/5.1/dse-admin/datastax_enterprise/install/installRHELdse.html

#### Add the DataStax Yum repository to /etc/yum.repos.d/datastax.repo:

```
[datastax] 
name = DataStax Repo for DataStax Enterprise
baseurl=https://dsa_email_address:password@rpm.datastax.com/enterprise
enabled=1
gpgcheck=0
```

#### Install DSE 5.0.7-1

```
$ sudo yum install dse-full-5.0.7-1
```

#### Configure DSE 5.0.7-1


* Stop DSE

```
$ sudo systemctl stop dse.service
```

* Clear Cassandra logs

```
$ echo -n | sudo tee /var/log/cassandra/system.log
```

* Remove cluster metadata

```
$ sudo rm -rf /var/lib/cassandra/*
```

* Modify Cassandra config file

```
$ sudo nano /etc/dse/cassandra/cassandra.yaml
```

node_ip is the IP address used by Cassandra nodes to communicate with each other

```
cluster_name: 'indigo_cluster'
num_tokens: 128
- seeds: "node_ip"
listen_address: node_ip
broadcast_address: node_ip
```

* Modify DSE config file

```
$ sudo nano /etc/default/dse
```

```
GRAPH_ENABLED=1
```

* Restart DSE

```
$ sudo systemctl start dse.service
```

## Install Indigo libs

### Install dependencies

```
$ sudo yum install -y automake gcc gcc-c++ kernel-devel git htop iotop python-pip python-virtualenv
``` 

### Install RPM

```
$ sudo rpm -ivh indigo-1.2-1.noarch.rpm 
``` 

## Install Indigo-web package

### Install dependencies

Install Indigo lib

```
$ sudo yum install -y expect nginx openldap-devel
``` 

### Install RPM

```
$ sudo rpm -ivh indigo-web-1.2-1.noarch.rpm 
``` 





