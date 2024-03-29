# pyspark on k8s

---
该项目是介绍在k8s集群中运行pyspark任务。
包含了以下部分：
1. spark-operator，用于在k8s集群安装 spark 运行环境。
2. spark-history-operator，用于在k8s集群安装 spark-history。
3. spark-runtime， 用于构建 spark 运行环境的镜像，目前支持 spark-3.1.1，以及腾讯 COS 对象存储、MinIO 对象存储以及AWS S3
。


## 1. 部署 Minio 对象存储以及 Spark Job

---

#### 1.1 Minio 部署
```yaml
version: '3.9'
services:
    minio:
        command: 'server /data --console-address ":9001"'
        image: quay.io/minio/minio
        environment:
            - MINIO_ROOT_USER=console
            - MINIO_ROOT_PASSWORD=console123
        volumes:
            - './data:/data'
        container_name: minio
        ports:
            - '9001:9001'
            - '9000:9000'
```
##### 1.1.1 Minio 配置
```shell
mc config host add myminio http://127.0.0.1:9000 console console123
mc mb myminio/spark-test
mc cp test.json myminio/spark-test/test.json
```
#### 1.1.2 Spark-runtime 构建
```shell
cd spark-runtime
docker build -t pyspark-runner:v1 .
```

#### 1.1.3 构建 Spark Job 镜像
例如:
```shell
docker build -t pyspark-demo:dev-v.0.1 .
```


#### 1.1.4 Spark Job 部署

**协议映射**
对于不同的对象存储，需要配置不同的协议，文件地址写法不同。例如：
```shell
hdfs://a/b/c
cosn://a/b/c
s3a://a/b/c
```

```python
PROTOCOL_MAP = {
    'cos': 'cosn',
    's3': 's3a',
    'hdfs': 'hdfs',
    'http': 'http',
    'https': 'https',
    'file': 'file'
}
```


根据实际情况修改 `pyspark-demo` 文件中的配置信息，然后部署 SparkApplication, 例如：

```yaml
apiVersion: sparkoperator.k8s.io/v1beta2
kind: SparkApplication
metadata:
  name: pyspark-demo
  namespace: default
spec:
  driver:
    coreLimit: "2"
    cores: 1
    labels:
      version: 3.1.1
    memory: 2G
    serviceAccount: my-spark-spark
  executor:
    cores: 2
    instances: 4
    labels:
      version: 3.1.1
    memory: 2G
    serviceAccount: my-spark-spark
  image: pyspark-demo:dev-v.0.1
  imagePullPolicy: Always
  imagePullSecrets: 
  - [secret-name] # 私有仓库的 secret
  mainApplicationFile: local:///opt/spark/work-dir/demo.py
  arguments:
    - '' # Kafka broker
    - '' # Kafka topic
    - '' # Kafka consumer group
    - '' # Minio access key
    - '' # Minio secret key
    - '' # Minio endpoint
    - '' # Minio bucket
    - '' # Minio region
    - '' # Minio storage type
    - '' # Minio prefix 
  mode: cluster
  pythonVersion: "3"
  restartPolicy:
    onFailureRetries: 3
    onFailureRetryInterval: 10
    onSubmissionFailureRetries: 5
    onSubmissionFailureRetryInterval: 20
    type: OnFailure
  sparkConf:
    spark.hadoop.fs.s3a.access.key:  # Minio access key
    spark.hadoop.fs.s3a.secret.key:  # Minio secret key
    spark.hadoop.fs.s3a.path.style.access: true 
    spark.hadoop.fs.s3a.connection.ssl.enabled: false
    spark.hadoop.fs.s3a.block.size: 512M
    spark.hadoop.fs.s3a.buffer.dir: file:///tmp
    spark.hadoop.fs.s3a.committer.magic.enabled: false
    spark.hadoop.fs.s3a.committer.name: directory
    spark.hadoop.fs.s3a.committer.staging.abort.pending.uploads: true
    spark.hadoop.fs.s3a.committer.staging.conflict-mode: append
    spark.hadoop.fs.s3a.committer.staging.tmp.path: /tmp/staging
    spark.hadoop.fs.s3a.committer.staging.unique-filenames: true
    spark.hadoop.fs.s3a.committer.threads: 2048
    spark.hadoop.fs.s3a.connection.establish.timeout: 5000
    spark.hadoop.fs.s3a.connection.maximum: 8192
    spark.hadoop.fs.s3a.connection.timeout: 200000
    spark.hadoop.fs.s3a.endpoint: # Minio endpoint
    spark.hadoop.fs.s3a.fast.upload.active.blocks: 2048
    spark.hadoop.fs.s3a.fast.upload.buffer: disk
    spark.hadoop.fs.s3a.fast.upload: true
    spark.hadoop.fs.s3a.impl: org.apache.hadoop.fs.s3a.S3AFileSystem
    spark.hadoop.fs.s3a.max.total.tasks: 2048
    spark.hadoop.fs.s3a.multipart.size: 512M
    spark.hadoop.fs.s3a.multipart.threshold: 512M
    spark.hadoop.fs.s3a.socket.recv.buffer: 65536
    spark.hadoop.fs.s3a.socket.send.buffer: 65536
    spark.hadoop.fs.s3a.threads.max: 2048
    spark.eventLog.dir: s3a://spark-test/spark-history # spark history
    spark.eventLog.enabled: "true"
    spark.history.fs.logDirectory: s3a://spark-test/spark-history
    spark.history.ui.port: "18080"
  sparkVersion: 3.1.1
  type: Python
```

