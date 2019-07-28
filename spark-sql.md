# Spark SQL 사용하기

```python
# -*- coding: utf-8 -*-
import pyspark
from pyspark.sql import SQLContext

sc = pyspark.SparkContext.getOrCreate()
sc.setSystemProperty("com.amazonaws.services.s3.enableV4", "true")

hadoopConf = sc._jsc.hadoopConfiguration()
hadoopConf.set("fs.s3a.impl", "org.apache.hadoop.fs.s3a.S3AFileSystem")
hadoopConf.set("fs.s3a.endpoint", 's3.ap-northeast-2.amazonaws.com')

sqlContext = SQLContext(sc)

df = sqlContext.read.load('s3a://skyer9-test/sample_us.tsv', format='csv', sep='\t', header='true')

df.createOrReplaceTempView('tmp_ratingdata')

sql = """
    SELECT customer_id
    FROM tmp_ratingdata
"""
df = sqlContext.sql(sql)
cnt = df.count()
print("Totel counts in csv file: %i" % (cnt))

sc.stop()
```
