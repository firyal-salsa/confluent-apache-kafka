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

### Memastikan services berjalan
Untuk mengetahui list services yang tersedia, gunakan perintah berikut
```bash
systemctl list-units --type=service --all
```
Untuk mengetahui deskripsi suatu services, gunakan perintah berikut
```bash
systemctl status confluent-zookeeper.service
```
status bisa diganti dengan start, restart, dan stop sesuai dengan kebutuhan

Jika sudah di start, cek portnya dengan perintah berikut
```bash
netstat -tunlp
```
Secara default, berikut masing-masing port yang ada di confluent:

| Service                  | Default Port | Description                                         |
|--------------------------|--------------|-----------------------------------------------------|
| Kafka Broker             | 9092         | Port untuk berkomunikasi dengan Kafka clients.      |
| Zookeeper                | 2181         | Port default untuk komunikasi Zookeeper.            |
| Kafka REST Proxy         | 8082         | API REST untuk berkomunikasi dengan Kafka.          |
| Schema Registry          | 8081         | Menyimpan dan menyediakan akses ke schema.          |
| Confluent Control Center | 9021         | UI untuk mengelola dan memonitor Kafka cluster.     |
| Kafka Connect            | 8083         | Digunakan untuk integrasi data dari/ke Kafka.       |
| ksqlDB Server            | 8088         | Layanan ksqlDB untuk pemrosesan data streaming.     |


## Memasang SSL 
Pada contoh kasus kali ini, terdapat tiga instances / virtual machines dalam satu cluster. Dalam memasang SSL, langkah awal yang harus dipersiapkan adalah memuat key store dan trust store untuk server dan juga client menggunakan self-signed CA. 
Pastikan di file /etc/hosts sudah terdefinisi setiap node beserta toolsnya : <br>
<nama_ip> node1.zookeeper node1.kafka node1.schema node1.connect node1.ksql node1.rest <br>
<nama_ip> node2.zookeeper node2.kafka node2.schema node2.connect node2.ksql node2.rest node2.cc <br>
<nama_ip> node3.zookeeper node3.kafka node3.schema node3.connect node3.ksql node3.rest <br>

1. Buat setifikat root CA
   ```bash
   sudo openssl req -new -x509 -keyout ca-root.key -out ca-root.crt -days 365 -subj '/CN=server/OU=FS/O=ADI/L=JAKBA
   R/ST=DKI/C=ID' -passin pass:confluent -passout pass:confluent
   ```
2. Buat sertifikat key store untuk brokers dengan sertifikat yang ditandatangani oleh CA. Jika menggunakan wildcard (*) hostname, key store yang sama akan bisa digunakan untuk semua brokers.
   ```bash
   sudo keytool -genkey -noprompt -alias server -dname "CN=server,OU=FS,O=ADI,L=JAKBAR,S=DKI,C=ID" -ext "SAN=dns:no
   de1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.schema,dns:node1.connect,dns:node1.
   ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.connect,dns:node2.ksql,dns:node2.res
   t,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,dns:node3.ksql,dns:node3.rest" -k
   eystore kafka.server.keystore.jks -keyalg RSA -storepass confluent -keypass confluent -storetype pkcs12
   ```
3. Membuat CSR
  ```bash
  sudo keytool -keystore kafka.server.keystore.jks -alias server -certreq -file server.csr -storepass confluent -k
  eypass confluent -ext "SAN=dns:node1,dns:node2,dns:node3,dns:localhost,dns:node1.zookeeper,dns:node1.kafka,dns:node1.sc
  hema,dns:node1.connect,dns:node1.ksql,dns:node1.rest,dns:node2.zookeeper,dns:node2.kafka,dns:node2.schema,dns:node2.con
  nect,dns:node2.ksql,dns:node2.rest,dns:node2.cc,dns:node3.zookeeper,dns:node3.kafka,dns:node3.schema,dns:node3.connect,
  dns:node3.ksql,dns:node3.rest"
  ```
4. Langkah selanjutnya, buat file server.cnf dengan isi :
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
5. Buat sertifikat SSL/TLS yang ditandatangani untuk server Kafka berdasarkan CSR yang telah dibuat sebelumnya
  ```bash
   sudo openssl x509 -req -CA ca-root.crt -CAkey ca-root.key -in server.csr -out server-signed.crt -sha256 -days 365 -CAcreateserial -passin pass:confluent -extensions v3_req -extfile server.cnf
   ```
6. Import CA ke keystore java
  ```bash
sudo keytool -noprompt -keystore server.keystore.jks -alias caroot -import -file ca-root.crt -storepass confluent -keypass confluent
```
7. Menggabungkan kedua file tersebut ke full-chain.xrt
   ```bash
   sudo sh -c 'cat server-signed.crt ca-root.crt > full-chain.crt'
   ```
8. Import sertifikat ke keystore java
  ```bash
  sudo keytool -noprompt -keystore kafka.server.keystore.jks -alias server -import -file full-chain.crt -storepass confluent -keypass confluent
  ```
9. Impor sertifikat root Certificate Authority (CA) ke dalam truststore Java
    ```bash
 sudo keytool -noprompt -keystore kafka.server.truststore.jks -alias caroot -import -file ca-root.crt -storepass confluent -keypass confluent
    ```
11. Salin files tersebut ke beberapa brokers dengan perintah :
  ```bash
   scp -r confluent@<alamat_ip>:/var/ssl/private /var/ssl/private
   ```

## Konfigurasi Services
Secara keseluruhan, kita akan membuat konfigurasi di dalam path /etc/kafka untuk semua services yang ada. Bisa dilihat [disini](https://github.com/firyal-salsa/confluent-apache-kafka/tree/main/kafka). 
Setelah membuat files konfigurasi, kita akan membuat folder /dataku dan buat folder-folder services confluent kafka di dalam folder tersebut.
Tambahkan permission untuk setiap services, contoh untuk zookeeper:
```bash
sudo chown -R cp-kafka:confluent /dataku/zookeeper
```
```bash
sudo chmod -R 755 /dataku/zookeeper
```
Khusus untuk zookeeper, tambahkan file berikut yang berisi nomor dari node sekarang
```bash
touch /data/zookeeper/myid
```

### Zookeeper

### Broker

### Schema Registry

### Kafka Connect

### Kafka REST


