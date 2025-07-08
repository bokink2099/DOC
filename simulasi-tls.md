# 🔐 Simulasi Pembuatan Sertifikat TLS Lengkap Secara Lokal (Dengan CA Sendiri)

## 🎯 Tujuan
Membuat infrastruktur TLS secara penuh seperti di environment production, tapi hanya menggunakan tools lokal (`openssl`). Flow-nya akan melibatkan:
1. Membuat CA (Certificate Authority) lokal
2. Membuat CSR (Certificate Signing Request)
3. Menandatangani sertifikat server dengan CA
4. Menguji sertifikat melalui `curl` dan `nginx`
5. (Opsional) Menambahkan CA ke sistem agar dianggap trusted

## 📂 Struktur Direktori

tls-local/

├── ca/

│ ├── myCA.key # Private key CA

│ └── myCA.crt # Public cert CA (self-signed)

├── server/

│ ├── mysite.key # Private key server (localhost)

│ ├── mysite.csr # Certificate Signing Request

│ └── mysite.crt # Sertifikat final ditandatangani oleh CA

└── README.md

---

## 🧱 1. Buat Certificate Authority (CA) Lokal

### 🔑 Generate Private Key untuk CA
```bash
openssl genrsa -out ca/myCA.key 2048

📄 Buat Sertifikat Self-Signed untuk CA

openssl req -x509 -new -nodes -key ca/myCA.key -sha256 -days 3650 -out ca/myCA.crt

Isi metadata sesuai kebutuhan. Common Name (CN) bisa diisi: My Local CA

🧾 2. Buat Key dan CSR untuk Server (localhost)
🔑 Generate Private Key untuk Server

openssl genrsa -out server/mysite.key 2048
📄 Buat CSR (Certificate Signing Request)

openssl req -new -key server/mysite.key -out server/mysite.csr

Isikan Common Name: localhost
Ini penting agar cocok dengan domain saat dipakai di curl atau nginx.

✍️ 3. Tanda Tangani CSR Menggunakan CA

openssl x509 -req -in server/mysite.csr \
  -CA ca/myCA.crt -CAkey ca/myCA.key -CAcreateserial \
  -out server/mysite.crt -days 825 -sha256

🌐 4. Testing dengan Nginx
Contoh Konfigurasi Nginx

server {
    listen 443 ssl;
    server_name localhost;

    ssl_certificate     /full/path/server/mysite.crt;
    ssl_certificate_key /full/path/server/mysite.key;

    location / {
        return 200 "TLS OK";
    }
}
Coba Akses Pakai curl

curl --cacert ca/myCA.crt https://localhost

Tanpa --cacert, curl akan anggap cert tidak trusted.

🖥️ (Opsional) Trust CA di MacOS

sudo security add-trusted-cert -d -r trustRoot \
  -k /Library/Keychains/System.keychain ca/myCA.crt

Setelah ini, lo bisa akses:

curl https://localhost

Tanpa warning lagi.

🧠 Penjelasan Singkat Tiap File
File	Deskripsi
myCA.key	Private key CA (jangan dishare)
myCA.crt	Sertifikat publik CA
mysite.key	Private key milik server (localhost)
mysite.csr	Permintaan tanda tangan cert dari server
mysite.crt	Cert final server, ditandatangani oleh CA

🔄 Diagram Alur TLS Lokal

[ CA (myCA.key + myCA.crt) ]
         ↓
  [ Sign CSR (mysite.csr) ]
         ↓
[ Output: mysite.crt + mysite.key ] → digunakan di Nginx / TLS server

🧪 Next Steps (Lanjutan Opsional)

✅ Tambahkan SAN (Subject Alternative Name)

✅ Buat intermediate CA

✅ Simulasi expired / revoked certificate

✅ Coba TLS di Docker container

✅ Validasi sertifikat dengan openssl:

openssl verify -CAfile ca/myCA.crt server/mysite.crt
📌 Referensi Singkat Perintah

# Generate private key (CA/server)
openssl genrsa -out file.key 2048

# Generate self-signed cert (CA)
openssl req -x509 -new -nodes -key file.key -out file.crt -days 365

# Generate CSR (server)
openssl req -new -key server.key -out server.csr

# Sign CSR with CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 825 -sha256

# Verify TLS handshake (client)
curl --cacert ca.crt https://localhost
🔒 End Result
✔️ Lo punya CA sendiri
✔️ Lo ngerti flow sertifikat dari key → CSR → signed cert
✔️ Lo bisa setup HTTPS lokal tanpa warning
✔️ Lo bisa debug TLS kayak di production
