# Install RPM version of Indigo lib and Indigo Web Server

## Vagrant configuration

The vagrant file configures a fedora 27 box with a private network that will
be used by Cassandra.

### Create and provision the box 

```
$ vagrant up
```

### Log to the box

```
$ vagrant ssh
```

### Troubleshooting

If there's an error for the install of the virtualbox guest additions you can try
a manual install. vbguest is useful to share folders to the box.

```
$ vagrant up
$ vagrant vbguest --do install --no-cleanup
```

Then in the box

```
$vagrant ssh
```

Run the installer manually

```
$ sudo /mnt/VBoxLinuxAdditions.run
```


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
(192.168.50.4 from the vagrantfile)

```
cluster_name: 'indigo_cluster'
num_tokens: 128
seeds: "node_ip"
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
$ cd /vagrant
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

### Create the virtual environment for the web app

As the user indigo (`sudo -i -u indigo`)

```
$ virtualenv /var/lib/indigo/web
$ ln -s /var/lib/indigo/indigo-web /var/lib/indigo/web/project
```

### Install Indigo lib in the the virtual env

As the user indigo (`sudo -i -u indigo`)

```
$ cd /var/lib/indigo/indigo
$ /var/lib/indigo/web/bin/pip install -r requirements.txt
$ /var/lib/indigo/web/bin/python setup.py develop

```

### Install requirements for Indigo-web in the the virtual env

As the user indigo (`sudo -i -u indigo`)

```
$ cd /var/lib/indigo/web/project
$ /var/lib/indigo/web/bin/pip install -r requirements.txt


### Create the Cassandra database

As the user indigo (`sudo -i -u indigo`)

```
$ /var/lib/indigo/web/bin/iadmin create

```


### Collect static pages (Django)

As the user indigo (`sudo -i -u indigo`)

```
$ /var/lib/indigo/web/bin/python /var/lib/indigo/web/project/manage.py collectstatic --noinput

```

### Initialise Django database

As the user indigo (`sudo -i -u indigo`)

```
$ /var/lib/indigo/web/bin/python /var/lib/indigo/web/project/manage.py syncdb

```

### Create default users

As the user indigo (`sudo -i -u indigo`)

indigo is an administrator

```
$ /var/lib/indigo/web/bin/iadmin mkuser indigo
$ /var/lib/indigo/web/bin/iadmin mkuser guest
```

### Create default groups

As the user indigo (`sudo -i -u indigo`)

```
$ /var/lib/indigo/web/bin/iadmin mkgroup admins
$ /var/lib/indigo/web/bin/iadmin mkgroup guest
```

### Add users to groups


As the user indigo (`sudo -i -u indigo`)

```
$ /var/lib/indigo/web/bin/iadmin atg admins indigo
$ /var/lib/indigo/web/bin/iadmin atg guest guest
```

## Serve Indigo-web with gunicorn

```
$ cd /var/lib/indigo/web/project/
$ source /var/lib/indigo/web/bin/activate
$ gunicorn -b 192.168.50.4 indigo_ui.wsgi
```

Indigo should be available at http://192.168.50.4:8000

Eventually you may have to stop the firewall (or configure it)

```
$ sudo systemctl stop firewalld
```

### Serve using Nginx

```
$ sudo mkdir /etc/nginx/sites-available
$ sudo mkdir /etc/nginx/sites-enabled
$ sudo cp conf/nginx_redhat /etc/nginx/nginx.conf
$ sudo cp conf/indigo-web.nginx /etc/nginx/sites-available/indigo_http
$ sudo ln -s /etc/nginx/sites-available/indigo_http /etc/nginx/sites-enabled/indigo_http
$ sudo systemctl restart nginx.service

```

### Configure Indigo-web init script (systemctl)

```
$ sudo cp conf/indigo-web.service.redhat /etc/systemd/system/indigo-web.service
$ sudo systemctl daemon-reload
$ sudo systemctl restart indigo-web.service
```

