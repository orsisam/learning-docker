# Docker Compose

Docker Compose adalah toold yang digunakan untuk mendefinisikan dan menjalankan
multiple Docker Container secara bersamaan. Dengan menggunakan Docker Compose,
kita bisa menggunakan file YAML untuk melakukan konfigurasi Docker Container.
Dengan sebuah perintah, kita bisa membuat semua Docker Container dan
menjalankannya sekaligus dari file konfigurasi tersebut. Dengan cara ini kita
tidak perlu mengetikkan perintah docker create secara manual ketika ingin
membuat docker container.

> Docker Compose menyederhanakan kontrol dari keseluruhan stack aplikasi kita,
> memberikan kemudahan untuk mengatur layanan, jaringan, dan volume dalam file konfigurasi
> YAML tunggal. Kemudian dengan sebuah perintah, kita buat dan mulai semua
> layanan dari file konfigurasi.

## Fitur Docker Compose

Docker Compose memiliki fitur **Multiple Isolated Environment** dalam satu
docker host/server, atau dibilang project. Hal ini memungkinkan kita bisa
membuat banyak jenis environment untuk Docker Compose. Secara default nama
project akan menggunakan nama folder konfigurasi.

Hanya membuat container yang berubah. Jadi Docker Compose bisa mendeteksi
container mana yang harus dibuat ulang dari perubahan file konfigurasi. Misal
kita baru saja melakukan perubahan konfigurasi, maka compose sudah bisa
mengetahuinya dan membuat ulang secara otomatis.


## Kapan Menggunakan Docker Compose

Kapan Docker Compose perlu digunakan? 

- Membuat Development Environment. Ketika kita mengembangkan aplikasi, kita
  sering butuh tool-tool yang berbeda untuk tiap project. Kita bisa gunakan
  Docker Compose untuk melakukan setup-nya .
- Automated Testing. Ketika kita menggunakan tes otomatis pada aplikasi, banyak
  sekali hal yang harus kita jalankan secara manual. Docker Compose bisa
  membantu kita untuk otomatisasi proses setup.
- Deployment. Kita tidak perlu lagi start manual aplikasi kita di server. Karena
  semua lingkungan dan layanan yang dibutuhkan sudah jadi satu dalam satu project.



## Configuration File

Docker Compose menyimpan konfigurasinya dalam bentuk file YAML :
[https://yaml.org](https://yaml.org). File ini mirip dengan JSON namun lebih
sederhana. Biasanya file konfigurasi disimpan dalam file bernama
`docker-compose.yaml`, dan nama project secara default akan menggunakan nama
folder dimana file konfigurasi berada.

## Membuat Container

Sebelumnya untuk membuat container, kita selalu menggunakan perintah `docker
container create`. Namun sekarang kita bisa membuat container hanya menggunakan
configuration file di Docker Compose. Pada file yaml, kita bisa tambahkan bagian
services untuk menentukan container-nya. Dalam service tersebut, kita bisa
tentukan container name dan image untuk docker container yang akan kita buat.

### Contoh Penerapan

Untuk mencobanya kita buat folder baru dan buat file `docker-compose.yaml` di dalamnya.

**docker-compose.yaml**
```yaml
services:
    nginx-example:
        container_name: nginx-example
        image: nginx:latest
```

Setelah file konfigurasi dibuat tidak otomatis container dibuat, kita harus
membuatnya dengan menggunakan Docker Compose, yakni dengan perintah:

```bash
docker compose create
```

### Menjalankan Container

Setelah container dibuat tidak serta merta container berjalan, kita harus
menjalankannya secara manual. Bisa menggunakan perintah `docker container start`
atau dengan docker compose. Jika menggunakan compose maka perintahnya

```bash
# Pastikan anda berada di lokasi yang sama dengan file docker-compose.yaml
docker compose start
```

Sebenarnya ada beberapa cara, salah satunya adalah dengan menggunakan perintah
`docker compose up`. Lalu apa bedanya dengan perintah sebelumnya? Perbedaan
terletak pada proses di dalamnya. Jika `compose start` hanya menjalankan
container kemudian kembali ke terminal. Sedangkan `compose up` jika container
belum dibuat maka docker akan membuatnya terlibih dahulu kemudian
menjalankannya, kemudian memberikan status/log dari container yang berjalan. Hal
ini dikarenakan secara default perintah `compose up` berjalan di foreground,
agar berjalan di background kita perlu menambahkan parameter `-d` atau mode
detached. Dan kita juga bisa tetap memantau status container dengan menggunakan
perintah `docker compose logs -f` atau `--follow`.

### Melihat Container

Biasanya kita melihat container dengan menggunakan perintah `docker container
ls`. Namun menggunakan perintah ini akan menampilkan semua container, baik yang
dibuat menggunakan compose ataupun menggunakan container create. Ada perintah
yang bisa kita gunakan untuk menampilkan hanya container yang ada di file
konfigurasi docker compose dengan menggunakan perintah `docker compose ps`

**Melihat compose container**
```bash
docker compose ps
```


### Menghentikan Container

Untuk menghentikan Container dari compose file. kita menggunakan perintah:

**Menghentikan Compose Container**
```bash
docker compose stop
```

Menggunakan perintah ini hanya akan menghentikan container namun tidak
menghapusnya. Jika ingin menghapus semua container yang ada pada compose file
maka gunakan perintah.


### Menghapus Container

Jika sudah tidak butuh container yang terdapat di file konfigurasi, kita bisa
menghapusnya dengan menggunakan perintah.

```bash
# Menghapus container secara manual
docker container rm nama_container

# Menghapus menggunakan compose
docker compose down
```

Lalu apa bedanya menghapus container menggunakan perintah `rm` dan `down`. Jika
menggunakan `rm` hanya container yang terhapus. Kelemahannya adalah kita
mengetikkan nama containernya secara manual satu per satu. Sedangkan jika
menggunakan `compose down` kita akan menghapus semua container pada project dir
dan sekaligus menghapus Network dan Volume docker yang terhubung dengan container.


## Project Name

Seperti penjelasan di awal, saat kita menggunakan Docker Compose, informasi
konfigurasi Docker Compse akan disimpan dalam project. Secara default nama
project adalah nama folder lokasi file `docker-compose.yaml`. Untuk melihat
daftar project yang sedang berjalan, kita bisa menggunakan perintah

```bash
# Melihat daftar project yang sedang berjalan
docker compose ls
```

### Lebih Jauh Tentang Project Name

Project Name pada Dcocker Compose adalah nama unik yang digunakan untuk
mengelompokkan dan mengisolasi seluruh sumber daya (_network, container,
volume_, hingga _secret_) dari sebuah aplikasi.

Ketika kita menjalankan Docker Compose, semua resource yang dibuat akan diberi
awalan (*prefix*) berdasarkan project name tersebut.

#### Mengapa Project Name Penting?

Tanpa project name, Docker Compose tidak tahu container atau volume mana yang
saling memiliki keterkaitan. Project name memberikan isolasi lingkungan
(*environment isolation*). 

Dengan isolasi ini, kita bisa:
1. Menjalankan multiple instance dari proyek yang sama di satu komputer. Kita
   bisa menjalankan Laravel versi `dev`, `staging`, atau `feature-a` di satu
   mesin tanpa bentrok nama container atau network.
2. Mencegah tabrakan namas (*Name Collision*): Docker mewajibkan nama container
   bersifat unik di satu host. Prefix nama project memastikan nama container
   kita tidak saling tumpang tindih dengan proyek lain.

#### Nilai Default & Naming Convention

Secara default, Docker Compose akan menentukan project name dari nama
folder/direktori tempat berkas `docker-compose.yaml` berada.

Docker Compse juga mengubah nama project menjadi huruf kecil (lowecase) dan
menghapus karakter khusus (hanya menyisihkan huruf, angka, `-`, dan `_`).

#### Cara Mengubah/Menentukan Nama Project

Kita bisa menentukan nama project dengan beberapa method

1. **Menggunakan variable `name:` di dalam file `docker-compose.yaml`
   (Top-level-attribute)**
   Ini adalah cara terbaik dan paling bersih jika ingin nama project konsisten
   untuk semua anggota tim, tanpa terpengaruh nama direktori lokal mereka.

   ```yaml
    name: my-custom-app

    services:
      app:
        image: php:8.2-fpm
      db:
        image: mysql:8.0

   ```

2. **Menggunakan Flag `-p`/`--project-name` saat mengeksekusi CLI**
   Digunakan jika kita ingin menentukan nama proyek secara terpisah saat
   menjalankan perintah terminal.

   ```bash
   docker compose -p my-staging-app up -d

   ```

   > **Catatan**: Jika menggunakan cara ini, kita juga harus menyertakan flag
   > `-p` yang sama saat menjalankan perintah lain seperti `docker compose -p`
   > `my-staging-app down` atau `logs`.

3. **Menggunakan Variable Lingkungan / Environment Variable
   `COMPOSE_PROJECT_NAME`**
   Kita bisa menetapkan environment variable di terminal sebelum menjalankan
   perintah Compose:

   ```bash
   export COMPOSE_PROJECT_NAME=my-feature-branch
   docker compose up -d

   ```

4. **Menggunakan File `.env`**
   Kita bisa membuat file `.env` di direktori yang sama dengan
   `docker-compose.yaml` dan menambahkan kunci `COMPOSE_PROJECT_NAME`.

   ```text
   # File .env
   COMPOSE_PROJECT_NAME=pos-system-dev
   
   ```

   Docker Compose akan membaca file `.env` secara otomatis saat kita mejalankan
   `docker compose up`.

#### Matriks Hierarki / Order of Precedence

Jika kita mengkonfigurasi nama proyek menggunakan beberapa cara sekaligus.
Docker Compose menentukan nama mana yang dipakai berdasarkan urutan prioritas
(Order of Precedence) dari tertinggi ke terendah:

1. Flag CLI (`-p` / `--project-name`) -- Prioritas Utama (Menimpa semua)
2. Variable Lingkungan `COMPOSE_PROJECT_NAME` (di terminal).
3. Variable `COMPOSE_PROJECT_NAME` di file `.env`.
4. Atribut `name:` di file `docker-compose.yml`.
5. Nama folder/direktori tempat file berada - Prioritas terakhir (Fallback).


## Services in Docker Compose

Service adalah komponen/layanan dalam aplikasi (seperti web server, backend,
atau database) yang bisa diperbanyak (scale) atau diganti secara mandiri tanpa
mengganggu komponen lainnya. Service disandarkan pada kumpulan dari beberapa container.

Point-point penting dari service:

1. **Service Dijalankan oleh Container**
    - Setiap *service* dijalankan menggunakan satu atau beberapa container.
    - Konfiguraasi *service* menentukan Docker image apa yang dipakai serta
      aturan saat aplikasi berjalan. Semua *container* di dalam satu service
      yang sama akan dibuat dengan aturan yang persis sama.
2. **Wajib Didefinisikan di Berkas Compose**
    - Berkas `docker-compose.yaml` wajib memiliki bagian utama bertuliskan `serivices:`.
    - Di dalam bagian ini, kita bisa menuliskan nama-nama service (seperti `app`, `db`, atatu `nginx`) 
      beserta konfigurasi untuk masing-masing service tersebut.
3. **Bagian `build` (Opsional)**
    - Bagian ini digunakan jika kita ingin Docker Compose membuat (build) Docker
      Image sendiri berdasarkan berkas `Dockerfile`.
    - Jika tidak ingin digunakan (misalnya ingin memakai image yang sudah ada
      dari docker hub), bagian ini bisa dilewati dan berkas Compose tetap valid.
4. **Bagian `deploy` (Opsional)**
    - Bagaian ini digunakan untkuk mengatur aturan penyebaran aplikasi, seperti
      berapa banyak salinan (replication) yang ingin dijalankan dan pembatasan
      penggunaan sumberdaya (seperti RAM/CPU).
    - Jika bagian ini tidak ditulis, Docker Compose akan mengabaikannya dan
      berkas Compose tetap berjalan dengan normal.

### Atribut-atribut Service

#### `annotations`

Atribut `annotations` pada Docker Compose digunakan untuk menyematkan metadata
kustom (berupa pasangan key-value) langsung ke container atau service yang dibuat.

Fitur ini diperkenalkan agar alat (tool) dari pihat ketiga, sistem monitoring,
atau *orchestrator* dapat membaca konfigurasi khusus tanpa perlu mengubah
struktur utaama dari Docker Compose itu sendiri.

Lalu apabedanya `annotations` dengan `labels` dan `environment`?

| Atribut | Tujuan Utama | Siapa yang Mengosumsi |
| ------- | ------------ | --------------------- |
|`environment` | Variabel lingkungan (env var) di dalam OS Container | Aplikasi/skrip di dalam container (misal: DB_PASSWORD)|
|`labels` | Metadata Docker untuk objek Docker (Container, Image, Volume, Network) | Docker Engine, Traefik, Watchtower, Portainer. |
|`annotations` | Metadata tingkat lanjut/spesifik untuk platform orchestrator (seperti Kubernetes/Openshift) atau OCI (Open Container Initiative) | Kubernetes, OpenShift, CRI (Container Runtime Interface), Mesh Network. |


##### Kegunaan Utama `asnnotations`

1. **Integrasi dengan Kubernetes / OpenShift**
    Saat kita mengonversi berkas Docker Compose ke Kubernetes (misalnya
    menggunakan tool seperti `kompose` atau platform e2e), atribut `annotations`
    akan langsung dipetakan menjadi Kubernetes Annotations pada Pod atau Deployment.

    Contoh Penggunaan:
    * Mengatur Strategi *ingress* atau *load balancer*.
    * Mengonfigurasi *sidecar proxy* (seperti Istio atau Linkerd).
    * Menentukan kebijakan *security policy* khusus di cluster Kubernets.

2. **Mengonfigurasi Container Runtime (OCI)**
    Beberapa *container runtime* (seperti `runC`, `crio`, atau `containerd`)
    membaca *annotations* pada level OCI untuk mengaktifkan fitur kernel
    tertentu atau batasan keamanan khusus saat container dinyalakan.

    **Contoh Penulisan Sintaks**
    Atribut `annotations` dapat ditulis dalam bentuk **Map (Dictionary)** atau
    **List Array** di dalam blok sevice:

    **Contoh 1: Bentuk Map**
    ```yaml
    service:
        web:
            image: nginx:alpine
            annotations:
                kubernetes.io/ingress.class: "nginx"
                prometheus.io/scrape: "true"
                prometheus.io/port: "80"

    ```

    **Contoh 2: Bentuk List**
    ```yaml
    services:
        web:
            image: nginx:alpine
            annotations:
                - "kubernetes.io/ingress.class=nginx"
                - "prometheus.io/scrape=true"

    ```

##### Kapan Menggunakan Annotations?
* Gunakan `annotations` jika: Kamu menggunakan Compose sebagai definisi aplikasi
  yang nantinaya akan di-deploy ke Kubernetes/OpenShift, atau menggunakan tool
  yang secara eksplisit meminta OCI Annotations.
* Gunakan `labels` jika: Kamu hanya menggunakan Docker biasa/Docker Swarm dan
  butuh integrasi dengan alat seperti Traefik (untuk HTTPS otomatis), Watchtower
  (auto update container) atau Portainer.


#### `attach`
Ketika `attach` didefinisikan dan di-set ke `false` Compose tidak akan
mengumpulkan log service, hingga kita memintanya.

Secara default `attach` bernilai `true`.


#### `build`
`build` digunakan untuk menentukan konfigurasi build untuk membuat image
container dari source, sebagaimana didefinisikan di [Compose Build
Specification](https://docs.docker.com/reference/compose-file/build/).


#### `bkio_config`
Attribute `blkio_conig` mendefinisikan seperangkat opsi konfigurasi to mengeset
block I/O limit untuk service.

```yaml
services:
  foo:
    image: busybox
    blkio_config:
       weight: 300
       weight_device:
         - path: /dev/sda
           weight: 400
       device_read_bps:
         - path: /dev/sdb
           rate: '12mb'
       device_read_iops:
         - path: /dev/sdb
           rate: 120
       device_write_bps:
         - path: /dev/sdb
           rate: '1024k'
       device_write_iops:
         - path: /dev/sdb
           rate: 30

```

* `divice_read_bps`, `device_write_bps` -> Menentukan batasan operasi read/write
  dalam ukuran bytes per second. Tiap item di dalam list harus punya dua kunci:
  - `path`: Mendifinisikan simbolic path ke perangkat yang terpengaruh.
  - `rate`: Baik sebagai nilai integer yang mewakili angka dalam bytes atau
    sebagai string yang mengekspresikan sebuah nilai byte.

* `device_read_iops`, `device_write_iops` -> Menentukan batas operasi pada
  operasi read/write dalam hitungan per second. Tiap item pada list harus punya
  dia key ini:
  - `path`: Mendefinisikan sybolic path dari device yang terpengaruh.
  - `rate`: Nilai integer yang mewakili angka yang diperbolehkan dalam operasi
    per second.

* `weight` -> Mengubah proporsi bandwidth yang dialokasikan untuk suatu layanan
  relatif terhadap layanan lainnya. Menggunakan nilai integer antara 10 sampai
  1000, dengan 500 sebagai nilai default.

* `weight_device` -> Sesuaikan alokasi bandwidth berdasarkan perangkat. Setiap
  item dalam daftar harus memiliki dua key:
  - `path`: Menentukan path symbolic ke perangkat tertaut.
  - `weight`: Nilai integer antara 10 dan 1000.

#### `cpu_count` 
Menentukan jumlah CPU yang bisa digunakan untuk service container.

#### `cpu_percent` 
Menentukan persentase CPU yang bisa digunakan dari CPU yang tersedia.

#### `cpu_shares` 
Menentukan,sebagai nilai integer, bobot CPU relatif dari
sebuah serice container dengan container lainnya.

#### `cpu_period` 
Mengonfigurasi periode CPU CFS (Completely Fair Schedular)
ketika platform berbasis kernel Linux.

#### `cpu_quota` 
Mengonfigurasi quota CPU CFS (Completely Fair Scheduler) ketika
platform berbasis kernel Linux.

#### `cpu_rt_runtime` 
Mengofigurasi parameter alokasi CPU untuk platform dengan
dukungan penjadwalan real-time. Bisa menggunakan nilai integer menggunakan
microsecond sebagai unit atau sebagai durasi.

  ```yaml
  cpu_rt_runtime: '400ms'
  cpu_rt_runtime: '95000'

  ```

#### `cpu_rt_period` 
Mengonfigurasi parameter alokasi CPU untuk platform dengan
dukungan real-time scheduler. Nilai tersebut bisa berupa integer dengan satuan
mikrodetik atau durasi.

  ```yaml
  cpu_rt_period: '1400us'
  cpu_rt_period: '11000'

  ```

#### `cpus` 
Menentukan jumlah CPU (yang berpotensi virtual) yang akan
dialokasikan ke layanan container. Bernilai angka pecahan. Jika bernilai
`0.000` maka artinya tidak ada batasan.

Jika attribut ini didefinisikan, `cpus` harus konsisten dengan atribut `cpu`
pada [Deploy Specification](https://docs.docker.com/reference/compose-file/deploy/#cpus)

#### `cpuset` 
Menentukan CPU ekplisit yang diizinkan untuk menjalankan perintah.
Dapat berupa rentang `0-3` atau berupa list `0,1`.

#### `cap_add` 
Menentukan kapabilitas tambahan pada container, nilai berupa
string. Atribut ini digunakan untuk menambahkan hak akses khusus (Linux
Capabilities) ke dalam sebuah container.

Secara bawaan, Docker menjalankan container dengan hak akses terbatas demi
keamanan. Meskipun di dalam container kita login sebagai root, tapi user ini
tidak punya hak akses penuh ke sistem host.

Jika kita menggunakan atribut `privileged: true`, maka memberikan container
hak akses penuh tanpa batas ke kernel host, yang bisa jadi celah keamanan
serius. Solusinya ya pakai `cap_add`. Contoh:

```yaml
services:
vpn:
    image: opnvpn
    cap_add:
        - NET_ADMIN # Hanya memberi izin pengelolaan jaringan

```
Contoh Linux Capabilities yang sering digunakan:
| Capability | Kegunaan | Contoh Kasus |
| ---------- | -------- | ------------ |
| `NET_ADMIN` | Mengelola antarmuka jaringan (network interface), aturan firewall (iptables), atau routing table. | Container VPN (OpenVPN/WireGuard). Wiremock, atau alat Simulasi network |
| `SYS_TIME` | Mengubah jam dan waktu pada sistem (clock hsot) | Container server NTP atau sinkronisasi waktu khusus. |
| `SYS_PTRACE` | Memungkinkan proses untuk memantau atau men-debug proses lain (process tracing). | Tool debugging seperti `gdb`, `strace`, atau pemantau peforma APM. |
| `SYS_ADMIN` | Memberikan serangkaian akses administratif luas (seperti melakukan mount sistem berkas). | Container yang perlu melakukan `mount` drive NFS/SMB atau alat pembawa sistem default. |
| `NET_RAW` | Memungkinkan penggunaan socker RAW dan PACKET (misal: mengirim paket ICMP) | Aplikasi yang perlu menjalankan perintah `ping` atau analisis lalu lintas jaringan. |


#### `cap_drop`
`cap_drop` Punya peran yang sama dengan `cap_add` hanya saja beda tujuan,
cap_drop digunakan untuk menghapus capabilities dari container. 


#### `cgroup`
`cgroup` menentukan namespace cgroup yang akan digabungkan. Ketika tidak
ditentkukan, maka itu merupakan keputusan runtime container untuk memilih
namcespace cgroup mana yang digunakan, jika didukung.

Atribut `cgroup` pada Docker Compose digunakan untuk menentukan atau mengatur
pengelompokkan **Control Groups (cgroups)** pada sistem operasi Linux tempat
container tersebut berjalan.

Nilai dari atribut ini biasanya bernilai:

* `private` (Default): Container memiliki tampilan hierarki `cgroup`-nya sendiri
  yang terisolasi dari host.
* `host`: Container dapat melihat struktur `cgroup` milik mesin host secara
  langsung. Pilihan ini biasa dipakai jika kita menjalankan alat monitoring
  infrastruktur atau alat manajemen container di dalam container itu sendiri
  (konsep *Docker-in-Docker / System Monitoring*).

##### Apa itu cgroup (Control Groups)?
**cgroup** disini bukan atribut dari docker, melainkan fitur tingkat dasar
**Kernel Linux** yang berfungsi untuk membatasi, mencatat (accounting), dan
mengisolasi penggunaan sumber daya fisik sistem (seperti CPU, Memori/RAM, IO/
Disk, dan Network) bagi sekelompok proses.

Tanpa `cgroup`, satu container yang mengalami kebocoran memori (memory leak)
atau menjalankan proses berat secara tak terbatas (infinite loop) dapat
menghabiskan seluruh RAM/CPU komputer induk (host), yang berakibat pada *crash*
pada seluruh sistem.

Ketika Docker berjalan di Linux, Docker secara otomatis memanfaatkan `cgroup` di
latar belakang untuk membuat pembatasan isolasi tiap container.

##### Fungsi Utama cgroup dalam Docker
Secara umum, `cgroiup` memiliki 4 fungsi medasar:
1. **Resource Limiting (Pembatasan Sumber Daya):** Membatasi batas maksimum
   penggunaan memori, CPU, atau I/O disk untuk container.
2. **Prioritization (Penentuan Prioritas):** Mengatur alikasi hak akses sumber
   daya jika terjadi perebuatan (contention), misalnya memberri porsi CPU lebih
   besar pada container database dibanding container log aggregator.
3. **Accounting & Monitoring (Pencatatan & Pemantauan):** Mengukur berapa banyak
   RAM, CPU, atau ruang disk yang sudah digunakan oleh proses di dalam container
   (digunakan oleh perintah `docker stats`).
4. **Control (Pengendalian Proses)**: Memungkinkan penghentian (freeze),
   pembatalan, atau restart seluruh proses di dalam satu kelompok `cgroup`
   secara bersamaan.


##### Apa beda `cgroup` dengan Atribut `mem_limit` / `deploy.resource`
Meskipun `cgroup` adalah teknologi di balik layar yang mengatur pembatasan,
dalam penggunaan sehari-hari Docker Compose menyediakan atribut tingkat tinggi
(high-level) yang mudah dibaca untuk mengonfigurasi `cgroup`.

```yaml
services:
  db:
    image: mysql:8.0
    # Cara mudah mengonfigurasi cgroup untuk membatasi RAM & CPU:
    deploy:
      resources:
        limits:
          cpus: '1.5'
          memory: 1024M
```

Di belakang layar, Docker Compose akan menerjemahkan nilai `cpus: '1.5'` dan
`memory: 1024M` tersebut menjadi aturan cgroup di kernel Linux host.


#### `cgroup_parent`
Atribut `cgroup_parent` berbeda dengan atribut `cgroup` sebelumnya. Jika atribut
`cgroup` hanya menentukan mode cgroup namespace yang digunakan container.
Maka `cgroup_parent` digunakan untuk menentukan **parent_cgroup** kustom tempat container tersebut akan dimasukkan.

Lebih detailnya, atribut `cgroup_parent` di Docker Compose adalah konfigurasi
yang menentukan **ruang/kelompok induk (parent node)** dalam struktur hierarki
Control Groups (cgroup) di sistem operasi Linux tempat container tersebut akan
ditempatkan.

Secara bawaan, Docker menempatkan semua container di bawah satu cgroup induk
bawaan (biasanya bernama `/docker` atau `system.slice`). Dengan atribut
`cgroup_parent`, kita bisa memindahkan posisi container tersebut ke cgroup
kustom di level OS.

##### Memahami Struktukr Hierarki cgroup (Sistem Pohon)
Di dalam Linux, cgroup bekerja seperti struktur direktori atau pohon (tree):

```
/ (root cgroup)
├── system.slice
│   ├── ssh.service
│   └── nginx.service
├── user.slice
└── docker / (cgroup default bawaan Docker Engine)
    ├── container_A (Laravel)
    └── container_B (MySQL)
```

Di atas adalah contoh struktur cgroup. Hampir mirip dengan hasil yang saya dapat
ketika menjalankan perintah `systemd-cgls`. Perintah ini digunakan untuk melihat
struktur cgroup di Ubuntu.

Pada kasus Docker Compose, Jika kita tidak menentukan `cgroup_parent`, Docker
Engine akan memasukkan semua container baru ke dalam group induk bawaannya
(yaitu direktori `/docker`). Namun pada docker terbaru hierarki docker berada di
bawah `system.slice`, jadi container diserahkan ke sistem operasi.

##### Apa yang Terjadi Saat Menggunakan `cgroup_parent`?
Misalnya kita mengatur `cgroup_parent: backend_app.slice` pada file
`docker-compose.yaml`:

```yaml
services:
  app:
    image: my-laravel-app
    cgroup_parent: backend_apps.slice

  db:
    image: mysql:8.0
    cgroup_parent: backend_apps.slice

```

Struktur hierarki di kernel Linux akan berubah menjadi:

```
CGroup /:
-.slice
├─backend_apps.slice
│ ├─docker-f0f40e637f10837cfc2d40a19bca4fd3f7275c0d4e68a8880317b068047e5492.scope …
│ │ ├─720958 nginx: master process nginx -g daemon off;
│ │ ├─721104 nginx: worker process
│ │ ├─721105 nginx: worker process
│ │ ├─721106 nginx: worker process
│ │ ├─721107 nginx: worker process
│ │ ├─721108 nginx: worker process
│ │ ├─721109 nginx: worker process
│ │ ├─721110 nginx: worker process
│ │ ├─721111 nginx: worker process
│ │ ├─721112 nginx: worker process
│ │ ├─721113 nginx: worker process
│ │ ├─721114 nginx: worker process
│ │ └─721115 nginx: worker process
│ └─docker-b5c249530e5f17b085c047503c83c25c1cac6a5db3bf7078beaf045bb5689011.scope …
│   └─720966 mongod --bind_ip_all
```

##### Kapan dan Mengapa Perlu Menggunakan `cgroup_parent`
Penggunaan `cgroup_parent` sangat berguna dalam skenario administrasi server
lanjut (Advanced System Administration):

- **Pembatasan sumber Daya Tingkat Kelompok (Group-Level Rate Limiting)**
    Misalkan kita mengelola server multi-tenant (satu server dipakai oleh banyak
    tim/klien). Daripada membatasi RAM/CPU untuk tiap container satu per satu,
    kita bisa membuat cgroup di Linux bernama `klien_A.slice` dan membatasinya
    maksimum 4 GB RAM di level OS.

    Dengan memasukkan semua container milik Klien A ke `cgroup_parent:
    klien_A.slice`. total gabungan penggunaan RAM seluruh container Klien A
    tidak akan pernah melibihi 4 GB.

- **Integrasi dengan `systemd`**
    Banyak distribusi Linux modern (Ubuntu, Debian, RHEL) menggunakan
    **systemd** untuk mengelola proses. Dengan `cgroup_parent`, kita bisa
    mendaftarkan container Docker langsung ke dalam slice milik `systemd`
    (misalnya `/system.slice/my-app.slice`) agar sistem pencatatan resource OS
    dan perintah seperti `systemd-cgtop` dapat memantau container tersebut
    secara akurat.

- **Alokasi Prioritas CPU (CPU Weight/Shares)
    Kita dapat mengelompokkan aplikasi kritikal (misalnya Core Banking API) ke
    cgroup induk yang memiliki prioritas CPU tinggi, dan aplikasi latar belakang
    (misalnya Log Collector) ke cgroup induk berprioritas rendah.

##### Membuat cgroup Secara Manual di Host
Kita tahu bahwa saat menggunakan cgroup_parent kita memanfaatkan berkas
`.slice`. Ketika `cgroup_parent` ditentukan pada Docker Compose maka Docker
Engine akan membuat cgroup tersebut pada memori. Namun cara ini tidak memberikan
kemampuan berupa limitation resource untuk container pada cgroup tersebut. Agar
bisa menerapkan fitur ini kita harus membuat file konfigurasi secara manual pada
direktori `/etc/systemd/system/`.

Saat membuat berkas `.slice` secara manual di bawah direktori
`/etc/systemd/system/`, konfigurasi yang dimasukkan berada di dalam blok `[Slice]`.

Berikut adalah daftar konfigurasi utama yang tersedia untuk dimasukkan ke dalam
berkas `.slice`:

1. **Pembatasan Memori (RAM & Swap)**
    Konfigurasi ini mengontrol batas penggunaan RAM agar container/proses di
    dalam slice tidak mengahabiskan memori induk (host).
    
    - `MemoryMin=`: Menjamin alokasi RAM minimum yang tidak boleh diambil oleh
      proses lain atau page cache.
    - `MemoryLow=`: Batas perlindungan memori (*protection threshold*). Jika
      penggunaan RAM berada di bawah batas ini, kernel tidak akan mengambil
      memorinya kecuali benar-benar terdesak.
    - `MemoryHigh=`: Batas peringatan (*soft limit*). Jika penggunaan RAM
      melebihi nilai ini, proses di dalam slice akan diperlambat (*throttled*)
      dan kernel secara agresif akan membersihkan cache.
    - `MemoryMax=`: Batas keras (*hard limit*). Jika total penggunaan RAM
      melibihi nilai ini, mekanisme **OOM Killer** (*Out of Memory*) akan aktif
      untuk menghentikan (kill) proses di dalam slice.
    - `MemorySwapMax=`: Batas maksimum penggunaan memori swap.

2. **Pambatasan dan Alokasi CPU**
    Konfigurasi ini mengatur porsi pemrosesan CPU yang boleh digunakan.

    - `CPUWeight=`: Mengatur prioritas relatif pembagian CPU (range nilai: `1`
      sampai `10000`, nilai bawaan: `100`). Semakin tinggi nilainya, semakin
      besar porsi CPU yang didapat saat terjadi perebutan sumber daya.
    - `CPUQuota=`: Membatasi penggunaan total waktu CPU dalam bentuk presentase
      (*hard cap*).
      * Contoh: `CPUQuota=50%` (maksimum setengah core CPU).
      * Contoh: `CPUQuota=200%` (maksimum 2 core CPU secara penuh).
    - `AllowedCPUs=`: Menentukan core CPU fisik mana saja yang boleh digunakan
      oleh slice ini (teknik CPU affinity).
      * Contoh: `AllowedCPUs=0-2` (hanya boleh berjalan di core 0, 1, dan 2).

3. **Pembatasan I/O Disk (Baca/Tulis Drive)**
    Mengatur prioritas dan batas kecepatan akses membaca/menulis ke media
    penyimpanan (storege).

    - `IOWeight=`: Mengatur prioritas akses I/O relatif (`1` sampai `10000`,
      bawaan: `100`).
    - `IOReadBandwidthMax=`: Membatasi kecepatan membaca dari disk. Format
      penulisan: `/path/device <kecepatan>` (misal: `/dev/sdb1 10M` untuk batas
      10 MB/s).
    - `IOWriteBandwidthMax=`: Membatasi kecepatan menulis ke disk. parameter
      sama dengan `IOReadBandwidthMax`.
    - `IOReadIOPSMax=`: / `IOWriteIOPSMax=`: Membatasi jumlah operasi read/write
      per detik (IOPS).

4. **Pambatasan Jumlah Proses (Tasks)**
    Mencegah serangan Fork Bomd atau aplikasi yang membuat thread/proses liar
    tak terbatas.

    - `TaskMax=`: Membatasi jumlah maksimum proses/thread yang boleh berjalan
      secara bersamaan di dalam slice tersebut (bisa diisi angka pasti seperti
      `1000` atau presentase dari batas maksimum OS seperti `20%`).

##### **Contoh Berkas `backend_apps.slice`**

Berikut adalah contoh penerapan berkas `/etc/systemd/system/backend_apps.slice`
yang menggabungkan beberapa pembatasan di atas:

```ini
[Unit]
Description=Specialized Slize for Backend Application Groups
Documentation=man:systemd.slice(5)
Before=slices.target

[Slice]
# Pembatasan Memori
MemoryHigh=2G
MemoryMax=3G
MemorySwapMax=500M

# Pembatasan CPU
CPUWeight=200
CPUQuota=150%
AllowedCPUs=0,1

# Pembatasan I/O Disk
#IOWriteBandwidthMax=/dev/sda 50M

# Pembatasan Jumlah proses
TasksMax=500
```

Setelah itu buat berkas `docker-compose.yaml` yang memiliki atribut
`cgroup_parent=backend_apps.slice`.

```yaml
name: test-cgroup

services:
    webserver:
        image: nginx:alpine
        container_name: test_nginx
        ports:
            - "8088:80"
        cgroup_parent: backend_apps.slice

    database:
        image: mongo:latest
        container_name: test_mongo
        cgroup_parent: backend_apps.slice
```

Ini adalah hasil dari perintah `systemd-cgls`:

```
CGroup /:
-.slice
├─backend_apps.slice
│ ├─docker-23f1d3107567bd92edaa5a391bafb18d34eee133dc08da463643b2d5806196b4.scope …
│ │ └─1282801 mongod --bind_ip_all
│ └─docker-83aaaee230abc0ce23c3fa53fa3bfcf5a33b74a9cf7b7a22f47ffcd48dc4e8dd.scope …
│   ├─1282808 nginx: master process nginx -g daemon off;
│   ├─1282941 nginx: worker process
│   └─1282942 nginx: worker process
```
Hasil di atas menunjukkan bahwa container telah berjalan dalam lingkungan cgroup
kustom yang telah kita buat sebelumnya. coba akses `localhost:8088`.


#### `command`

`command` mengubah perintah/command bawaan yang dideklarasikan oleh image
container, contohnya oleh perintah `CMD`.

```yaml
command: bundle exec thin -p 300
```

Jika nilainya `null`, maka command bawaan dari image akan digunakan. Namun jika
nilai berupa `[]` (list kosong) atau `''` (string kosong), maka command bawaan
yang dideklrasikan oleh image diabaikan, dengan kata lain tidak menjalankan
perintah apapun karena di-override perintah kosong.

> **Catatan**: Tidak seperti instuksi `CMD` pada Dockerfile, command pada docker
> compose tidak berjalan dalam konteks instruksi `SHELL` seperti pada image.
> Jika perintah yang kita jalankan pada docker compose butuh konteks shell maka
> harus diikutsertakan secara jelas.
> ```
> command: /bin/sh -C 'echo "hello $$HOSTNAME"'
> ```
> 


#### `configs`

Atribut `configs` digunakan untuk menyisipkan berkas atau data konfigurasi 
(seperti `.conf`, `.json`, `.yaml`, atau file konfigurasi lainnya) ke dalam 
container tanpa perlu membuat ulang (rebuild) Docker Image atau bergantung pada
bind mount volume tradisional.

##### `configs` vs Volume?
Mengapa menggunakan `configs` dibandingkan dengan menggunakan volume?

1. **Aman & Read-Only secara Default:** Berkas yang dimasukkan lewat `configs`
   di-mount secara read-only di dalam container. Aplikasi tidak bisa secara
   tidak sengaja mengubah atau merusak file konfigurasi tersebut.
2. **Decoupling (Pemisahan Tanggung Jawab):** Memisahkan logika kode aplikasi
   (di dalam image) dari konfigurasi lingkungan (environment config).
3. **Fleksibilitas Input:** Pada standar Compose terbaru, isi konfigurasi tidak
   hanya bisa diambil dari file fisik lokal, tetapi juga bisa dituliskan
   langsung (inline) di dalam file `docker-compose.yaml`.

##### 2 Komponen Utama `configs`
Sama seperti atribut `volumes` atau `networks`, konfigurasi `configs` harus
dideklrasikan di dua tempat:

1. **Top-Level** `configs`: Tempat mendefinisikan sumber konfigurasi (source).
2. **Service-Level** `services.<service_name>.configs`: Tempat memasangkan
   konfigurasi tersebut ke service yang membutuhkan, lebih tepatnya ada di bawah
   atribut container service.


##### Sintaks `configs`

Atribut `configs` punya dua macam sintaks:

1. **Sintaks Rignkas (Short Syntax)**
    Sintaks ini digunakan jika kita hanya ingin menyisipkan file ke dalam
    container dengan direktori default (`/<nama_config>`).

    ```yaml
    services:
        webserver:
            image: nginx:alpine
            ports:
                - "8080:80"
            configs:
                - my_nginx_conf  # Cukup sebutkan nama config

    configs:
        my_nginx_conf:
            file: ./nginx.conf  # Berkas fisik di host
    ```

    > Catatan: Pada sintaks ringkas ini, berkas `nginx.conf` akan muncul di
    > dalam container pada lokasi `/my_nginx_conf`.

2. **Sintaks Lengkap (Long Syntax)**
    Sintaks ini digunakan untuk menentukan lokasi target (path), pemilik file
    (UID/GID), dan hak akses (permission) secara spesifik di dalam container.

    ```yaml
    services:
        webserver:
            image: nginx:alpine
            ports:
                - "8080:80"
            configs:
                - source: my_nginx_conf
                  target: /etc/nginx/conf.d/default.conf # Lokasi tujuan di dalam container
                  uid: "101"
                  gid: "101"
                  mode: 0440

    configs:
        my_nginx_conf:
            file: ./site.conf
    ```

    **Opsi Parameter Long Syntax**


    | Parameter | Tipe Data | Fungsi |
    | --------- | --------- | ------ |
    | `source` | String | Nama config yang dideklarasikan di bagian top-level |
    | `target` | String | Path/lokasi absolut berkas diletakkan di dalam container |
    | `uid` | String/Int | User ID pemilik berkas di dalam container |
    | `gid` | String/Int | Group ID pemilik berkas di dalam container | 
    | `mode` | Octal/Int | Hak akses file falam format okta (misal: `0644` atau `0400`). Default `044` (read-only oleh semua). |


##### Tipe Sumber pada Top-Level `configs`

Di bagian top-level configs, kita bisa mendefinisikan sumber data konfigurasi
dengan 3 cara:

1. **Tipe `file` (Berkas Lokal)**
    Mengambil isi dari file yang ada di mesin host.

    ```yaml
    configs:
        app_settings:
            file: ./configs/app.json

    ```


2. **Tipe `content` (Inline Text) -- Fitur Compose Spec Modern**
    Memungkinkan kita menuliskan isi teks konfigurasinya secara langsung di
    dalam file `docker-compose.yaml` tanpa perlu membuat file fisik di host.

    ```yaml
    configs:
        custom_nginx_page:
            content: |
                server {
                    listen 80;
                    server_name localhost;
                    location / {
                        return 200 'Hello dari Inline Compose Config';
                        add_header Content-Type text/plain;
                    }
                }

    ```

3. **Tipe `external`**
    Digunakan jika config sudah dubuat sebelumnya di luar Compose (misalnya via
    CLI `docker config create` pada Swarm cluster).

    ```yaml
    configs:
        shared_configs:
            external: true

    ```

##### Pengujian 

Kita akan buat pengujian sederhana menggunakan fitur `content` (inline config)
untuk membuktikan cara kerja `configs`.

1. Buat file `docker-compose.yaml` terlebih dahulu:
    ```yaml
    name: test-config-demo

    services:
        web:
            image: nginx:alpine
            ports:
                - "8088:80"
            configs:
                - source: inline_nginx_conf
                  target: /etc/nginx/conf.d/default.conf
                  mode: 0444

    configs:
        inline_nginx_conf:
            content: |
                server {
                  listen 80;
                  location / {
                    return 200 "Atribut 'configs' Docker Compose Berhasil Diuji!\n";
                    add_header Content-Type text/plain;
                  }
                }
    ```

2. **Jalankan Container:**
    ```bash
    docker compose up -d
    ```

3. **Uji Hasilnya:**
    ```bash
    curl http:localhost:8088
    ```

    jika berhasi akan menghasilkan output berupa string return pada config.


**?Permasalahan 1**

Ketika melakukan pengujian, ternyata terdapat sebuah keanehan. Yakni meski kita
mengubah mode config menjadi read-only (`0444`), akan tetapi ketika kita testing
dengan mengubah file `default.conf` milik nginx, file tersebut berhasil diubah.
Hal ini dikarenakan container berjalan sebagai user *root* yang memiliki hak
akses penuh.

**Solusi 1**

Cara pertama yang dilakukan adalah dengan menjadikan container berjalan dengan
user non-root. Bahkan ketika file config memiliki ownership user yang aktif,
dengan mode read-only maka file konfigurasi tersebut tidak akan bisa diubah oleh
user aktif. User bisa merubahnya karena user ini punya kapabilitas kernel
`CAP_DAC_OVERRIDE` untuk menembus isin baca/tulis.

Maka untuk skenario pertama adalah dengan login di container sebagai user
non-root dan set ownership config file menjadi user non-root dengan mode read-only.

```yaml
name: test-non-root

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:8080"  # Menggunakan port >1024 karena non-root tidak bisa bind port 80
    user: "1000:1000" # <-- MENJALANKAN CONTAINER SEBAGAI NON-ROOT USER
    configs:
      - source: inline_nginx_conf
        target: /etc/nginx/conf.d/default.conf
        uid: "1000"   # Pemilik file config (User ID)
        gid: "1000"   # Pemilik file config (Group ID)
        mode: 0444    # Read-only permission

configs:
  inline_nginx_conf:
    content: |
      server {
          listen 8080;
          location / {
              return 200 "Berjalan sebagai Non-Root User!\n";
              add_header Content-Type text/plain;
          }
      }

```

**!Error Result**

Ternyata hal ini menimbulkan permasalahan baru. Saya baca log compose dan saya
menemukan pesan error di logs:

```
web-1  | nginx: [emerg] mkdir() "/var/cache/nginx/client_temp" failed (13: Permission denied)
```

Ini adalah error access denied, ternyata user tidak bisa mengakses memory/cache,
karena folder-folder ini milik user root. AI memberikan dua solusi, **pertama**
dengan menggunakan Image resmi nginx-unprivileged (nginx dengan skenario non-root).
Dan cara yang **kedua** dengan memanfaatkan atribut `tmpfs`, yang berfungsi melakukan 
mounting folder cache dan PID ke memori sementara (`tmpfs`) agar user `1000`
memiliki hak akses tulis ke lokasi-lokasi tersebut.

Cara kedua yang saya pilih karena ada kemungkinan image lain tidak menyediakan
image resmi non-root layaknya nginx. 

```yaml
name: test-config-demo

services:
    web:
        image: nginx:alpine
        ports:
            - "8088:8080"
        user: "1000:1000"
        # Memberikan akses tulis di memori RAM untuk folder cache & PID Nginx
        tmpfs:
            - /var/cache/nginx:uid=1000,gid=1000
            - /var/run:uid=1000,gid=1000
        configs:
            - source: inline_nginx_conf
              target: /etc/nginx/conf.d/default.conf
              uid: "1000"
              gid: "1000"
              mode: 0444

configs:
    inline_nginx_conf:
        content: |
            server {
              listen 8080;
              location / {
                return 200 "Atribut 'configs' Docker Compose Berhasil Diuji!\n";
                add_header Content-Type text/plain;
              }
            }

```

Selanjutnya drop semua container sebelumnya, dan jalankan ulang compose container.

```bash
docker compose down && docker compose up -d
```

Kita coba ubah file konfigurasi nginx.

```bash
docker compose exec web /bin/sh -c "echo 'test' > /etc/nginx/conf.d/default.conf" 
```

Jika yang muncul adalah pesan error

```
sh: can't create /etc/nginx/conf.d/default.conf: Permission denied
```

Maka pengujian telah berhasil.


##### Pengetahuan Tambahan

Masih berhubungan dengan permasalahan config di atas. Yang menjadi pertanyaan
apakah aman menjalankan docker container dengan user non-root dan mengapa?

Menjalankan web server menggunakan user non-root jauh lebih aman dan merupakan
*best practice* dalam dunia container security. 

Mekanisme mounting `tmpfs` untuk folder cache/PID bukanlah sebuah celah
keamanan, melainkan langkah isolasi yang justru meningkatkan proteksi sistem
secara keseluruhan.

Berikut adalah analisis mengapa **non-root + tmpfs** jauh lebih aman
dibandingkan berjalan sebagai root:

1. **Mencegah Bahaya Container Escape (Pelarian dari Container)**
    Jika web server punya celah keamanan (misal, Remote Code Execution / RCE
    pada aplikasi web atau nginx):

    - **Jika Berjalan sebagai `root` (UID 0): Penyerang yang berhasil masuk
      melalui RCE akan langsung memiliki hak akses `root` di dalam container.
      Jika penyerang menemukan celah di level kernel Linux (kernel exploit) atau
      konfigurasi Docker yang kurang ketat, mereka dapat melakukan **Container
      Escape** untuk menguasai server induk (host) sebagai `root`.
    - **Jika Berjalan sebagai Non-Root User (UID 1000):** Penyerang hanya
      terisolasi sebagai user biasa tanpa hak istimewa. Bahkan jika penyerang
      berhasil menembus container, dampaknya terbatas di dalam container dan
      mereka tidak bisa menyentuh kernel host atau OS induk.

##### Kesimpulan Ujicoba

1. **Fungsi Utama & Keunggulan Arsitektur**
    - **Pemisahan Konfigurasi & Image (Decoupling)**: Atribut `configs`
      memungkinkan file konfigurasi disisipkan secara terpisah tanpa perlu
      rebuild Docker Image atau berhantung pada bind mount volume biasa.
    - **Mendukung Inline Content (`content`)**: Pada Compose Specification, isi
      file konfigurasi bisa ditulis langsung di dalam file `docker-compose.yaml`
      (tanpa membuat file fisik terpisah di host).
    - **Manajemen Metadata**: Mendukung pengaturan `target` (lokasi file di
      container). `uid`/`gid` (pemilik file), serta `mode` (izin akses
      POSIX/Unix okta).

2. **Temua Kritis Seputar Kemanan (`mode: 0444` & User Context)**
    |   |   |   |
    | ----------- | -------------------- | ------------------ |
    | Parameter / Kondisi | Menggunakan User `root` (Default Docker) | Menggunakan User `non-root` (`user: "1000:1000"`) |
    | **Penerapan `mode: 0444`** | Bisa ditembus / Dianulir | Bekerja Efektif |
    | **Penyebab Kernel** | `root` memiliki kapabilitas `CAP_DAC_OVERRIDE` yang mengabaikan aturan permission file POSIX | Non-root tidak memiliki kapabilitas root, sehingga kernel menolak akses read/write (`permission denied`) |
    | **Kesimpulan Proteksi** | Pengaturan `mode: 0444` menjadi sia-sia jika container tetap berjalan sebagai `root` di mode standalone. | Mengombinasikan `mode: 0444` dengan `user: non-root` menjamin file konfigurasi bersifat read-only secara mutlak |

3. **Solusi Praktis Menjalankan Non-Root Web Server (Nginx)**
    Ketika memindahkan container Nginx ke user non-root (`user: "1000:1000"`).
    Nginx standar akan mengalami error `permission denied` pada direktori cache
    dan file PID (`/var/cache/nginx` & `/var/run`).

    Dua pendekatan solusi teruji:

    - **Menggunakan Image Unpriviliged**:
        Menggunakan `nginxinc/nginx-unprivileged:alpine` yang secara native
        sudah didesain tanpa memerlukan akses root.

    - **Mounting `tmpfs` pada Container Standar:
        Menyediakan memori RAM sementara (`tmpfs`) untuk direktori kerja Nginx:

        ```yaml
        tmpfs:
            - /var/cache/nginx:uid=1000,gid=1000
            - /var/run:uid=1000,gid=1000

        ```
        - Mengapa `tmpfs` aman? `tmpfs` berbasis RAM (hilang saat restart),
          terisolasi pada direktori cache/PID saja, dan tidak memberikan akses
          tulis ke root filesystem container maupun host.


#### `container_name`

Secara default, ketika kita tidak menentukan atribut `container_name`, Docker
Compose akan membuat nama container secara otomatis menggunakan pola penamaan standar:

```
<project_name>-<service_name>-<container_name>
```

Contoh: jika nama project (folder) kita adalah `myapp`, nama serice di Compose
adalah `web`, dan ini adalah container pertama, maka nama otomatisnya menjadi
`myapp-web-1`.

Atribut `container_name` digunakan untuk menimpa (override) nama kustom secara
ekplisit untuk container yang dihasilkan oleh service tersebut.

```yaml
services:
    web:
        image: nginx:alpine
        container_name: custom_web_server

```

##### Kapan `container_name` Digunakan & Kapan Sebaiknya Dihindari?

Meskipun terlihat sederhana dan rapi, penggunaan `container_name` memiliki
dampak arsitektural yang signifikan.

1. **Kapan Sebaiknya Digunakan?**
    - **Integrasi dengan Tools / Scripts Eksternal**: ketika kita memiliki script
      automasi (misal Bash script, CI/CD pipeline, atau monitoring tool seperti
      Portainer/Prometheus) yang memanggil container berdasarkan nama spesifiknya
      secara konsisten via `docker exec custom_web_server ....` atau `docker logs
      custom_web_server`.
    - **Kemudahan Debugging Lokal**: Memudahkan pengembang untuk mengenali
      container saat menjalankan perintah `doctor ps` di terminal tanpa perlu
      mengingat atau mengetikkan nomor indeks project.

2. **Kapan Harus Dihindari?**
    - **Skalabilitas (`docker compose scale` / `deploy.replicas`)**: Nama
      container di Docker harus unik di seluruh Docker daemon host. Jika kita
      menggunakan `container_name`, kita tidak bisa melakukan scaling service
      tersebut (misalnya `docker compose up --scale web=3`) karena Docker akan
      error akibar bentrok nama (name collision).
    - **Multi-Environment / Parallel Staging**: Jika beberapa project Compose di
      server yang sama menggunakan `container_name` yang identik (misalnya `backend-api`), project kedua
      akan gagal menyala karena nama container sudah dipakai oleh project pertama.

Salah satu pemahaman yang sering keliru adalah menganggap `container_name`
dibutuhkan agar container lain bisa saling berkomunikasi. Ini tidak benar.

Di dalam jaringan internal Docker Compose (Docker Bridge Network):

1. **Service Name (`service.<name>`)**: adalah nama panggil DNS default di dalam
   jaringan Compose. Container lain cukup memanggil nama service ini (misal:
   `http://web:8080` atau `mysql://db:3306`).
2. `container_name` hanya label nama untuk Docker Daemon Host (terlihat saat
   `docker ps`).
3. `hostname` menentukan nama host sistem operasi internal di dalam container
   itu sendiri (yang muncul saat kita menjalankan perintah `hostname` di dalam container).

**Contoh Perbandingan dalam `docker-compose.yaml`**

```yaml
services:
  api:
    image: node:alpine
    container_name: my_custom_api_container  # Terlihat di 'docker ps' di host OS
    hostname: api-node-v1                    # Terlihat saat 'hostname' di dalam container
    environment:
      - DB_HOST=db                           # Menggunakan SERVICE NAME 'db' untuk koneksi DNS!

  db:
    image: postgres:alpine
    container_name: my_custom_postgres_container
```

##### Pengujian Praktis

Kita akan melakukan pengujian sederhana untuk membuktikan perilaku
`container_name` serta batasan skalabilitasnya.

1. Buat file `docker-compose.yaml`:

```yaml
name: test-container-name

services:
  app:
    image: alpine
    container_name: custom_app_node
    command: sleep infinity

```

2. Jalankan Service:

```bash
docker compose up -d
```

3. Cek Nama Container yang terbuat:

```bash
docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

Output: kita akan melihat nama containernya tepat sesuai atribut - `custom_app_node`.

4. Uji Melakukan Scaling (Gagal):

Perintah berikut akan memberikan pesan error, perintah ini digunakan untuk
memperbanyak instance service.

```bash
docker compose up -d --scale app=2

```

Perintah di atas akan menghasilkan pesan error:

```text
WARNING: The "app" service is using the custom container name "custom_app_name". Docker requires each container to have a unique name. Remove the custom name to scale the service.
```



#### `credential_spec

Atribut `credential_spec` digunakan khusus untuk manajemen identitas &
otentikasi akun terpusat, terutama saat menghubungkan container ke domain Active
Directory (AD) menggunakan mekanisme gMSA (Group Managed Service Accounts).

##### Apa Fungsi Utama `credential_spec`?

Secara default, container berjalan dengan indentitas lokal yang terisolasi dari
domain jaringan. Namun, pada aplikasi enterprise (seperti web service .NET, SQL
Server, atau apalikasi Windows/Linux yang terintegrasi AD), container sering
kali perlu mengakses sumber daya jaringan terpusat (seperti file share SMB,
database SQL Server, atau intranet) tanpa menyimpan credential
(username/password) secara hardcoded di dalam berkas atau environment variables.

Di sinilah `credential_spec` berperan:

* Mengizinkan container menggunakan **gMSA** untuk melakukan autentikasi ke
  Active Directory secara otomatis.
* Menghilangkan kebutuhan menyimpan password di dalam image atau controller.
* Mengelola rotasi password akun domain secara otomatis melalui Windows Domain Controllerl.

##### Kapan `credential_spec` Digunakan?

- **Windows Containers**: Digunakan secara luas pada Windows Container yang
  bergabung (joined) atau terhubung ke Active Directory domain untuk menjalankan
  services IIS, .NET, atau SQL Server.
- **Linux Containers**: Pada versi Docker/Compose modern, Linux Container juga
  dapat memanfaatkan gMSA (melalui kerberos/Active Directory Integration) jika
  Docker daemon host terhubung ke domain AD.

##### Opsi & Sintaks Penulisan `credential_spec`

Atribut `credential_spec` mendukung beberapa format sumber tergantung di mana
berkas spesifikasi kredensial diletakkan.

1. **Tipe `file` (Berkas Lokal di Host)**
    Mengambil file JSON `credential_spec` yang tersimpan di sistem berkas host.

    ```yaml
    services:
      app:
        image: my-enterprise-app:latest
        credential_spec:
          file: gmsa-config.json
    ```

2. **Tipe `registry` (Windows Registry - Khusus Windows Host)**
    Mengambil data `credential_spec` yang sudah didaftarkan ke dalam Windows
    Registry host (`HKLM\SOFTWARE\Microsoft\Windows
    NT\CurrentVersion\Virtualization\Containers\CredentialSpecs`).

    ```yaml
    services:
      web:
        image: mcr.microsoft.com/dotnet/framework/aspnet:4.8
        credential_spec:
          registry: my_gmsa_account
    ```

3. **Tipe `config` (Integrasi dengan Top-Level `configs` Compose)**
    Menggunakan mekanisme `configs` terpusat milik Docker Compose (pendekatan
    paling clean & modern).

    ```yaml
    services:
      api:
        image: my-backend-api
        credential_spec:
          config: gmsa_spec_config

    configs:
      gmsa_spec_config:
        file: ./policies/gmsa-credential-spec.json
    ```

##### Contoh Implementasi Lengkap (Windows / Enterprise Architecture)

Berikut contoh penggunaan `credential_spec` dalam skenario aplikasi .NET yang
terhubung ke SQL Server menggunakan gMSA:

```yaml
name: enterprise-app

services:
  web-app:
    image: mcr.microsoft.com/dotnet/aspnet:8.0-windowsservercore-ltsc2022
    ports:
      - "80:80"
    # Menghubungkan container ke gMSA Active Directory
    credential_spec:
      config: my_gmsa_spec
    security_opt:
      - "credentialspec=config:my_gmsa_spec"

configs:
  my_gmsa_spec:
    file: ./gmsa/webapp_gmsa.json
```


##### Ringkasan dan Perbandingan

| Parameter | credential_spec | secrets / env_file |
| --------- | --------------- | ------------------ |
| **Fokus Utama** | Otentikasi identitas domain/AD berbasis token/Kerberos (gMSA). | Menyimpan string teks rahasia (API Key, password statis, Sertifikat). |
| **Rotasi Password** | Ditangani otomatis oleh Active Directory Controller. | Harus diperbarui secara manual atau via CI/CD pipeline. |
| **Ketergantungan OS/Infra** | Membutuhkan infrastruktur Active Directory / Domain | Agnostik (Bisa digunakan di mana saja tanpa infrastruktur AD). |




#### `depends_on`

Atribut `depends_on` digunakan untuk mengekspresikan ketergantungan
antar-service (**service dependencies**), yang mengontrol urutan startup
(pembukaan) dan shutdown (penutupan) container oleh Docker Compose.

##### Mengapa `depends_on` Sangat Penting?

Secara default, saat kita menjalankan `docker compose up`, Docker Compose akan
menyalakan seluruh service secara paralel bersamaan.

Dalam aplikasi dunia nyata, hampir selalu ada ketergantungan urutan:

- Aplikasi **Backend/API** tidak boleh menyala sebelum Database
  (MySQL/PostgreSQL) siap meneripa koneksi.
- Service **Search Engine** (Elasticsearch) butuh waktu booting lebih laam
  sebelum service Web berjalan.

`depends_on` memastikan Docker Compose mengeksekusi container berdasarkan
hirarki urutan yang benar.

##### Bentuk Penulisan `depends_on`

1. **Short Syntax (Sintaks Ringkas)**
    Mengatur urutan eksekusi murni berdasarkan status pembentukan container.

    ```yaml
    services:
      web:
        image: my-app:latest
        depends_on:
          - db
          - redis

      db:
        image: postgres:alpine

      redis:
        image: redis:alpine
    ```

    **Jebakan Short Syntax** -> Pada short syntax hanya menjalamin container
    `db` sudah mulai menyala (started) sebelum `web` dinyalakan. Compose tidak
    menunggu sampai database di dalam container `db` tersebut benar-benar siap
    (ready/healthy) menerima koneksi port.

2. **Long Sytax dengan `condition` (Sintaks Lengkap & Direkomendasikan)**
    Untuk mengatasi kelemahan *short syntax*, Compose Specification modern
    menyediakan atribut `condition` yang dikombinasikan dengan fitur
    `healthcheck`.

    ```yaml
    services:
      web:
        image: my-app:latest
        ports:
          - "8080:8080"
        depends_on:
          db:
            condition: service_healthy # Menunggu sampai db berstatus HEALTHY
          redis:
            condition: service_started # Cukup menunggu redis menyala

      db:
        image: postgres:alpine
        environment:
          POSTGRES_PASSWORD: secretpassword
        # Mendefinisikan pengujian kesehatan database
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres"]
          interval: 5s
          timeout: 5s
          retries: 5

      redis:
        image: redis:alpine

    ```

    **Pilihan Nilai `condition` pada Long Syntax**

    | Nilai `condition` | Perilaku Docker Compose |
    | --------------- | ----------------------- |
    | `service_started` | Menganggap ketergantungan terpenuhi segera setelah |
    |                 | container target berhasil dibuat dan dinyalakan (sama |
    |                 | seperti short syntax). |
    | `service_healthy` | Menganggap ketergantungan terpenuhi hanya setelah |
    |                 | perintah `healthcheck` pada container target |
    |                 | mengembalikan status healthy. |
    | `service_completed_successfully` | Menganggap ketergantungan terpenuhi |
    |                 | jika container target selesai berjalan dengan  |
    |                 | status exit 0 (Sangat berguna untuk container |
    |                 | migration/seeder yang berjalan sekali lalu mati). | 


    - **Kasus Khusus: Database Migration / Seeder (`service_completed_successfully`)** 
        Salah satu pola arsitektur paling populer menggunakan
        `service_completed_successfully` adalah menjalankan skrip migrasi
        database otomatis sebelum aplikasi web menyala:

        ```yaml
        services:
          app:
            image: my-laravel-app
            ports:
              - "8000:8000"
            depends_on:
              db:
                condition: service_healthy
              migration:
                condition: service_completed_successfully # Tunggu sampai migrasi selesai!

          migration:
            image: my-laravel-app
            command: php artisan migrate --force
            depends_on:
              db:
                condition: service_healthy

          db:
            image: mysql:8.0
            environment:
              MYSQL_ROOT_PASSWORD: secret
              MYSQL_DATABASE: myapp
            healthcheck:
              test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
              interval: 3s
              timeout: 3s
              retries: 10
        ```

        **Urutan Eksekusi Otomatis Compose**:
        1. Menyalakan `db`.
        2. Menunggu `db` hingga `mysqladmin ping` berhasil (service_healthy).
        3. Menyalakan `migration` (`php artisan migrate`).
        4. Menunggu perintah migrasi selesai dengan sukses (service_completed_successfully).
        5. Menyalakan `app` utama.

##### Dampak `depends_on` pada Perintah CLI

- `docker compose up <service>`: Jika kita hanya menjalankan `docker compose up app`. 
  Compose secara otomatis akan ikut menyalakan `db` dan `migration` karena
  terdartar di `depends_on`.
- `docker compose stop` / `down`: Urutan pemadaman (shutdown) dilakukan secara
  kebalikan (reverse order). Service `app` akan dimatikan terlebih dahulu, baru
  kemudian `db` dimatikan.

##### Ringkasan

1. **Selalu Gunakan Long Syntax + `healthcheck`**: Jangan mengandalkan *short syntax* untuk
   service kritis seperti Database/Message Queue. Selalu padukan `condition: service healthy`
   dengan `healthcheck`.
2. **Aplikasi Tetap Harus Punya Retry Logic**: Jangan menggantungkan 100%
   ketahanan aplikasi pada `depends_on`. Kode aplikasi (backend) sebaiknya tetap
   memiliki logika reconnect/koneksi ulang ke database jika sewaktu-waktu
   database restart di tengah jalan.



