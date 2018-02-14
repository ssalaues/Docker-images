# Inspired by https://hub.docker.com/r/singularities/spark/~/dockerfile/

FROM openjdk:8-jdk
MAINTAINER Lauren Spiegel <lauren.spiegel@scality.com>

# Update packages
RUN apt-get -qq -y update

# Software installation
RUN apt-get -qq -y install build-essential telnet iperf build-essential wget curl less rsync vim

# Install hadoop
ENV HADOOP_VERSION=2.8.1
ENV HADOOP_DIR=/opt/hadoop
# note that HADOOP_PREFIX is used when calling bin/hadoop commands so must be defined
ENV HADOOP_HOME=$HADOOP_DIR/hadoop-$HADOOP_VERSION
RUN mkdir -p $HADOOP_DIR
RUN curl -fSL "http://mirrors.gigenet.com/apache/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz " -o /tmp/hadoop.tar.gz
RUN tar -xf /tmp/hadoop.tar.gz -C $HADOOP_DIR && rm -f /tmp/hadoop.tar.gz

# Add AWS jar file to classpath
RUN sed -i '43i export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:/opt/hadoop/hadoop-$HADOOP_VERSION/share/hadoop/tools/lib/*' "${HADOOP_HOME}/etc/hadoop/hadoop-env.sh"

# Add s3 endpoint and accessKey/secretKey info
ADD core-site.xml.template $HADOOP_HOME/etc/hadoop/core-site.xml

# Add hadoop to path
ENV PATH=$HADOOP_HOME/bin:$PATH

# Install Spark
ENV SPARK_VERSION=2.2.0
ENV SPARK_HOME=/opt/spark/spark-$SPARK_VERSION

RUN mkdir -p "${SPARK_HOME}" \
  && curl -fSL "https://archive.apache.org/dist/spark/spark-$SPARK_VERSION/spark-$SPARK_VERSION-bin-without-hadoop.tgz" -o spark.tgz \
  &&  tar -xvf spark.tgz -C $SPARK_HOME --strip-components 1 \
  && rm -rf spark.tgz
COPY spark-env.sh $SPARK_HOME/conf/spark-env.sh
ENV PATH=$SPARK_HOME/bin:$PATH

# Add entrypoint script
COPY entrypoint.sh /entrypoint.sh

WORKDIR $SPARK_HOME

# Run script to update s3 configuration in hadoop with env variables sent to docker run command
ENTRYPOINT ["/entrypoint.sh"]
# Start spark shell
CMD bin/spark-shell
