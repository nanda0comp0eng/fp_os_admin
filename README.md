# Internal Live-Chat server
Live chat server yang dapat digunakan untuk internal perusahaan guna menjaga kerahasiaan serta keamanan perusahaan.

## Topologi Arsitektur
![Topologi Infrastruktur](/img/infra.png)

## Deskripsi Projet
Implementasi server chatting internal berbasis open-source yang aman, scalable, dan mudah dikelola untuk komunikasi perusahaan.

## Service yang Digunakan

### 1. Rocket.Chat
- **Versi**: 6.x LTS
- **Deskripsi**: Platform open-source untuk komunikasi tim 
- **Fitur Utama**:
  - Obrolan real-time
  - Saluran pribadi dan umum
  - Integrasi dengan layanan eksternal
  - Keamanan end-to-end

### 2. MongoDB
- **Versi**: 6.0 Community Edition
- **Deskripsi**: Basis data dokumentasi NoSQL 
- **Keunggulan**:
  - Penyimpanan data fleksibel
  - Performa tinggi
  - Skalabilitas horizontal
  - Replikasi otomatis

### 3. Nginx
- **Versi**: 1.24.x Stable
- **Deskripsi**: Web server dan reverse proxy
- **Fungsi**:
  - Load balancing
  - Terminasi SSL
  - Caching
  - Keamanan lapis aplikasi

### 4. Certbot
- **Versi**: Latest (0.40.x)
- **Deskripsi**: Alat manajemen sertifikat Let's Encrypt
- **Kemampuan**:
  - Otomatisasi perpanjangan sertifikat
  - Dukungan HTTPS penuh
  - Konfigurasi sederhana

### 5. Fail2Ban
- **Versi**: 0.11.x
- **Deskripsi**: Sistem pencegahan instrusi
- **Fitur**:
  - Blokir IP yang mencurigakan
  - Perlindungan brute-force
  - Kustomisasi aturan keamanan

### 6. SSH
- **Versi**: OpenSSH 8.x
- **Deskripsi**: Protokol akses jarak jauh
- **Keamanan**:
  - Autentikasi kunci publik
  - Enkripsi koneksi
  - Pembatasan akses

### 7. Cron
- **Versi**: Varian Linux standar
- **Fungsi**:
  - Backup terjadwal
  - Manajemen tugas berkala
  - Pemeliharaan sistem otomatis

## OS yang digunakan
- CPU: 2 Core
- RAM: 4 GB
- Sistem Operasi: Ubuntu 22.04 LTS

## Lisensi
Open-source dengan lisensi MIT
