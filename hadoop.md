# Hadoop 설정하기

Hadoop 을 이용하여 노드간 파일을 공유할 수 있습니다.

```sh
vi /home/ec2-user/.config/flintrock/config.yaml
```

```yaml
services:
  spark:
    version: 2.4.4
    download-source: "http://apache.mirror.cdnetworks.com/spark/spark-{v}/spark-{v}-bin-hadoop2.7.tgz"
    # executor-instances: 1
  hdfs:
    version: 2.8.5
    download-source: "http://apache.mirror.cdnetworks.com/hadoop/common/hadoop-{v}/hadoop-{v}.tar.gz"

provider: ec2

providers:
  ec2:
    key-name: keyname
    identity-file: /home/ec2-user/keyname.pem
    instance-type: m5.large
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

```sh
flintrock launch bigdata-cluster
Launching 2 instances...
[54.180.88.16] SSH online.
[13.125.242.238] SSH online.
[54.180.88.16] Configuring ephemeral storage...
[13.125.242.238] Configuring ephemeral storage...
[54.180.88.16] Installing Java 1.8...
[13.125.242.238] Installing Java 1.8...
[54.180.88.16] Installing HDFS...
[13.125.242.238] Installing HDFS...
[54.180.88.16] Installing Spark...
[13.125.242.238] Installing Spark...
[172.31.31.136] Configuring HDFS master...
[172.31.31.136] Configuring Spark master...
HDFS online.
Spark online.
launch finished in 0:02:00.
Cluster master: ec2-54-180-88-16.ap-northeast-2.compute.amazonaws.com
Login with: flintrock login bigdata-cluster
```

아래 주소를 접속하면 하둡의 상태를 확인할 수 있습니다.

```url
http://<마스터노드 퍼블릭 아이피>:50070/dfshealth.html#tab-overview
```

아래의 명령으로 하둡 파일시스템에 파일을 복사해 넣을 수 있습니다.

```sh
flintrock login bigdata-cluster
wget https://s3.amazonaws.com/amazon-reviews-pds/tsv/sample_us.tsv
hadoop fs -copyFromLocal sample_us.tsv /
hdfs dfs -ls -R /
```

pyspark 를 설치합니다.

```sh
# 모든 스파크 클러스터 노드에 pyspark 설치
flintrock run-command bigdata-cluster 'sudo yum -y install python37'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pypandoc'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pyspark'

# support library
flintrock run-command bigdata-cluster 'sudo pip-3.7 install numpy'

# pyspark python 버전 지정
flintrock run-command bigdata-cluster 'echo "export PYSPARK_PYTHON=/usr/bin/python3" >> ~/.bashrc'
```

jar 파일을 설치합니다.

``` sh
exit
wget https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk/1.7.4/aws-java-sdk-1.7.4.jar
wget https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/2.7.6/hadoop-aws-2.7.6.jar
flintrock copy-file bigdata-cluster aws-java-sdk-1.7.4.jar ./spark/jars/
flintrock copy-file bigdata-cluster hadoop-aws-2.7.6.jar ./spark/jars/
```

`test-spark-sql-hadoop.py` 를 생성합니다.

```sh
vi test-spark-sql-hadoop.py
```

```python
# -*- coding: utf-8 -*-
import pyspark
from pyspark.sql import SQLContext

sc = pyspark.SparkContext.getOrCreate()

sqlContext = SQLContext(sc)

# 출처 : https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt
df = sqlContext.read.load('hdfs:///skyer9-test/sample_us.tsv', format='csv', sep='\t', header='true')

df.createOrReplaceTempView('tmp_ratingdata')

sql = """
    SELECT
        customer_id, product_id, product_category, star_rating, review_date
    FROM
        tmp_ratingdata
    ORDER BY
        customer_id, product_id
"""
result = sqlContext.sql(sql)
result.show()

sc.stop()
```

```sh
flintrock copy-file bigdata-cluster test-spark-sql-hadoop.py ./

# 마스터 노드에 로그인
flintrock login bigdata-cluster
```

```sh
spark-submit --master spark://172.31.8.129:7077 \
               test-spark-sql-hadoop.py
```

## Hadoop - Name node is in safe mode. 에러 해결

```sh
flintrock login bigdata-cluster
hadoop/bin/hadoop dfsadmin -safemode leave
hadoop/sbin/stop-all.sh
hadoop/sbin/start-all.sh
hadoop/bin/hadoop dfsadmin -safemode leave
```
