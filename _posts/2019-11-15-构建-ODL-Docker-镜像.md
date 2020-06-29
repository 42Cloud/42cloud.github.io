---
title: 构建 ODL Docker 镜像
date: 2019-11-15 22:01:06
categories: [SDN, 虚拟化]
tags:
---

```dockerfile
FROM ubuntu:16.04
 
RUN apt-get update &&\
    apt-get install unzip -y &&\
    apt-get install net-tools -y &&\ 
    apt-get install iputils-ping -y &&\ 
    apt-get install vim -y &&\
    apt-get autoclean &&\
    apt-get autoremove
COPY opendaylight-0.11.0.zip jdk-8u212-linux-x64.tar.gz /home/
WORKDIR /home
RUN tar -zxvf jdk-8u212-linux-x64.tar.gz &&\
    rm jdk-8u212-linux-x64.tar.gz &&\
    unzip opendaylight-0.11.0.zip &&\
    rm opendaylight-0.11.0.zip	
 
ENV JAVA_HOME=/home/jdk1.8.0_212
ENV JRE_HOME=$JAVA_HOME/jre 
ENV CLASSPATH=.:$JAVA_HOME/lib:$JRE_HOME/lib:$CLASSPATH
ENV PATH=$JAVA_HOME/bin:$JRE_HOME/bin:$PATH
 
EXPOSE 8212 6653 6633

CMD ./opendaylight-0.11.0/bin/karaf


```

```dockerfile
FROM anapsix/alpine-java:8_jdk

WORKDIR /home
COPY opendaylight-0.11.0.zip ./

RUN apk add --no-cache gcc g++ make libc-dev python-dev openssl unzip && \
    apk add maven --update-cache --repository http://dl-3.alpinelinux.org/alpine/edge/community/ && \
    unzip opendaylight-0.11.0.zip && \
    rm opendaylight-0.11.0.zip	&& \
    apk del gcc make python-dev libc-dev g++ maven unzip && \
    rm -rf /var/cache/apk/*

EXPOSE 8181 6633 8101

CMD ./opendaylight-0.11.0/bin/karaf
```


```mininet
docker run -it --rm --privileged -e DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /lib/modules:/lib/modules iwaseyusuke/mininet
```

https://hub.docker.com/r/iwaseyusuke/mininet


```opendaylight

docker run -dit -p 6633:6633 -p 8181:8181 -p 8101:8101 --name=opendaylight opendaylight_alpine

feature:install odl-restconf odl-dluxapps-applications odl-openflowplugin-flow-services-ui odl-l2switch-switch-ui odl-mdsal-apidocs

ssh -p 8101 karaf@localhost Default password is "karaf"

http://localhost:8181/index.html

```
https://hub.docker.com/r/glefevre/opendaylight

feature:install odl-openflowplugin-all


Windows 下 Docker 虚拟机存储的移动，管理工具->Hyper V 管理->选定 Docker 虚拟机->右键选择移动

[jdk 下载地址](https://github.com/frekele/oracle-java/releases)

[构建 docker 镜像](https://www.sdnlab.com/22554.html)