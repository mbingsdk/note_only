# ✅ Setup OpenVPN Server (VPS) untuk MikroTik Clients

**Platform**: Ubuntu 22

---

## 🧩 1. Instalasi OpenVPN dan Easy-RSA

```bash
apt update && apt install -y openvpn easy-rsa iptables-persistent
```

---

## 🔐 2. Setup PKI (Public Key Infrastructure)

```bash
make-cadir /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa

./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full server nopass
./easyrsa gen-dh
./easyrsa gen-crl
```

---

## 📂 3. Salin Sertifikat & Kunci ke Direktori OpenVPN

```bash
cp pki/ca.crt pki/private/server.key pki/issued/server.crt /etc/openvpn/
cp pki/dh.pem /etc/openvpn/dh.pem
cp pki/crl.pem /etc/openvpn/crl.pem
chown nobody:nogroup /etc/openvpn/crl.pem
```

---

## ⚙️ 4. Konfigurasi `/etc/openvpn/server.conf` (Khusus TCP untuk MikroTik)

Gunakan konfigurasi ini:

```bash
cat > /etc/openvpn/server.conf <<EOF
port 1194
proto tcp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh.pem
crl-verify crl.pem
user nobody
group nogroup
persist-key
persist-tun
auth SHA1
cipher AES-256-CBC
data-ciphers AES-256-CBC
data-ciphers-fallback AES-256-CBC
verify-client-cert none
topology subnet
auth-user-pass-verify /etc/openvpn/login.sh via-env
username-as-common-name
script-security 3
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
keepalive 10 120
status openvpn-status.log
log-append /var/log/openvpn.log
verb 3
EOF
```

---

## 🔄 5. Aktifkan IP Forwarding

```bash
echo 'net.ipv4.ip_forward=1' >> /etc/sysctl.conf
sysctl -p
```

---

## 🌐 6. Setup NAT & Port Forwarding (Akses MikroTik dari VPS)

```bash
# Forward ke MikroTik di VPN
# Bisa di set terakhir karna 10.8.0.2 adalah vpn cilent/mikrotik

iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 10001 -j DNAT --to-destination 10.8.0.2:8728
iptables -A FORWARD -p tcp -d 10.8.0.2 --dport 8728 -j ACCEPT

iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 10002 -j DNAT --to-destination 10.8.0.2:8291
iptables -A FORWARD -p tcp -d 10.8.0.2 --dport 8291 -j ACCEPT

# NAT Masquerade
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE

# Simpan aturan firewall
netfilter-persistent save
```

---

## 👤 7. Tambahkan User Login VPN

```bash
cat >> /etc/openvpn/psw-file <<EOF
userVpn passs
EOF
```

### Buat Script Login

```bash
cat > /etc/openvpn/login.sh <<EOF
#!/bin/sh
USERNAME=$common_name
PASSWORD=$password
if grep -q "^$USERNAME $PASSWORD" /etc/openvpn/psw-file; then
  exit 0
else
  exit 1
fi
EOF

chmod +x /etc/openvpn/login.sh
```

---

## 🚀 8. Jalankan OpenVPN Server

```bash
systemctl enable openvpn@server
systemctl restart openvpn@server
```

> ❗ **Catatan**: Jika muncul error `REMOVED OPTION: --client-cert-not-required`, gunakan `verify-client-cert none` seperti pada konfigurasi di langkah 4.

---

## 📡 9. Contoh Konfigurasi MikroTik OpenVPN Client

```bash
/interface ovpn-client add \
    name=ovpn-to-vps \
    connect-to=VPS_IP \
    port=1194 \
    user=userVpn \
    password=passs \
    protocol=tcp-client \
    profile=default-encryption \
    certificate=none \
    add-default-route=no \
    auth=sha1 cipher=aes256
```

---

## 🔎 10. Tes Koneksi Port MikroTik di Jaringan VPN

```bash
nmap 10.8.0.2 -p 8291,8728
```

---

## 🌐 11. (Opsional) Akses MikroTik via Subdomain (Reverse Proxy)

### Buat file konfigurasi Nginx:

`/etc/nginx/sites-available/mikrotik.example.com`

```nginx
server {
    listen 80;
    server_name mikrotik.example.com;

    location / {
        proxy_pass http://10.8.0.2;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Aktifkan dan reload Nginx

```bash
ln -s /etc/nginx/sites-available/mikrotik.example.com /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### Aktifkan HTTPS (Let’s Encrypt)

```bash
certbot --nginx -d mikrotik.example.com
```

---

## ✅ Selesai

- MikroTik sekarang dapat terkoneksi ke VPS sebagai OpenVPN client.
- Akses MikroTik via VPS (misalnya melalui Winbox/API): `ipvps:10001` atau `ipvps:10002`.

---

**Catatan:** Jangan lupa mengganti username/password dan domain sesuai kebutuhan produksi!
