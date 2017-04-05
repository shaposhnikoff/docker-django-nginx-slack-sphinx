FROM ubuntu
MAINTAINER Max I. Shaposhnikoff <max@shaposhnikoff.info>

RUN rm /bin/sh && ln -s /bin/bash /bin/sh

RUN sed -i 's%archive.ubuntu.com%ua.archive.ubuntu.com%' /etc/apt/sources.list

ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
	apt-get -y install software-properties-common python-software-properties sudo git jq dnsutils iputils-ping


RUN add-apt-repository -y ppa:webupd8team/java

RUN apt-get update 
RUN echo debconf shared/accepted-oracle-license-v1-1 select true | sudo debconf-set-selections 
RUN echo debconf shared/accepted-oracle-license-v1-1 seen true | sudo debconf-set-selections 
RUN apt-get -y install oracle-java8-installer

ADD hybris-commerce-suite-5.5.1.4.tar.gz /opt/
COPY mysql-connector-java-5.1.37-bin.jar /opt/hybris/bin/platform/lib/dbdriver/mysql-connector-java-5.1.37-bin.jar

RUN apt-get update && \
	apt-get -y install npm

RUN ln -s /usr/bin/nodejs /usr/bin/node
