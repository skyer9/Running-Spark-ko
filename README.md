# Running Spark

빅데이타 분석을 위한 스파크 사용법을 설명합니다.

## flintrock 설치

[flintrock](./flintrock.md) 을 이용해 스파크 클러스터를 생성합니다.

## pyspark 설치

스파크 클러스터에 [pyspark](./pyspark.md) 를 설치합니다.

## aws s3 접근하기

[AWS S3](./s3.md) 에 데이타 파일을 올려서 데스트 데이타로 사용합니다.

## Spark SQL 사용하기

[Spark SQL](./spark-sql.md) 를 이용해 데이타를 분석할 수 있습니다.

## Jupyter 로 스파크 실행하기

[Jupyter](./jupyter.md) 를 이용해 스파크를 실행할 수 있습니다.

## Hadoop 설정하기

[Hadoop](./hadoop.md) 을 이용하여 노드간 파일을 공유할 수 있습니다.

## Python vs Scala

Python 보다 Scala 로 코딩을 하면 10배 더 빠르다고 합니다.

하지만, Scala 보다 Python 개발자가 10배 더 많습니다.

이것이 어떤 결과를 가져올지 생각해 보시면 어떤 언어를 선택할 지 감이 오실 듯 합니다.

## port 정리

Spark UI

```url
http://<마스터노드 퍼블릭 아이피>:8080
```

Hadoop UI

```url
http://<마스터노드 퍼블릭 아이피>:50070
```

## yarn 설정하기

[yarn]('/yarn.md) 을 이용해 클러스터를 컨트롤 할 수 있습니다.
