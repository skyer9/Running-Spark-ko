# Jupyter 로 스파크 실행하기

`Jupyter` 를 이용해 스파크를 실행할 수 있습니다.

우선 마스터 노드에 접속합니다.

```sh
flintrock login bigdata-cluster
```

jupyter 를 설치합니다.

```sh
sudo pip-3.7 install jupyter
jupyter notebook --generate-config
vi /home/ec2-user/.jupyter/jupyter_notebook_config.py
```

원격 접속을 허용해줍니다.

**모든 아이피에서 접속을 허용하므로 별도의 접속제한 수단을 가져야 한다.**

```text
......
c.NotebookApp.ip = '*'
......
c.NotebookApp.port = 8082
......
```

환경변수를 설정합니다. 설정파일에 등록하면 `spark-submit` 실행시 오류를 발생시키므로 추가하지 않습니다.

```sh
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='notebook'
# vi ~/.bashrc
```

노트북을 백그라운드에서 실행하도록 합니다.

```sh
nohup pyspark &
cat nohup.out
```

노트북에서 새 노트북을 생성하고 아래 코드를 실행해 볼 수 있습니다.

```python
import random
num_samples = 100
def inside(p):
  x, y = random.random(), random.random()
  return x*x + y*y < 1
count = sc.parallelize(range(0, num_samples)).filter(inside).count()
pi = 4 * count / num_samples
print(pi)
```
