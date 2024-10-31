# Confluent Apache Kafka 
Dari proses instalasi, memasang SSL, SASL, sampai mengakses suatu topic dengan ACLs


## Install

Menggunakan Ubuntu 
```bash
sudo nano /etc/apt/sources.list.d/archive_uri-https_packages_confluent_io_clients_deb-noble.list
```
ubah noble menjadi focal
```bash
sudo apt-get update && \
sudo apt-get install confluent-platform && \
sudo apt-get install confluent-security
```


Menggunakan RHEL / CentOs / Rocky / Amazon Linux
```bash
sudo yum clean all && \
sudo yum install confluent-platform && \
sudo yum install confluent-security
```


## Memasang SSL 

Pastikan di file /etc/host/ sudah terdefinisi setiap node beserta toolsnya : <br>
<nama_ip> node1.zookeeper node1.kafka node1.schema node1.connect node1.ksql node1.rest <br>
<nama_ip> node2.zookeeper node2.kafka node2.schema node2.connect node2.ksql node2.rest node2.cc <br>
<nama_ip> node3.zookeeper node3.kafka node3.schema node3.connect node3.ksql node3.rest <br>

```bash
sudo openssl req -new -x509 -keyout ca-root.key -out ca-root.crt -days 365 -subj '/CN=server/OU=FS/O=ADI/L=JAKBA
R/ST=DKI/C=ID' -passin pass:confluent -passout pass:confluent
```
```bash

```
```bash
sudo keytool -genkey -noprompt -alias server -dname "CN=server,OU=FS,O=ADI,L=JAKBAR,S=DKI,C=ID" -ext "SAN=dns:no
de1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.schema,dns:node1.connect,dns:node1.
ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.connect,dns:node2.ksql,dns:node2.res
t,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,dns:node3.ksql,dns:node3.rest" -k
eystore kafka.server.keystore.jks -keyalg RSA -storepass confluent -keypass confluent -storetype pkcs12
```
```bash
sudo keytool -keystore kafka.server.keystore.jks -alias server -certreq -file server.csr -storepass confluent -k
eypass confluent -ext "SAN=dns:node1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.sc
hema,dns:node1.connect,dns:node1.ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.con
nect,dns:node2.ksql,dns:node2.rest,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,
dns:node3.ksql,dns:node3.rest"
```
Buat file server.cnf 
```bash
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no
[req_distinguished_name]
CN = server
[v3_req]
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = node1.zookeeper
DNS.2 = node1.kafka
DNS.3 = node1.schema
DNS.4 = node1.connect
DNS.5 = node1.ksql
DNS.6 = node1.rest
DNS.7 = node2.zookeeper
DNS.8 = node2.kafka
DNS.9 = node2.schema
DNS.10 = node2.connect
DNS.11 = node2.ksql
DNS.12 = node2.rest
DNS.13 = node2.cc
DNS.14 = node3.zookeeper
DNS.15 = node3.kafka
DNS.16 = node3.schema
DNS.17 = node3.connect
DNS.18 = node3.ksql
DNS.19 = node3.rest
```
```bash
sudo openssl x509 -req -CA ca-root.crt -CAkey ca-root.key -in server.csr -out server-signed.crt -sha256 -days 365 -CAcreateserial -passin pass:confluent -extensions v3_req -extfile server.cnf
```
```bash
sudo keytool -noprompt -keystore server.keystore.jks -alias caroot -import -file ca-root.crt -storepass confluent -keypass confluent
```
Menggabungkan kedua file tersebut ke full-chain.xrt
```bash
sudo sh -c 'cat server-signed.crt ca-root.crt > full-chain.crt'
```
```bash
sudo keytool -noprompt -keystore kafka.server.keystore.jks -alias server -import -file full-chain.crt -storepass confluent -keypass confluent
```
```bash
 sudo keytool -noprompt -keystore kafka.server.truststore.jks -alias caroot -import -file ca-root.crt -storepass 
confluent -keypass confluent
```

```bash
scp -r confluent@<alamat_ip>:/var/ssl/private /var/ssl/private
```

### Zookeeper
```bash
sudo nano /etc/kafka/zookeeper.properties
```

Edit file /etc/kafka/zookeeper.properties
```bash
4lw.commands.whitelist=*
admin.enableServer=false
authProvider.x509=org.apache.zookeeper.server.auth.X509AuthenticationProvider
autopurge.purgeInterval=1
autopurge.snapRetainCount=10
dataDir=/data1/zookeeper/
initLimit=5
maxClientCnxns=0
secureClientPort=2182
server.1=node1.zookeeper:2888:3888
server.2=node2.zookeeper:2888:3888
server.3=node3.zookeeper:2888:3888
serverCnxnFactory=org.apache.zookeeper.server.NettyServerCnxnFactory
ssl.clientAuth=none
ssl.keyStore.location=/var/ssl/private/kafka.server.keystore.jks
ssl.keyStore.password=confluent
ssl.trustStore.location=/var/ssl/private/kafka.server.truststore.jks
ssl.trustStore.password=confluent
syncLimit=2
```

Buat file /data1/zookeeper dengan akses sebagai berikut
```bash
sudo chown -R cp-kafka:confluent /data1/kafka
```

Tambahkan file berikut yang berisi nomor dari node sekarang
```bash
touch /data/zookeeper/myid
```

```bash
systemctl daemon-reload
```
```bash
systemctl start confluent-zookeeper.service
```

### Broker

Edit [/etc/kafka/server.properties](https://github.com/firyal-salsa/confluent-apache-kafka/blob/main/kafka/server.properties)

```bash
sudo chown -R cp-kafka:confluent /data1/kafka
```

```bash
sudo chmod -R 755 /data1/kafka
```

```bash
sudo systemctl start confluent-server.service
```
