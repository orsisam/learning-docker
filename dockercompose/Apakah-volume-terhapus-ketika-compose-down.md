# Apakah Volume akan Ikut Terhapus Ketika jalankan Perintah `docker composedown` ?

Perintah `docker compose down` secara default TIDAK MENGHAPUS Volume Data.

Secara standar, Docker memisahkan filecycle (siklus hidup) antara
Compute/Runtime (Container & Network) dengan Storage (Volume).

## Perilaku Bawaan `docker compose down`

Saat kita menjalankan perintah:

```bash
docker compose down
```

Docker Compose hanya akan menghentikan dan menghapus:

- Container
- Network internal yang dibuat oleh Compose
- Anonymous Volume (volume tanpa nama yang dibuat otomatis)

**Named Volume** (volume bernama yang dideklarasikan di bagian top-level `volumes:`)
akan tetap utuh di dalam disk server (`/var/lib/docker/volumes/...`).

Ketika kita menjalankan `docker compose up -d` kembali setelah melakukan `docker
compose down`, Docker akan otomatis menyambungkan container ke Named Volume lama
tersebut, sehingga database kita akan aman dan tidak hilang.


## Kapan Data Volume Bisa Hilang?

Data akan hilang jika kita secara eksplisit menambahkan flag `-v` atau
`--volumes` saat mematikan service:

```bash
# HATI-HATI: Perintah ini AKAN MENGHAPUS VOLUME & seluruh Data Database!
docker compose down -v
```

Flag ini sangat dihindari ketika ada di lingkungan production, kecuali ketika
kita sedang bersih-bersih (cleanup) di lingkungan lokal/development.


## Best Practice untuk Mencegah Kehilangan Data

Meskipun `docker compose down` aman, di project nyata kita tetap perlu
menggunakan beberapa lapisan perlindungan tambahan agar data tidak hilang akibat
kesalahan manusia (human error) atau kerusakan hardware:

1. **Menggunakan External Volume**
    Di file `docker-compose.yaml`, kita bisa menandai volume sebagai `external:`
    `true`. Artinya, volume tersebut harus dibuat secara manual terlebih dahulu
    di luar Compose (misal via CLI `docker volume create my_db_data`).

    ```yaml
    services:
      db:
        image: postgres:alpine
        volumes:
          - production_db_data:/var/lib/postgresql/data

    volumes:
      production_db_data:
        external: true # Compose TIDAK AKAN PERNAH menghapus volume ini meski di-down -v!

    ```

    **Efek Kemanan**: Jika ada orang yang tidak sengaja mengetik 
    `docker compose down -v`, Docker Engine akan menolak menghapus volume
    tersebut karena statusnya adalah exsternal.

2. **Menggunakan Bind Mounts ke Direktori Spesifik Host**
    Selain Named Volume, kita bisa memasangkan folder khusus di mesin host ke
    dalam container:

    ```yaml
    services:
      db:
        image: postgres:alpine
        volumes:
          - /var/app/data/postgres:/var/lib/postgresql/data # Bind mount ke folder host

    ```

    Perintah `docker compose down -v` tidak bisa menghapus folder fisik yang ada
    di `/var/app/data/postgres`.

3. **Strategi Backup Otomatis Terpisah (Automated Database Backup)**
    Di project nyata, menyimpan data di volume Docker saja tidak cukup (karena
    server fisik/SSD tetap bisa rusak). Kita bisa menerapkan **Automated
    Backup** untuk menanganinya:

    - Menjalankan *cronjob* bulanan/harian di server host yang melakukan
      `pgdump` atau `msqldump`.
    - Mengompres file dump tersebut dan secara otomatis diunggah ke Object
      Storage terpisah (seperti AWS S3, Cloudflare R2, atau S3-compatible
      storage lainnya).

