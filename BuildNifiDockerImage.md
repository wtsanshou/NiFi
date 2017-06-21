# Build Nifi Docker image

## Prerequisities
* OS: Ubuntu 16.04
* NiFi: 1.3.0 
* Docker: 1.12+

# Install Docker if you don't have Docker

Follow the <a>Docker Website</a>

or

Script:
```bash
#!/bin/bash

## Update and Upgrade
sudo apt-get update
sudo apt-get upgrade -y

## Install Docker Engine
DOCKER_VERSION=1.12.3-0~xenial
apt-get update
apt-get install -y apt-transport-https ca-certificates
apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
echo '' > /etc/apt/sources.list.d/docker.list
echo 'deb https://apt.dockerproject.org/repo ubuntu-xenial main' > /etc/apt/sources.list.d/docker.list
apt-get update
apt-get install -y linux-image-extra-$(uname -r)
apt-get install -y docker-engine=$DOCKER_VERSION
service docker start
usermod -aG docker ubuntu
```

**Note:** If you have NiFi installed and saw this error:
```
insserv: warning: script 'S65nifi' missing LSB tags and overrides
insserv: warning: script 'nifi' missing LSB tags and overrides
insserv: Starting nifi depends on plymouth and therefore on system facility `$all' which can not be true!
insserv: Starting nifi depends on plymouth and therefore on system facility `$all' which can not be true!
```

Change the name of nifi in `/etc/init.d/nifi` to anyother name
You can change it back after installation

## Use custom docker registry server
```bash
## For Ubuntu 16.04 & Debian Jessie, append `--insecure-registry 192.168.200.167:5000` in the line of ExecStart=/usr/bin/dockerd -H fd:// in the file of /lib/systemd/system/docker.service
##NOTE, to add multiple custom registry servers, just specify multiple insecure registries by lining them up:
ExecStart=/usr/bin/dockerd -H fd:// --insecure-registry IP1:PORT --insecure-registry IP2:PORT --insecure-registry IP3:PORT

##After modifying the file, restart the docker service:
sudo systemctl daemon-reload
sudo service docker restart

## if you see error: "http: server gave HTTP response to HTTPS client root@black-pearl:/# vi /lib/systemd/system/docker.service"
## check docker status
sudo service docker status
```

# Create Docker file (It is not working in Openstack, Don't know why)
```bash
cd dockerfile/
sudo docker build -f docker-nifi-<os> -t 192.168.200.167:5000/docker-nifi .
docker push 192.168.200.167:5000/docker-nifi
```

makefile e.g:
```
FROM ubuntu:latest
MAINTAINER rob nolan robertnolan@research.ait.ie

#install yum repos
RUN apt-get update &&\
    apt-get install -y default-jdk curl python

ENV JAVA_HOME /usr/lib/jvm/java-1.8.0-openjdk-amd64
RUN export JAVA_HOME

ENV NIFI_VERSION=1.3.0 \
        NIFI_HOME=/opt/nifi \
        MIRROR_SITE=http://apache.spinellicreations.com

# Picked the recommended mirror from Apache for the distribution.
# Import the Apache NiFi release keys, download the release, and set up a user to run NiFi.
RUN set -x \
        && curl -Lf https://dist.apache.org/repos/dist/release/nifi/KEYS -o /tmp/nifi-keys.txt \
        && gpg --import /tmp/nifi-keys.txt \
        && curl -Lf ${MIRROR_SITE}/nifi/${NIFI_VERSION}/nifi-${NIFI_VERSION}-bin.tar.gz -o /tmp/nifi-bin.tar.gz \
        && mkdir -p ${NIFI_HOME} \
        && tar -z -x -f /tmp/nifi-bin.tar.gz -C ${NIFI_HOME} --strip-components=1 \
        && rm /tmp/nifi-keys.txt \
        && sed -i -e "s|^nifi.ui.banner.text=.*$$|nifi.ui.banner.text=Docker NiFi ${NIFI_VERSION}|" ${NIFI_HOME}/conf/nifi.properties \
        && groupadd nifi \
        && useradd -r -g nifi nifi \
        && bash -c "mkdir -p ${NIFI_HOME}/{database_repository,flowfile_repository,content_repository,provenance_repository}" \
        && chown nifi:nifi -R ${NIFI_HOME}

# These are the volumes (in order) for the following:
# 1) user access and flow controller history
# 2) FlowFile attributes and current state in the system
# 3) content for all the FlowFiles in the system
# 4) information related to Data Provenance
# You can find more information about the system properties here - https://nifi.apache.org/docs/nifi-docs/html/administration-guide.html#system_properties
VOLUME ["${NIFI_HOME}/database_repository", \
        "${NIFI_HOME}/flowfile_repository", \
        "${NIFI_HOME}/content_repository", \
        "${NIFI_HOME}/provenance_repository"]

# Open port 8081 for the HTTP listen
USER nifi
WORKDIR ${NIFI_HOME}
EXPOSE 8080 8081
CMD ["bin/nifi.sh", "start"]
```

# Run NiFi docker image
```bash
sudo docker exec -it -p 8888:8080 <docker_nifi_image_name> bash
```

# Access NiFi UI
In host machine, open the port 8888
```
sudo nc -l 8888
```

In browser

```
http://<host_IP>:8888/nifi
```

