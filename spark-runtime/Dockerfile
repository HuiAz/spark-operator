FROM apache/spark-py:v3.1.3

USER root
COPY spark/sources.list /etc/apt/sources.list
COPY spark/*.jar /opt/spark/jars/

RUN apt update && apt install -y vim telnet iputils-ping net-tools wget curl