# Flintrock 을 이용해 스파크 클러스터를 생성

`AWS EC2` 인스턴스에서 스파크 클러스터를 생성합니다.

`Flintrock` 이라는 툴을 이용해 클러스터를 생성합니다.

## flintrock 설치

```sh
# pip 를 이용해 설치한다.
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
    version: 3.2.0
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
  # install-hdfs: True
  # install-spark: False

debug: false
```

## 스파크 클러스터 생성

클러스터를 생성합니다. 처음 생성하면 에러가 발생합니다.

```sh
flintrock launch bigdata-cluster
......
Do you want to terminate the 2 instances created by this operation? [Y/n]:
```

보안그룹을 보면 새로 생성된 `flintrock` 이 있습니다.

여기에 클러스터를 생성 시도하고 있는 인스턴스의 아이피를 추가해 줍니다.(모든 TCP, 0-65535, 인스턴스 프라이빗 아이피)

위 보안그룹을 수정후 다시 생성을 시도하면 정상적으로 클러스터가 생성됩니다.

```sh
flintrock launch bigdata-cluster
Launching 2 instances...
[15.164.103.149] SSH online.
[15.164.103.149] Configuring ephemeral storage...
[15.164.103.149] Installing Java 1.8...
[13.125.236.42] SSH online.
[13.125.236.42] Configuring ephemeral storage...
[13.125.236.42] Installing Java 1.8...
[13.125.236.42] Installing Spark...
[15.164.103.149] Installing Spark...
[172.31.23.241] Configuring Spark master...
Spark online.
launch finished in 0:01:20.
Cluster master: ec2-15-164-103-149.ap-northeast-2.compute.amazonaws.com
Login with: flintrock login bigdata-cluster
```

생성 후 마스터 노드에 접속할 수 있습니다.

```sh
flintrock login bigdata-cluster
Warning: Permanently added '15.164.103.149' (ECDSA) to the list of known hosts.
Last login: Sat Jul 27 21:39:03 2019 from ip-172-31-14-36.ap-northeast-2.compute.internal

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
3 package(s) needed for security, out of 8 available
Run "sudo yum update" to apply all updates.


spark-shell
19/07/27 21:39:52 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://ec2-15-164-103-149.ap-northeast-2.compute.amazonaws.com:4040
Spark context available as 'sc' (master = local[*], app id = local-1564263602036).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.4.3
      /_/

Using Scala version 2.11.12 (OpenJDK 64-Bit Server VM, Java 1.8.0_201)
Type in expressions to have them evaluated.
Type :help for more information.

scala> println("Hello, world!")
Hello, world!

scala> :q
```
