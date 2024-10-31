# Instalasi Services
Berikut panduan instalasi services seperti zookeeper, broker, kafka connect distributed, schema registry, rest, dan ksql serta pemasangan SASL SSL untuk setiap services.

## Install Confluent Apache Kafka
Dengan mengggunakan confluent, Administrator dapat menginstall semua services tanpa perlu repot-repot menginstall satu persatu. Berikut instalasi confluent menggunakan Ubuntu dan Rocky :
### Instalasi Confluent di Ubuntu
Saat ini confluent belum support ubuntu versi 24 atau noble, versi yang compatible adalah sebagai berikut : <br>

<div align="center">
 <img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1730379972/osrequirement_t4texo.jpg" align="center" width="60%" /> <br>
Source : <a href="https://docs.confluent.io/platform/current/installation/system-requirements.html#:~:text=Confluent%20Platform%20is%20supported%20on,Linux%20and%20Ubuntu%20Operating%20Systems">Confluent</a>
</div>
<br>
Jika hanya memiliki ubuntu versi terbaru, maka ubah file berikut <br>

```bash
sudo nano /etc/apt/sources.list.d/archive_uri-https_packages_confluent_io_clients_deb-noble.list
```

Ubah kata 'noble' menjadi 'focal', mengubah file ini bukan berarti downgrade sistem secara keseluruhan, hanya  mengubah repositori sumber paket untuk Confluent.
<br>
Sekarang kita akan menginstall confluent menggunakan systemd di ubuntu, berikut langkah-langkahnya:
1. Instal public key Confluent. Kunci ini digunakan untuk menandatangani paket di repositori APT.
```bash
wget -qO - https://packages.confluent.io/deb/7.7/archive.key | sudo apt-key add -
```
2. Tambahkan repositori ke /etc/apt/sources.list dengan menjalankan perintah ini:
```bash
sudo add-apt-repository "deb [arch=amd64] https://packages.confluent.io/deb/7.7 stable main"
sudo add-apt-repository "deb https://packages.confluent.io/clients/deb $(lsb_release -cs) main"
```
3. Update apt-get dan install services confluent dengan RBAC
```bash
sudo apt-get update && \
sudo apt-get install confluent-platform && \
sudo apt-get install confluent-security
```
### Instalasi Confluent di Rocky 8
Berikut panduan install confluent menggunakan systemd di Rocky
1. Install curl dan which
   ```bash
   sudo yum install curl which
   ```
3. Install confluent platform menggunakan public key
   ```bash
   sudo rpm --import https://packages.confluent.io/rpm/7.7/archive.key
   ```
5. Di path /etc/yum.repos.d/ tambahkan file confluent.repo yang berisi
   ```bash
   [Confluent]
   name=Confluent repository
   baseurl=https://packages.confluent.io/rpm/7.7
   gpgcheck=1
   gpgkey=https://packages.confluent.io/rpm/7.7/archive.key
   enabled=1
   
   [Confluent-Clients]
   name=Confluent Clients repository
   baseurl=https://packages.confluent.io/clients/rpm/centos/$releasever/$basearch
   gpgcheck=1
   gpgkey=https://packages.confluent.io/clients/rpm/archive.key
   enabled=1
   ```
7. Bersihkan caches YUM dan install confluent platform dengan RBAC
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
