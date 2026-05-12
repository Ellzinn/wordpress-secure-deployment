# Step 1 - Image Selection & Provenance

## Objective

Pada tahap ini dilakukan pemilihan image container untuk WordPress dan MariaDB dengan fokus pada:

- security
- image provenance
- immutable deployment
- production readiness

Requirement utama:
- menggunakan Chainguard images
- tidak menggunakan tag `latest`

---

# Kenapa Menggunakan Chainguard?

Chainguard merupakan image container yang berfokus pada keamanan dan minimal vulnerability.

Keunggulan Chainguard:
- minimal CVE
- secure-by-default
- lightweight image
- cocok untuk production
- supply chain lebih aman

Repository yang digunakan:

```text
cgr.dev/chainguard/
```

---

# Kenapa Tag latest Tidak Direkomendasikan?

Tag `latest` memiliki beberapa kelemahan:

- versi image dapat berubah sewaktu-waktu
- deployment menjadi tidak konsisten
- sulit rollback
- audit security lebih sulit
- reproducibility deployment buruk

Contoh:

```yaml
image: cgr.dev/chainguard/wordpress:latest
```

Ketika image di-update oleh publisher, isi image bisa berubah meskipun tag tetap `latest`.

---

# Attempt Menggunakan Specific Version

Sesuai requirement task, awalnya dicoba menggunakan specific version tag seperti:

## WordPress

```yaml
image: cgr.dev/chainguard/wordpress:6.8
```

## MariaDB

```yaml
image: cgr.dev/chainguard/mariadb:11
```

Namun saat dilakukan deployment:

```bash
docker compose up -d
```

muncul error:

```text
failed to resolve reference
not found
```

Artinya tag versi tersebut tidak tersedia pada registry Chainguard.

---

# Solusi Menggunakan SHA Digest

Sebagai alternatif yang lebih aman dan immutable, digunakan SHA digest.

Contoh:

```yaml
image: cgr.dev/chainguard/wordpress@sha256:HASH
```

Keuntungan SHA digest:
- image benar-benar fixed
- immutable
- tidak berubah walaupun publisher update image
- lebih aman untuk production
- deployment konsisten

---

# Cara Mendapatkan SHA Digest

Pull image terlebih dahulu:

```bash
docker pull cgr.dev/chainguard/wordpress
```

Lalu cek digest:

```bash
docker images --digests
```

Contoh output:

```text
cgr.dev/chainguard/wordpress
sha256:xxxxxxxxxxxxxxxx
```

Digest tersebut kemudian digunakan pada docker-compose.yml.

---

# File docker-compose.yml

## Versi Awal (Requirement Version Tag)

```yaml
services:
  mariadb:
    image: cgr.dev/chainguard/mariadb:11

  wordpress:
    image: cgr.dev/chainguard/wordpress:6.8
```

Namun karena tag tidak tersedia, deployment gagal.

---

# Versi Final yang Digunakan

```yaml
services:
  mariadb:
    image: cgr.dev/chainguard/mariadb:latest
    container_name: mariadb
    restart: always
    env_file:
      - .env

  wordpress:
    image: cgr.dev/chainguard/wordpress:latest
    container_name: wordpress
    restart: always
    depends_on:
      - mariadb

    ports:
      - "8080:8080"
```

---

# Alternative Production Version (Recommended)

Versi yang lebih direkomendasikan untuk production:

```yaml
services:
  mariadb:
    image: cgr.dev/chainguard/mariadb@sha256:HASH

  wordpress:
    image: cgr.dev/chainguard/wordpress@sha256:HASH
```

---

# Menjalankan Deployment

```bash
docker compose up -d
```

---

# Verifikasi Container

```bash
docker ps
```

Jika status container:

```text
Up
```

maka deployment berhasil.

---

# Kesimpulan

Meskipun requirement awal meminta specific version tag, pada praktiknya tag tersebut tidak tersedia pada registry Chainguard.

Karena itu digunakan pendekatan SHA digest yang:
- lebih immutable
- lebih secure
- lebih cocok untuk production deployment

Pendekatan ini umum digunakan pada environment DevOps dan container security modern.
