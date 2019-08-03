# 한방 정리

위 내용을 한번에 정리 해 봅니다.

## flintrock 설치

Python 3.X 가 설치되어 있는지 확인 후, flintrock 을 설치합니다.

```sh
python3 -V
Python 3.6.2

sudo pip-3.6 install flintrock
flintrock --help
```

## flintrock 설정

스파크 클러스터 생성을 위한 설정을 합니다.

```sh
flintrock configure
vi /home/ec2-user/.config/flintrock/config.yaml
```

```yaml
services:
  spark:
    version: 2.4.3
    download-source: "http://apache.mirror.cdnetworks.com/spark/spark-{v}/spark-{v}-bin-hadoop2.7.tgz"
    # executor-instances: 1
  hdfs:
    version: 2.7.7
    download-source: "http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-{v}/hadoop-{v}.tar.gz"

provider: ec2

providers:
  ec2:
    key-name: keyname
    identity-file: /home/ec2-user/keyname.pem
    instance-type: r4.large
    region: ap-northeast-2
    # availability-zone: <name>
    ami: ami-095ca789e0549777d  # Amazon Linux 2, ap-northeast-2
    user: ec2-user
    tenancy: default  # default | dedicated
    ebs-optimized: no  # yes | no
    instance-initiated-shutdown-behavior: terminate  # terminate | stop
    # user-data: /path/to/userdata/script

launch:
  num-slaves: 1
  install-hdfs: True
  install-spark: True

debug: false
```

## 스파크 클러스터 생성

클러스터를 생성합니다. 생성에 실패하면 [여기](./flintrock.md) 를 참조하여 보안그룹을 추가합니다.

```sh
flintrock launch bigdata-cluster
```

## pyspark 모듈 설치하기

아래와 같이 `pyspark` 패키지를 설치합니다.

```sh
# 모든 스파크 클러스터 노드에 pyspark 설치
flintrock run-command bigdata-cluster 'sudo yum -y install python37'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pypandoc'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pyspark'

# pyspark python 버전 지정
flintrock run-command bigdata-cluster 'echo "export PYSPARK_PYTHON=/usr/bin/python3" >> ~/.bashrc'
```

## aws jar 파일 다운받아 설치하기

```sh
wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar
wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.7.6/hadoop-aws-2.7.6.jar
flintrock copy-file bigdata-cluster aws-java-sdk-1.7.4.jar ./spark/jars/
flintrock copy-file bigdata-cluster hadoop-aws-2.7.6.jar ./spark/jars/
```

## yarn 설정하기

```sh
vi yarn-site.xml
```

```xml
<configuration>
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>마스터 노드 프라이빗 아이피</value>
</property>
</configuration>
```

```sh
vi enable-yarn.sh
```

```sh
#!/bin/sh

export HADOOP_PREFIX=/home/ec2-user/hadoop

echo "export HADOOP_PREFIX=$HADOOP_PREFIX" >> ~/.bashrc
echo "export HADOOP_HOME=$HADOOP_PREFIX" >> ~/.bashrc
echo "export HADOOP_COMMON_HOME=$HADOOP_PREFIX" >> ~/.bashrc
echo "export HADOOP_CONF_DIR=$HADOOP_PREFIX/conf" >> ~/.bashrc
echo "export HADOOP_HDFS_HOME=$HADOOP_PREFIX" >> ~/.bashrc
echo "export HADOOP_MAPRED_HOME=$HADOOP_PREFIX" >> ~/.bashrc
echo "export HADOOP_YARN_HOME=$HADOOP_PREFIX" >> ~/.bashrc
```

생성한 파일을 클러스터에복사합니다.

```sh
flintrock copy-file bigdata-cluster yarn-site.xml ./hadoop/conf/
flintrock copy-file bigdata-cluster enable-yarn.sh ./
flintrock run-command bigdata-cluster 'sh ~/enable-yarn.sh'
```

resourcemanager 는 마스터 노드에만 실행시켜 줍니다.

```sh
flintrock login bigdata-cluster
$HADOOP_PREFIX/sbin/yarn-daemon.sh start resourcemanager
exit
```

모든 노드에 nodemanager 를 실행시켜 줍니다.

```sh
flintrock run-command bigdata-cluster '$HADOOP_PREFIX/sbin/yarn-daemon.sh start nodemanager'
```

```sh
flintrock login bigdata-cluster
wget https://s3.amazonaws.com/amazon-reviews-pds/tsv/sample_us.tsv
hadoop fs -copyFromLocal sample_us.tsv /
```

```sh
vi test-hadoop.py
```

```python
# -*- coding: utf-8 -*-
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('abc').getOrCreate()

df = sparkSession.read.option("sep", "\t").csv('hdfs://<마스터 노드 프라이빗 아이피>/sample_us.tsv')
df.write.csv("hdfs://<마스터 노드 프라이빗 아이피>/output.csv")

spark.stop()
```
