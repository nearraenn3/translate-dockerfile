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

Under the hood
เมื่อสร้างภาพโดยใช้พื้นที่เก็บข้อมูล Git ระยะไกลเป็นบริบทการสร้างนักเทียบท่าทำการโคลน git ของพื้นที่เก็บข้อมูลบนเครื่องท้องถิ่นและส่งไฟล์เหล่านั้นเป็นบริบทการสร้างไปยัง daemon คุณลักษณะนี้ต้องการให้ติดตั้ง git บนโฮสต์ที่คุณเรียกใช้คำสั่ง docker build
