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

## Hadoop - Name node is in safe mode. 에러 해결

```sh
flintrock login bigdata-cluster
hadoop/bin/hadoop dfsadmin -safemode leave
hadoop/sbin/stop-all.sh
hadoop/sbin/start-all.sh
hadoop/bin/hadoop dfsadmin -safemode leave
```
