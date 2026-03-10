# Lab 9: Distributed Stream Processing with Apache Spark on Docker

<b>Course:</b> 06026213 Big Data Systems

<b>Term: `02/2025`</b>

Lecturer: `Sirasit Lochanachit`

Lab materials prepared by `Sila (TA) - 2025`

### 🎯 วัตถุประสงค์ (Objectives)

1. เพื่อให้นักศึกษาเข้าใจสถาปัตยกรรมแบบ Distributed System ผ่านการใช้งาน Apache Spark Cluster (1 Master, 3 Workers)
2. เพื่อฝึกปฏิบัติการสร้าง Environment สำหรับ Big Data ด้วย Docker และ Docker Compose
3. เพื่อเรียนรู้การประมวลผลข้อมูลแบบ Streaming (Structured Streaming) และการจัดการ Unstructured Data (JSON Text)
4. เพื่อทดลอง Monitor การทำงานของ Cluster ผ่าน Spark Web UI Dashboard

### 🏗️ System Architecture

เราจะจำลองเครื่องคอมพิวเตอร์ 5 เครื่อง (Containers) ทำงานร่วมกันใน Network เดียวกัน:

1. spark-master: ทำหน้าที่เป็น Resource Manager และ Driver สำหรับ Cluster
2. spark-worker-1: หน่วยประมวลผลที่ 1
3. spark-worker-2: หน่วยประมวลผลที่ 2
4. spark-worker-3: หน่วยประมวลผลที่ 3
5. data-generator: โปรแกรม Python ที่ทำหน้าที่จำลองข้อมูล Social Media Post (Image URL + Caption) และส่งออกมาทาง TCP Socket (Port 9999) อย่างต่อเนื่อง

### 🛠️ Step 1: Prepare Project Directory

เปิด Terminal (Ubuntu/Unix) และสร้างโฟลเดอร์สำหรับโปรเจกต์นี้

```
mkdir bds_spark_lab
cd bds_spark_lab
mkdir apps
```

`bds_spark_lab`: โฟลเดอร์หลัก
`apps`: โฟลเดอร์สำหรับเก็บ Python Script ที่เราจะเขียน (Mapping volume เข้าไปใน Container)

### 🐳 Step 2: Setup Docker Environment

สร้างไฟล์ `docker-compose.yml` เพื่อกำหนดโครงสร้างของ Cluster
(ให้นักศึกษาสร้างไฟล์นี้ในโฟลเดอร์ `bds_spark_lab`)

```
services:
  # --- 1. Spark Master (Driver/Cluster Manager) ---
  spark-master:
    image: bitnami/spark:3.5
    container_name: spark-master
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
    ports:
      - "8080:8080"  # Spark Master Web UI
      - "7077:7077"  # Spark Master Port
    volumes:
      - ./apps:/opt/bitnami/spark/apps # Map โค้ดของเราเข้าไป
    networks:
      - spark-network

  # --- 2. Spark Workers (3 Nodes) ---
  spark-worker-1:
    image: bitnami/spark:3.5
    container_name: spark-worker-1
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - spark-master
    networks:
      - spark-network

  spark-worker-2:
    image: bitnami/spark:3.5
    container_name: spark-worker-2
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - spark-master
    networks:
      - spark-network

  spark-worker-3:
    image: bitnami/spark:3.5
    container_name: spark-worker-3
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark-master:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
    depends_on:
      - spark-master
    networks:
      - spark-network

  # --- 3. Data Generator (Fake Social Media Stream) ---
  data-generator:
    image: python:3.9-slim
    container_name: data-generator
    volumes:
      - ./apps:/app
    working_dir: /app
    command: python data_producer.py
    ports:
      - "9999:9999"
    networks:
      - spark-network

networks:
  spark-network:
    driver: bridge

```

### 📡 Step 3: Create Data Producer (Fake Stream)

สร้างไฟล์ Python ชื่อ `data_producer.py` ในโฟลเดอร์ `apps/`
Script นี้จะทำหน้าที่เป็น TCP Server ที่รอให้ Spark เชื่อมต่อเข้ามา แล้วส่งข้อมูล JSON (ชื่อ, รูป, แคปชั่น) กลับไปเรื่อยๆ ทุกๆ 1 วินาที

<b>File:</b> `apps/data_producer.py`

```
import socket
import time
import json
import random

# ข้อมูลตัวอย่างสำหรับสุ่ม
USERS = ["Alice", "Bob", "Charlie", "Dave", "Eve"]
ACTIONS = ["posted a photo", "shared a memory", "updated status"]
HASHTAGS = ["#travel", "#food", "#coding", "#bigdata", "#spark", "#kmitl", "#life"]

def generate_data():
    """สร้างข้อมูล Fake JSON"""
    data = {
        "user": random.choice(USERS),
        "timestamp": time.time(),
        "image_url": f"[https://fake-img.com/](https://fake-img.com/){random.randint(100, 500)}.jpg",
        "caption": f"Just {random.choice(ACTIONS)}! {random.choice(HASHTAGS)} {random.choice(HASHTAGS)}",
        "likes": random.randint(0, 1000)
    }
    return json.dumps(data)

def start_server(host='0.0.0.0', port=9999):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind((host, port))
    s.listen(1)
    print(f"Listening on port {port}...")

    while True:
        conn, addr = s.accept()
        print(f"Connected by {addr}")
        try:
            while True:
                # สร้างข้อมูลและแปลงเป็น Bytes พร้อม Newline (\n)
                message = generate_data() + "\n"
                conn.sendall(message.encode('utf-8'))
                print(f"Sent: {message.strip()}")
                time.sleep(1) # ส่งข้อมูลทุกๆ 1 วินาที
        except socket.error as e:
            print(f"Client disconnected: {e}")
            conn.close()

if __name__ == "__main__":
    start_server()
```

### ⚡ Step 4: Create Spark Streaming Processor

สร้างไฟล์ Python ชื่อ `spark_processor.py` ในโฟลเดอร์ `apps/`
โค้ดนี้จะอ่านข้อมูลจาก Socket, แปลง JSON, และทำการคำนวณ (Aggregation) แบบ Real-time

<b>File:</b> `apps/spark_processor.py`

```
from pyspark.sql import SparkSession
from pyspark.sql.functions import from_json, col, split, explode, count
from pyspark.sql.types import StructType, StringType, IntegerType, FloatType

# 1. Initialize Spark Session
spark = SparkSession.builder \
    .appName("SocialMediaStreamAnalytics") \
    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")

# 2. Define Schema for JSON Data
# ตรงกับโครงสร้าง JSON ที่ส่งมาจาก data_producer.py
schema = StructType() \
    .add("user", StringType()) \
    .add("timestamp", FloatType()) \
    .add("image_url", StringType()) \
    .add("caption", StringType()) \
    .add("likes", IntegerType())

# 3. Read Stream from Socket (Data Generator Container)
# host: "data-generator" คือชื่อ service ใน docker-compose
raw_stream = spark.readStream \
    .format("socket") \
    .option("host", "data-generator") \
    .option("port", 9999) \
    .load()

# 4. Parse JSON Data
# ข้อมูลจาก Socket จะเข้ามาใน column ชื่อ 'value'
parsed_stream = raw_stream.select(from_json(col("value"), schema).alias("data")).select("data.*")

# ---------------------------------------------------------
# Processing Task: นับจำนวน Hashtag ที่เกิดขึ้นแบบ Real-time
# ---------------------------------------------------------

# แยกคำใน caption ออกมาเป็นคำๆ
words = parsed_stream.select(
    explode(split(col("caption"), " ")).alias("word")
)

# Filter เอาเฉพาะคำที่เป็น Hashtag (#)
hashtags = words.filter(col("word").startswith("#"))

# นับจำนวน Hashtag
hashtag_counts = hashtags.groupBy("word").count().orderBy(col("count").desc())

# 5. Output Sink (Show result to Console)
# Output Mode: Complete (แสดงผลลัพธ์ทั้งหมดใหม่ทุกครั้งที่มีการ update)
query = hashtag_counts.writeStream \
    .outputMode("complete") \
    .format("console") \
    .option("truncate", "false") \
    .start()

print(">>> Streaming Application Started... Waiting for data...")
query.awaitTermination()
```

### 🚀 Step 5: Run the Lab

#### 1. Start Containers

เปิด Terminal ในโฟลเดอร์ `bds_spark_lab` แล้วรันคำสั่ง:

```
docker-compose up -d
```

รอสักครู่ให้ Container ทั้งหมดสถานะเป็น Up (ตรวจสอบด้วย `docker-compose ps`)

#### 2. Verify Spark Dashboard

เปิด Web Browser ไปที่ http://localhost:8080

- คุณควรจะเห็นหน้า Spark Master UI
- ตรวจสอบส่วน Workers: ต้องมี 3 Workers สถานะเป็น ALIVE

#### 3. Submit Spark Job

เราจะสั่งให้ `spark-master` รันโค้ด `spark_processor.py `โดยส่งงานไปให้ Workers ช่วยกันทำ

รันคำสั่งนี้ใน Terminal:

```
docker exec -it spark-master spark-submit \
  --master spark://spark-master:7077 \
  /opt/bitnami/spark/apps/spark_processor.py
```

#### 4. Monitor Output & Dashboard

หลังจาก Submit job:

1. ใน Terminal: คุณจะเห็นตารางข้อมูล Batch update ขึ้นมาเรื่อยๆ ทุกๆ 1-2 วินาที แสดงอันดับ Hashtag ที่ถูกพูดถึงมากที่สุด

   ```
   -------------------------------------------
   Batch: 5
   -------------------------------------------
   +---------+-----+
   |word     |count|
   +---------+-----+
   |#spark   |12   |
   |#bigdata |10   |
   |#kmitl   |8    |
   ...
   ```

2. ใน Browser (Spark UI):

- Refresh หน้า http://localhost:8080
- ดูที่ส่วน Running Applications: จะเห็น App ชื่อ SocialMediaStreamAnalytics กำลังรันอยู่
- คลิกเข้าไปที่ Application ID เพื่อดูรายละเอียด Jobs, Stages และ Executor (Workers) ที่กำลังทำงาน

#### 🧹 Step 6: Clean Up

เมื่อทำ Lab เสร็จแล้ว ให้กด Ctrl+C ที่ Terminal เพื่อหยุดโปรแกรม Spark จากนั้นรันคำสั่งเพื่อปิด Container:

```
docker-compose down
```

### 📝 Assignment (การบ้านท้าย Lab)

1. แก้โค้ด `spark_processor.py` ให้ทำการคำนวณ <b>"ยอด Likes รวม ของแต่ละ User"</b> แทนการนับ Hashtag

2. Capture หน้าจอ Terminal ที่แสดงผลลัพธ์ และหน้าจอ Spark Master UI ที่เห็น Workers 3 ตัวทำงาน
