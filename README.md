# Securing services on Confluent Apache Kafka using the SCRAM-SHA-256 method
<h3 align="center">
  <img src="https://res.cloudinary.com/dvehyvk3d/image/upload/v1730172698/confluent-images_bruliy.png" width="50%"/>
</h3>

> Security is an aspect of the system that must be addressed for the system as a whole, rather than component by component. The security of a system is only as strong as the weakest link, and security processes and policies must be enforced across the system, including the underlying platform. The customizable security features in Kafka enable integration with existing security infrastructure to build a consistent security model that applies to the entire system. (Kafka The Definitive Guide, hal 265)

<br>

Repositori ini menyediakan panduan instalasi confluent apache kafka, konfigurasi keamanan, pengujian keamanan, sampai penggunaan setiap services yang ada di confluent. Untuk mencegah penyadapan data dan memodifikasi data tanpa izin, maka pemasangan security menggunakan standar SSL (Secure Socket Layer) dan SASL (Simple Authentication and Security Layer) diperlukan untuk melindungi data dari sisi client maupun server. SSL mengenkripsi message selama transmisi, sementara SASL memberikan mekanisme otentikasi. Ada beberapa cara untuk otentikasi menggunakan SASL, yaitu dengan cara GSSAPI, OAUTHBREAKER, SCRAM-SHA-256 dan SCRAM-SHA-512. Pada kesempatan kali ini, kita akan uji coba menggunakan SCRAM-SHA 256 yang mana user perlu menginput kredensial seperti username dan password untuk mengakses service tertentu. Metode ini menggabungkan hashing dengan salting untuk meningkatkan keamanan.

Penggunaan metode SCRAM-SHA-256 sangat cocok digunakan untuk excercise di 3 VM di 1 cluster karena belum tahap production. Berikut contoh kasus :
<ul>
  <li>Administrator adalah super user, yang berarti user ini memiliki hak akses penuh seperti membaca dan memodifikasi data</li>
  <li>Alice hanya bisa produce / menginput data</li>
  <li>Bob hanya bisa consume atau read data</li>
   Alice dan Bob memiliki akses terbatas untuk mencegah potensi penyalahgunaan
</ul>


## Daftar Isi
<ol>
  <li><a href="https://github.com/firyal-salsa/confluent-apache-kafka/blob/main/install-services-ssl.md">Instalasi Confluent Kafka menggunakan Linux Ubuntu & Rocky serta memasang SSL</a></li>
  <li>Konfigurasi produce - consume message menggunakan ACLs </li>
  <li>Mengamankan beberapa services yang ada di confluent control center menggunakan SCRAM-SHA-256</li>
  <li>Mengoperasikan Kafka Connect</li>
  <li>Mengoperasikan schema registry</li>
  <li>Mengoperasikan REST</li>
</ol>


## Sumber
<ol>
  <li> Kafka The Definitive Guide 2nd Edition </li>
  <li> Confluent Documentation </li>
</ol>

## Kontributor
