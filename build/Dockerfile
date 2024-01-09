FROM registry.cn-hangzhou.aliyuncs.com/publicliuyi/spark-operator:v1beta2-1.3.8-3.1.1

USER root
ENV SPARK_HOME=/opt/spark
ENV PATH=$PATH:${SPARK_HOME}/bin
RUN mkdir -p /opt/spark/conf
COPY hadoop-cos-3.1.0-8.3.5.jar /opt/spark/jars/
COPY cos_api-bundle-5.6.137.2.jar /opt/spark/jars/
COPY ./spark-defaults.conf /opt/spark/conf/spark-defaults.conf

RUN groupadd -g 185 spark && \
    useradd -u 185 -g 185 spark

USER 185

ENTRYPOINT ${SPARK_HOME}/sbin/start-history-server.sh