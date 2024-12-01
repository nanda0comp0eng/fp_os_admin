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

## Script Install dengan menggunakan Bash

```bash
#!/bin/bash

# Set variabel untuk konfigurasi
DOMAIN="chat.perusahaan.com"
ROCKET_VERSION="latest"

# Fungsi untuk menampilkan pesan error
error_exit() {
    echo "Error: $1" >&2
    exit 1
}

# Pastikan script dijalankan sebagai root
if [[ $EUID -ne 0 ]]; then
   error_exit "Script ini harus dijalankan dengan sudo" 
fi

# Update sistem
apt update && apt upgrade -y || error_exit "Gagal update sistem"

# Instal dependensi umum
apt install -y curl wget software-properties-common gnupg2 ca-certificates lsb-release || error_exit "Gagal instal dependensi"

# Instal MongoDB
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | apt-key add - || error_exit "Gagal impor kunci MongoDB"
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu $(lsb_release -cs)/mongodb-org/6.0 multiverse" | tee /etc/apt/sources.list.d/mongodb-org-6.0.list
apt update
apt install -y mongodb-org || error_exit "Gagal instal MongoDB"

# Jalankan dan aktifkan MongoDB
systemctl start mongod
systemctl enable mongod

# Instal dependensi Node.js
curl -fsSL https://deb.nodesource.com/setup_16.x | bash - || error_exit "Gagal setup Node.js"
apt install -y nodejs || error_exit "Gagal instal Node.js"

# Unduh dan ekstrak Rocket.Chat
wget https://releases.rocket.chat/${ROCKET_VERSION}/download -O rocket.chat.tar.gz || error_exit "Gagal unduh Rocket.Chat"
tar -xzf rocket.chat.tar.gz || error_exit "Gagal ekstrak Rocket.Chat"

# Buat direktori dan pengguna
mkdir -p /opt/Rocket.Chat
mv bundle/* /opt/Rocket.Chat/
useradd -M rocketchat
chown -R rocketchat:rocketchat /opt/Rocket.Chat

# Instal Nginx
apt install -y nginx || error_exit "Gagal instal Nginx"

# Instal Certbot
apt install -y certbot python3-certbot-nginx || error_exit "Gagal instal Certbot"

# Instal Fail2Ban
apt install -y fail2ban || error_exit "Gagal instal Fail2Ban"

# Instal alat backup
apt install -y mongodb-clients || error_exit "Gagal instal alat backup MongoDB"

# Bersihkan file unduhan
rm rocket.chat.tar.gz

echo "Instalasi Rocket.Chat Server selesai!"
```

## Lisensi
Open-source dengan lisensi MIT
