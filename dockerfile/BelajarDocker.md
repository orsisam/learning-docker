# Belajar Docker

## 1. Docker Dasar

### Docker Images

Docker image diibaratkan adalah installer bagi docker container.

**Perintah-perintah docker image**

```bash
docker image ls  # Untuk melihat daftar image yang sudah didownload

docker image pull <nama_image>:<tag> # Untuk mengunduh image baru, jika tag
                                    #tidak didefinisikan maka akan menggunakan latest sebagai nilai default.

# Untuk menghapus image
docker image rm <image_name>:<tag>
```

### Docker Container

Jika Docker Image seperti installer aplikasi, maka Docker Container mirip
seperti aplikasi hasil installernya. Satu Docker Image bisa digunakan untuk
membuat beberapa Docker Container, asalkan nama containernya berbeda.

> Jika kita sudah membuat Docker Container, maka docker image yang kita gunakan
> tidak bisa dihapus, hal ini dikerenakan sebenarnya Docker Container tidak
> mengcopy isi Docker Image, tapi hanya menggunakan isinya saja.


**Perintah-perintah di dalam Docker Container**

```bash
# Melihat container apa saja yang sedang berjalan.
docker container ls

# Melihat semua container, baik yang sedangn running atau tidak.
docker container ls -a

# Membuat container
docker container create --name <container_name> <image_name>:<tag>
```

### Container Exec

Saat kita membuat container, aplikasi yang terdapat di dalam container hanya
bisa diakses dari dalam container. Oleh karena itu kita perlu masuk ke dalam
container itu sendiri. Untuk masuk ke dalam container, kita bisa menggunakan
fitur *Container Exec*.

Sebenarnya Container Exec bukan digukanan untuk masuk ke dalam container,
melainkan untuk menjalankan perintah di dalam container. Namun dengan
menjalankan bash, kita bisa mengakses command shell pada linux.

Perintah yang digunakan untuk masuk ke dalam container:

```bash
docker container exec -i -t <containerId/container_name> /bin/bash

```

* `-i` adalah argument interaktif, menjaga input tetpap aktif.
* `-t` adalah argument untuk alokasi pseudo-TTY (terminal akses)
* `/bin/bash` contoh kode program yang terdapat di dalam container (karena
  kebanyakan container dibuild dari linux).


### Container Port

Saat menjalankan container, container tersebut terisolasi di dalam Docker.
Artinya sistem Host (mis. Komputer kita) tidak bisa mengakses aplikasi yang ada
di dalam container secara langsung, salah satu caranya adalah menggunakan
Container Exec untuk masuk ke dalam container.

Biasanya, sebuah aplikasi berjalan pada port tertentu, misal saat kita
menjalankan aplikasi Redis, dia berjalan pada port 6379, kita bisa melihat port
apa yang digunakan ketika melihat semua daftar container. Namun ini adalah port
internal yang tidak bisa diakses dari luar.

#### Port Forwarding

Docker punya kemampuan untuk melakukan port forwarding, yaitu meneruskan sebuah
port yang terdapat di sistem Host nya ke dalam Docker Container. Cara ini cocok
jika kita ingin mengekspos port yang terdapat di container ke luar melalui
sistem Host nya.

Jika ingin membuat port forwarding pada container yang sudah jadi maka harus
menghapus container yang sudah ada dan kemudian membuat baru dengan perintah
berikut.

```bash
docker container create --name <container_name> --publish
<porthost>:<port_container> <image>:<tag>

# Contoh
docker container create --name contohnginx --publish 8090:80 nginx:latest
```

### Container Environment Variable

Saat membuat aplikasi, menggunakan *environment variable* adalah salah satu
teknik agar konfigurasi aplikasi diubah secara dinamis. Dengan menggunakan
environment variable, kita bisa mengubah-ubah konfigurasi aplikasi, tanpa harus
mengubah kode aplikasi. Docker container memiliki parameter yang yang bisa kita
gunakan untuk mengirim environment variable ke aplikasi yang terdapat di dalam container.

Untuk menambahkan environment variable gunakan perintah berikut (contoh
menggunakan mongodb):

```bash
docker container create --name contohmongo --publish 27017:27017 --env
MONGO_INITDB_ROOT_USERNAME=orsisam --env MONGO_INITDB_ROOT_PASSWORD=password
mongo:latest
```

Kita bisa melihat konfigurasi environment variable dari tiap aplikasi pada situs
docker hub.


### Container Stats

Saat menjalankan beberapa container, di sistem Host, penggunaan resource seperti
CPU dan Memory hanya terlihat digunakan oleh Docker saja. Kadang kita ingin
melihat  detail dari penggunaan resource untuk tiap container. Misal saja,
kenapa docker menggunakan resource yang sangat tinggi, kita perlu melihat
container mana yang menggunakan resource tinggi. Untungnya docker memiliki
kemampuan untuk melihat penggunaan resource untuk tiap container yang sedang
berjalan. Kita bisa gunakan perintah.

```bash
docker container stats
```


### Container Resource Limit

Saat membuat container, secara default dia akan menggunakan semua CPU dan Memory
yang diberikan ke Docker (Mac dan Windows), dan akan menggunakan semua CPU dan
Memory yang tersedia di sistem Host (Linux). Jika terjadi kesalahan, misal
container terlalu banya memakan CPU dan Memory, maka bisa berdampak terhadap
performa container lain, atau bahkan ke sistem host. Oleh karena itu ada baiknya
ketika kita membuat container, kita memberikan resource limit terhadap containernya.

Saat membuat container, kita bisa menentukan jumlah memory yang bisa digunakan
oleh container, dengan menggunakan perintah `--memory` diikuti dengan angka
memory yang diperbolehkan untuk digunakan.

Kita bisa menambahkan ukuran dalam bentuk **b** (bytes), **k** (kilo bytes),
**m** (mega bytes), atau **g** (giga bytes), misal 100m artinya 100 mega bytes.

Selain mengatur memory, kita bisa menentukan berapa jumlah CPU yang bisa
digunakan oleh container dengan parameter `--cpus`. Jika misal kita set dengan
nilai 1.5 artinya container bisa menggunakan satu dan setengah CPU core.

Contoh:

```bash
docker container create --name smallnginx --publish 8091:80 --memory 100m --cpus
0.5 nginx:latest
```


### Bind Mounts

Bind Mounts merupakan kemampuan melakukan mounting (sharing) file atau folder
yang terdapat di sistem host ke container yang terdapat di docker. Fitur ini
sangat berguna ketika kita ingin mengirim konfigursi dari luar container, atau
juga ketika menyimpan data yang dibuat aplikasi di dalam container ke dalam
folder di sistem host (Mis. pada aplikasi database, jadi ketika container
dihapus data tetap ada). Jika file atau folder tidak ada di sistem host, secara
otomatis akan dibuatkan oleh Docker. Untuk melakukan mounting, kita bisa
menggunakan parameter `--mount` ketika membuat container. Isi dari parameter ini
punya aturan tersendiri.

**Parameter Mount**

| Parameter | Keterangan |
|-----------|------------|
| type | Tipe mount, bind, atau volume |
| source | Lokasi file atau folder di sistem host |
| destination | Lokasi file atau folder di container |
| readonly | Jika ada, maka file atau folder hanya bisa dibaca di container, tidak bisa ditulis |


Untuk membuat container dengan mounting sintaksnya seperti berikut:

```bash
docker container create --name <container_name> --mount
"type=bind,source=folder,destination=folder,readonly" <image>:<tag>
```

Contoh:

```bash
docker container create --name mongodata \
--mount "type=bind,source=/home/orsisam/.dockerdata/mount/mongodata,destination=/data/db" \
--publish 27018:27018 \
--env MONGO_INITDB_ROOT_USERNAME=orsisam \
--env MONGO_INITDB_ROOT_PASSWORD=password \
mongo:latest
```

> **Catatan**: karena kita udah buat storage khusus untuk data dari database,
> jadi ketika container tersebut dihapus maka data dari database yang sudah kita
> buat tidak akan hilang. Kita hanya perlu membuat container lagi degan perintah
> yang sama (dengan mount folder yang sama), sehingga data tidak akan hilang.



### Docker Volume

Fitur Bind Mount sudah ada sejak Docker versi awal, di versi  terbaru
direkomendasikan menggunakan *Docker Volume*. Docker Volume mirip dengan Bind
Mounts, bedanya adalah terdapat managament Volume, dimana kita bisa membuat
Volume, melihat daftar Volume dan menghapus Volume. Volume sendiri bisa dianggap
storage yang digunakan untuk menyimpan data, bedanya dengan Bind Mounts, pada
bind mount, data disimpan pada sistem host, sedangkan pada volume, data diatur
oleh Docker.

Saat kita membuat container, secara default semua data container disimpan di
dalam volume. Jika kita mencoba melihat docker volume, kita akan lihat bahwa ada
banyak volume yang sudah terbuat, walaupun kita belum pernah membuatnya sama
sekali. Kita bisa gunakan perintah berikut untuk melihat daftar volume.

```bash
docker volume ls

# Membuat volume baru
docker volume create <volume_name>
```

Volume yang tidak digunakan oleh container bisa kita hapus, tapi jika volume
digunakan oleh container, maka tidak bisa dihapus sampai containernya dihapus.

Untuk menghapus volume, kita bisa gunakan perintah:

```bash
docker volume rm <volume_name>
```

Volume yang sudah kita buat, bisa kita gunakan di container. Keuntungan
menggunakan volume adalah, jika kita hapus container, data akan tetap aman di
volume. Cara menggunakan volume di container sama dengan menggunakan bind mount,
kita bisa menggunakan parameter `--mount` namun dengan menggunakan type volume
dan source berupa nama volume.

Contoh: 

```bash
# Buat volume
docker volume create mongodata

# Lanjut buat container dengan volume
docker container create --name mongovolume \
--mount "type=volume,source=mongodata,destination=/data/db" \
--publish 27019:27017 \
--env MONGO_INITDB_ROOT_USERNAME=orsisam \
--env MONGO_INITDB_ROOT_PASSWORD=password \
mongo:latest
```

#### Backup Volume

Docker tidak menyediakan fitur backup volume sampai saat ini, tidak ada cara
otomatis melakukan backup volume yang sudah kita buat. Namun kita bisa
memanfaatkan container untuk melakukan backup data yang ada di dalam volume ke
dalam archive seperti zip dan tar.gz.

Ada beberapa cara untuk melakukan backup volume, namun sebelum melakukan backup
Pastikan container yang menggunakan volume dalam keadaan mati. Buat container
baru dengan dua mount, volume yang ingin kita backup dan bind mount folder dari
sistem host. Lakukan backup menggunakan container dengan cara meng-archive isi
volume dan disimpan di bind mount folder. Isi file backup sekarang ada di folder
sistem host. Hapus container yang kita gunakan untuk melakukan backup.

Contoh langakah-langkah untuk melakukan backup:

```bash
# Hentikan dahulu container yang akan dilakukan backup
docker container stop mongovolume

# Buat container baru untuk backup data, disini kita menggunakan
# image nginx (sebenarnya bisa menggunakan image apapun)
docker container create \
--name nginxbackup \
--mount "type=bind,source=/home/orsisam/Documents/Learn/Docker/backup,destination=/backup" \
--mount "type=volume,source=mongodata,destination=/data" \
nginx:latest

# Jalankan container nginxbackup yang baru saja kita buat.
docker container start nginxbackup

# Masuk ke terminal container nginxbackup menggunakan exec
docker container exec -i -t ngincbackup /bin/bash

# Pada terminal container nginxbackup lakukan perintah backup ataupun copy
tar cvf /backup/backup.tar.gz /data

# Matikan container backup
docker container stop nginxbackup
docker container rm nginxbackup

# Jalankan kembali container mongovolume
docker container start mongovolume

```

Cara di atas memang cukup banyak dan kompleks, karena kita harus melakuakan tiap
langkah secara manual dan menuliskan baris kodenya satu demi satu.

Kita bisa memanfaatkan perintah `run` untuk menjalankan perintah di container
dan gunakan parameter `--rm` untuk menghapus secara otomatis container setelah
perintahnya selesai berjalan. Image yang akan digunakan pada perintah run ini
kita akan menggunakan ubuntu docker image, hal ini dikarenakan kita butuh
container yang tidak jalan begitu dibuat, tidak seperti nginx yang akan terus
jalan.

```bash
docker container run --rm --name ubuntu24 \
--mount "type=bind,source=/home/orsisam/Documents/Learn/Docker/udemy/backup,destination=/backup" \
--mount "type=volume,source=mongodata,destination=/data" \
ubuntu:24.04 \
tar cvf /backup/backup-run.tar.gz /data
```

### Restore Volume

Ketika telah membuat backup dari volume lama, tentunya kita juga perlu untuk
melakukan restore data jika diperlukan. Ada beberapa tahapan melakukan restore.

**Tahapan Melakukan Restore**
1. Buat volume baru untuk lokasi restore data backup.
2. Buat container baru dengan dua mount, volume baru untuk restore backup, dan bind mount folder dari sistem host yasng berisi file backup.
3. Lakukan restore menggunakan container dengan cara meng-extract isi backup file ke dalam volume.
4. Setelah isi file backup sudah di-restore ke volume. Hapus container sementara
   yang barusan kita buat untuk melakukan restore.
5. Volume baru yang berisi file backup siap digunakan oleh container baru.

Untuk menjalankan langkah-langkah tersebut kita cukup membuat sebuah baris
command run.

```bash
# Buat volume baru
docker volume create mongodatabackup

# Jalankan proses restore sesuai dengan tahapan di atas.
docker container run --rm \
--name ubuntu_restore \
--mount "type=bind,source=/home/orsisam/Documents/Learn/Docker/udemy/backup,destination=/backup" \
--mont "type=volume,source=mongodatabackup,destination=/data" \
ubuntu:24.04 \
/bin/bash -c "cd /data && tar xvf /backup/backup.tar.gz --strip 1"
```


### Docker Network

Saat kita membuat container di docker , secara default container akan saling
terisolasi satu sama lain, jadi jika kita mencoba memanggil antar container,
kita tidak akan bisa melakukannya. Docker memiliki fitur Network yang bisa
digunakan untuk membuat jaringan di dalam Docker. Dengan menggunakan Network,
kita bisa mengkoneksikan container dengan container lain dalam satu Network yang
sama. Ketika beberapa container pada satu jaringan yang sama, maka secara
otomatis container tersebut bisa saling berkomunikasi.

Saat kita membuat Network di Docker, kita perlu menentukan driver mana yang akan
kita gunakan.

#### Network Driver

* `bridge` (Default)
     - **Cara Kerja**: Docker membuat virtual switch (jembatan) di dalam komputer.
       Container yang terhubung ke jarignan bridge mendapatkan alamat IP private
       tersendiri. Container yang terkoneksi pada bridge network yang sama bisa
       saling berkomunikasi.
     - **Penggunaan**: Digunakan untuk container yang berjalan di satu host yang
       sama. Agar bisa diakses dari luar, port container harus di-publish (`-p
       8080:80`)
* `host`
    - **Cara Kerja**: Menghilangkan isolasi jaringan antara container dan
      komputer host. Container langsung menggunakan alamat IP dan port dari OS
      host. Tipe driver host hanya bisa jalan di Docker Linux, tidak bisa
      digunakan di Mac atau Windows.
    - **Penggunaan**: Memberikan performa I/O jaringan tercepat, cocok untuk
      aplikasi yang membutuhkan throghput sangat tinggi.
* `none`
    - **Cara Kerja**: Mematikan semua fungsi jaringan pada container. Container
      hanya memiliki loopback interface(`127.0.0.1`).
    - **Penggunaan**: Cocok untuk tugas yang membutuhkan keamanan tinggi
      (misalnya menjalankan script/batch rahasia yang tidak boleh menyentuh internet)
* `overlay`
    - **Cara Kerja**: Menghubungkan beberapa daemon Docker di mesin
      physical/host yang berbeda agar bisa saling terhubung dalam satu jaringan.
    - **Penggunaan**: Digunakan pada arsitektur multi-host seperti Docker Swarm
      atau kluster terdistribusi.


```bash
# Melihat daftar network pada docker.
docker network ls

# default syntax
# buat network baru
# Jika driver tidak didefiniskan maka akan menggunakan default driver (bridge)
docker network create --driver <drivername> <network_name>

# Hapus network
docker network rm <network_name>
```

**Contoh Penerapan**

```bash
# Buat network dengan nama "mongonetwork"
docker network create --driver bridge mongonetwork

# Buat mongodb container yang terhubung ke network
docker container create --name mongodb \
--network mongonetwork \
--env MONGO_INITDB_ROOT_USERNAME=admin \
--env MONGO_INITDB_ROOT_PASSWORD=123456 \
mongo:latest

# Buat mongodbexpress container (mongodb client)
docker container create \
--name mongoexpress \
--network mongonetwork \
--publish 8088:8081 \
--env ME_CONFIG_MONGODB_URL="mongodb://admin:admin@mongodb:27017" \
--env ME_CONFIG_BASICAUTH_USERNAME='admin' \
--env ME_CONFIG_BASICAUTH_PASSWORD='admin' \
mongo-express:latest

# Jalankan mongodb dan mongo-express
docker container start mongodb
docker container start mongoexpress

```

Jika diperlukan, kita bisa juga menghapus konesksi container ke sebuah network
atau menghubungkan kembali sebuah container ke network.
Caranya adalah demikian.

```bash
# Format perintah
docker network disconnect <network_name> <container_name>

# Format perintah connect
docker network connect <network_name> <container_name>

# Contoh peritah
docker network disconnect mongonetwork mongodb
```

Lalu bagaimana kita bisa mengetahui suatu network terhubung dengan container
mana saja.
1. Dengan menggunakan *network inspect*, dengan ini kita bisa melihat container
   mana saja yang terhubung dengan network yang kita masukkan pada perintah:
    ```bash
    docker network inspect <network_name>
    ```
2. Cara yang kedua, dengan melakukan inspect ke container. Cara ini adalah
   kebalikan dari cara sebelumnya. Degan cara ini kita mengetahui network mana
   saja yang terhubung ke container yang kita inspect.
   ```bash
   docker inspect <container_name>/<container_id>
   ```


### Docker Inspect

Setelah kita mengunduh image, atau membuat network, volume dan container.
Terkadang kita ingin melihat detail dari tiap hal tersebut. Misal kita ingin
melihat detail dari image, perintah apa yang digunakan oleh image tersebut,
environment variable apa yang digunakan, atau port apa yang digunakan? Misalkan
saja, kita ingin melihat detail dari container, seperti volumen, environment
variable, port forwarding, dan network.

Docker punya fitur yang bernama *inspect*, yang bisa digunakan di image,
container, volume, dan network. Dengan fitur ini kita bisa melihat detail dari
tiap hal yang ada di Docker.

**Cara menggunakan Inspect**
```bash
# Untuk melihat detail dari image
docker image inspect <image_name>

# Untuk melihat detail container
docker container inspect <container_name> 

# Untuk melihat detail volume
docker volume inspect <volume_name>

# Untuk melihat detail network
docker network inspect <network_name>
```



### Docker Prune

Saat menggunakan Docker, kita perlu untuk membersihkan hal-hal yang sudah tidak
digunakan lagi di Docker, misal container yang sudah dihentikan, image yang
tidak digunakan oleh container, atau volume yang tidak digunakan oleh container.

Fitur untuk membersihkan secara otomatis di Docker bernama prune. Hampir semua
perintah di Docker mendukung prune.

**Perintah-perintah Prune**
```bash
# Untuk menghapus semua container yang sudah distop
docker container prune

# Untuk menghapus semua image yang tidak digunakan container
docker image prune

# Untuk menghapus semua network yang tidak digunakan container
docker network prune

# Untuk menghapus semua volume yang tidak digunakan container
docker volume prune

# Jika ingin menhapus container, network, image yang tidak digunakan
docker system prune
```


---

## Dockerfile

Sebelumnya kita menggunakan image yang tersedia di docker hub. Tapi sebenarnya
kita juga bisa membuat docker image sendiri. Pembuatan Docker Image bisa
dilakukan menggunakan instruksi yang kita simpan di dalam Dockerfile. Nanti
Dockerfile tersebut akan dieksekusi sebagai perintah untuk membuat Docker Image.

#### Docker Build

Kita bisa menggunakan perintah `docker build` untuk membuat Docker Image dari
Dockerfile. Saat membuat Docker Image dengan docker build, nama image secara
otomatis akan dibuat random, namun kita bisa mengubahnya dengan menambahkan
nama/tag pada image dengan menggunakan parameter tambahan `-t`. Misal berikut
contoh cara menggunakan docker build:

```bash
docker build -t orsisam/app:1.0.0 folder-dockerfile

# Dari satu docker image kita bisa membuat 2 nama yang berbeda
docker build -t orsisam/app.1.0.0 -t orsisam/app:latest folder-dockerfile
```


### Dockerfile Format

Dockerfile biasanya dibuat dalam sebuah file dengan nama `Dockerfile`, tidak
memiliki extension apapun. Walaupun sebenarnya bisa saja kita membuat dengan
nama lain, namun direkomendaasikan menggunakan nama Dockerfile.

#### Instruction Format

Secara sederhana berikut format untuk Dockerfile

```
# Komentar
INSTRUCTION arguments
```

* Tanda `#` digunakan untuk menambah komentar, kode dalam baris ini secara
  otomatis dianggap sebagai komentar dan tidak dieksekusi.
* `INSTRUCTION` adalah perintah yang digunakan Dockerfile, ada banyak perintah
  yang tersedia, dan penulisan perintahnya case insensitive, sehingga kita bisa
  gunakan huruf besar atau kecil. Namun direkomendasikan menggunakan UPPER CASE.
* `arguments` adalah data argument untuk instruction, yang menyesuaikan dengan
  instruction yang digunakan.


### From Instruction

Ketika kita membuat Docker Image, biasanya perintah pertama adalah melakukan
build stage dengan instruksi FROM. FROM digunakan untuk membuat build stage dari
image yang kita tentukan. Jarang sekali kita akan membuat Docker Image dari awal
mula (kosongan), biasanya kita akan membuat Docker Image dari image yang sudah
tersedia. Untuk menggunakan FROM, kita bisa gunakan perintah:

```dockerfile
FROM image:version
```

Contoh penerapan dockerfile. Buat dockerfile di folder **from**.

```dockerfile
FROM alpine:3
```

Untuk membuat image kita menggunakan perintah `docker build`.

```bash
# Format perintah
docker build -t <dockerhub_username>/<image-name>:<image-tag>? <folder-dockerfile>

docker build -t orsisam/from:1.0.0 from

# Periksa image yang sudah jadi
docker image ls
docker image inspect orsisam/from:1.0.0
```


### Run Instruction

RUN adalah sebuah instruksi untuk mengeksekusi perintah di dalam image pada saat
build stage. Hasi dari perintah RUN akan di-*commit* dalam perubahan image
tersebut, jadi perintah RUN akan dieksekusi pada saat proses docker build saja,
setelah menjadi Docker Image, perintah tersebut tidak akan dijalankan lagi. Jadi
ketika kita menjalankan Docker Container dari Image tersebut, maka perintah RUN
sudah tidak dijalankan.

Perintah RUN biasa digunakan untuk menginstall aplikasi ke dalam image atau
memodifikasi konfigurasi bawaan aplikasi.

**Format Perintah**

Perintah RUN punya punya 2 format seperti berikut:

```bash
# Format 1
RUN <command>

# Format 2
RUN ["executable", "argument", "..."]
```

**Contoh Penerapan**

```dockerfile
FROM alpine:latest

RUN mkdir hello
RUN echo "Hello Docker" > "hello/world.txt"
RUN cat "hello/world.txt"
```

Lalu build dockerfile tersebut dengan menggunakan perintah

```bash
docker build --progress=plain -t orsisam/run:1.0 run
```

Maksud dari perintah build di atas adalah buat image dengan nama orsisam/run
dengan tag 1.0 di sedangkan `run` paling akhir adalah folder dimana dockerfile
berada. Kita juga memberikan argument `--progress=plain` yang berfungsi untuk
melihat hasil dari perintah secara detail, terutama perintah terakhir `cat
"hello/world.txt"`. Karena secara default docker tidak memberikan detail dari
hasil running.

Kita juga bisa memanfaatkan argument `--no-cache` jika tidak ingin melakukan
cache saat build maka berikan parameter ini saat build. Karena ketika kita
melakukan build ulang image, jika perintah RUN tidak ada perubahan maka docker
akan menggunakan cache dari perintah yang dijalankan sebelumnya.

![docker run](run/run.png) 



### Command Instruction

CMD atau Command ini merupakan instruksi yang digunakan ketika Docker Container
berjalan. Perintah CMD tidak dijalankan ketika proses build tapi dijalankan
ketika Docker Container berjalan.

Di dalam Dockerfile, kita tidak bisa menambah lebih dari satu perintah CMD, jika
kita menambahkan lebih dari satu instruksi CMD, maka instruksi yang dijalankan
adalah instruksi CMD terakhir pada dockerfile.

#### Format Instruksi Command

Instruksi CMD memiliki 3 jenis format

```bash
# Format 1
CMD <command> <param1> <param2>

# Format 2
CMD ["executable", "param1", "param2"]

# Format 3 - menggunakan executable ENTRY POINT
CMD ["param1", "param2"]
```


**Contoh penerapan CMD**

```dockerfile
FROM alpine:latest

RUN mkdir hello
RUN echo "Hello Docker" > "hello/world.txt"

CMD ["cat", "hello/world.txt"]
```

Kemudian untuk memastikan kalau perintah command tersebut dijalankan, langkah
pertama yang harus dilakukan adalah melakukan build image dari Dockerfile yang
sudah kita buat sebelumnya. Sejalanjutnya adalah dengan membuat container baru
berdasarkan image yasng baru saja kita build.

Langkah ke-3 adalah jalankan container tersebut (ketika container dijalankan
maka akan langsung kembali ke keadaan stop, karena perintah command telah
selesai). Untuk memeriksanya apakah command berhasil dijalankan kita bisa lihat
dari log container.

```bash
# Build image command
docker build -t orsisam/command:1.0 command

# Buat container dari image orsisam/command:1.0
docker container create --name command orsisam/command:1.0

# Jalankan container
docker start command

# Baca log dari container
docker container logs command

# Jika berhasil maka akan menampilkan teks "Hello Docker"
```


### Label Instruction

Instruksi LABEL merupakan instruksi yang digunakan untuk menambahkan metadata ke
dalam Docker Image yang kita buat. Metadata adalah informasi tambahan, misal
seperti nama aplikasi, pembuat, website, perusahaan, lisensi, dll.

Metada berfungsi hanya sebagai informasi image saja, tidak digunakan ketika kita
menjalankan Docker Container.

#### Format Instruksi Label

Instruksi label punya format instruksi seperti berikut

```bash
LABEL <key>=<value>

LABEL <key1>=<value1> <key2>=<value2> ...
```


### Add Instruction

ADD adalah instruksi yang dapat digunakan untuk menambahkan file dari source ke
dalam folder destination di Docker Image. Perintah ADD bisa mendeteksi apakah
sebuah file source merupakan file kompresi seperti `tar.gz, gzip`, dll.  Jika
mendeteksi file source adalah file kompresi, maka secara otomatis file tersebut
akan di-ekstrak dalam folder destination.

Perintah ADD juga bisa mendukung banyak penambahan file sekaligus. Penambahan
file sekaligus di instruksi ADD menggunakan pattern di GoLang
[https://pkg.go.dev/path/filepath#Match](https://pkg.go.dev/path/filepath#Match) 

File source yang digunakan bisa berasal dari url ataupun dari direktori local
host (komputer kita).

#### Format Instruksi Add

Instruksi ADD memiliki format sebagai berikut:

```dockerfile
ADD <source> <destination>
```

Contoh:

```dockerfile
ADD world.txt hello  # Menambahkan file world.txt ke folder hello
ADD *.txt hello     # Menambahkan semua file format txt ke dalam folder hello
```

#### Penerapan

Kita akan buat folder `add` dimana di dalamnya terdapat `Dockerfile`.

```dockerfile
FROM alpine:latest

RUN mkdir hello

# Kita gunakan match pattern untuk menambahkan semua file txt ke dalam folder hello
ADD text/*.txt hello

# Catatan: Saya gunakan perintah array karena ada warning dari docker ketika
# menggunakan perintah plain
CMD ["cat", "hello/world.txt"]
```

Setelah selesai kita buat file Dockerfile, kita buat folder text pada direktori
yang sama dengan Dockerfile, kemudian tambahkan file `world.txt` ke dalam folder
text yang akan kita gunakan untuk proses add.

Selanjutnya kita akan build image dan membuat container serta menjalankannya.

```bash
# Build image
docker image build -t orsisam/add add

# Buat container based on image yang sudah kita buat.
docker container create --name add orsisam/add

# Jalankan container add
docker container start add

# Periksa logs untuk memastikan file txt berhasil dibaca
docker container logs add
```


### Docker Ignore File

`.dockeringnore` adalah dokumen yang dibaca pertama kali sebelum kita melakukan
COPY atau ADD file dari source. File `.dockerignore` ini mirip sekali dengan
file `.gitignore` yang digunakan oleh git, dimana kita bisa menyebutkan
file-file apa saja yang yang ingin kita hiraukan. Jika ada file yang kita sebut
di dalam file `.dockerignore`, secara otomatis file tersebut tidak akan di ADD
atau di-COPY. File `.dockerignore` juga mendukung ignore folder atau
mengguanakan regex.

#### Praktik

Untuk mecoabanya kita buat folder dengan nama `ignore`, di dalamnya buat file
`Dockerfile`, `.dockerignore`, serta folder `text`. Strukturnya akan seperti
pada gambar ini:

![GambarDockerignore](images/dockerignore.png)

Selanjutnya adalah dengan mem-build image -> running container -> check log.

```bash
# Build image
docker image build -t orsisam/ignore ignore

# Create container
docker container create --name ignore orsisam/ignore

# Run container
docker container start ignore

# Check logs
docker container logs ignore

# Result must be
-rw-rw-r--    1 root     root            51 Sep  6 12:11 world.txt
```


### Expose Instruction

Instruksi EXPOSE adalah instruksi untuk memberitahu bahwa container akan listen
port pada nomor tertentu dan protocol tertentu. Instruksi EXPOSE tidak akan
mem-publish port apapun sebenarnya. Instruksi EXPOSE hanya digunakan sebagai
dokumentasi untuk memberitahu yang membuat Docker Container, bahwa Docker Image
ini akan menggunakan port tertentu ketika dijalankan menjadi Docker Container.

#### Format Perintah

Berikut adalah format perintah dari instruksi EXPOSE

```dockerfile
EXPOSE <port> # Default menggunakan protocol TCP
EXPOSE <port>/<protocol>
```

#### Penerapan

Pada contoh penerapannya kita akan gunakan website sederhana menggunakan bahasa
pemrograman Golang.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from Golang server")
}
```

Simpan file tersebut dengan nama `main.go`. Lalu buat Dockerfile pada direktori
yang sama.

```dockerfile
FROM golang:alpine

RUN mkdir app
COPY main.go app

EXPOSE 8080

CMD ["go", "run", "app/main.go"]
```

Setelah membuat file-file tersebut, maka selanjutnya adalah menjalankan proses
build sampai mempersiapkan containernya agar aplikasi bisa jalan dan bisa diakses.

```bash
# Build Image
docker image build -t orsisam/expose

# Check exposed port
docker image inspect orsisam/expose

# Untuk memastikan aplikasi berhasil berjalan
# Buat container
docker container create --name expose --publish 8085:8080 orsisam/expose

# Jalankan container
docker container start expose
```


Lalu periksa di browser dengan url
[http://localhost:8085/](http://localhost:8085/)



### Environment Variable Instruction

ENV adalah instruksi yang digunakan untuk mengubah environment variable, baik
itu ketika tahapan build atau ketika dalam Docker Container. ENV yang sudah
didefinisikan di dalam Dockerfile bisa digunakan kembali dengan menggunakan
sintaks `${ENV_NAME}`. Environment Variable yang dibuat menggunakan instruksi
ENV disimpan di dalam Docker Image dan bisa dilihat dengan menggunakan perintah
docker image inspect. Selain itu, environment variable juga bisa diganti ketika
pembuatan Docker Container dengan menambahkan argument `--env key=value`.

#### Format Instruksi Env

Berikut format instruksi ENV

```dockerfile
ENV key=value
# Atau bisa dengan 
ENV key1=value1 key2=value2
```

#### Implementasi

Contoh implementasi kita akan menggunakan server sederhana menggunakan golang.
Sama seperti pada bab sebelumnya, hanya saja disini kita bisa merubah default port.

```go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	port := os.Getenv("APP_PORT")
	fmt.Println("Run app in port : ", port)
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":"+port, nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello world, using Environment Variable")
}
```

Simpan file di atas dengan nama `main.go`.

Kemudian buat file `Dockerfile` pada direktori yang sama.

```dockerfile
FROM golang:alpine

ENV APP_PORT=8080

RUN mkdir app
COPY main.go app

EXPOSE ${APP_PORT}
CMD ["go", "run", "app/main.go"]
```

Secara default aplikasi server kita akan menggunakan port 8080, namun dengan
membuat container yang diberi argument `--env` kita bisa menentukan port yang
berbeda secara manual.

Selanjutnya kita lakukan perintah docker di terminal.

```bash
# Build docker image
docker image build -t orsisam/env env

# Check image menggunakan inspect
# Disini kita bisa lihat environment variable apa saja yang bisa kita 
# gunakan pada image. Kita akan gunakan APP_PORT.
docker image inspect orsisam/env

# Create docker container with custom port
docker container create \
--name env \
--env APP_PORT=9090 \
--publish 9090:9090 \
orsisam/env

# Jalankan container
docker container start env

# Coba akses server dengan menggunakan curl atau browser
# dengan mengakses localhost:9090

# Check port dengan melihat log container
docker container logs env
```



### Instruksi Volume

VOLUME merupakan instruksi yang digunakan untuk membuat volume secara otomatis
ketika kita membuat Docker Container. Semua file yang terdapat di volume secara
otomatis akan otomatis di salin ke Docker Volume, walaupun kita tidak membuat
Docker Volume ketika membuat Docker Container-nya. Ini sangat cocok pada kasus
ketika aplikasi kita misal menyimpan data di dalam file, sehingga data bisa
secara otomatis aman berada di Docker volume.

#### Format Instruksi Volume

Berikut format instruksi volume

```dockerfile
VOLUME /lokasi/folder
VOLUME /lokasi/folder1 /lokasi/folder2 ....
VOLUME ["/lokasi/folder1", "/lokasi/folder2", "..."]
```


#### Implementasi




### Working Direktory Instruction

WORKDIR adalah instruksi untuk menentukan direktori/folder untuk menjalankan
instruksi RUN, CMD, ENTRYPOINT, COPY, dan ADD. Jika WORKDIR tidak ada, secara
otomatis direktorinya akan dibuat, dan selanjutnya setelah kita tentukan lokasi
WORKDIR-nya, direktori tersebut dijadikan tempat menjalankan instruksi selanjutnya.

Jika lokasi WORKDIR adalah relative path, maka secara otomatis dia akan masuk ke
direktori dari WORKDIR sebelumnya. WORKDIR juga bisa digunakan sebagai path
untuk likasi pertama kali ketika kita masuk ke dalam Docker Container.


#### Format Instruksi WORKDIR

**Absolute path**
```dockerfile
WORKDIR /app    # artinya working derektori adalah folder /app
```
Ini berarti working direktori saat ini adalah `/app`. dan ketika kita memberikan
nama folder tanpa diberikan slash `/`.

```dockerfile
WORKDIR src # menggunakan relative path, jadi working direktori saat ini /app/src
```

Kita juga bisa langsung mendefinisikan working direktori seperti ini:
```dockerfile
WORKDIR /home/app # artinya working direktori saat ini /home/app
```


#### Contoh Implementasi

Kita akan menggunakan contoh web server go pada contoh sebelumnya.

**main.go**
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from workdir")
}
```

Kemudian buat docker filenya
```dockerfile
FROM golang:alpine

WORKDIR /app
COPY main.go /app

EXPOSE 8080
CMD ["go", "run", "main.go"]

```

Pada perintah CMD kita tidak lagi memberi full path pada main.go, karena sudah
ditentukan oleh instuksi WORKDIR.

Selanjutnya tinggal build, buat container dan jalankan container-nya.

```bash
# build container
docker image build -t orsisam/workdir

# Create container
docker container create \
--name workdir \
--publish 8088:8080 \
orsisam/workdir

# Jalankan container
docker container start workdir

# Selain menggunakan browser, kita juga bisa memanfaatkan curl untuk testing
curl localhost:8088
```

Kalau kita pahami instruksi WORKDIR ini mirip dengan perintah `cd` pada
terminal. 

Oke, selanjutnya kita akan masuk ke container menggunakan perintah exec. Dan
untuk memastikan ada di direktori mana kita saat masuk ke container, kita
jalankan perintah `pwd` pada terminal.


```bash
docker container exec -i -t workdir /bin/sh

# Di dalam container jalankan perintah
pwd

# Seharusnya menghasilkan:
/app
```


### User Instruction

USER adalah instruksi yang digunakan untuk mengubah user atau user group ketika
Docker Image dijalankan. Secara default, Docker akan menggunakan user root,
namun pada kasus tertentu mungkin ada aplikasi yang tidak ingin jalan dalam user
root. Maka kita bisa mengubah user menggunakan instruksi USER.

> **Catatan**: Perlu diperhatikan, sebelum kita pindah user pastikan kita sudah
> membuat user tersebut saat image creation. Karena jika tidak user yang dituju
> itu tidak ada.

#### Format Instruksi USER
Berikut adalah format instruksi USER:

```dockerfile
USER <user> # mengubah user
USER <user>:<group> # mengubah user dan group
```

#### Implementasi Instruksi USER
Pada contoh ini kita akan memanfaatkan aplikasi go server sebelumnya. 

**main.go**
```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from workdir")
}
```

Kemudian buat Dockerfile pada direktori yang sama.

```dockerfile
FROM golang:alpine

RUN mkdir /app

RUN addgroup -S orsigroup
RUN adduser -S -D -h /app orsi orsigroup
RUN chown -R orsi:orsigroup /app

USER orsi

COPY main.go /app

EXPOSE 8080

CMD ["go", "run", "/app/main.go"]
```

Pada Dockerfile, pertama kali kita akan buuat image dari base image yang sudah
ada, kita akan pakai image `golang:alpine`.

`RUN addgroup -S orsigroup` merupakan perintah tambah group dengan parameter -S
yang berarti system group.

`RUN adduser -S -D -h /app orsi orsigroup` adalah perintah untuk membut user
baru. Parameter "-S" digunakan untuk mendefinisikan user sebagai system user.
Mirip dengan tipe system group, namun dengan tambahan seperti tidak punya home
folder selayaknya user biasa dan biasanya tidak punya passwork. Parameter "-D"
artinya disabled password. dan parameter terakhir "-h" untuk menentukan home
folder yakni pada direktori "/app"


> Informasi tambahan: istilah system group dibuat untuk keperluan aplikasi,
> service atau daemon sistem (seperti nginx, Docker, Portgresql, atau aplikasi
> server, dll). Medapatkan GID (Group ID) bernilai rendah, biasanya di bawah
> 1000, tergantung alikasi sistem. Di desain agar tidak berbenturan dengan
> daftar group pengguna manusia (*regular users*)

Setelah kedua file telah dibuat maka langkah selanjutnya adalah melakukan build
image -> create container -> testing.

```bash
# Build image
docker image build -t orsisam/user user

# Create container
docker container create \
--name user \
--publish 8088:8080 \
orsisam/user

# Jalankan container
docker container start user

# Masuk ke shell container dan lakukan testing untuk memastikan
# user sudah sesuai dengan user yang kita buat di dockerfile.
docker container exec -i -t user /bin/sh

# Di dalam container jalankan perintah 
whoami

# Perintah di atas digunakan untuk mendapatkan user yang kita gunakan 
# saat ini. Jika sudah sesuai maka implementasi sudah benar.

```


### Argument Instruction

ARG merupakan instruksi yang digunakan untuk mendefinisikan variabel yang bisa
digunakan oleh pengguna untuk dikirim ketika melakukan proses docker build
menggunakan perintah `--build-arg key=value`. Mirip dengan instruksi ENV, tapi
ada ARG nilainya hanya bisa digunakan saat build

Cara mengakses variable dari ARG sama seperti mengakses variable dari ENV, yakni
dengan menggunakan `${variable_name}`

#### Format Instruksi ARG
Berikut adalah format untuk instruksi ARG

```dockerfile
ARG key # Membuat argument variable

ARG key=default_value # membuat argument variable dengan default value jika
                      # tidak diisi.
```


#### Praktik

Kita akan menggunakan file `main.go` yang sama dengan user.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from workdir")
}
```

Selanjutnya kita buat file Dockerfile

```dockerfile
FROM golang:apline

ARG app=main

RUN mkdir app
COPY main.go app
RUN mv app/main.go app/${app}.go

EXPOSE 8080

CMD ["go", "run", "app/${app}.go"]
```

Langkah selanjutnya adalah dengan menjalankan beberapa perintah docker.

```bash
# Build image 
docker image build -t orsisam/arg arg --build-arg app=testarg

# Create container
docker container create --name arg --publish 8088:8080 orsisam/arg

# Jalankan container
docker container start arg
```

Saat dijalankan ternyata container tidak berhasil, alias tidak jalan. Mengapa
hal ini bisa terjadi. Kita akan cek pada log container.

```bash
docker container logs arg

# Output
# stat app/${app}.go: no such file or directory

# Inspect image untuk memeriksa perintah CMD di image
docker image inspect orsisam/arg

# Output:
# "Cmd": [
#                "go",
#                "run",
#                "app/${app}.go"
#            ],

```

Hal ini dikarenakan variable nya tidak dikenali oleh container. Seperti yang
disampaikan di awal ARG hanya bisa diakses pada waktu build time, sedangkan CMD
itu dijalankan pada saat runtime. Jadi ketika kita menggunakan variable ARG di
CMD, maka kita perlu memasukkan variable ARG ke variable ENV.

Jadi kita akan memperbaiki `Dockerfile` sebelumnya, sehingga nantinya saat
runtime CMD mengenali variable ini.

**Dockerfile terbaru**
```dockerfile
FROM golang:alpine

ARG app=main

RUN mkdir app
COPY main.go app
RUN mv app/main.go app/${app}.go

EXPOSE 8080

ENV app=${app}
CMD ["/bin/sh", "-c", "go run app/${app}.go"]
```

Perubahan ada pada penambahan instruksi ENV yang mengambil dari variable
instuksi ARG. Dan langkah selanjutnya adalah dengan menjalankan peritah-perintah
docker yang kita jalankan sebelumnya mulai dari build image, creata container,
dan jalankan containernya. Jangan lupa untuk menghapus container dan image sebelumnya.


### Health Check Instruction

HEALTHCHECK adalah instruksi yang digunakan untuk memberitahu Docker untuk
mengecek apakah Container masih berjalan dengan baik atau tidak. Jika kita
membuat container dengan menggunakan HEALTHCHECK, secara otomatis container akan memilih status `health`, ketika awal
bernilai `starting`, jika berhasil bernilai `healthy`, jika gagal akan bernilai
`unhealthy`.

#### Format Instruksi

HEALTHCHECK punya dua bentuk instruksi

```dockerfile
# Bentuk default
HEALTHCHECK NONE # status disabled atau mengikuti dari baase image

# Bentuk kompleks
# Periksa container health dengan menjalankan perintah di dalam container
HEALTHCHECK [OPTIONS] CMD command
```

Instruksi HEALTHCHECK memberitahu Docker bagaimana mengetes container untuk
memeriksa apakah container tersebut masih bekerja dengan baik. Hal ini bisa
digunakan untuk medeteksi semacam kasus web server yang stuck di infinite loop
dan tidak bisa menangani koneksi baru, meskipun status server masih berjalan.

Opsi yang bisa digunakan sebelum perintah CMD antara lain:
* `--interval=DURATION` (default=30s)
* `--duration=DURATION` (default=30s)
* `--start-period=DURATION` (default: 0s), digunakan untuk menentukan jeda waktu
  kapan HEALTHCHECK dijalankan setelah aplikasi dimulai, tidak disarankan
  menggunakan nilai `0s` karena aplikasi memiliki jeda waktu sampai benar-benar
  berjalan.
* `--start-interval=DURATION` (default: `5s`)
* `--retries=N` (default: `3`), pengulangan yang dilakukan ketika perintah CMD
  mengindikasikan gagal. Jika sampai percobaan ke N masih gagal maka Docker akan
  memberikan status `unhealthy` pada container.

> **Catatan**: HEALTHCHECK hanya bisa diberikan sekali pada Dockerfile, jika ada
> lebih dari 1 instruksi HEALTHCHECK pada Dockerfile maka instruksi terakhiar
> lah yang dieksekusi.


#### Praktik Healthcheck

Disini kita akan membuat webserver golang namun dengan tambahan fungsi healthcheck.

**main.go**
```go
package main

import (
	"fmt"
	"net/http"
)

var counter = 0

func main() {
	http.HandleFunc("/", HelloServer)
	http.HandleFunc("/health", HealthCheck)

	http.ListenAndServe(":8080", nil)
}

func HealthCheck(w http.ResponseWriter, r *http.Request) {
	counter = counter + 1
	if counter > 5 {
		w.WriteHeader(500)
		fmt.Fprintf(w, "KO")
	} else {
		fmt.Fprintf(w, "OK")
	}
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from healthcheck.")
}
```

Server go tersebut akan secara otomatis memberikan response kode 500 ketika
HEALTHCHECK instruction telah dipanggil lebih dari 5x. Jadi ketika hasil
tersebut bernilai 500 (merupakan http kode untuk internal server error) maka
container akan punya status unhealthy.

Oke, sekarang kita lanjut menjalankan langkah-langkah pengujian.

```bash
# Build image
docker image build -t orsisam/health health

# Buat container 
docker container create --name health --publish 8088:8080 orsisam/health

# Jalankan container
docker container start health

# Periksa status container dengan menggunakan perintah
docker container ls

# jalankan perintah di atas berulang kali sehingga status menjadi unhealthy

```



### Entrypoint Instruction

ENTRYPOINT adalah instruksi untuk menentukan executable file yang akan
dijalankan oleh container. Biasanya ENTRYPOINT itu erat kaitannya dengan
instruksi CMD. Karena saat kita membuat instruksi CMD tapi tidak menentukan file
executable-nya, maka secara otomatis CMD akan menggunakan executable file dari ENTRYPOINT.


#### Format Instruksi ENTRYPOINT
Instruksi ENTRYPOINT punya dua format yang tersedia:

```dockerfile
# Exec form (recomended form)
ENTRYPOINT ["executable", "param1", "param2"]

# Shell form
ENTRYPOINT command param1 param2

```

#### Contoh Penerapan
Sama dengan contoh-contoh sebelumnya, kita akan menggunakan go server sederhana
untuk memastikan peritahnya bekerja dengan baik.

Pertama-tama kita buat file **main.go**.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from workdir")
}

```

Selanjutnya dengan membuat **Dockerfile** dimana kita akan menerapkan instruksi
ENTRYPOINT

```dockerfile
FROM golang:alpine

RUN mkdir /app/
COPY main.go /app/

EXPOSE 8080

ENTRYPOINT ["go", "run"]

CMD ["/app/main.go"]

```

Sebagaimana dijelaskan sebelumnya, pada instruksi entrypoint kita hanya
menempatkan file executable yakni perintah `go run`. Dan pada instruksi CMD kita
hanya menempatkan parameter yang merupakan lokasi dimana file `main.go` berada.

Setelah semua file yang dibutuhkan telah selesai dibuat, langkah selanjutnya
adalah membuat image dan container dari dockerfile yang sudah dibuat. Baru kita
melakukan pengujian apakah aplikasi kita berhasil dijalankan dengan menggunakan
instruksi ENTRYPOINT.

```bash
# Build image
docker image build -t orsisam/entrypoint entrypoint

# Create container
docker container create --name entrypoint \
--publish 8088:8080 \
orsisam/entrypoint:latest

# Jalankan container
docker container start entrypoint

# Periksa apakah container berhasil dijalankan
docker container ls

# Jika ditemukan maka proses running aplikasi menggunakan 
# instruksi ENTRYPOINT berhasil.
# Coba cek menggunakan curl untuk mendapatkan pesan dari server
curl localhost:8088

```


### Multi-Stage build

Saat kita membuat Dockerfile dari base image yang besar, secara otomatis ukuran
image-nya pun akan menjadi besar juga. Pada percobaan yang sudah kita lakukan
sebelumnya kita menggunakan image dari golang, dan ukuran image-nya cukup besar.
Coba jalankan perintah `docker image ls | grep golang` dan lihat ukuran image
sekitar 250MB. Dan coba cek juga image yang kita buat sebelumnya juga dan
bandingkan.

Lalu apa itu multi-stage build? Multi-stage build merupakan salah satu fitur
Docker yang memungkinkan kita menggunakan beberapa instruksi FROM dalam satu
Dockerfile. Setiap instruksi FROM memulai stage(tahap) baru dengan base image
yang berbeda.

Kita akan menggunakan permasalahan yang ada pada contoh-contoh penerapan yang
sebelumnya kita buat. Pada saat kita membuat aplikasi sederhana  yang
memanfaatkan docker image dari golang, file image yang kita buat dari base image
tersebut terlalu besar untuk ukuran aplikasi sederhana kita. Solusinya kita bisa
membuat image dari alpine yang ringkas dan kecil. Sedangkan file `main.go` kita
compile terlebih dahulu baru kita salin ke image file kita dengan basis alpine.

Tapi dari cara ini timbul permasalahan baru lagi. Kita harus mengkompilasi file
`main.go` ini di sistem operasi alpine atau di os yang sama dimana kita akan
mengensekusi binari filenya. Maka kita bisa memanfaatkan fitur docker
**multi-stage build**. 

Berikut bentuk `Dockerfile` yang akan kita buat.

```dockerfile
FROM golang:alpine AS builder

WORKDIR /app/
COPY main.go .
RUN go build -o main main.go


FROM alpine:latest
WORKDIR /app/
COPY --from=builder /app/main ./
EXPOSE 8080
CMD ["./main"]

```

Dan kita buat juga file `main.go`, kita ambil yang simpel.

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", HelloServer)
	http.ListenAndServe(":8080", nil)
}

func HelloServer(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello from workdir")
}

```

Seperti biasa kita lanjut build dan create container untuk mengecek image bisa
dijalankan atau tidak, meskipun start container tidak perlu karena kita hanya
perlu memastikan ukuran image jadi lebih kecil.

```bash
# Build image
docker image build -t orsisam/builstage buildstage

# Cek ukuran image yang sudah jadi
docker image ls | grep multistage

# Ini hasil yang saya dapat
# orsisam/multistage:latest    76381f5e1e58       16.4MB             0B   U

```

Bisa kita lihat ukuran file jadi sangat kecil dibanding kita build menggunakan
image golang. Jika ingin memeriksa apakah image berhasil jalan di container
jalankan perintah ini:

```bash
# Create docker container
docker container create --name multistage \
--publish 8088:8080 \
orsisam/multistage:latest

# cek apakah sudah running
docker container ls
curl localhost:8088

```



