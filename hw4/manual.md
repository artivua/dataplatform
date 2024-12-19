# Мануал по использованию Apache Spark под управлением YARN для чтения, трансформации и записи данных

## Шаг 1: Подключение к серверу
1. Подключитесь к jump-node через SSH:

## Шаг 2: Запуск Apache Spark в режиме YARN
1. Перейдите в виртуальное окружение:
   ```bash
   source venv/bin/activate
   ```
2. Запустите интерактивную оболочку IPython:
   ```bash
   ipython
   ```
3. Инициализируйте SparkSession с поддержкой YARN:
   ```python
   from pyspark.sql import SparkSession
   import pyspark.sql.functions as F
   from onetl.connection import SparkHDFS
   from onetl.file import FileDFReader

   # Создание SparkSession
   spark = SparkSession.builder \
       .appName("spark-with-yarn") \
       .master("yarn") \
       .config("spark.sql.warehouse.dir", "/user/hive/warehouse") \
       .config("spark.hive.metastore.uris", "thrift://team-5-dn-01:9083") \
       .enableHiveSupport() \
       .getOrCreate()

   # Подключение к HDFS
   hdfs = SparkHDFS(host="tmpl-nn", port=9000, spark=spark, cluster="test")
   hdfs.check()

   # Чтение данных из HDFS
   reader = FileDFReader(spark, format="csv", path="hdfs://192.168.1.23:9000/tmp/user_data.csv")
   data = reader.read(header=True, inferSchema=True)
   data.printSchema()
   data.show(5)

   # Применение трансформаций
   filtered_data = data.filter(F.col("registration_date") >= "2024-10-01")
   upper_case_data = filtered_data.withColumn("name", F.upper(F.col("name")))
   with_year = upper_case_data.withColumn("year", F.year(F.to_date(F.col("registration_date"), "yyyy-MM-dd")))
   aggregated_data = with_year.groupBy("year").agg(F.count("*").alias("count"))
   sorted_data = aggregated_data.orderBy("year")

   # Сохранение данных в HDFS
   sorted_data.write \
       .mode("overwrite") \
       .partitionBy("year") \
       .parquet("hdfs://192.168.1.23:9000/user/hive/warehouse/processed_users")
   ```

## Шаг 3: Проверка данных в Hive CLI
1. Перейдите в Hive CLI на NameNode, запустив команду:
   ```bash
   hive
   ```
2. Создайте внешнюю таблицу:
   ```sql
   CREATE EXTERNAL TABLE IF NOT EXISTS processed_users (
       name STRING,
       count INT
   )
   PARTITIONED BY (year INT)
   STORED AS PARQUET
   LOCATION 'hdfs://192.168.1.23:9000/user/hive/warehouse/processed_users';
   ```
3. Выполните SQL-запрос для проверки данных:
   ```sql
   SELECT * FROM processed_users;
   ```
