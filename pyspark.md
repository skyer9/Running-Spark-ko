# pyspark 설치

## pyspark 모듈 설치하기

아래와 같이 `pyspark` 패키지를 설치합니다.

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

## 테스트 파일 생성하기

`test-pyspark.py` 를 생성합니다.

```sh
vi test-pyspark.py
```

```python
# -*- coding: utf-8 -*-
import pyspark

sc = pyspark.SparkContext.getOrCreate()

# 출처 : https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt
df = sc.textFile("file:///home/ec2-user/sample_us.tsv").map(lambda x: x.split("\t"))

print(df.take(10))

sc.stop()
```

## 테스트 파일 실행하기

```sh
# 샘플파일 다운로드
wget https://s3.amazonaws.com/amazon-reviews-pds/tsv/sample_us.tsv

# 모든 스파크 클러스터 노드에 실행파일 복사
flintrock copy-file bigdata-cluster test-pyspark.py ./
flintrock copy-file bigdata-cluster sample_us.tsv ./

# 마스터 노드에 로그인
flintrock login bigdata-cluster
```

```sh
spark-submit --master spark://172.31.29.43:7077 \
               test-pyspark.py
```
