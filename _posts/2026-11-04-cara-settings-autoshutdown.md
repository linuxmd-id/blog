---
layout: post
title: "Cara bikin Timer auto-shudown"
date: 2026-11-04
excerpt: "Membuat fitur Auto-Shutdown di Linux menggunakan Systemd User Service agar laptop mati otomatis setiap hari pukul 22.00 tanpa perlu dimatikan secara manual."
image: /assets/images/covers/image.png
cover: /assets/images/covers/cover.png
---

# Membuat Timer Auto-Shutdown Otomatis Setiap Pukul 22.00 di Linux Menggunakan Systemd

## 1. Pendahuluan

Pernahkah kalian menggunakan laptop hingga larut malam, kemudian tanpa sadar tertidur dan lupa mematikan laptop?

Hal seperti ini cukup sering terjadi, terutama bagi pengguna yang terbiasa mengerjakan tugas, menonton film, bermain game, atau melakukan aktivitas lainnya menggunakan laptop pada malam hari.

Ketika laptop dibiarkan menyala sepanjang malam, daya listrik tetap digunakan meskipun komputer sudah tidak digunakan secara aktif.

Dari permasalahan tersebut, saya mencoba membuat sebuah solusi berupa **Timer Auto-Shutdown**, yaitu fitur yang memungkinkan laptop Linux mati secara otomatis pada waktu yang telah ditentukan.

Pada eksperimen ini, saya membuat sistem agar laptop melakukan shutdown secara otomatis setiap hari pukul **22.00 atau jam 10 malam**.

Untuk membuatnya, saya memanfaatkan Systemd User Service, yaitu fitur pada Linux yang memungkinkan pengguna mengatur layanan dan menjalankan perintah secara otomatis.

Dengan adanya fitur ini, pengguna tidak perlu lagi mengingat untuk mematikan laptop setiap malam karena proses shutdown sudah dijadwalkan oleh sistem.

## 2. Mengenal Systemd dan Timer

Sebelum masuk ke proses pembuatan, kita perlu mengenal teknologi yang digunakan.

**Systemd** merupakan sistem pengelola layanan pada Linux yang bertanggung jawab mengatur berbagai proses dan layanan di dalam sistem operasi.

Salah satu fitur yang dimiliki systemd adalah Timer. Fitur ini memungkinkan sebuah service dijalankan berdasarkan jadwal tertentu.

Dalam eksperimen ini, terdapat dua komponen utama:

### 1. Service

Service merupakan bagian yang bertugas menjalankan perintah ketika dipanggil. Dalam proyek ini, service digunakan untuk menjalankan perintah shutdown pada komputer.

### 2. Timer

Timer bertugas menentukan kapan sebuah service harus dijalankan. Pada proyek ini, timer dikonfigurasi agar memanggil service setiap hari pukul 22.00.

Kedua komponen tersebut bekerja sama untuk membuat sistem Auto-Shutdown berjalan secara otomatis.

## 3. Persiapan Pembuatan

Langkah pertama adalah memastikan Linux yang digunakan sudah mendukung systemd.

Untuk memeriksanya, buka Terminal kemudian jalankan:

```bash
systemctl --version
```

Setelah itu, buat direktori khusus untuk menyimpan konfigurasi Systemd User Service:

```bash
mkdir -p ~/.config/systemd/user
```

Direktori tersebut digunakan untuk menyimpan file konfigurasi service dan timer yang akan dibuat.

## 4. Membuat File Service

Setelah direktori berhasil dibuat, langkah selanjutnya adalah membuat file service.

File ini bertanggung jawab menjalankan perintah shutdown ketika timer mencapai jadwal yang telah ditentukan.

Buat file dengan perintah:

```bash
nano ~/.config/systemd/user/auto-shutdown.service
```

Kemudian masukkan konfigurasi berikut:

```ini
[Unit]
Description=Daily Auto Shutdown Service

[Service]
Type=oneshot
ExecStart=/usr/bin/systemctl poweroff
```

Simpan file tersebut.

### Penjelasan konfigurasi

- `[Unit]` digunakan untuk memberikan informasi mengenai layanan.
- `Description` merupakan deskripsi atau nama layanan.
- `[Service]` berisi konfigurasi mengenai bagaimana layanan dijalankan.
- `Type=oneshot` berarti service hanya menjalankan satu tugas kemudian selesai.
- `ExecStart` merupakan perintah yang dijalankan ketika service dipanggil.

Pada bagian `ExecStart`, terdapat perintah `systemctl poweroff` yang berfungsi mematikan komputer.

## 5. Membuat File Timer

Setelah file service selesai dibuat, selanjutnya kita membuat file timer.

Timer ini akan mengatur jadwal kapan service Auto-Shutdown dijalankan.

Buat file menggunakan perintah:

```bash
nano ~/.config/systemd/user/auto-shutdown.timer
```

Masukkan konfigurasi berikut:

```ini
[Unit]
Description=Daily Shutdown at 10 PM

[Timer]
OnCalendar=*-*-* 22:00:00
Persistent=false
Unit=auto-shutdown.service

[Install]
WantedBy=timers.target
```

Simpan file tersebut.

### Penjelasan konfigurasi

- `[Unit]` berisi informasi mengenai timer.
- `Description` memberikan nama atau deskripsi timer.
- `[Timer]` merupakan bagian yang mengatur jadwal pelaksanaan.
- `OnCalendar=*-*-* 22:00:00` menentukan jadwal agar timer berjalan setiap hari pukul 22.00.
- `Persistent=true` membuat systemd tidak mengejar jadwal yang terlewat ketika timer sebelumnya tidak aktif,jika pakai false maka systemd mengejar jadwal yang tertinggal
- `Unit=auto-shutdown.service` menentukan service yang akan dijalankan ketika waktu timer tercapai.
- `[Install]` menentukan target aktivasi timer.

Hal yang menarik dari konfigurasi `OnCalendar` adalah kita tidak perlu membuat jadwal untuk setiap hari secara terpisah.

Tanda `*` pada bagian tanggal menunjukkan bahwa jadwal berlaku untuk semua tanggal, sedangkan `22:00:00` menunjukkan pukul 10 malam.

Dengan demikian, timer akan memiliki jadwal yang berulang setiap hari.

## 6. Mengaktifkan Timer Auto-Shutdown

Setelah kedua file berhasil dibuat, langkah selanjutnya adalah memuat ulang konfigurasi systemd.

Gunakan perintah:

```bash
systemctl --user daemon-reload
```

Perintah ini digunakan agar systemd mengenali file konfigurasi baru yang sudah dibuat.

Kemudian aktifkan timer:

```bash
systemctl --user enable --now auto-shutdown.timer
```

Perintah tersebut memiliki dua fungsi:

- `enable` membuat timer aktif secara otomatis ketika sesi pengguna dimulai.
- `--now` langsung menjalankan timer tanpa harus menunggu sesi berikutnya.

Setelah berhasil diaktifkan, systemd akan mengelola jadwal Auto-Shutdown berdasarkan konfigurasi yang sudah dibuat.

## 7. Memeriksa Jadwal Timer

Untuk memastikan timer sudah berjalan, kita dapat memeriksa daftar timer menggunakan perintah:

```bash
systemctl --user list-timers
```

Perintah tersebut akan menampilkan daftar timer yang aktif beserta informasi waktu pelaksanaan berikutnya.

Kita juga dapat memeriksa status timer secara langsung:

```bash
systemctl --user status auto-shutdown.timer
```

Jika konfigurasi berhasil dikenali dan timer aktif, status layanan akan menunjukkan bahwa timer sedang berjalan.

## 8. Bagaimana Sistem Ini Bekerja Setiap Hari?

Setelah timer berhasil diaktifkan, systemd akan mengelola jadwal shutdown secara otomatis.

Sebagai contoh, berikut simulasi penggunaannya:

| Waktu | Aktivitas |
|---|---|
| 08.00 | Laptop dinyalakan |
| 12.00 | Laptop digunakan seperti biasa |
| 18.00 | Pengguna masih melakukan aktivitas |
| 21.00 | Laptop masih dapat digunakan |
| 22.00 | Timer memanggil service |
| 22.00 | Sistem menjalankan proses shutdown |

Setelah itu, jadwal akan berlaku kembali pada hari berikutnya.

Dengan sistem ini, pengguna tidak perlu mengatur ulang timer setiap malam.

## 9. Menghentikan atau Menonaktifkan Timer

Selain membuat sistem otomatis, kita juga perlu mengetahui cara menghentikannya.

Jika ingin menghentikan timer sementara, gunakan:

```bash
systemctl --user stop auto-shutdown.timer
```

Jika ingin menonaktifkan timer agar tidak aktif otomatis ketika sesi pengguna dimulai, gunakan:

```bash
systemctl --user disable --now auto-shutdown.timer
```

Dengan demikian, pengguna tetap memiliki kendali untuk mengaktifkan atau menonaktifkan fitur sesuai kebutuhan.

## 10. Hal yang Perlu Diperhatikan

Ada beberapa hal yang perlu diperhatikan sebelum menggunakan fitur Auto-Shutdown ini.

1. Pastikan semua pekerjaan sudah disimpan sebelum pukul 22.00 karena komputer akan menjalankan proses pematian secara otomatis.
2. Systemd User Service memiliki keterkaitan dengan sesi pengguna. Jika ingin timer tetap aktif meskipun pengguna logout, diperlukan konfigurasi tambahan seperti `loginctl enable-linger`.
3. Perintah shutdown dari User Service dapat membutuhkan izin tambahan, tergantung konfigurasi dan kebijakan otorisasi Linux yang digunakan.
4. Jika laptop dalam kondisi sleep ketika pukul 22.00 tiba, timer tidak selalu dapat membangunkan laptop untuk menjalankan shutdown. Perilaku tersebut bergantung pada konfigurasi sistem dan mekanisme timer.

## 11. Kesimpulan

Dari eksperimen ini, saya mencoba membuat sebuah fitur sederhana yang dapat membantu mengatasi kebiasaan lupa mematikan laptop pada malam hari.

Dengan memanfaatkan Systemd User Service dan Systemd Timer, saya dapat mengatur agar komputer Linux menjalankan proses shutdown secara otomatis setiap hari pukul 22.00.

Eksperimen ini menunjukkan bahwa Linux memiliki fitur bawaan yang dapat dimanfaatkan untuk membuat otomatisasi sesuai kebutuhan pengguna.

Meskipun konsepnya sederhana, proyek ini memberikan pemahaman mengenai penggunaan service, timer, konfigurasi sistem, serta bagaimana sebuah perintah dapat dijalankan secara terjadwal.

Harapannya, fitur Auto-Shutdown ini dapat menjadi solusi praktis bagi pengguna Linux yang ingin mengatur waktu penggunaan laptop dengan lebih teratur dan tidak perlu lagi mengingat untuk mematikan komputer secara manual setiap malam.
