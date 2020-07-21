# Best practices for writing Dockerfiles -TH

  เอกสารนี้ครอบคลุมแนวทางปฏิบัติที่ดีที่สุดและวิธีการที่แนะนำสำหรับการสร้างภาพที่มีประสิทธิภาพ
 
 Docker สร้างภาพโดยอัตโนมัติโดยการอ่านคำแนะนำจาก Dockerfile - ไฟล์ข้อความที่มีคำสั่งทั้งหมดตามลำดับที่จำเป็นในการสร้างภาพที่กำหนด Dockerfile ยึดตามรูปแบบเฉพาะและชุดคำสั่งที่คุณสามารถหาได้ในการอ้างอิง Dockerfile
  
  อิมเมจ Docker ประกอบด้วยเลเยอร์แบบอ่านอย่างเดียวแต่ละอันแสดงถึงคำสั่ง Dockerfile เลเยอร์จะซ้อนกันและแต่ละชั้นเป็นเดลต้าของการเปลี่ยนแปลงจากเลเยอร์ก่อนหน้า พิจารณา Dockerfile นี้:
```js
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
แต่ละคำสั่งจะสร้างหนึ่งเลเยอร์:

* FROM สร้างเลเยอร์จากอิมเมจ ubuntu: 18.04
* COPY เพิ่มไฟล์จากไดเรกทอรีปัจจุบันของไคลเอ็นต์ Docker
* RUN สร้างแอปพลิเคชันของคุณด้วย make
* CMD ระบุคำสั่งที่จะเรียกใช้ภายในคอนเทนเนอร์

เมื่อคุณเรียกใช้รูปภาพและสร้างคอนเทนเนอร์คุณเพิ่มเลเยอร์ที่เขียนใหม่ได้ (“ เลเยอร์คอนเทนเนอร์”) ที่ด้านบนของเลเยอร์ที่ขีดเส้นใต้ การเปลี่ยนแปลงทั้งหมดที่เกิดขึ้นกับคอนเทนเนอร์ที่กำลังทำงานเช่นการเขียนไฟล์ใหม่การแก้ไขไฟล์ที่มีอยู่และการลบไฟล์จะถูกเขียนไปยังเลเยอร์คอนเทนเนอร์แบบเขียนได้บางนี้

สำหรับข้อมูลเพิ่มเติมเกี่ยวกับเลเยอร์ภาพ (และวิธีที่ Docker สร้างและจัดเก็บรูปภาพ) ให้ดูที่เกี่ยวกับไดรเวอร์อุปกรณ์จัดเก็บข้อมูล
## General guidelines and recommendations
### Create ephemeral containers
รูปภาพที่กำหนดโดย Dockerfile ของคุณควรสร้างคอนเทนเนอร์ที่ไม่สำคัญที่สุดเท่าที่จะทำได้ โดย“ ephemeral” เราหมายถึงคอนเทนเนอร์สามารถหยุดและทำลายจากนั้นสร้างใหม่และแทนที่ด้วยการตั้งค่าและการกำหนดค่าขั้นต่ำที่แน่นอน

อ้างถึงกระบวนการภายใต้วิธีการของแอพสิบสองปัจจัยเพื่อให้เกิดความรู้สึกถึงแรงจูงใจในการใช้ภาชนะบรรจุในแบบไร้รัฐ
### Understand build context
เมื่อคุณใช้คำสั่ง docker build ไดเร็กทอรีการทำงานปัจจุบันจะเรียกว่าบริบทการสร้าง โดยค่าเริ่มต้น Dockerfile จะถือว่าอยู่ที่นี่ แต่คุณสามารถระบุตำแหน่งอื่นด้วยแฟล็กไฟล์ (-f) ไม่ว่าตำแหน่งของ Dockerfile นั้นจะอยู่ที่ใดเนื้อหาเนื้อหาซ้ำทั้งหมดของไฟล์และไดเรกทอรีในไดเรกทอรีปัจจุบันจะถูกส่งไปยัง Docker daemon เป็นบริบทการสร้าง

* Build context example
สร้างไดเรกทอรีสำหรับบริบทการสร้างและซีดีลงไป เขียน“ hello” ลงในไฟล์ข้อความชื่อ hello และสร้าง Dockerfile ที่รัน cat ไว้ สร้างภาพจากภายในบริบทการสร้าง (.):
```js
mkdir myproject && cd myproject
echo "hello" > hello
echo -e "FROM busybox\nCOPY /hello /\nRUN cat /hello" > Dockerfile
docker build -t helloapp:v1 .
```

ย้าย Dockerfile และสวัสดีไปยังไดเรกทอรีที่แยกต่างหากและสร้างอิมเมจเวอร์ชันที่สอง (โดยไม่ต้องพึ่งพาแคชจากการสร้างครั้งล่าสุด) ใช้ -f เพื่อชี้ไปที่ Dockerfile และระบุไดเร็กทอรีของบริบทบิลด์:
```js
mkdir -p dockerfiles context
mv Dockerfile dockerfiles && mv hello context
docker build --no-cache -t helloapp:v2 -f dockerfiles/Dockerfile context
```

การรวมไฟล์ที่ไม่จำเป็นสำหรับการสร้างรูปภาพโดยไม่ได้ตั้งใจส่งผลให้เกิดบริบทการสร้างที่ใหญ่ขึ้นและขนาดรูปภาพที่ใหญ่ขึ้น สิ่งนี้สามารถเพิ่มเวลาในการสร้างภาพเวลาที่จะดึงและดันและขนาดของรันไทม์ของคอนเทนเนอร์ หากต้องการดูว่าบริบทการสร้างของคุณมีขนาดใหญ่เพียงใดให้ค้นหาข้อความเช่นนี้เมื่อสร้าง Dockerfile ของคุณ:
```js
Sending build context to Docker daemon  187.8MB
```

### Pipe Dockerfile through stdin
นักเทียบท่ามีความสามารถในการสร้างอิมเมจโดย pipingfile ผ่าน stdin พร้อมบริบทการสร้างโลคัลหรือรีโมต การวาง Piperfile ผ่าน stdin อาจเป็นประโยชน์ในการสร้างครั้งเดียวโดยไม่ต้องเขียน Dockerfile ลงในดิสก์หรือในสถานการณ์ที่สร้าง Dockerfile ขึ้นมาและไม่ควรคงอยู่ต่อไป
ตัวอย่างในส่วนนี้ใช้เอกสารที่นี่เพื่อความสะดวก แต่สามารถใช้วิธีใดก็ได้เพื่อให้ Dockerfile บน stdin สามารถใช้ได้

ตัวอย่างเช่นคำสั่งต่อไปนี้เทียบเท่า:
```js
echo -e 'FROM busybox\nRUN echo "hello world"' | docker build -
```
```js
docker build -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```
คุณสามารถแทนที่ตัวอย่างด้วยวิธีการที่คุณต้องการหรือวิธีที่เหมาะกับกรณีการใช้งานของคุณ
* BUILD AN IMAGE USING A DOCKERFILE FROM STDIN, WITHOUT SENDING BUILD CONTEXT
ใช้ไวยากรณ์นี้เพื่อสร้างภาพโดยใช้ Dockerfile จาก stdin โดยไม่ต้องส่งไฟล์เพิ่มเติมเป็นบริบทการสร้าง ยัติภังค์ (-) รับตำแหน่ง PATH และสั่งให้ Docker อ่านบริบทการสร้าง (ซึ่งมีเฉพาะ Dockerfile) จาก stdin แทนไดเรกทอรี:

```js
docker build [OPTIONS] -
```
ตัวอย่างต่อไปนี้สร้างภาพโดยใช้ Dockerfile ที่ส่งผ่าน stdin ไม่มีไฟล์ถูกส่งเป็น build บริบทให้กับ daemon
```js
docker build -t myimage:latest -<<EOF
FROM busybox
RUN echo "hello world"
EOF
```
การข้ามบริบทการสร้างอาจมีประโยชน์ในสถานการณ์ที่ Dockerfile ของคุณไม่ต้องการให้คัดลอกไฟล์ลงในภาพและปรับปรุงความเร็วในการสร้างเนื่องจากไม่มีไฟล์ใดถูกส่งไปยัง daemon

หากคุณต้องการปรับปรุงความเร็วในการสร้างโดยการแยกไฟล์บางไฟล์ออกจากโครงสร้างบริบทโปรดอ้างถึงการแยกด้วย. docerignore

หมายเหตุ: ความพยายามในการสร้าง Dockerfile ที่ใช้ COPY หรือ ADD จะล้มเหลวหากใช้ไวยากรณ์นี้ ตัวอย่างต่อไปนี้แสดงให้เห็นถึงสิ่งนี้:

```js
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

docker build -t myimage:latest -<<EOF
FROM busybox
COPY somefile.txt .
RUN cat /somefile.txt
EOF

# observe that the build fails
...
Step 2/3 : COPY somefile.txt .
COPY failed: stat /var/lib/docker/tmp/docker-builder249218248/somefile.txt: no such file or directory
```
```js

```
* BUILD FROM A LOCAL BUILD CONTEXT, USING A DOCKERFILE FROM STDIN
ใช้ไวยากรณ์นี้เพื่อสร้างภาพโดยใช้ไฟล์บนระบบไฟล์โลคัลของคุณ แต่ใช้ Dockerfile จาก stdin ไวยากรณ์ใช้ตัวเลือก -f (หรือ --file) เพื่อระบุ Dockerfile ที่จะใช้โดยใช้ยัติภังค์ (-) เป็นชื่อไฟล์เพื่อสั่งให้ Docker อ่าน Dockerfile จาก stdin:
```js
docker build [OPTIONS] -f- PATH
```
ตัวอย่างด้านล่างใช้ไดเรกทอรีปัจจุบัน (.) เป็นบริบทการสร้างและสร้างภาพโดยใช้ Dockerfile ที่ส่งผ่าน stdin โดยใช้เอกสาร
```js
# create a directory to work in
mkdir example
cd example

# create an example file
touch somefile.txt

# build an image using the current directory as context, and a Dockerfile passed through stdin
docker build -t myimage:latest -f- . <<EOF
FROM busybox
COPY somefile.txt .
RUN cat /somefile.txt
EOF
```

* BUILD FROM A REMOTE BUILD CONTEXT, USING A DOCKERFILE FROM STDIN	
<<<<<<< HEAD
ใช้ไวยากรณ์นี้เพื่อสร้างภาพโดยใช้ไฟล์จากที่เก็บ git ระยะไกลโดยใช้ Dockerfile จาก stdin ไวยากรณ์ใช้ตัวเลือก -f (หรือ --file) เพื่อระบุ Dockerfile ที่จะใช้โดยใช้ยัติภังค์ (-) เป็นชื่อไฟล์เพื่อสั่งให้ Docker อ่าน Dockerfile จาก stdin:
```js
docker build [OPTIONS] -f- PATH
```
ไวยากรณ์นี้มีประโยชน์ในสถานการณ์ที่คุณต้องการสร้างภาพจากพื้นที่เก็บข้อมูลที่ไม่มี Dockerfile หรือถ้าคุณต้องการสร้างด้วย Dockerfile แบบกำหนดเองโดยไม่ต้องดูแลพื้นที่เก็บข้อมูลของคุณเอง

ตัวอย่างด้านล่างนี้สร้างรูปภาพโดยใช้ Dockerfile จาก stdin และเพิ่มไฟล์ hello.c จากที่เก็บ Git“ hello-world” บน GitHub
```js
docker build -t myimage:latest -f- https://github.com/docker-library/hello-world.git <<EOF
FROM busybox
COPY hello.c .
EOF
```
** Under the hood
เมื่อสร้างภาพโดยใช้พื้นที่เก็บข้อมูล Git ระยะไกลเป็นบริบทการสร้างนักเทียบท่าทำการโคลน git ของพื้นที่เก็บข้อมูลบนเครื่องท้องถิ่นและส่งไฟล์เหล่านั้นเป็นบริบทการสร้างไปยัง daemon คุณลักษณะนี้ต้องการให้ติดตั้ง git บนโฮสต์ที่คุณเรียกใช้คำสั่ง docker build

=======
### Exclude with .dockerignore //Short
### Use multi-stage builds
### Don’t install unnecessary packages //Short
### Decouple applications
### Minimize the number of layers
### Sort multi-line arguments
* การแบ่งโค้ดให้เป็นหลายบรรทัด
* การทำแบบนี้จะช่วยให้หลีกเลี่ยงการพิมซ้ำ
* ง่ายต่อการที่จะอ่าน แก้ไข และปรับปรุง
* นอกจากนั้นยังจะทำให้คนที่จะกดยอมรับ pull request เข้าใจได้ง่ายและสามารถให้คำแนะนำได้อย่างถูกต้อง
* ให้พื้นที่ว่างก่อนที่จะใส่เครื่องหมาย \ (แบ็กสแลช) ตัวอย่างเช่น
    #### GOOD
    ```bash
    > $docker container run -d --name mongo \
      --network demo-network  \
      -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \
      -e MONGO_INITDB_ROOT_PASSWORD=secret \
      -e MONGO_INITDB_DATABASE=mydb \
      mongo:4.2.8 
    ```
    #### BAD
    ```bash
    > $docker container run -d --name mongo --network demo-network -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=secret -e MONGO_INITDB_DATABASE=mydb mongo:4.2.8 
    ```
### Leverage build cache
* Docker จะทำตามคำสั่งใน **Dockerfile** ในขณะที่สร้าง Image เพื่อดำเนินการตามลำดับที่ระบุ เมื่อตรวจสอบคำสั่งแต่ละครั้ง Docker จะค้นหาภาพที่มีอยู่ในแคช (Caching) ที่สามารถนำมาใช้ซ้ำได้แทนที่จะสร้างภาพใหม่

* หากไม่ต้องการใช้แคช สามารถใช้ **--no-cache = true** บนคำสั่ง **docker build --no-cache = true**  อย่างไรก็ตามถ้าคุณปล่อยให้ Docker ใช้แคชมันเป็นสิ่งสำคัญที่จะต้องเข้าใจเมื่อมันสามารถและไม่สามารถหาภาพที่ตรงกันได้ตามกฎพื้นฐานที่ Docker มีดังต่อไปนี้:

  * เริ่มต้นด้วย **Parent image** ที่มีอยู่ในแคชแล้วหลังจากนั้นจะเปรียบเทียบกับ **Child image** ทั้งหมดที่มีในฐานข้อมูลนั้นเพื่อดูว่าภาพใดภาพหนึ่งถูกสร้างขึ้นโดยใช้คำสั่งเดียวกันไหม ถ้าไม่จะทำให้แคชไม่ทำงาน

  * ในกรณีส่วนใหญ่เพียงการเปรียบเทียบใน Dockerfile กับหนึ่งใน **Child image** ก็เพียงพอ

  * สำหรับคำสั่ง **ADD** และ **COPY** เนื้อหาของไฟล์ใน Image จะถูกตรวจสอบสำหรับแต่ละไฟล์ เวลาที่แก้ไขและเข้าถึงครั้งล่าสุดของไฟล์จะไม่ถูกพิจารณาในการตรวจสอบเหล่านี้ ในระหว่างการค้นหาแคชการตรวจสอบจะเปรียบเทียบกับการตรวจสอบใน Image ที่มีอยู่หรือเคยสร้างไปแล้ว หากมีการเปลี่ยนแปลงอะไรในไฟล์เช่นเนื้อหาและข้อมูลแสดงว่า แคชจะไม่ทำงาน

  * นอกเหนือจากคำสั่ง **ADD** และ **COPY** แล้วการตรวจสอบแคชไม่ได้ดูไฟล์ในคอนเทนเนอร์ ตัวอย่างเช่นเมื่อประมวลผลคำสั่ง **RUN apt-get -y update** ไฟล์ที่อัพเดตในคอนเทนเนอร์จะไม่ถูกตรวจสอบเพื่อพิจารณาว่ามีการเข้าใช้แคชหรือไม่ ในกรณีนั้นเพียงใช้สตริงคำสั่งเพื่อค้นหาข้อมูลที่ตรงกัน

* เมื่อแคชไม่ถูกต้องคำสั่ง Dockerfile ที่ตามมาทั้งหมดจะสร้างรูปภาพใหม่โดยไม่ใช้ **Caching**
![image](https://codefresh.io/wp-content/uploads/2017/03/Docker-cache-benchmark-1024x365.png)
## Dockerfile instructions
* คำแนะนำเหล่านี้ออกแบบมาเพื่อช่วยคุณสร้าง Dockerfile ที่มีประสิทธิภาพและบำรุงรักษาได้ง่าย
### FROM 
* คำสั่ง **FROM** เริ่มต้นการสร้างและการติดตั้ง **Base Image** ดังนั้น Dockerfile ที่ถูกต้องจะต้องเริ่มต้นด้วยคำสั่ง **FROM** การที่จะทำให้ **Image** เป็น **Image** ที่ถูกต้อง ควรที่จะเริ่มต้นโดยการดึง **Image** จาก **Public Repositories**
    * **FROM** สามารถใช้ได้หลายครั้งภายใน **Dockerfile** เดียวเพื่อสร้าง **Image** หรือใช้หนึ่งสเตจ **Build** เพื่อเป็นการอ้างอิงสำหรับอีก **Image** หนึ่ง เพียงแค่จดบันทึก **Image ID** ล่าสุดโดยการคอมมิตก่อนแต่ละคำสั่ง **FROM** อันใหม่ คำสั่ง **FROM** แต่ละครั้งจะล้างสเตจใด ๆ ที่สร้างขึ้นโดยคำสั่งก่อนหน้า
    * คุณสามารถตั้งชื่อให้กับสเตจสร้างใหม่โดยเพิ่ม **AS name** ไปยังคำสั่ง **FROM** โดยชื่อสามารถใช้ในภายหลังคำสั่ง **FROM** และ **COPY --from = <name | index>** เพื่ออ้างอิงรูปภาพที่สร้างขึ้นในสเตจนี้
    * **tag** หรือ **digest** เป็นทางเลือก หากคุณไม่เลือกตัวใดตัวหนึ่ง **Builder** จะใช้แท็ก **latest** เป็นค่าเริ่มต้น **Builder** จะส่งคืนข้อผิดพลาดหากไม่พบค่าแท็ก
  * **--platform** สามารถใช้ เพื่อระบุแพลตฟอร์มของ **Image** ในกรณีที่ **FROM** อ้างอิง **Image** แบบหลายแพลตฟอร์ม ตัวอย่างเช่น **linux/amd64, linux/arm64 หรือ windows/amd64** โดยค่าเริ่มต้นแพลตฟอร์มเป้าหมายของคำขอสร้างจะถูกใช้ ข้อโต้แย้งการสร้างระดับ Global สามารถนำมาใช้ในค่าของการตั้งค่าสเตจนี้ตัวอย่างเช่น automatic platform ARGs ช่วยให้สามารถบังคับให้สเตจสร้างแพลตฟอร์มแบบเนทีฟ (--platform = $ BUILDPLATFORM) และใช้เพื่อรวบรวมคอมไพล์ไปยังแพลตฟอร์มเป้าหมายภายในสเตจ
  * Example
    ```bash
    > FROM [--platform=<platform>] <image> [AS <name>]
    > FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
    > FROM node:12.18.2-stretch
    ```
### LABEL
  * คุณสามารถเพิ่มป้ายชื่อให้กับ Image ของคุณเพื่อช่วยจัดระเบียบ Image ในโปรเจ็ค เพื่อช่วยในการ **Automation** หรือด้วยเหตุผลอื่น สำหรับแต่ละป้ายชื่อให้ใช้คำสั่งขึ้นต้นด้วย **LABEL** และคู่ **key-value** อย่างน้อยหนึ่งคู่ ตัวอย่างต่อไปนี้แสดงรูปแบบที่แตกต่างกัน
    ```bash
    > # Set one or more individual labels
      # การติดตั้งป้ายชื่อโดยใช้คำสั่ง LABEL ทุกครั้งต่อการติดตั้งป้ายชื่อ 1 ครั้ง
      LABEL com.example.version="0.0.1-beta"
      LABEL vendor1="ACME Incorporated"
      LABEL vendor2=ZENITH\ Incorporated
      LABEL com.example.release-date="2015-02-12"
      LABEL com.example.version.is-production=""

    > # Set multiple labels on one line
      # การติดตั้งป้ายชื่อหลายครั้งโดยใช้คำสั่ง LABEL เพียงครั้งเดียว
      LABEL com.example.version="0.0.1-beta" com.example.release-date="2015-02-12"

    > # Set multiple labels at once, using line-continuation characters to break long lines
      # การติดตั้งป้ายชื่อหลายๆ ครั้งภายในคำสั่ง LABEL เดียว แต่ใช้ \ เข้ามาทำให้โค้ดอ่านง่ายขึ้น
      LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
    ```
### RUN
  * คำสั่ง RUN มี 2 รูปแบบ คือ
    ```bash
    > RUN <command> (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)
    > RUN ["executable", "param1", "param2"] (exec form)
    ```
    * คำสั่ง RUN จะดำเนินการคำสั่งใด ๆ ในเลเยอร์ใหม่ด้านบนของ Image ปัจจุบันและยืนยันผล Image และ Image ที่ถูกยืนยันแล้วจะถูกนำไปใช้สำหรับขั้นตอนต่อไปใน Dockerfile
    * คำแนะนำการสร้าง RUN และการสร้างการยอมรับสอดคล้องกับแนวคิดหลักของ Docker ที่การยืนยันนั้นง่ายและสามารถสร้าง Container ได้ทุกที่ในช่วง Image ก่อนๆ เหมือนกับการควบคุมแหล่งที่มา
    * ฟอร์ม exec จะหลีกเลี่ยง shell string munging และคำสั่ง RUN โดยใช้ฐานข้อมูลของ Image ที่ไม่มีเชลล์เฉพาะที่สามารถทำงานได้
    * เชลล์เริ่มต้นสำหรับฟอร์มเชลล์สามารถเปลี่ยนแปลงได้โดยใช้คำสั่ง **SHELL**
    * ในรูปแบบเชลล์คุณสามารถใช้ \ (แบ็กสแลช) เพื่อดำเนินการคำสั่ง RUN เดียวบนบรรทัดถัดไป ตัวอย่างเช่นพิจารณาสองบรรทัดนี้:
      ```bash
      > RUN /bin/bash -c 'source $HOME/.bashrc; \
        echo $HOME'
      > # สามารถเขียนรวมให้เป็นบรรทัดเดียวได้ดังนี้:
        RUN /bin/bash -c 'source $HOME/.bashrc; echo $HOME'
      > # หากต้องการใช้เชลล์อื่นนอกจาก ‘/ bin / sh’ ให้ใช้แบบฟอร์ม exec ที่ส่งผ่านในเชลล์ที่ต้องการ ตัวอย่างเช่น:
        RUN ["/bin/bash", "-c", "echo hello"]
      ```
    > NOTE
      * แบบฟอร์ม exec ถูกวิเคราะห์เป็นอาร์เรย์ JSON ซึ่งหมายความว่าคุณต้องใช้เครื่องหมาย (“) รอบคำที่ไม่ใช่เครื่องหมาย (‘)
### APT-GET
* APT-GET
Use-case ที่น่าจะเกิดขึ้นอย่างแน่นอนสำหรับ RUN คือแอพพลิเคชันของ apt-get. เพราะคือการติดตั้ง packages คำสั่ง  RUN apt-get   มีหลากหลาย gotchas เพื่อที่จะหลีกเลี่ยง RUN apt-get upgrade และ dist-upgrade มี package ที่มีความจำเป็นหลาย package จากparent images ที่ไม่สามารถอัพเกรดข้างในunprivileged container ได้ ถ้า package contained ใน parent image คือการล้าสมัย(เลิกใช้แล้ว) ให้ติดต่อผู้ดูแล  ถ้าคุณรู้จัก

รวมกันเสมอ RUN apt-get update กับ  apt-get install ในคำสั่ง  RUN เหมือนกัน ยกตัวอย่าง:

RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
ใช้คำสั่ง apt-get update เพียงอย่างเดียวใน RUN เพราะปัญหาการแคชและต่อมาคือ apt-get install คำสั่งล้มเหลว ยกตัวอย่างเชื่อ , ไปที่ Dockerfile:
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl
หลังจากสร้าง  image, layers ทั้งหมดใน Docker cache. สมมติว่าคุณแก้ไขในภายหลัง apt-get install โดดยการเพิ่ม  extra package:
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y curl nginx

* USING PIPES
### CMD
### EXPOSE
### ENV
### ADD or COPY
### ENTRYPOINT
การใช้งาน ```ENTRYPOINT``` ที่ดีที่สุดคือการตั้งค่าคำสั่งหลักของimageเพื่อให้สามารถเรียกใช้imageเหมือนเป็นคำสั่งนั้น (จากนั้นใช้ CMD เป็นค่าเริ่มต้น)

เริ่มด้วยตัวอย่างของimageของcommand line tool ```s3cmd```:

```bash
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```

โดยimageสามารถทำงานได้เช่นนี้เพื่อแสดงความช่วยเหลือของคำสั่ง:

```bash
$ docker run s3cmd
```

หรือใช้พารามิเตอร์ที่เหมาะสมเพื่อใช้คำสั่ง:

```bash
$ docker run s3cmd ls s3://mybucket
```

วิธีนี้มีประโยชน์อย่างมากเพราะชื่อของimageจะเพิ่มเป็นสองเท่า ดังที่แสดงในคำสั่งด้านบน
คำสั่ง ```ENTRYPOINT``` ยังสามารถใช้ร่วมกับตัวช่วยสคริปต์ ซึ่งสามารถทำงานได้ในลักษณะเดียวกันกับคำสั่งด้านบน ถึงแม้ว่าช่วงเริ่มอาจจะใช้มากกว่าหนึ่งขั้นตอน
ตัวอย่างเช่น imageอย่างเป็นทางการของ ```Postgres``` ใช้สคริปต์ต่อไปนี้เป็น ```ENTRYPOINT```

```bash
#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```

ตัวช่วยสคริปต์จะถูกคัดลอกลงในคอนเทนเนอร์และถูกเรียกใช้ผ่าน ```ENTRYPOINT``` เมื่อเริ่มต้นคอนเทนเนอร์:

```bash
COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["postgres"]
```


สคริปต์นี้อนุญาตให้ผู้ใช้โต้ตอบกับ ```Postgres``` ได้หลายวิธี
โดยสามารถเริ่ม ```Postgres``` จาก :

```bash
$ docker run postgres
```

หรือสามารถใช้เพื่อรัน ```Postgres``` และส่งต่อพารามิเตอร์ไปยังเซิร์ฟเวอร์:

```bash
$ docker run postgres postgres --help
```


และสุดท้ายก็สามารถใช้เพื่อเริ่มเครื่องมือที่แตกต่างอย่างสิ้นเชิงเช่น Bash:

```bash
$ docker run --rm -it postgres bash
```

### VOLUME  //Short
การใช้คำสั่ง ```bashVOLUME``` จะใช้เพื่อแสดงพื้นที่ของฐานข้อมูล,การจัดเก็บการกำหนดค่า,ไฟล์ หรือโฟลเดอร์ที่สร้างโดยdocker container โดยควรใช้```VOLUME```สำหรับส่วนของ```image```ที่เปลี่ยนแปลงง่าย
และ/หรือส่วนที่userไม่สามารถแก้ไขได้
### USER
ถ้า service นั้นสามารถใช้ได้แบบไม่มีสิทธิพิเศษ(privilage) จะสามารถใช้ ```USER``` เพื่อเปลี่ยนเป็นผู้ใช้แบบ non-root โดยเริ่มต้นด้วยการสร้าง```USER```และจัดกลุ่ม Dockerfile อย่างเช่น

```bash
RUN groupadd -r postgres && useradd --no-log-init -r -g postgres postgres.
```
หลีกเลี่ยงการติดตั้งหรือการใช้ sudo มีลักษณะการทำงานของ TTY ที่คาดเดาไม่ได้และลักษณะการส่งต่อซึ่งอาจทำให้เกิดปัญหาได้ หากคุณต้องการฟังก์ชั่นที่คล้ายกับ sudo เช่นการเริ่มต้น ```daemon``` เป็นรูท แต่รันเป็น non-root ให้ลองใช้ “gosu”
สุดท้ายเพื่อลดความซับซ้อน ให้หลีกเลี่ยงการสลับ USER ไปมาบ่อยๆ
### WORKDIR //Short
เพื่อความชัดเจนและความน่าเชื่อถือ ควรใช้เส้นทางที่แน่นอนสำหรับ WORKDIR  นอกจากนี้ควรใช้ WORKDIR แทนคำแนะนำที่เพิ่มขึ้นเช่น  ``` RUN cd ... &&- ```ซึ่งยากต่อการอ่าน,แก้ไขและบำรุงรักษา
### ONBUILD 
คำสั่ง ```ONBUILD``` จะดำเนินการหลังจากการสร้าง ```Dockerfile``` ปัจจุบันเสร็จ ```ONBUILD``` จะดำเนินการในchild imageใด ๆ ที่ได้รับจาก image ปัจจุบัน โดย Docker Build จะเรียกใช้งานคำสั่ง ```ONBUILD``` ก่อนคำสั่งใด ๆ ใน child  ```ONBUILD``` มีประโยชน์สำหรับ```image```ที่กำลังจะสร้างจาก```image```ที่กำหนด ตัวอย่างเช่นคุณจะใช้```bash ONBUILD``` สำหรับimageแบบstackภาษาที่สร้างซอฟแวร์ผู้ใช้ที่เขียนด้วยภาษานั้นภายใน Dockerfile ดังที่คุณเห็นในRuby’s ONBUILD รุ่นต่างๆ 

รูปภาพที่สร้างด้วย``` ONBUILD ```ควรได้รับTagแยกต่างหากตัวอย่างเช่น:``` ruby: 1.9-onbuild ```หรือ ```ruby: 2.0-onbuild```
ควรระวังเมื่อ ```ADD ```หรือ ```COPY``` ลงใน ```ONBUILD ```image“ onbuild” ล้มเหลว หากบริบทของบิลด์ใหม่ไม่มีทรัพยากรที่เพิ่มเข้ามาการเพิ่มแท็กแยกตามที่แนะนำไว้ข้างต้นจะช่วยลดสิ่งนี้ได้โดยอนุญาตให้ผู้เขียน ```Dockerfile``` 
>>>>>>> ef246e8c8d634fa01b74123b77e41bd80ae8956c
