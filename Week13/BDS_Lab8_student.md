# Lab 8: Introduction to Ubuntu Linux and Docker

<b>Course:</b> 06026213 Big Data Systems

<b>Term: `02/2025`</b>

Lecturer: `Sirasit Lochanachit`

Lab materials prepared by `Sila (TA) - 2025`

### 🎯 วัตถุประสงค์ (Objectives)

1. เข้าใจและสามารถใช้งานคำสั่ง Linux พื้นฐานในการจัดการไฟล์ได้
2. สามารถติดตั้ง Docker Engine และ Docker Compose บน Ubuntu Linux ได้อย่างถูกต้อง
3. เข้าใจหลักการทำงานพื้นฐานของ Docker (Pull, Run, Stop, Remove)

### 🏗️ System Architecture

เป้าหมายหลักคือเราจะจำลองเครื่องคอมพิวเตอร์ 5 เครื่อง (Containers) ทำงานร่วมกันใน Network เดียวกัน:

1. spark-master: ทำหน้าที่เป็น Resource Manager และ Driver สำหรับ Cluster
2. spark-worker-1: หน่วยประมวลผลที่ 1
3. spark-worker-2: หน่วยประมวลผลที่ 2
4. spark-worker-3: หน่วยประมวลผลที่ 3
5. data-generator: โปรแกรม Python ที่ทำหน้าที่จำลองข้อมูล Social Media Post (Image URL + Caption) และส่งออกมาทาง TCP Socket (Port 9999) อย่างต่อเนื่อง

แต่ในสัปดาห์นี้เราจะมาดูพื้นฐานก่อนที่จะเริ่มทำการจำลองกัน คือการติดตั้ง Linux Server เพื่อ host container

### 📚 Part 1: Ubuntu Linux Server Installation

1. ดาวน์โหลดและติดตั้ง Virual Box https://www.virtualbox.org/wiki/Downloads
2. ทำการติดตั้ง และเปิดโปรแกรม Virual Box
3. ดาวน์โหลด Ubuntu Server Images สำหรับใช้ในการติดตั้งใน Step ต่อไป https://ubuntu.com/download/server เลือกดาวน์โหลดเวอร์ชัน `Ubuntu 24.04.3 LTS`

<> สำหรับในห้อง XXX ไฟล์ Ubuntu Server Images จะอยู่ในเครื่องคอมพิวเตอร์ทุกเครื่องแล้วที่ :

4. เปิดโปรแกรม Virual Box จากนั้นกดที่ `New`

![alt text](images/image.png)

5. จากนั้นในหน้าต่าง New Virtual Machine ให้ตั้งค่าในส่วนของ `VM Name` / `ISO Image` และ เอาตัวเลือกในช่อง `Process with Unattended Installation` ออก จากนั้นกด `Next`

![alt text](images/image-1.png)

6. ตั้งค่า `Base Memory` / `CPUs` และ `Disk Size` ตามภาพ จากนั้นกด `Next`

![alt text](images/image-2.png)

7. ตรวจสอบ และกด `Finish`

![alt text](images/image-3.png)

8. หลังจากสร้าง Virtual Machine เสร็จ กด `Start` เพื่อเริ่มการทำงาน

![alt text](images/image-4.png)

9. หลังจากที่ VM เริ่มการทำงาน ให้เลือก `Try or Install Ubuntu Server` จากนั้นกด `Enter`

![alt text](images/image-5.png)

10. เลือกภาษาที่ต้องการ ในที่นี้แนะนำให้เลือกภาษา English จากนั้นกด `Enter`

![alt text](images/image-7.png)

11. เลือก `Done` และกด `Enter`

![alt text](images/image-6.png)

12. เลือก `Done` และกด `Enter`

![alt text](images/image-8.png)

13. เลือก `Done` และกด `Enter`

![alt text](images/image-9.png)

14. เลือก `Done` และกด `Enter`

![alt text](images/image-10.png)

15. เลือก `Done` และกด `Enter`

![alt text](images/image-11.png)

16. เลือก `Done` และกด `Enter`

![alt text](images/image-12.png)

17. เลือก `Continue` และกด `Enter`

![alt text](images/image-13.png)

18. ทำการตั้ง Your Name / Server's Name / Username และ Password (จำ Username และ Password ให้ดี ! เพราะต้องใช้ในขั้นตอนถัด ๆ ไปและใช้สำหรับ Login)

![alt text](images/image-14.png)

19. เลือก `Skip for now` จากนั้นเลือก `Continue` และกด `Enter`

![alt text](images/image-15.png)

20. เลือก `Done` และกด `Enter`

![alt text](images/image-16.png)

21. เลือก `Done` และกด `Enter`

![alt text](images/image-17.png)

22. รอจนกว่าการติดตั้งจะเสร็จสมบูรณ์ หรือขึ้นให้กด `Reboot Now`

![alt text](images/image-18.png)

23. จากนั้นเมื่อระบบปฎิบัติการเปิดขึ้นมาใหม่ ให้ทำการ Login โดยใช้ Username และ Password ตามที่ได้ตั้งค่าไปในข้อ 18.

![alt text](images/image-19.png)

---

### 📚 Part 2: Basic Linux Commands Playground

ก่อนจะเริ่มการติดตั้ง Docker ให้นักศึกษาลองฝึกใช้งานคำสั่ง Linux พื้นฐานตามลำดับขั้นตอน (Step-by-Step) ต่อไปนี้ เพื่อสร้างความคุ้นเคยกับการจัดการไฟล์และ Directory บน Server

#### Step 1: ตรวจสอบตำแหน่งปัจจุบันและสร้าง Directory

1. เช็คว่าปัจจุบันอยู่ที่ Path ไหน (Print Working Directory)

```
pwd
```

2. สร้าง Directory ใหม่ชื่อ 'Week10' (Make Directory)

```
mkdir Week10
```

3. ลอง List ดูว่าโฟลเดอร์ถูกสร้างขึ้นมาจริงไหม

```
ls -l
```

#### Step 2: เข้าไปทำงานใน Directory

4. ย้ายเข้าไปในโฟลเดอร์ Week10 (Change Directory)

```
cd Week10
```

5. เช็คอีกรอบเพื่อให้แน่ใจว่าเข้ามาแล้วจริงๆ (ต้องเห็น path จบด้วย /Week10)

```
pwd
```

#### Step 2: เข้าไปทำงานใน Directory

6. สร้างไฟล์เปล่าๆ ชื่อ bds.txt (Touch)

```
touch bds.txt
```

7. ลองอ่านไฟล์ดู (จะยังไม่เห็นข้อความอะไร เพราะเป็นไฟล์เปล่า)

```
cat bds.txt
```

8. ใส่ข้อความ "Hello Big Data" ลงไปในไฟล์ (ใช้ echo และเครื่องหมาย > เพื่อเขียนทับ)

```
echo "Hello Big Data Systems" > bds.txt
```

9. อ่านไฟล์อีกครั้ง (คราวนี้ต้องเห็นข้อความที่เราพิมพ์ไป)

```
cat bds.txt
```

#### Step 3: เข้าไปทำงานใน Directory

10. คัดลอกไฟล์ bds.txt ไปเป็นไฟล์ใหม่ชื่อ bds_backup.txt

```
cp bds.txt bds_backup.txt
```

11. เช็คดูว่ามีไฟล์เพิ่มมาไหม

```
ls -l
```

12. เปลี่ยนชื่อไฟล์ (Rename) จาก bds.txt เป็น readme.txt

```
mv bds.txt readme.txt
```

13. ลบไฟล์ backup ทิ้ง (Remove)

```
rm bds_backup.txt
```

14. เช็คไฟล์ที่เหลืออยู่เป็นครั้งสุดท้าย

```
ls -l
```
---

### 📚 Part 3: Docker Installation

1. Update package information

```
sudo apt-get update -y
```

เป็นการสั่งให้ระบบปฏิบัติการ Ubuntu ทำการ "เช็คชื่อ" หรืออัปเดตรายการซอฟต์แวร์ (Package Lists) ล่าสุดจาก Server หลัก เพื่อให้แน่ใจว่าเครื่องของเราเห็นรายการซอฟต์แวร์เวอร์ชันปัจจุบันก่อนที่จะเริ่มติดตั้งอะไรใหม่ๆ

2. Install prerequisites

```
sudo apt-get install -y ca-certificates curl gnupg
```

ติดตั้งโปรแกรมพื้นฐานที่จำเป็นสำหรับการดาวน์โหลดและยืนยันความปลอดภัยของไฟล์ติดตั้ง Docker

- `ca-certificates`: ใบรับรองความปลอดภัยสำหรับการเชื่อมต่อเว็บ (HTTPS)
- `curl`: เครื่องมือสำหรับดาวน์โหลดไฟล์ผ่าน Command Line
- `gnupg`: เครื่องมือสำหรับการเข้ารหัสและตรวจสอบลายเซ็นดิจิทัล (GPG Key)

3. Docker GPG Key

```
sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

ขั้นตอนนี้คือการเพิ่ม "กุญแจดิจิทัล" (GPG Key) ของ Docker เข้าไปในเครื่องเรา เพื่อเป็นการยืนยันตัวตนว่าซอฟต์แวร์ที่เรากำลังจะดาวน์โหลดนั้นมาจาก Docker Official จริงๆ และไม่ได้ถูกปลอมแปลงหรือดัดแปลงระหว่างทาง (เป็นเรื่องของ Security)

4. Add the Docker repository to apt sources

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

เป็นการบอก Ubuntu ว่า "ให้ไปดาวน์โหลด Docker จากที่ไหน"
โดยปกติ Ubuntu จะมี Docker ให้โหลดอยู่แล้วแต่เป็นเวอร์ชันเก่า คำสั่งนี้จะเป็นการเพิ่มที่อยู่ (Repository) ของ Docker โดยตรงลงไปในรายการแหล่งดาวน์โหลดของเครื่อง เพื่อให้เราได้ใช้ Docker เวอร์ชันล่าสุดเสมอ

5. Install Docker packages

```
sudo apt-get update -y

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

`apt-get update`: สั่งให้อัปเดตรายการซอฟต์แวร์อีกครั้ง (เพื่อให้เครื่องเห็น Docker Repo ที่เราเพิ่งเพิ่มไปในข้อ 4)

install ...: ติดตั้งองค์ประกอบหลักของ Docker ได้แก่:

- `docker-ce`: Docker Engine (ตัวทำงานหลัก)
- `docker-ce-cli`: เครื่องมือพิมพ์คำสั่ง Docker
- `containerd.io`: ตัวจัดการ Container Runtime
- `docker-compose-plugin`: ปลั๊กอินสำหรับจัดการหลาย Container พร้อมกัน

6. Enable and start the Docker service

```
sudo systemctl enable docker
sudo systemctl start docker
```

`enable`: สั่งให้ Docker เปิดตัวเองขึ้นมาทำงานอัตโนมัติทุกครั้งที่เราเปิดเครื่องคอมพิวเตอร์ (Boot)
`start`: สั่งให้ Docker เริ่มทำงาน "เดี๋ยวนี้" เลย

7. Docker group and permission

```
sudo usermod -aG docker $USER
sudo groupadd docker

sudo chmod 666 /var/run/docker.sock
```

เป็นการจัดการสิทธิ์การใช้งาน (Permission)

ปกติแล้วคำสั่ง Docker ต้องใช้สิทธิ์ผู้ดูแลระบบ (sudo) นำหน้าเสมอ ซึ่งไม่สะดวก

`usermod -aG docker $USER`: เพิ่ม User ของเราเข้าไปในกลุ่ม 'docker' เพื่อให้รันคำสั่งได้โดยไม่ต้องพิมพ์ sudo
`chmod 666 ...`: (ใน Lab นี้) เป็นการเปิดสิทธิ์ไฟล์ Socket ของ Docker ให้ทุกคนเขียน/อ่านได้ เพื่อแก้ปัญหา Permission Denied เบื้องต้นแบบเร่งด่วน

8. Install the Compose plugin

```
sudo apt-get install -y docker-compose-plugin
```

9. Print Docker and Docker Compose versions

```
docker --version
docker compose version
```
---

### 📚 Part 4: Docker Hello World

ในส่วนนี้ให้นักศึกษาทดลองใช้คำสั่งพื้นฐานของ Docker เพื่อทำความเข้าใจวงจรชีวิต (Lifecycle) ของ Container ตั้งแต่การดึง Image, การรัน, การหยุด, และการลบ

1. Docker Pull (ดาวน์โหลด Image) : คำสั่ง `pull` ใช้สำหรับดาวน์โหลด Image จาก Docker Hub มาเก็บไว้ที่เครื่องของเรา

```
docker pull hello-world
```

2. Docker Run (สร้างและรัน Container) : คำสั่ง `run` จะนำ Image มาสร้างเป็น Container และเริ่มทำงานทันที สำหรับ `hello-world` เมื่อทำงานเสร็จ (พิมพ์ข้อความ) มันจะปิดตัวเอง (Exit) ทันที

```
docker run hello-world
```

3. Docker PS (ตรวจสอบสถานะ Container) : คำสั่ง `ps` (Process Status) ใช้ดู Container ที่มีอยู่ในระบบ

- ตรวจสอบ Container ที่ กำลังทำงานอยู่ (จะไม่เห็น hello-world เพราะมันจบการทำงานแล้ว)

  ```
  docker ps
  ```

- ตรวจสอบ Container ทั้งหมด (รวมถึงที่จบการทำงานไปแล้ว)

  ```
  docker ps -a
  ```

4. Run Detached Container (รันแบบเบื้องหลัง) : เพื่อให้ทดลองคำสั่ง Stop ได้ เราจะลองรัน `nginx` (Web Server) ซึ่งเป็นโปรแกรมที่รันค้างไว้ได้ โดยใช้ option `-d` (Detached mode)

```
docker run -d --name my-webserver nginx

docker ps
```

(หมายเหตุ: หากเครื่องยังไม่มี Image nginx ระบบจะทำการ pull ให้เองอัตโนมัติ)

5. Docker Stop (หยุด Container)

```
docker stop my-webserver
```

6. Docker Remove (ลบ Container)

```
docker rm my-webserver
docker container prune -f
```

7. Clear Images (ลบ Image ต้นฉบับ)

```
docker images
docker rmi hello-world nginx
```
---