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
1. **Lihat Semua Services**
   dan pilih services mana saja yang akan dihapus.
   ```bash
   systemctl list-units --type=service --all
   ```

2. **Nonaktifkan dan Hentikan Service**
    yang ingin dihapus.
   ```bash
   sudo systemctl stop <nama_service>
   sudo systemctl disable <nama_service>
   ```

3. **Hapus Service**
    terkait dengan service tersebut.
   ```bash
   sudo apt-get purge <nama_aplikasi>
   sudo apt-get autoremove
   ```

4. **Hapus Folder Konfigurasi**
    dari aplikasi yang sudah dihapus.
   ```bash
   sudo rm -rf /etc/<nama-aplikasi> /var/lib/<nama-aplikasi>
   ```

5. **Hapus Cache**
    untuk mengosongkan ruang.
   ```bash
   sudo apt clean
   ```

6. **Atur Ulang Hak Akses File**
   ```bash
   sudo chown root:root /etc/<somefile>
   sudo chmod 644 /etc/<somefile>
   ```

7. **Hapus Pengguna yang Tidak Dibutuhkan**
   ```bash
   sudo deluser <nama-user>
   sudo rm -rf /home/<nama-user>
   ```

8. **Hapus Log yang Lebih dari 7 Hari**
   untuk menghemat ruang.
   ```bash
   sudo journalctl --vacuum-time=7d 
   ```

9. **Periksa Cron Jobs**
   jika cron job diatur, periksa dan hapus jika tidak diperlukan.
   ```bash
   crontab -e
   ```

10. **Hapus File Sementara**
    ```bash
    sudo rm -rf /tmp/*
    sudo rm -rf /var/tmp/*
    ```

11. **Hapus File Konfigurasi Pengguna**
    yang tertinggal di direktori home pengguna.
    ```bash
    rm -rf ~/.config/<nama-aplikasi> ~/.<nama-aplikasi>
    ```

12. **Cek Proses yang Masih Berjalan**
     untuk menghentikan service yang tidak diperlukan.
    ```bash
    ps aux
    ```

13. **Periksa `.bashrc` atau `.profile` untuk Perubahan yang Tidak Diperlukan**

14. **Restart Server**
    untuk menerapkan perubahan.
    ```bash
    sudo reboot
    ```
