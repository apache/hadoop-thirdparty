
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Dockerfile for installing the necessary dependencies for building Hadoop.
# See BUILDING.txt.

FROM ubuntu:bionic

WORKDIR /root

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

#####
# Disable suggests/recommends
#####
RUN echo APT::Install-Recommends "0"\; > /etc/apt/apt.conf.d/10disableextras
RUN echo APT::Install-Suggests "0"\; >>  /etc/apt/apt.conf.d/10disableextras

ENV DEBIAN_FRONTEND noninteractive
ENV DEBCONF_TERSE true

######
# Install common dependencies from packages. Versions here are either
# sufficient or irrelevant.
#
# WARNING: DO NOT PUT JAVA APPS HERE! Otherwise they will install default
# Ubuntu Java.  See Java section below!
######
# hadolint ignore=DL3008
RUN apt-get -q update \
    && apt-get -q install -y --no-install-recommends \
        apt-utils \
        bats \
        build-essential \
        bzip2 \
        clang \
        cmake \
        curl \
        doxygen \
        fuse \
        g++ \
        gcc \
        git \
        gnupg-agent \
        libbz2-dev \
        libcurl4-openssl-dev \
        libfuse-dev \
        libsasl2-dev \
        libssl-dev \
        libtool \
        libzstd1-dev \
        locales \
        make \
        pinentry-curses \
        pkg-config \
        python \
        python2.7 \
        python-pip \
        python-pkg-resources \
        python-setuptools \
        python-wheel \
        rsync \
        shellcheck \
        software-properties-common \
        sudo \
        valgrind \
        zlib1g-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


#######
# OpenJDK 8
#######
# hadolint ignore=DL3008
RUN apt-get -q update \
    && apt-get -q install -y --no-install-recommends openjdk-8-jdk libbcprov-java \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

######
# Install Apache Maven 3.9.12 (Ubuntu Bionic ships 3.6.0 which is too old;
# pom.xml requires [3.6.3,))
######
RUN curl -s -S -o /tmp/maven.tar.gz \
        "https://archive.apache.org/dist/maven/maven-3/3.9.12/binaries/apache-maven-3.9.12-bin.tar.gz" \
    && echo "0a1be79f02466533fc1a80abbef8796e4f737c46c6574ede5658b110899942a94db634477dfd3745501c80aef9aac0d4f841d38574373f7e2d24cce89d694f70  /tmp/maven.tar.gz" \
        | sha512sum -c - \
    && tar -xz -C /opt -f /tmp/maven.tar.gz \
    && rm /tmp/maven.tar.gz \
    && ln -sf /opt/apache-maven-3.9.12/bin/mvn /usr/bin/mvn \
    && ln -sf /opt/apache-maven-3.9.12/bin/mvnDebug /usr/bin/mvnDebug
ENV MAVEN_HOME /opt/apache-maven-3.9.12
# JAVA_HOME must be set in Maven >= 3.5.0 (MNG-6003)
# Use a symlink to support both amd64 and arm64 builds
RUN ln -sf "$(ls -d /usr/lib/jvm/java-8-openjdk-* | head -1)" /usr/lib/jvm/java-8-openjdk
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk

######
# Install findbugs 3.1.0 (3.1.0 ships with Bionic)
# Ant is needed for findbugs
######
# hadolint ignore=DL3008
RUN apt-get -q update \
    && apt-get -q install -y --no-install-recommends findbugs ant \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
ENV FINDBUGS_HOME /usr

####
# Install pylint at fixed version (2.0.0 removed python2 support)
# https://github.com/PyCQA/pylint/issues/2294
#
# weichiu: as of 05/12/2021, downloading pylint downloads isort>=4.2.5 which
# points to isort 5.8.0 whic doesn't support python 2.7.
# Instead, download the last isort 4.x version.
####
RUN pip2 install isort==4.3.21
RUN pip2 install setuptools-scm==5.0.2
RUN pip2 install wrapt==1.11.2
RUN pip2 install lazy-object-proxy==1.4.3
RUN pip2 install pylint==1.9.2

####
# Install dateutil.parser
####
RUN pip2 install python-dateutil==2.7.3

###
# Install hadolint
####
RUN curl -L -s -S \
        https://github.com/hadolint/hadolint/releases/download/v1.11.1/hadolint-Linux-x86_64 \
        -o /bin/hadolint \
   && chmod a+rx /bin/hadolint \
   && shasum -a 512 /bin/hadolint | \
        awk '$1!="734e37c1f6619cbbd86b9b249e69c9af8ee1ea87a2b1ff71dccda412e9dac35e63425225a95d71572091a3f0a11e9a04c2fc25d9e91b840530c26af32b9891ca" {exit(1)}'

###
# Avoid out of memory errors in builds
###
ENV MAVEN_OPTS -Xms256m -Xmx1536m

###
# Everything past this point is either not needed for testing or breaks Yetus.
# So tell Yetus not to read the rest of the file:
# YETUS CUT HERE
###

# Hugo static website generator (for new hadoop site and Ozone docs)
RUN curl -L -o hugo.deb https://github.com/gohugoio/hugo/releases/download/v0.30.2/hugo_0.30.2_Linux-64bit.deb \
    && echo "4a63254d90fb44722d4dabd0a935f749248b9cf182740480836a53f27c30d90df13895361512af204382d8b8cc76aacadbbddd77d99aa8b93608d8e1ece021eb  hugo.deb" \
        | sha512sum -c - \
    && dpkg --install hugo.deb \
    && rm hugo.deb


# Add a welcome message and environment checks.
COPY hadoop_env_checks.sh /root/hadoop_env_checks.sh
RUN chmod 755 /root/hadoop_env_checks.sh
# hadolint ignore=SC2016
RUN echo '${HOME}/hadoop_env_checks.sh' >> /root/.bashrc
