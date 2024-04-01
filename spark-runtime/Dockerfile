FROM apache/spark-py:v3.1.3

USER root
COPY spark/sources.list /etc/apt/sources.list
COPY spark/*.jar /opt/spark/jars/
RUN apt update && apt install -y vim telnet iputils-ping net-tools wget curl && \
    rm -f /opt/spark/jars/httpclient-4.5.6.jar && \
    groupadd -g 185 spark && \
    useradd -u 185 -g 185 spark && \
    mkdir -p /opt/spark/logs && \
    chown -R 185 /opt/spark
USER 185
ENV SPARK_NO_DAEMONIZE=false
ENV SPARK_HOME=/opt/spark
ENV PATH=$PATH:${SPARK_HOME}/bin
