# Setup NiFi

## Overview

This document is to show the process of setup NiFi.

## Prerequisities
* OS: Ubuntu 16.04
* NiFi: 1.3.0 
* Java: recent Java 8 (or newer) JDK

## Setup

### Download NiFi
```bash
weget http://ftp.heanet.ie/mirrors/www.apache.org/dist/nifi/1.3.0/nifi-1.3.0-bin.tar.gz

tar vxzf nifi-1.3.0-bin.tar.gz
```

Recommend editing the nifi.properties file and entering a password for the nifi.sensitive.props.key under the folder  `<installdir>/conf`

### Install Java jre and JDK, if you don't have
```bash
sudo apt-get update

sudo apt-get install default-jre

sudo apt-get install default-jdk

## Setting environment
## put "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" into ~/.bashrc
source ~/.bashrc
```

### Nifi commands

```
    start: starts NiFi in the background

    stop: stops NiFi that is running in the background

    status: provides the current status of NiFi

    run: runs NiFi in the foreground and waits for a Ctrl-C to initiate shutdown of NiFi

    install: installs NiFi as a service that can then be controlled via

        service nifi start

        service nifi stop

        service nifi status
```

### Install Nifi
Go to folder `<installdir>/bin`
```bash
./nifi.sh install 
```

Check the status of NiFi, should see something like:
```
Java home: /usr/lib/jvm/java-8-openjdk-amd64
NiFi home: /home/ubuntu/nifi/nifi-1.3.0

Bootstrap Config File: /home/ubuntu/nifi/nifi-1.3.0/conf/bootstrap.conf

2017-06-20 12:00:08,075 INFO [main] org.apache.nifi.bootstrap.Command Apache NiFi is currently running, listening to Bootstrap on port 36499, PID=14365
```

## Access UI of NiFi
Go to browser

`http://<your_ip>:8080/nifi`

