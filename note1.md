# 🌐 Setup VPS Webserver: NGINX + React + Node.js (Tanpa cPanel)

Panduan lengkap setup **VPS** sebagai server production dengan:

* **Node.js (Express Backend)**
* **React (Frontend)**
* **NGINX (Reverse Proxy & SSL)**
* Tanpa menggunakan **cPanel**

---

## 1. 📟 Login sebagai Root & Buat User Baru

Login ke VPS via SSH:

```bash
ssh root@your-vps-ip
```

Update sistem dan buat user baru (untuk keamanan dan menghindari error konfigurasi saat root):

```bash
apt-get update
adduser your_username
usermod -aG sudo your_username
su - your_username
```

---

## 2. 🔧 Install Node.js

Install Node.js versi 22.x:

```bash
curl -sL https://deb.nodesource.com/setup_22.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt install nodejs
```

Cek versi:

```bash
node -v
npm -v
```

---

## 3. 🔐 Firewall & Keamanan Dasar

Konfigurasi UFW (Uncomplicated Firewall):

```bash
sudo apt install ufw
sudo ufw allow OpenSSH
sudo ufw allow 80,443/tcp
sudo ufw enable
```

---

## 4. 🚀 Install dan Konfigurasi NGINX

### Install NGINX:

```bash
sudo apt install nginx
```

### Buat file konfigurasi NGINX untuk domain kamu:

```bash
sudo nano /etc/nginx/sites-available/myapp
```

Paste konfigurasi berikut:

```nginx
server {
  listen 80;
  server_name yourdomain.com;

  location / {
    proxy_pass http://localhost:5000/; # Port backend kamu
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}
```

Simpan dan keluar (`Ctrl+X`, `Y`, `Enter`)

---

## 5. ⚙️ Aktifkan Konfigurasi NGINX

Aktifkan config yang sudah dibuat:

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

Uji konfigurasi:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 6. 🔒 SSL (HTTPS) dengan Certbot

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx
```

Generate dan pasang SSL:

```bash
sudo certbot --nginx -d yourdomain.com
```

---

## 7. 🟢 Jalankan Server dengan PM2

Install PM2 secara global:

```bash
sudo npm install -g pm2
```

Jalankan backend (misal `server.js`):

```bash
pm2 start server.js --name your-app-name
pm2 save
pm2 startup
```

Ikuti perintah yang ditampilkan setelah `pm2 startup` untuk mengaktifkan saat booting.

---

## 8. 🌍 Setting DNS Domain

Masuk ke panel domain dan atur DNS Record sesuai kebutuhan:

### Jika subdomain:

```
Type: A
Name: subdomain (contoh: api)
Value: your_vps_ip
```

### Jika domain utama:

```
Type: A
Name: @
Value: your_vps_ip
```

Tunggu propagasi DNS 1–30 menit.

---

## ✅ Selesai!

Server kamu sekarang bisa mengakses:

* `http://yourdomain.com` (otomatis redirect ke backend port 5000 via NGINX)
* `https://yourdomain.com` jika SSL aktif

---

### 🔄 Bonus: Build & Deploy React Frontend (opsional)

Jika kamu ingin React frontend jalan di domain utama dan API di subdomain:

* Build React:

  ```bash
  npm run build
  ```
* Copy isi `build/` ke folder publik, misal `/var/www/html/`
* Tambahkan konfigurasi `location /` pada NGINX sesuai folder tersebut

---

🧠 Butuh auto-deploy via GitHub/Webhook/CI? Tambahkan saja, nanti bisa bantu juga setup-nya.
