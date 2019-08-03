# yarn 설정하기

yarn 을 이용해 클러스터를 컨트롤 할 수 있습니다.

**yarn 을 실행시키기 위해서는 16G 이상의 메모리를 가지는 인스턴스를 생성하기를 권장합니다.**

```sh
vi yarn-site.xml
```

```xml
<configuration>

    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>마스터 노드 프라이빗 아이피</value>
    </property>
    <property>
        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
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

cp $HADOOP_PREFIX/etc/hadoop/capacity-scheduler.xml $HADOOP_PREFIX/conf/
cp $HADOOP_PREFIX/etc/hadoop/log4j.properties $HADOOP_PREFIX/conf/
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

정상 작동을 확인합니다.

```sh
vi hello.py
```

```python
# -*- coding: utf-8 -*-
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName('abc').getOrCreate()

print('Hello, World!')
```

```sh
flintrock copy-file bigdata-cluster hello.py ./
flintrock login bigdata-cluster
spark-submit --master yarn \
             --deploy-mode cluster \
               hello.py
```

아래 명령을 이용해 로그를 확인할 수 있습니다.

```sh
yarn logs -applicationId application_1564819346305_0012
```
