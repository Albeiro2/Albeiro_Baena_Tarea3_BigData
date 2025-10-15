# Albeiro_Baena_Tarea3_BigData
El dataset escogido para la actividad es: 
Movies Dataset (TMDB) – Ratings, Popularity, Votes

# Problema
Se quiere analizar el raitng y votos que recibieron las peliculas mas populares.
tambien simular un flujo de raitin con kafka y en streaming calcular estadisticas 
en vivo para el promedio de raiting que esta teniendo una pelicula en tiempo real

# Proceso
#Aseguramos de tener el sistema actualizado e instalar las dependencias

```bash
sudo apt update -y
```
<img width="920" height="581" alt="image" src="https://github.com/user-attachments/assets/bce29501-1dc0-44ea-86dd-bc9995f3ed40" />

```bash
sudo apt install python3-pip unzip -y
```
<img width="921" height="587" alt="image" src="https://github.com/user-attachments/assets/071df7f5-6602-48bd-afa4-aaba6bdb8c23" />

#Instalamos Kaggle para traer el dataset

```bash
pip install kaggle
```
<img width="920" height="574" alt="image" src="https://github.com/user-attachments/assets/44280d95-1bd7-4c28-978f-11d86b2b7293" />

#Creamos la carpeta de configuración para Kaggle

```bash
mkdir -p ~/.kaggle
```
<img width="625" height="188" alt="image" src="https://github.com/user-attachments/assets/d3b0ceab-92b4-4ebd-b9cd-c1b5dcd0105a" />

#Creamos el archivo Kaggle.json para ingresar la key y username de mi cuenta

```bash
nano ~/.kaggle/kaggle.json
```
<img width="920" height="464" alt="image" src="https://github.com/user-attachments/assets/494c01fe-b510-44f1-91b8-c9851e460c4a" />
#Ctrl + O y enter para guardar y luego Ctrl + X para salir

Asignamos permisos al archivo y creemos la carpeta para el dataset

```bash
chmod 600 ~/.kaggle/kaggle.json
mkdir -p ~/datasets/movies
```
<img width="705" height="88" alt="image" src="https://github.com/user-attachments/assets/a25923fd-167e-4363-a3e8-7490e0d95af4" />

#Descargamos el dataset 

```bash
kaggle datasets download -d kajaldeore04/movies-dataset-tmdb-ratings-popularity-votes -p ~/datasets/movies
```
<img width="921" height="253" alt="image" src="https://github.com/user-attachments/assets/7e801802-459a-44ae-b83f-dc3ca01a26ce" />

#Descomprimir el datase

```bash
unzip ~/datasets/movies/movies-dataset-tmdb-ratings-popularity-votes.zip -d ~/datasets/movies/
```

<img width="921" height="532" alt="image" src="https://github.com/user-attachments/assets/b0390b92-e6bc-4080-af13-5d906b6e4de3" />

#Verificar el archivo

```bash
ls ~/datasets/movies/
```

#Deberías ver un archivo llamado  “movies.csv”

#Creamos un directorio de trabajo para Spark Kafka y nos vemos adentro 

```bash
cd ~
mkdir -p spark_kafka_project/code
cd spark_kafka_project/code
```
<img width="920" height="299" alt="image" src="https://github.com/user-attachments/assets/333fb79b-3900-4aed-8d32-a7310210e859" />

#Creamos el archivo que contendrá el script para el análisis exploratorio de los datos con DataFrames 

```bash
nano spark_batch_processing.py
```

#Copiamos y pegamos el siguiente codigo

```bash
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, count

# Crear la sesión de Spark
spark = SparkSession.builder \
    .appName("Movies Batch Processing") \
    .getOrCreate()

# Ruta del dataset
path_movies = "/home/vboxuser/datasets/movies/movies.csv"

# Cargar datos
df = spark.read.option("header", True).option("inferSchema", True).csv(path_movies)

# Mostrar esquema
df.printSchema()

# Mostrar primeras filas
print("Primeras filas del dataset:")
df.show(5)

# Limpiar datos (eliminar duplicados y nulos)
df_clean = df.dropDuplicates().dropna(subset=["vote_average", "popularity"])

# Análisis exploratorio básico
print("Estadísticas de columnas numéricas:")
df_clean.describe(["vote_average", "popularity"]).show()

# Calcular promedio de votos por idioma original
avg_votes = df_clean.groupBy("original_language").agg(
    avg("vote_average").alias("promedio_votos"),
    count("*").alias("cantidad_peliculas")
)

# Mostrar resultados
print("Promedio de votos por idioma:")
avg_votes.show()

# Guardar los resultados en formato CSV
output_path = "/home/ vboxuser /spark_kafka_project/output/batch_results"
avg_votes.coalesce(1).write.mode("overwrite").option("header", True).csv(output_path)

print(f"✅ Resultados guardados en: {output_path}")

spark.stop()
```
<img width="920" height="484" alt="image" src="https://github.com/user-attachments/assets/dd018b81-15c7-4d08-bf57-3a61d398bd53" />

#Ctrl + O y enter para guardar y Ctrl + X para salir

#Ejecutamos el procesamiento 

```bash
spark-submit spark_batch_processing.py
```
<img width="921" height="623" alt="image" src="https://github.com/user-attachments/assets/8ef7b40b-ecf6-4a20-a636-33d6dd1f8c6f" />

#Deberias ver los siguientes Resultados
<img width="920" height="322" alt="image" src="https://github.com/user-attachments/assets/f306b364-d3f9-4156-ab28-047f4c95d98f" />

<img width="672" height="253" alt="image" src="https://github.com/user-attachments/assets/b3aa8030-6211-46c7-87fd-88c5bab0c73b" />

#Para verificar los resultados guardados ejecuta

```bash
ls ~/spark_kafka_project/output/batch_results/
```
# Para el procesamiento en Tiempo Real (Spark Streaming + Kafka)

#Abrimos otra sesión para Zookeeper 
#Nos movemos a la caperta donde tenemos Kafka y ejecutamos Zookeeper

```bash
cd  /opt/Kafka
bin/zookeeper-server-start.sh config/zookeeper.properties
```
<img width="921" height="591" alt="image" src="https://github.com/user-attachments/assets/7b6dc495-bdc7-4dc2-abf6-896f9d93ca1e" />

#Abrimos otra sesion para Kafka
#Nos movemos a la caperta donde tenemos Kafka y ejecutamos kafka

```bash
cd  /opt/Kafka
bin/kafka-server-start.sh config/server.properties
```
<img width="921" height="588" alt="image" src="https://github.com/user-attachments/assets/93838000-9ff8-4bb0-8260-118b0f3fc9ef" />

#Abrimos una ultima sesion para el Producer
#Nos dirigimos a la carpeta donde tenemos los proyectos de Kafka 
#Creamos el archivo que generara datos en tiempo real

```bash
cd ~/spark_kafka_project/code
nano kafka_generator.py
```

#Copiamos y pegamos el siguiente codigo

```bash
from kafka import KafkaProducer
import json, time, random

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

# Generar mensajes simulando puntuaciones de películas
while True:
    data = {
        "movie_id": random.randint(1000, 9999),
        "rating": round(random.uniform(1.0, 10.0), 1),
        "timestamp": time.strftime("%Y-%m-%d %H:%M:%S")
    }
    producer.send("movies_topic", data)
    print(f"Enviado: {data}")
    time.sleep(2)

```
<img width="921" height="476" alt="image" src="https://github.com/user-attachments/assets/d21b0b46-0b46-48d2-8fe2-ba79a74678b0" />
#Ctrl + O y enter para guardar y Ctrl + X para salir

#Nos movemos a la carpeta donde tenemos Kafka y creamos el topic

```bash
cd /opt/Kafka/
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --topic movies_topic --partitions 1 --replication-factor 1
```
<img width="921" height="461" alt="image" src="https://github.com/user-attachments/assets/35c05bfc-7e69-4813-967f-921366969c39" />

# Ejecutamos el generador de datos 

#Volvemos a la carpeta de los proyectos y ejecutamos

```bash
cd ~/spark_kafka_project/code
python3 kafka_generator.py
```
<img width="921" height="226" alt="image" src="https://github.com/user-attachments/assets/8226c8d5-14c6-448e-b503-308c9281bda7" />

# Producer
<img width="920" height="472" alt="image" src="https://github.com/user-attachments/assets/434f2e3a-18d4-47f0-ab04-6b194cf88d34" />

Volvemos a la sesión que hemos usado al principio, se podría decir que la sesión principal

#Vamos a la carpeta de los proyectos y creamos el script de Spark Streaming

```bash
cd ~/spark_kafka_project/code
nano spark_streaming.py
```
#Copiamos y pegamos el siguiente codigo

```bash
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, avg
from pyspark.sql.types import StructType, StructField, StringType, DoubleType

# Crear la sesión de Spark
spark = SparkSession.builder.appName("Kafka Spark Streaming").getOrCreate()
spark.sparkContext.setLogLevel("WARN")

# Definir esquema de los datos
schema = StructType([
    StructField("movie_id", StringType()),
    StructField("rating", DoubleType()),
    StructField("timestamp", StringType())
])

# Leer el stream desde Kafka
df_raw = spark.readStream.format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "movies_topic") \
    .load()

# Extraer el valor y convertirlo en JSON
df_values = df_raw.selectExpr("CAST(value AS STRING) as json_value")

df_parsed = df_values.select(from_json(col("json_value"), schema).alias("data")).select("data.*")

# Calcular promedio de rating en tiempo real
avg_rating = df_parsed.groupBy(“movie_id”).agg(avg("rating").alias("promedio_rating"))

# Mostrar resultados en consola
query = avg_rating.writeStream.outputMode("complete").format("console").start()

query.awaitTermination()

```
<img width="921" height="598" alt="image" src="https://github.com/user-attachments/assets/8dd5fd7d-5980-4060-af7e-a0aa84b653de" />
#Ctrl + O y enter para guardar y Ctrl + X para salir

#Por último, ejecutamos el streaming

```bash
spark-submit --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.0 spark_streaming.py
```
<img width="921" height="584" alt="image" src="https://github.com/user-attachments/assets/d0900881-d2fe-4606-8631-34690e97033d" />

# Resultado 
<img width="630" height="1355" alt="image" src="https://github.com/user-attachments/assets/4ed02f8d-05ce-427f-bf26-3d61a7a8dd50" />

Simulamos el flujo de datos con kafka y en Spark Streaming calculamos el raitin de 
cada pelicula en tiempo real.

Trabajo presentado a el tutor: Handry Orozco
Big Data
Grupo: 202016911A_2034

