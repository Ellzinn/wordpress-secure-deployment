
# Tujuan Step 1

Pada tahap ini saya melakukan pemilihan image container untuk WordPress dan MariaDB.

Fokus utama pada step ini:
- menggunakan image yang lebih aman
- tidak menggunakan tag latest
- memastikan deployment lebih stabil dan konsisten

---

# Apa Itu Docker Image?

Docker image adalah template aplikasi yang digunakan untuk membuat container.

Contoh:
- image WordPress → berisi aplikasi WordPress
- image MariaDB → berisi database MariaDB

Docker nantinya menjalankan image tersebut menjadi container.

Analogi sederhana:

- image = file installer
- container = aplikasi yang sedang berjalan

---

# Apa Itu Container?

Container adalah aplikasi yang sedang berjalan di Docker.

Pada project ini terdapat 2 container:
- WordPress container
- MariaDB container

Kedua container tersebut saling terhubung.

WordPress digunakan sebagai website.

MariaDB digunakan sebagai database untuk menyimpan data WordPress.

---

# Kenapa Menggunakan Chainguard?

Task meminta menggunakan image dari Chainguard.

Chainguard merupakan image container yang lebih fokus pada security dan minimal vulnerability.

Keunggulan Chainguard:
- lebih aman
- minimal CVE
- lightweight
- cocok untuk production
- lebih fokus pada container security

Repository yang digunakan:

```text
hub.docker.com/u/chainguard
```

---

# Kenapa Tidak Menggunakan latest?

Tag latest berarti menggunakan versi paling baru secara otomatis.

Contoh:

```yaml
image: chainguard/wordpress:latest
```

Masalah dari latest:
- versi bisa berubah sewaktu-waktu
- deployment menjadi tidak konsisten
- sulit rollback
- lebih sulit untuk audit security

Karena itu lebih baik menggunakan:
- specific version
- atau SHA digest

---

# Menggunakan Specific Version

Sesuai requirement task, digunakan specific version.

## WordPress

```yaml
image: chainguard/wordpress:6.8
```

## MariaDB

```yaml
image: chainguard/mariadb:11
```

Keuntungan specific version:
- deployment lebih stabil
- konsisten
- mudah troubleshooting
- lebih aman dibanding latest

---

# Attempt Sebelumnya

Pada awal percobaan digunakan repository:

```yaml
cgr.dev/chainguard/wordpress
cgr.dev/chainguard/mariadb
```

Namun beberapa version tag tidak tersedia sehingga deployment gagal dengan error:

```text
not found
failed to resolve reference
```

Setelah mendapatkan informasi tambahan dari panitia, digunakan repository Chainguard melalui Docker Hub.

---

# Apa Itu SHA Digest?

SHA digest adalah sidik jari unik dari image Docker.

Contoh:

```text
sha256:xxxxxxxxxxxxxxxx
```

SHA digest memastikan image:
- benar-benar spesifik
- immutable
- tidak berubah otomatis
- lebih aman untuk production

Analogi sederhana:

- version tag = nama HP
- SHA digest = IMEI HP

SHA digest lebih spesifik dibanding version tag.

---

# Cara Mendapatkan SHA Digest

Pull image terlebih dahulu:

```bash
docker pull chainguard/wordpress:6.8
```

Kemudian cek digest:

```bash
docker images --digests
```

Contoh hasil:

```text
sha256:abc123xxxxxxxx
```

---

# Contoh Penggunaan SHA Digest

```yaml
image: chainguard/wordpress@sha256:abc123xxxxxxxx
```

Keuntungan SHA digest:
- image tidak berubah
- deployment lebih konsisten
- cocok untuk production environment
- lebih aman dibanding latest


 # Deployment Steps

## 1. Buka Terminal

Gunakan shortcut:

```bash
Ctrl + Alt + T
```

---

## 2. Buat Folder Project

```bash
mkdir secure-wordpress
cd secure-wordpress
```

---

## 3. Buat Folder Volume

```bash
mkdir wordpress_data
mkdir mariadb_data
```

Folder ini digunakan untuk persistent storage container.

---

## 4. Set Permission untuk Non-Root User

Container Chainguard berjalan menggunakan non-root user dengan UID/GID `65532`.

```bash
sudo chown -R 65532:65532 wordpress_data
sudo chown -R 65532:65532 mariadb_data
```

Verifikasi:

```bash
ls -l
```

Tujuan:
Agar container dapat menulis data ke volume tanpa error `Permission Denied`.

---

## 5. Buat File Environment (.env)

Buat file `.env`:

```bash
nano .env
```

Isi file:

```env
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=passwordku123
MYSQL_ROOT_PASSWORD=rootpassword123
```

Simpan file:
- CTRL + O
- Enter
- CTRL + X

Tujuan:
Password database tidak ditulis langsung di `docker-compose.yml`.

---

## 6. Buat File docker-compose.yml

Buat file:

```bash
nano docker-compose.yml
```

Isi file:

```yaml
services:
  mariadb:
    image: chainguard/mariadb:11.4
    container_name: mariadb_secure

    env_file:
      - .env

    environment:
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}

    volumes:
      - ./mariadb_data:/var/lib/mysql

    networks:
      - backend_net

    restart: unless-stopped

    mem_limit: 512m
    cpus: "0.5"

    user: "65532:65532"

  wordpress:
    image: chainguard/wordpress:6.8
    container_name: wordpress_secure

    depends_on:
      - mariadb

    env_file:
      - .env

    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}

    volumes:
      - ./wordpress_data:/var/www/html

    ports:
      - "8080:8080"

    networks:
      - frontend_net
      - backend_net

    restart: unless-stopped

    mem_limit: 512m
    cpus: "0.5"

    user: "65532:65532"

networks:
  frontend_net:
  backend_net:
```

Simpan file:
- CTRL + O
- Enter
- CTRL + X

---

## 7. Verifikasi File Compose

```bash
cat docker-compose.yml
```

---

## 8. Jalankan Container

```bash
docker compose up -d
```

---

## 9. Verifikasi Container Berjalan

```bash
docker ps
```

Pastikan:
- `wordpress_secure` berjalan
- `mariadb_secure` berjalan

---

## 10. Verifikasi MariaDB Tidak Mengekspos Port

Perhatikan output:

```bash
docker ps
```

MariaDB tidak boleh memiliki:

```text
0.0.0.0:3306->3306
```

Tujuan:
Agar database tidak dapat diakses langsung dari host maupun internet.

---

## 11. Verifikasi Network

Lihat daftar network:

```bash
docker network ls
```

---

## 12. Inspect Backend Network

```bash
docker network inspect secure-wordpress_backend_net
```

Pastikan:
- WordPress terhubung
- MariaDB terhubung

---

## 13. Inspect Frontend Network

```bash
docker network inspect secure-wordpress_frontend_net
```

Pastikan:
- Hanya WordPress yang terhubung

---

## 14. Verifikasi Log Container

Cek log WordPress:

```bash
docker logs wordpress_secure
```

Cek log MariaDB:

```bash
docker logs mariadb_secure
```

Pastikan:
- Tidak ada error
- Tidak ada `Permission Denied`

---

## 15. Akses WordPress

Buka browser:

```text
http://localhost:8080
```

---

## 16. Verifikasi Resource Limitation

```bash
docker stats
```

Pastikan container memiliki batas:
- RAM: 512MB
- CPU: 0.5

---

## 17. Stop dan Hapus Container

```bash
docker compose down
```

# Kesimpulan

Pada step ini digunakan image WordPress dan MariaDB dari Chainguard karena lebih fokus pada security dan minimal vulnerability.

Specific version digunakan agar deployment lebih stabil dibanding latest.

Selain itu dipelajari juga penggunaan SHA digest sebagai metode deployment yang lebih immutable dan konsisten untuk production environment.
