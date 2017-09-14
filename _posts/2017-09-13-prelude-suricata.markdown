---
layout: post
title: "Prelude + Suricata"
date: 2017-09-13 16:36:17 +0000
categories: blog
location: Antarctica, Troll
description: This is simple manual about how to setup Prelude + Suricata
---
---
This is simple manual how to setup **Prelude + Suricata** 

* [Prelude-SIEM](https://www.prelude-siem.org/) - is a Universal "Security Information & Event Management" (SIEM) system. Prelude collects, normalizes, sorts, aggregates, correlates and reports all security-related events independently of the product brand or license giving rise to such events; Prelude is "agentless".
* [Suricata](https://suricata-ids.org/) - is a free and open source, mature, fast and robust network threat detection engine.

## Prelude

https://www.prelude-siem.org/projects/prelude/wiki/InstallingPackageRHEL

Prepare a server with CentOS 6 x86_64.

* Download the [YUM configuration package](https://www.prelude-siem.org/attachments/download/577/prelude-release-3.0.0-1.el6.noarch.rpm).
* It contains the YUM configuration file and the [PGP Public Key](https://www.prelude-siem.org/attachments/download/233/RPM-GPG-KEY-Prelude-IDS)

##### Install the RPM

```sh
$ rpm -i prelude-release-3.0.0-1.el6.noarch.rpm
```

##### Installation using RHEL/CentOS

```sh
$ yum install libprelude
```

##### Install LibpreludeDB

```sh
$ yum install libpreludedb
$ yum install libpreludedb-mysql
$ yum install libpreludedb-python
```

##### Install Prelude-Manager

```sh
$ yum install prelude-manager
$ yum install prelude-manager-db-plugin
$ yum install prelude-manager-xml-plugin
```

##### Install Prelude-LML

```sh
$ yum install prelude-lml prelude-lml-rules
```

##### Install Prelude-Correlator

```sh
$ yum install prelude-correlator
```

##### Install Prewikka

```sh
$ yum install prewikka
```
>  Note: if a prewikka can't be installed because python-mako dependencies,
>  try to download **prewikka.rpm** and install it with options "\-\-nodeps"
```sh
$ rpm --nodeps --force -ivh prewikka-4.0.0-1.el6.noarch.rpm
$ pip install Mako
```

### Configuration

##### Disable SELinux.

```sh
$ vim /etc/sysconfig/selinux ; SELINUX=disabled
```

##### Add to startup

```sh
$ /sbin/chkconfig --add mysqld
$ /sbin/chkconfig --add prelide-manager
$ /sbin/chkconfig --add prelude-manager
$ /sbin/chkconfig --add prelude-correlator
$ /sbin/chkconfig --add prelude-lml
```

##### Prepare database

```sql
mysql> CREATE database prelude;
            CREATE USER 'prelude'@'localhost' IDENTIFIED BY 'prelude';
            SET PASSWORD FOR 'prelude'@'localhost' = PASSWORD('prelude');
            GRANT ALL PRIVILEGES ON prelude.* TO prelude@'localhost';
            FLUSH PRIVILEGES;
```

```sh
$ mysql -u prelude prelude -p < /usr/share/libpreludedb/classic/mysql.sql
```

##### Configure files

$ vim /etc/prelude-manager/prelude-manager.conf

```ini
[db]
type = mysql
host = localhost
port = 3306
name = prelude
user = prelude
pass = prelude
```

##### Add manager

```sh
$ prelude-admin add "prelude-manager" --uid 61 --gid 498
```

##### Run server & Register Agents

```sh
$ prelude-admin registration-server prelude-manager

$ prelude-admin register "prelude-correlator" "idmef:rw" 1.1.1.51 --uid 497 --gid 498
$ prelude-admin register "prelude-lml" "idmef:rw" 1.1.1.51 --uid 498 --gid 498
```

##### Check status

```sh
$ prelude-admin list -l
```

##### Configure web framework Prewikka

$ vim /etc/prewikka/prewikka.conf

```yaml
[idmef_database]
type: mysql
host: localhost
user: prelude
pass: prelude
name: prelude

# Prewikka DB
[database]
type: mysql
host: localhost
user: prewikka
pass: prewikka
name: prewikka

[log file]

level: debug
file: /tmp/prewikka.log

[log syslog]
level: info
```

##### Start Prewikka

```sh
$ prewikka-httpd&
```

## Suricata


##### Suricata installation

```sh
$ sudo add-apt-repository ppa:oisf/suricata-stable
$ sudo apt-get update
$ sudo apt-get build-dep suricata
$ apt-get source suricata
$ sudo apt-get install build-essential fakeroot devscripts
$ apt-get install libprelude-dev
$ cd suricata-4.0.0
```
 

##### Add a  prelude options

```sh
$ vim debian/rules 

#add to CONFIGURE_ARGS --enable-prelude --with-libprelude-prefix=/usr/

$ debuild -us -uc -i -I
$ cd ..
$ dpkg -i suricata_4.0.0-0ubuntu10_amd64.deb
```

##### Configure Suricata

$ vim /etc/suricata/suricata.yaml

```yaml
 - alert-prelude:
      enabled: yes
      profile: suricata
      log-packet-content: yes
      log-packet-header: yes
```

On prelude manager server run

```sh
$ prelude-admin registration-server prelude-manager
```

On suricate server run

```sh
$ prelude-admin register suricata "idmef:w" <manager address> --uid 114 --gid 118 
$ prelude-admin list -l         - # check if suricata was added to prelude
$ /etc/init.d/suricata restart  - # restart suricata service
```

Check alerts on PC with suricata

```sh
$ curl -A "BlackSun" www.google.com
```

