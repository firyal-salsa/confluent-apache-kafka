# Configuring ACLS
Menggunakan ACL untuk authorization, dengan skenario sebagai berikut : <br>
  <ul>
    <li>user alice hanya bisa produce, tidak bisa consume</li>
    <li>user bob hanya bisa consume, tidak bisa produce</li>
  </ul>

Create topic
```bash
kafka-topics --bootstrap-server node2.kafka:9092 --create --topic SASL-ACLS --partitions 3 --replication-factor 2 --command-config ssl-client.properties
```

Lihat list topic yang tersedia
```bash
kafka-topics --bootstrap-server node3.kafka:9092 --list --command-config client.properties
```
Deskripsi dari topic yang dipilih
<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728897781/describetopic_wiwltm.jpg" alt="image" />

file client.properties ini berguna untuk menghubungkan klien ke broker Kafka secara aman dari ancaman sniffing (penyadapan lalu lintas), berikut isi file tersebut:
```bash
bootstrap.servers=node3.kafka:9092

# Security protocol to use (SASL_SSL for encrypted communication with authentication)
security.protocol=SASL_SSL

# SSL settings for secure communication
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent

sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="cp-kafka" password="confluentsatu";
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
```
Seperti yang telah disampaikan sebelumnya bahwa user <b>Alice</b> hanya bisa produce dan tidak dapat consume, berikut langkah-langkah yang dilakukan untuk memenuhi requirement diatas:
Buat user Alice
```bash
kafka-configs --bootstrap-server node2.kafka:9092 --alter --add-config 'SCRAM-SHA-256=<your_password>' --entity-type users --entity-name alice --command-config client.properties
```
Buat file alice.properties di /etc/kafka/
```bash
bootstrap.servers=node3.kafka:9092
key.serializer=org.apache.kafka.common.serialization.StringSerializer
value.serializer=org.apache.kafka.common.serialization.StringSerializer
security.protocol=SASL_SSL
ssl.truststore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.truststore.password=confluent
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="alice" password="alicesatu";
```
Tambahkan permission WRITE untuk user Alice
```bash
kafka-acls --bootstrap-server node2.kafka:9092 --add --allow-principal User:alice --operation WRITE --topic ACLS --command-config client.properties
```
Produce message untuk user Alice :
```bash
kafka-console-producer --topic ACLS --bootstrap-server node3.kafka:9092 --producer.config alice.properties
```
<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728899591/aliceproduce_pm1znm.jpg" alt="" />


Untuk user <b>Bob</b> yang hanya bisa consume dan tidak bisa produce,

<img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1728893315/image_awc87f.png" alt="image" />
