# Sort multi-line arguments
* การแบ่งโค้ดให้เป็นหลายบรรทัด
* การทำแบบนี้จะช่วยให้หลีกเลี่ยงการพิมซ้ำ
* ง่ายต่อการที่จะอ่าน แก้ไข และปรับปรุง
* นอกจากนั้นยังจะทำให้คนที่จะกดยอมรับ pull request เข้าใจได้ง่ายและสามารถให้คำแนะนำได้อย่างถูกต้อง
* ให้พื้นที่ว่างก่อนที่จะใส่เครื่องหมาย \ ตัวอย่างเช่น
    #### GOOD
    * $docker container run -d --name mongo \\\
      --network demo-network  \\\
      -e MONGO_INITDB_ROOT_USERNAME=mongoadmin \\\
      -e MONGO_INITDB_ROOT_PASSWORD=secret \\\
      -e MONGO_INITDB_DATABASE=mydb \\\
      mongo:4.2.8 

    #### BAD
    * $docker container run -d --name mongo --network demo-network -e MONGO_INITDB_ROOT_USERNAME=mongoadmin -e MONGO_INITDB_ROOT_PASSWORD=secret -e MONGO_INITDB_DATABASE=mydb mongo:4.2.8 

# Leverage build cache

# Dockerfile instructions

# FROM

# LABEL

# RUN

