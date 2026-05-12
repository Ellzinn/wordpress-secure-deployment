
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

---

# File docker-compose.yml

```yaml
services:

  mariadb:
    image: chainguard/mariadb:11
    container_name: mariadb
    restart: always

    env_file:
      - .env

    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}

  wordpress:
    image: chainguard/wordpress:6.8
    container_name: wordpress
    restart: always

    depends_on:
      - mariadb

    env_file:
      - .env

    environment:
      WORDPRESS_DB_HOST: mariadb
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}

    ports:
      - "8080:8080"
```

---

# Menjalankan Deployment

Menjalankan container:

```bash
docker compose up -d
```

Penjelasan:
- docker compose = menjalankan multi-container
- up = menjalankan container
- -d = berjalan di background

---

# Verifikasi Container

Cek container:

```bash
docker ps
```

Jika status menunjukkan:

```text
Up
```

berarti container berhasil berjalan.

---

# Kesimpulan

Pada step ini digunakan image WordPress dan MariaDB dari Chainguard karena lebih fokus pada security dan minimal vulnerability.

Specific version digunakan agar deployment lebih stabil dibanding latest.

Selain itu dipelajari juga penggunaan SHA digest sebagai metode deployment yang lebih immutable dan konsisten untuk production environment.
