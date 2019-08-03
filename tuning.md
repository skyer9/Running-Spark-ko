# Spark 튜닝하기

Spark 튜닝 을 위한 파라미터를 확인해 봅니다.

## 용어정리

task : 최소 작업단위입니다. 하나의 core(= vcore, thread) 에서 실행됩니다.

executor : 하나의 executor(= container) 가 여러 개의 task 를 동시에 실행합니다.

```sh
spark-submit --master yarn
             --deploy-mode cluster
             --num-executors 2
             --executor-cores 3
             --driver-memory 2g
             --executor-memory 1g
             hdfs://<host>:<port>/home/hduser/count.py /data.txt
```

위 예제에서 executor 는 2개이고, 각각의 executor 는 3개의 core 를 실행합니다.

## 튜닝하기

경험적으로 하나의 executor 는 5개 이하의 task 를 실행하는 것이 좋다고 합니다.

서버가 6대, 각 서버의 CPU 는 8개, 각 서버의 메모리는 64G 인 클러스터가 있다고 가정합니다.

`서버 6대 * CPU 8개 = 48` 이고, 각 서버에 OS 등에서 사용하기 위한 CPU 를 남겨 두어야 하므로, `executor 10개 * core 4개 = 40` 으로 설정하는 것이 좋습니다.

executor 를 10개로 했으므로, 각 서버에 대략 2개의 executor 가 실행될 듯 합니다. 따라서, `서버 메모리 64G / 2 - 2 = 30G` 로 설정하는 것이 좋습니다. 2를 뺀 것은 역시 OS 등에서 사용할 여분의 메모리입니다.

최종적으로 아래와 같은 설정이 됩니다.

```sh
spark-submit --master yarn
             --deploy-mode cluster
             --num-executors 10
             --executor-cores 4
             --driver-memory 2g
             --executor-memory 30g
             hdfs://<host>:<port>/home/hduser/count.py /data.txt
```
