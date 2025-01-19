
# Dokumentasi Instalasi Odoo dengan Docker

Dokumentasi ini memberikan langkah-langkah untuk mengatur Odoo dan PostgreSQL menggunakan Docker, termasuk file konfigurasi dan pengaturan jaringan.

## Prasyarat
- WSL (Windows Subsystem for Linux) <a href="https://learn.microsoft.com/en-us/windows/wsl/install">Lihat tutorial disini</a>
- Docker Desktop sudah terinstal di sistem Anda <a href="https://www.docker.com/products/docker-desktop/">Lihat tutorial disini</a>

## 1. Setup Odoo dengan Docker

### `docker-compose.yml` untuk Odoo

Buat folder dengan nama `odoo` kemudian buat file bernama `docker-compose.yml` untuk konfigurasi layanan Odoo. Ini akan mendefinisikan layanan untuk Odoo dan pengaturan jaringannya.

```yaml
version: '3.1'
services:
  odoo:
    image: odoo:16.0
    ports:
      - "8000:8069"
    command: ["--log-level=debug"]
    container_name: "odoo-16"
    restart: "unless-stopped"
    volumes:
      - ./conf:/etc/odoo
      - ./addons:/mnt/extra-addons
    networks:
      - dockernet

networks:
  dockernet:
    name: dockernet
    external: true
```

### Penjelasan:
- **Image**: Menggunakan image Docker Odoo 16 atau bisa disesuaikan dengan versi yang diinginkan.
- **Ports**: Port 8069 pada container dipetakan ke port 8000 di host.
- **Command**: Menetapkan level log ke debug untuk memudahkan troubleshooting.
- **Volumes**: Direktori lokal (`./conf` dan `./addons`) dipasang untuk menyimpan konfigurasi Odoo dan addon tambahan.
- **Networks**: Odoo terhubung ke jaringan `dockernet`.

## 2. Setup PostgreSQL dengan Docker

### `docker-compose.yml` untuk PostgreSQL

Buat folder dengan nama `setup` dan buat file bername `docker-compose.yml` lainnya untuk konfigurasi PostgreSQL dan pgAdmin.

```yaml
version: '3.8'
services:
  db:
    image: postgres
    container_name: pgdb
    restart: always
    ports:
      - '5000:5432'
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: password
    volumes:
      - local_pgdata:/var/lib/postgresql/data
    networks: 
      - dockernet

  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin4
    restart: always
    ports:
      - '8080:80'
    environment:
      PGADMIN_DEFAULT_EMAIL: root@gmail.com
      PGADMIN_DEFAULT_PASSWORD: password
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    networks:
      - dockernet

volumes:
  local_pgdata:
  pgadmin-data:

networks:
  dockernet:
    external: true
```

### Penjelasan:
- **db**: Layanan PostgreSQL dengan pengguna (`root`) dan kata sandi (`password`) yang disesuaikan.
- **pgadmin**: pgAdmin untuk mengelola PostgreSQL.
- **Networks**: Kedua layanan ini terhubung ke jaringan `dockernet`.

## 3. File Konfigurasi Odoo

Buat file konfigurasi Odoo `odoo.conf` dan letakkan di direktori `./conf` (seperti yang direferensikan dalam `docker-compose.yml`).

```ini
[options]
db_host = db
db_port = 5432
db_user = root
db_password = password
admin_passwd = admin_password
addons_path = /mnt/extra-addons
data_dir = /etc/odoo
```

### Penjelasan:
- **db_host**: Nama host untuk layanan PostgreSQL (`db`).
- **db_port**: Port untuk PostgreSQL.
- **db_user**: Pengguna PostgreSQL (`root`).
- **db_password**: Kata sandi PostgreSQL (`password`).
- **admin_passwd**: Kata sandi admin Odoo.
- **addons_path**: Path ke direktori addon tambahan.
- **data_dir**: Path ke direktori data Odoo.

## 4. Menjalankan Layanan

Untuk memulai kontainer Odoo dan PostgreSQL dengan Docker Compose, ikuti langkah-langkah berikut:

1. Masuk ke direktori tempat file `docker-compose.yml` berada.
2. Jalankan perintah berikut untuk menjalankan kedua layanan:

```bash
docker-compose up -d
```

Perintah ini akan memulai Odoo, PostgreSQL, dan pgAdmin dalam mode detasemen.

## 5. Mengakses Odoo dan pgAdmin

- **Odoo**: Buka browser dan akses `http://localhost:8000` untuk masuk ke Odoo.
- **pgAdmin**: Buka browser dan akses `http://localhost:8080` untuk masuk ke pgAdmin. Login menggunakan kredensial yang telah ditentukan dalam file `docker-compose.yml`:
  - **Email**: `root@gmail.com`
  - **Password**: `password`

## 6. Menghentikan Layanan

Untuk menghentikan semua layanan yang berjalan, jalankan perintah berikut:

```bash
docker-compose down
```

Perintah ini akan menghentikan dan menghapus semua container yang didefinisikan dalam `docker-compose.yml`.
