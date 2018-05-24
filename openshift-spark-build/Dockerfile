# Copyright 2017 Red Hat
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and # limitations under the License.
#
# ------------------------------------------------------------------------
#
# This is a Dockerfile for the radanalyticsio/openshift-spark:2.3-latest image.


FROM centos:latest

# Environment variables
ENV JBOSS_IMAGE_NAME="radanalyticsio/openshift-spark" \
    JBOSS_IMAGE_VERSION="2.3-latest" \
    SPARK_HOME="/opt/spark" \
    APP_ROOT="/opt/app-root" \
    PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/opt/spark/bin:/opt/rh/rh-python36/root/usr/bin"


# Labels
LABEL name="$JBOSS_IMAGE_NAME" \
      version="$JBOSS_IMAGE_VERSION" \
      architecture="x86_64" \
      com.redhat.component="radanalyticsio-openshift-spark-docker" \
      maintainer="Emma Qiu <emqiu@redhat.com>" \
      sparkversion="2.3.0" \
      org.concrt.version="1.4.0"


USER root

# Install rh-python36 on centos7
RUN yum install -y centos-release-scl-rh && \
    yum-config-manager --enable centos-sclo-rh && \
    yum install -y rh-python36

# Enable the rh-python36
RUN mkdir -p ${APP_ROOT}/etc
RUN echo "source scl_source enable rh-python36" > ${APP_ROOT}/etc/scl_enable
RUN chmod 666 ${APP_ROOT}/etc/scl_enable
ENV BASH_ENV=${APP_ROOT}/etc/scl_enable \
    ENV=${APP_ROOT}/etc/scl_enable \
    PROMPT_COMMAND=". ${APP_ROOT}/etc/scl_enable"

USER root

# Install required RPMs and ensure that the packages were installed
RUN yum install -y java-1.8.0-openjdk wget curl tree java-headless bzip2 gnupg2 sqlite3 gcc gcc-c++ glibc-devel git mesa-libGL mesa-libGL-devel ca-certificates vim epel-release \
    && yum clean all \
    && rm -rf /var/cache/yum \
    && rpm -q java-1.8.0-openjdk wget

# Add all artifacts to the /tmp/artifacts
# directory
COPY \
     spark-2.3.0-bin-hadoop2.7.tgz \
     /tmp/artifacts/

# Add scripts used to configure the image
COPY modules /tmp/scripts

# Custom scripts
USER root
RUN [ "bash", "-x", "/tmp/scripts/spark/install" ]

USER root
RUN [ "bash", "-x", "/tmp/scripts/metrics/install" ]

RUN rm -rf /tmp/scripts
RUN rm -rf /tmp/artifacts
    
#Install the NLTK library for Spark
RUN pip install --no-cache-dir nltk

RUN export NLTK_DATA=/usr/share/nltk_data

#The NLTK library needs a language, this installs punkt
RUN /opt/rh/rh-python36/root/usr/bin/python -m nltk.downloader -d /usr/share/nltk_data punkt

RUN chmod 0755 /usr/share/nltk_data
		
#Install other packages		
RUN pip install --no-cache-dir pandas scipy seaborn scikit-learn protobuf ipywidgets 

USER 185

# Specify the working directory
WORKDIR /tmp

ENTRYPOINT ["/entrypoint"]

CMD ["/opt/spark/bin/launch.sh"]
