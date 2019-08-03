# Spark SQL 사용하기

`test-spark-sql.py` 를 생성합니다.

```sh
vi test-spark-sql.py
```

```python
# -*- coding: utf-8 -*-
import pyspark
from pyspark.sql import SQLContext

sc = pyspark.SparkContext.getOrCreate()

sqlContext = SQLContext(sc)

# 출처 : https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt
df = sqlContext.read.load('s3a://skyer9-test/sample_us.tsv', format='csv', sep='\t', header='true')

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
flintrock copy-file bigdata-cluster test-spark-sql.py ./

# 마스터 노드에 로그인
flintrock login bigdata-cluster
```

```sh
spark-submit --master spark://172.31.29.43:7077 \
               --conf spark.hadoop.fs.s3a.endpoint=s3.ap-northeast-2.amazonaws.com \
               --conf spark.executor.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.driver.extraJavaOptions=-Dcom.amazonaws.services.s3.enableV4=true \
               --conf spark.hadoop.fs.s3a.access.key=XXXXXXXXXXXXXXXXXXXXXX \
               --conf spark.hadoop.fs.s3a.secret.key=XXXXXXXXXXXXXXXXXXXXXXXXXXXXX \
               test-spark-sql.py
```
