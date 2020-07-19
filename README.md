 TOPIC : Best practices for writing Dockerfiles -TH
## General guidelines and recommendations
### Create ephemeral containers
### Understand build context
### Pipe Dockerfile through stdin
* BUILD AN IMAGE USING A DOCKERFILE FROM STDIN, WITHOUT SENDING BUILD CONTEXT
* BUILD FROM A LOCAL BUILD CONTEXT, USING A DOCKERFILE FROM STDIN
* BUILD FROM A REMOTE BUILD CONTEXT, USING A DOCKERFILE FROM STDIN	
### Exclude with .dockerignore //Short
### Use multi-stage builds
### Don’t install unnecessary packages //Short
### Decouple applications
### Minimize the number of layers
### Sort multi-line arguments //Short
### Leverage build cache
# Dockerfile instructions //Short
### FROM //Short
### LABEL
### RUN  //Short
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
### VOLUME  //Short
### USER
### WORKDIR //Short
### ONBUILD 