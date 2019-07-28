# pyspark 설치

## pyspark 모듈 설치하기

아래와 같이 `pyspark` 패키지를 설치합니다.

```sh
# 모든 스파크 클러스터 노드에 pyspark 설치
flintrock run-command bigdata-cluster 'sudo yum -y install python37'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pypandoc'
flintrock run-command bigdata-cluster 'sudo pip-3.7 install pyspark'
```

## 테스트 파일 생성하기

`test.py` 를 생성합니다.

```sh
vi test.py
```

```python
# -*- coding: utf-8 -*-
import pyspark

sc = pyspark.SparkContext.getOrCreate()
sc.setSystemProperty("com.amazonaws.services.s3.enableV4", "true")

hadoopConf = sc._jsc.hadoopConfiguration()
hadoopConf.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
hadoopConf.set("fs.s3a.endpoint", 's3.ap-northeast-2.amazonaws.com')

# 확장자는 csv, gz 등이 가능하다.
# 출처 : https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt
df = sc.textFile("file:///home/ec2-user/sample_us.tsv").map(lambda x: x.split("\t"))

print('\n\n------------------------------------------------\n\n')
print(df.take(10))
print('\n\n------------------------------------------------\n\n')

sc.stop()
```

## 테스트 파일 실행하기

```sh
# 샘플파일 다운로드
wget https://s3.amazonaws.com/amazon-reviews-pds/tsv/sample_us.tsv

# 모든 스파크 클러스터 노드에 실행파일 복사
flintrock copy-file bigdata-cluster test.py ./
flintrock copy-file bigdata-cluster sample_us.tsv ./

# 마스터 노드에 로그인
flintrock login spark
```

```sh
spark-submit --master spark://172.31.30.40:7077 \
               --conf spark.hadoop.fs.s3a.endpoint=s3.ap-northeast-2.amazonaws.com \
               --conf spark.hadoop.fs.s3a.impl=org.apache.hadoop.fs.s3a.S3AFileSystem \
               --conf spark.executor.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.driver.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.hadoop.fs.s3a.access.key=XXXXXXXXXXXXXXXXXXXXXX \
               --conf spark.hadoop.fs.s3a.secret.key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
               test.py
```
