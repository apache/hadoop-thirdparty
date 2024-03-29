
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
# Install Apache Maven 3.6.0 (3.6.0 ships with Bionic)
######
# hadolint ignore=DL3008
RUN apt-get -q update \
    && apt-get -q install -y --no-install-recommends maven \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
ENV MAVEN_HOME /usr
# JAVA_HOME must be set in Maven >= 3.5.0 (MNG-6003)
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

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
RUN pip2 install lazy-object-proxy==1.5.0
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
    && dpkg --install hugo.deb \
    && rm hugo.deb


# Add a welcome message and environment checks.
COPY hadoop_env_checks.sh /root/hadoop_env_checks.sh
RUN chmod 755 /root/hadoop_env_checks.sh
# hadolint ignore=SC2016
RUN echo '${HOME}/hadoop_env_checks.sh' >> /root/.bashrc
