# Configuring ACLs
ACL (Access Control List) digunakan untuk otorisasi, kita bisa create user dan memodifikasi apa saja yang bisa dilakukan user tersebut dalam kafka, kita bisa restrict apa saja hal yang bisa dilakukan dan tidak bisa dilakukan di cluster, topic, dan group tertentu. Kita bisa menggunakan ACL dengan skenario sebagai berikut : <br>
  <ul>
    <li>user alice hanya bisa produce, tidak bisa consume</li>
    <li>user bob hanya bisa consume, tidak bisa produce</li>
  </ul>

Berikut langkah-langkah yang harus dilakukan :
* Create topic
```bash
kafka-topics --bootstrap-server node2.kafka:9092 --create --topic SASL-ACLS --partitions 3 --replication-factor 2 --command-config client.properties
```
* Lihat list topic yang tersedia
```bash
kafka-topics --bootstrap-server node3.kafka:9092 --list --command-config client.properties
```
Deskripsi dari topic yang dipilih
<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728897781/describetopic_wiwltm.jpg" alt="image" />

file [client.properties](https://github.com/firyal-salsa/confluent-apache-kafka/blob/main/kafka/client.properties) ini berguna untuk menghubungkan klien ke broker Kafka secara aman dari ancaman sniffing (penyadapan lalu lintas), berikut isi file tersebut:

Seperti yang telah disampaikan sebelumnya bahwa user <b>Alice</b> hanya bisa produce dan tidak dapat consume, maka yang harus dilakukan adalah
* Membuat user Alice
```bash
kafka-configs --bootstrap-server node2.kafka:9092 --alter --add-config 'SCRAM-SHA-256=<your_password>' --entity-type users --entity-name alice --command-config client.properties
```
* Buat file [alice.properties](https://github.com/firyal-salsa/confluent-apache-kafka/blob/main/kafka/producer.properties) di /etc/kafka/
* Tambahkan permission WRITE untuk user Alice
```bash
kafka-acls --bootstrap-server node2.kafka:9092 --add --allow-principal User:alice --operation WRITE --topic ACLS --command-config client.properties
```
* Sekarang, kita coba produce message untuk user Alice :
```bash
kafka-console-producer --topic ACLS --bootstrap-server node3.kafka:9092 --producer.config alice.properties
```
<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728899591/aliceproduce_pm1znm.jpg" alt="" />


Untuk user <b>Bob</b> yang hanya bisa consume dan tidak bisa produce, ikuti langkah-langkah berikut :
* Membuat user Bob
```bash
kafka-configs --bootstrap-server node2.kafka:9092 --alter --add-config 'SCRAM-SHA-256=<your_password>' --entity-type users --entity-name bob --command-config client.properties
```
* Tambahkan file [consumer.properties](https://github.com/firyal-salsa/confluent-apache-kafka/blob/main/kafka/consumer.properties) di /etc/kafka/
* Tambahkan hak akses READ untuk user Bob:
```bash
kafka-acls --bootstrap-server node2.kafka:9092 --add --allow-principal Group:test-consumer-group --operation READ --topic ACLS --command-config client.properties
```

Berikut ilustrasi produce-consume:
<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728893315/image_awc87f.png" alt="image" />
