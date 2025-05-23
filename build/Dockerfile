# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ARG CK_BASE_IMAGE=eclipse-temurin:17-jdk

FROM $CK_BASE_IMAGE AS base

ARG IMAGE_ARCH

ARG MAVEN_DEFAULT_VERSION="3.8.6"
ARG MAVEN_HOME="/usr/share/maven"
ARG MAVEN_DIST_URL="https://archive.apache.org/dist/maven/maven-3/${MAVEN_DEFAULT_VERSION}/binaries/apache-maven-${MAVEN_DEFAULT_VERSION}-bin.zip"
ARG MVNW_DIR="/usr/share/maven/mvnw/"
ARG MVN_REPO="/etc/maven/m2"

USER 0

# Maven configuration
RUN mkdir -p ${MAVEN_HOME}
RUN mkdir -p ${MVN_REPO}

# Prepare the maven wrapper required in the build Pod strategy
COPY build/mvnw/mvnw.tar ${MVNW_DIR}
RUN tar -xC ${MVNW_DIR} -f ${MVNW_DIR}mvnw.tar \
    && rm -v ${MVNW_DIR}mvnw.tar \
    && sed -i "s;distributionUrl=.*;distributionUrl=${MAVEN_DIST_URL};" ${MVNW_DIR}.mvn/wrapper/maven-wrapper.properties
# Used by mvnw to download maven dist into it
ENV MAVEN_USER_HOME="${MAVEN_HOME}"
# Install a default mvnw distribution at build time
RUN ${MVNW_DIR}/mvnw --version

ADD build/_maven_output ${MVN_REPO}
# Fix https://github.com/moby/moby/issues/37965
RUN true

RUN chgrp -R 0 ${MVN_REPO} \
    && chown -R 1001:0 ${MVN_REPO} \
    && chmod -R 775 ${MVN_REPO} \
    && chgrp -R 0 ${MAVEN_HOME} \
    && chown -R 1001:0 ${MAVEN_HOME} \
    && chmod -R 775 ${MAVEN_HOME}

USER 1001:0

ADD build/_output/bin/kamel-${IMAGE_ARCH} /usr/local/bin/kamel

FROM golang:1.22 AS go

RUN go install github.com/go-delve/delve/cmd/dlv@latest

FROM base AS debug

COPY --from=go /go/bin/dlv /usr/local/bin/dlv
