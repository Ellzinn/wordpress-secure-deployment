
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

---Akses WordPress

# Apa Itu Container?

Container adalah aplikasi yang sedang berjalan di Docker.

Pada project ini terdapat 2 container:
- WordPress container
- MariaDB container

Kedua container tersebut saling terhubung.Akses WordPress

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

## 1. Image Selection
...

## Tujuan

Pada tahap ini dilakukan pemilihan image container yang aman dan sesuai dengan requirement tugas.

Fokus utama:
- menggunakan repository Chainguard
- tidak menggunakan tag `latest`
- menggunakan immutable image digest (`sha256`)
- memastikan image provenance
- meningkatkan reproducibility dan security deployment

---

# Buka Terminal

Gunakan shortcut:

```bash
Ctrl + Alt + T
```

---

#  Buat Folder Project

```bash
mkdir secure-wordpress
cd secure-wordpress
```

---

#  Pull Image MariaDB dari Chainguard

Pull image terlebih dahulu untuk mendapatkan digest SHA.

```bash
docker pull chainguard/mariadb:latest-dev
```

Tujuan:
- mendownload image dari repository Chainguard
- memastikan image tersedia secara lokal
- mendapatkan immutable digest SHA image

---

#  Pull Image WordPress

Karena image WordPress Chainguard tidak tersedia di Docker Hub fallback repository, digunakan official WordPress image.

```bash
docker pull wordpress:6.8
```

Tujuan:
- mendownload image WordPress
- memastikan image tersedia secara lokal
- mendapatkan immutable digest SHA image

---

#  Verifikasi Digest SHA Image

Jalankan:

```bash
docker images --digests
```

Contoh output:

```text
REPOSITORY             TAG          DIGEST
wordpress              6.8          sha256:abcd1234
chainguard/mariadb     latest-dev   sha256:efgh5678
```

Copy digest SHA masing-masing image.

---

#  2. Network Segregation (Isolasi Jaringan)

##  Tujuan
Tahap ini bertujuan untuk memisahkan trafik frontend dan backend agar WordPress dapat diakses dari luar, sementara MariaDB hanya dapat diakses secara internal melalui Docker network.

---

## Membuat dan Memverifikasi Network

Buat dua network terpisah:

```bash
docker network create frontend_net
docker network create backend_net
docker network ls
```

Pastikan terdapat:
- `frontend_net`
- `backend_net`

---

## Konfigurasi Network pada docker-compose.yml

Tambahkan konfigurasi berikut pada file `docker-compose.yml`:

```yaml
services:
  mariadb:
    networks:
      - backend_net

  wordpress:
    networks:
      - frontend_net
      - backend_net

networks:
  frontend_net:
    driver: bridge

  backend_net:
    driver: bridge
```

---

## Security Rule

MariaDB tidak boleh diekspos ke host maupun internet.

```text
❌ Jangan gunakan:
ports:
  - "3306:3306"
```

---

## Verifikasi Isolasi Network

Cek backend network:

```bash
docker network inspect backend_net
```

Pastikan:
- WordPress terhubung
- MariaDB terhubung

Cek frontend network:

```bash
docker network inspect frontend_net
```

Pastikan:
- Hanya WordPress yang terhubung

---

## Hasil Akhir

- WordPress dapat diakses dari frontend network
- WordPress dapat berkomunikasi dengan MariaDB melalui backend network
- MariaDB tidak memiliki akses langsung dari host atau internet

## 3. Security & Permission
- Non-root container (UID 65532)
- Volume ownership fixed using chown 65532:65532
- Verified using docker exec id


#  Buat Folder Volume

```bash
mkdir wordpress_data
mkdir mariadb_data
```

Folder ini digunakan untuk persistent storage container.

---

#  Set Permission untuk Non-Root User

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

## 4. Secret Management
...

#  Buat File Environment (.env)

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

#  Buat File docker-compose.yml

Buat file:

```bash
nano docker-compose.yml
```

Isi file:

```yaml
services:
  mariadb:
    image: chainguard/mariadb@sha256:PASTE_DIGEST_MARIADB_DISINI
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
    image: wordpress@sha256:PASTE_DIGEST_WORDPRESS_DISINI
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
      - "8080:80"

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

Ganti bagian:

```text
PASTE_DIGEST_MARIADB_DISINI
```

dengan digest SHA MariaDB.

Dan ganti:

```text
PASTE_DIGEST_WORDPRESS_DISINI
```

dengan digest SHA WordPress.

Contoh:

```yaml
image: wordpress@sha256:abcd1234
```

Simpan file:
- CTRL + O
- Enter
- CTRL + X

---

#  Verifikasi File Compose

```bash
cat docker-compose.yml
```

---


## 5. Resource Limitation
- CPU limit: 0.5
- Memory limit: 512MB
- Verified using docker stats
  

#  Jalankan Container

```bash
docker compose up -d
```

---

#  Verifikasi Container Berjalan

```bash
docker ps
```

Pastikan:
- `wordpress_secure` berjalan
- `mariadb_secure` berjalan

---

#  Verifikasi MariaDB Tidak Mengekspos Port

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


#  Verifikasi Log Container

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

#  Akses WordPress

Buka browser:

```text
http://localhost:8080
```

---

#  Verifikasi Resource Limitation

```bash
docker stats
```

Pastikan container memiliki batas:
- RAM: 512MB
- CPU: 0.5

---

#  Stop dan Hapus Container

```bash
docker compose down
```

# Kesimpulan

Pada step ini digunakan image WordPress dan MariaDB dari Chainguard karena lebih fokus pada security dan minimal vulnerability.

Specific version digunakan agar deployment lebih stabil dibanding latest.

Selain itu dipelajari juga penggunaan SHA digest sebagai metode deployment yang lebih immutable dan konsisten untuk production environment.
