## Services
```bash
systemctl list-units --type=service --all
```

## Keamanan


## ACLs
Produce message
```bash
kafka-console-producer --topic ACLS --bootstrap-server node3.kafka:9092 --producer.config produce.properties
```
Consume message
```bash
kafka-console-consumer --topic ACLS --bootstrap-server node3.kafka:9092 --consumer.config consumer.properties
```

## Clean up VM / instance
Ini adalah tata cara untuk membersihkan VM kembali seperti keadaan semula, sebaiknya gunakan backup atau snapshot terlebih dahulu agar lebih aman dan data bisa direstore.
### Ubuntu
1. Lihat semua services dan pilih services mana saja yang akan dihapus
   ```bash
   systemctl list-units --type=service --all
   ```
   
2. Nonaktifkan dan hentikan service
   ```bash
    sudo systemctl stop <nama_service>
    sudo systemctl disable <nama_service>
  ```

3. Hapus service
```bash
    sudo apt-get <purge nama_aplikasi>
    sudo apt-get autoremove
  ```

4. Hapus folder konfigurasi dari aplikasi yang sudah dihapus
  ```bash
     sudo rm -rf /etc/<nama-aplikasi> /var/lib/<nama-aplikasi>
  ```

5. Hapus cache
  ```bash
  sudo apt clean
  ```

6. Atur kembali hak akses sesuai kebutuhan
   ```bash
    sudo chown root:root /etc/<somefile>
    sudo chmod 644 /etc/<somefile>
    ```
   
7. Hapus pengguna yang tidak lagi dibutuhkan
  ```bash
  sudo deluser <nama-user>
  sudo rm -rf /home/<nama-user>
  ```

8. Hapus log yang lebih dari 7 hari
  ```bash
  sudo journalctl --vacuum-time=7d 
  ```

9. Jika cronjob di set, maka periksa cronjobnya
  ```bash
  crontab -e
  ```

10. Hapus file sementara
  ```bash
  sudo rm -rf /tmp/*
  sudo rm -rf /var/tmp/*
  ```

11. Hapus file konfigurasi
    ```bash
    rm -rf ~/.config/<nama-aplikasi> ~/.<nama-aplikasi>
    ```
12. Cek service yang masih berjalan, jika ada maka bisa hentikan proses yang berjalan
  ```bash
    ps aux
  ```
13. Periksa Pengaturan di .bashrc atau .profile
14. Restart server
```bash
reboot
```
