# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements. See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership. The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied. See the License for the
# specific language governing permissions and limitations
# under the License.
#

#FROM openjdk:8-jre
FROM centos:7.7

LABEL maintainer="Apache NiFi <dev@nifi.apache.org>"
LABEL site="https://nifi.apache.org"

ARG UID=1000
ARG GID=1000
ARG NIFI_VERSION=1.7.0
ARG MIRROR=https://archive.apache.org/dist

ENV NIFI_BASE_DIR /opt/nifi
ENV NIFI_HOME=${NIFI_BASE_DIR}/nifi-${NIFI_VERSION} \
    NIFI_BINARY_URL=/nifi/${NIFI_VERSION}/nifi-${NIFI_VERSION}-bin.tar.gz
ENV NIFI_PID_DIR=${NIFI_HOME}/run
ENV NIFI_LOG_DIR=${NIFI_HOME}/logs
ENV NIFI_LIB_DIR=${NIFI_HOME}/lib
# Added directory for executing start.sh file
ADD /nifi-rel-nifi-1.7.0/nifi-docker/dockerhub/sh/ ${NIFI_BASE_DIR}/scripts/

WORKDIR /opt/
# udpating file for building draco

RUN git clone https://github.com/pooja1pathak/fiware-draco.git && \
    cd fiware-draco
    sudo echo "*  hard  nofile  50000" >> /etc/security/limits.conf
    sudo echo "*  soft  nofile  50000" >> /etc/security/limits.conf
    sudo sysctl -w net.ipv4.ip_local_port_range="10000 65000"
    sudo sysctl -w net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait="1"
    sudo yum install java-1.8.0-openjdk
    sudo yum install java-1.8.0-openjdk-devel
    sudo echo "JAVA_HOME=$(readlink -f /usr/bin/java | sed "s:bin/java::")" | sudo tee -a /etc/profile
    source /etc/profile
    
    
#$$$$$$ Java installed$$$$$$$$

#Install Maven

    sudo cd /usr/local/src/
    sudo wget http://www-us.apache.org/dist/maven/maven-3/3.5.4/binaries/apache-maven-3.5.4-bin.tar.gz
    sudo tar -xf apache-maven-3.5.4-bin.tar.gz
    sudo mv apache-maven-3.5.4/ apache-maven/
    sudo echo "export JAVA_HOME=$JAVA_HOME" >> /etc/profile.d/maven.sh
    sudo echo "export M2_HOME=/usr/local/src/apache-maven" >> /etc/profile.d/maven.sh
    sudo echo "export PATH=${M2_HOME}/bin:${PATH}" >> /etc/profile.d/maven.sh
    sudo chmod +x /etc/profile.d/maven.sh
    source /etc/profile.d/maven.sh
    mvn --version
    
 #$$$$$$$$$$$$$ maven installed $$$$$$$$$$$$$$$$$$$$$
    
    cd /opt/fiware-draco/nifi-ngsi-bundle
    sudo mvn -T C2.0 clean install -Pinclude-grpc
    cd /opt
    sudo curl -kOL https://archive.apache.org/dist/nifi/1.7.0/nifi-1.7.0-bin.tar.gz
    sudo tar -xvzf nifi-1.7.0-bin.tar.gz
    sudo cp /opt/fiware-draco/nifi-ngsi-bundle/nifi-ngsi-nar/target/nifi-ngsi-nar-1.1.0.nar /opt/nifi-1.7.0/lib/
    sudo sed -i "s/8080/9090/g" /opt/nifi-1.7.0/conf/nifi.properties
    
CMD ENTRYPOINT ["./nifi.sh","start"]
    
# Setup NiFi user and create necessary directories
RUN groupadd -g ${GID} nifi || groupmod -n nifi `getent group ${GID} | cut -d: -f1` \
    && useradd --shell /bin/bash -u ${UID} -g ${GID} -m nifi \
    && mkdir -p ${NIFI_HOME}/conf/templates \
    && mkdir -p $NIFI_BASE_DIR/data \
    && mkdir -p $NIFI_BASE_DIR/flowfile_repository \
    && mkdir -p $NIFI_BASE_DIR/content_repository \
    && mkdir -p $NIFI_BASE_DIR/provenance_repository \
    && mkdir -p $NIFI_LOG_DIR \
    && mkdir -p $NIFI_LIB_DIR \
    && chown -R nifi:nifi ${NIFI_BASE_DIR} \
    && apt-get update \
    && apt-get install -y jq xmlstarlet procps

# Added file for adding Draco's functionality
#ADD nifi-rel-nifi-1.7.0/nifi-docker/dockerhub/nifi-ngsi-nar-1.1.0.nar ${NIFI_LIB_DIR}

#USER nifi

# Download, validate, and expand Apache NiFi binary.
#RUN curl -fSL ${MIRROR}/${NIFI_BINARY_URL} -o ${NIFI_BASE_DIR}/nifi-${NIFI_VERSION}-bin.tar.gz \
    && echo "$(curl https://archive.apache.org/dist/${NIFI_BINARY_URL}.sha256) *${NIFI_BASE_DIR}/nifi-${NIFI_VERSION}-bin.tar.gz" | sha256sum -c - \
    && tar -xvzf ${NIFI_BASE_DIR}/nifi-${NIFI_VERSION}-bin.tar.gz -C ${NIFI_BASE_DIR} \
    && rm ${NIFI_BASE_DIR}/nifi-${NIFI_VERSION}-bin.tar.gz

# Clear nifi-env.sh in favour of configuring all environment variables in the Dockerfile
#RUN echo "#!/bin/sh\n" > $NIFI_HOME/bin/nifi-env.sh

# Web HTTP(s) & Socket Site-to-Site Ports
EXPOSE 5050 6060 9090

#WORKDIR ${NIFI_HOME}

# Apply configuration and start NiFi
#
# We need to use the exec form to avoid running our command in a subshell and omitting signals,
# thus being unable to shut down gracefully:
# https://docs.docker.com/engine/reference/builder/#entrypoint
#
# Also we need to use relative path, because the exec form does not invoke a command shell,
# thus normal shell processing does not happen:
# https://docs.docker.com/engine/reference/builder/#exec-form-entrypoint-example
ENTRYPOINT ["../scripts/start.sh"]
