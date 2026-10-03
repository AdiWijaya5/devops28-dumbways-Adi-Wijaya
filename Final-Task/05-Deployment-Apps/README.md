# Deployment-App

## 1. Create Taks ansible-Playbook file name deploy-app.yaml

```yaml

---
- hosts: appserver
  become: true

  tasks:
    # ==========================================
    # 1. INSTALASI & KONFIGURASI DOCKER DI APP SERVER
    # ==========================================
    - name: Install required system packages for Docker
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present

    - name: Create directory for Docker keyrings
      ansible.builtin.file:
        path: /etc/apt/keyrings
        state: directory
        mode: "0755"

    - name: Add Docker official GPG key
      ansible.builtin.shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      args:
        creates: /etc/apt/keyrings/docker.gpg

    - name: Install Docker Engine
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
        update_cache: true
        state: present

    - name: Add user finaltask-adi to docker group
      ansible.builtin.user:
        name: "finaltask-adi"
        groups: docker
        append: true

    - name: Configure insecure-registries for local HTTP Docker Registry
      ansible.builtin.copy:
        dest: /etc/docker/daemon.json
        content: |
          {
            "insecure-registries": ["{{ registry_url }}"]
          }
        mode: "0644"
      register: docker_config

    - name: Restart Docker to apply insecure registry config
      ansible.builtin.systemd:
        name: docker
        state: restarted
      when: docker_config.changed

    - name: Ensure Docker service is started and enabled
      ansible.builtin.systemd:
        name: docker
        state: started
        enabled: true

    # ==========================================
    # 2. BUILD & PUSH FRONTEND (fe-dumbmerch)
    # ==========================================
    - name: Build Frontend Docker image
      community.docker.docker_image:
        name: "{{ registry_url }}/fe-dumbmerch"
        tag: "{{ tag_docker}}"
        source: build
        build:
          path: "{{ project_root }}/fe-dumbmerch"
          dockerfile: Dockerfile
        state: present

    - name: Push Frontend image to Gateway Registry
      community.docker.docker_image:
        name: "{{ registry_url }}/fe-dumbmerch"
        tag: "{{ tag_docker }}"
        source: local
        push: true
        state: present

    # ==========================================
    # 3. BUILD & PUSH BACKEND (be-dumbmerch)
    # ==========================================
    - name: Build Backend Docker image
      community.docker.docker_image:
        name: "{{ registry_url }}/be-dumbmerch"
        tag: "{{ tag_docker }}"
        source: build
        build:
          path: "{{ project_root }}/be-dumbmerch"
          dockerfile: Dockerfile
        state: present

    - name: Push Backend image to Gateway Registry
      community.docker.docker_image:
        name: "{{ registry_url }}/be-dumbmerch"
        tag: "{{ tag_docker }}"
        source: local
        push: true
        state: present

    # ==========================================
    # 4. DEPLOY / RUN FRONTEND CONTAINER
    # ==========================================
    - name: Remove old Frontend container if exists
      community.docker.docker_container:
        name: fe-dumbmerch-app
        state: absent

    - name: Run Frontend container
      community.docker.docker_container:
        name: fe-dumbmerch-app
        image: "{{ registry_url }}/fe-dumbmerch:{{ tag_docker }}"
        state: started
        restart_policy: always
        ports:
          - "3000:80"

    # ==========================================
    # 5. DEPLOY / RUN BACKEND CONTAINER
    # ==========================================
    - name: Remove old Backend container if exists
      community.docker.docker_container:
        name: be-dumbmerch-app
        state: absent

    - name: Run Backend container
      community.docker.docker_container:
        name: be-dumbmerch-app
        image: "{{ registry_url }}/be-dumbmerch:{{ tag_docker }}"
        state: started
        restart_policy: always
        ports:
          - "5000:5000"

  handlers:
    - name: Restart Docker
      ansible.builtin.systemd:
        name: docker
        state: restarted


```
### Penjelasan Otomatisasi Deployment - Dumbmerch Aplikasi

### Dokumentasi Teknis: Ansible Playbook Deployment (Dumbmerch App)

Skrip ini mengotomatiskan setup environment Docker, proses *build/push* ke Private Registry, hingga deployment aplikasi di `appserver`.

---

### Variabel Utama
* `registry_url` : Alamat IP dan port Private Docker Registry tujuan.
* `tag_docker`   : Label tag image (misal: `staging`, `production`).
* `project_root` : Jalur direktori source code aplikasi di server.

---

### Langkah Kerja Skrip (Tasks)

#### Tahap 1: Setup Environment Docker
* **Instalasi Paket Dependensi:** Mengunduh paket dasar Ubuntu untuk kebutuhan transport APT via HTTPS.
* **Import GPG Key:** Menambahkan kunci resmi Docker untuk memverifikasi keaslian paket [1cite: 2].
* **Instalasi Core Engine:** Menginstal runtime utama (`docker-ce`, `docker-cli`, `containerd.io`).
* **Hak Akses User:** Mendaftarkan user `finaltask-adi` ke grup docker agar bisa eksekusi command tanpa `sudo`.
* **Insecure Registry:** Menulis konfigurasi `/etc/docker/daemon.json` untuk mengizinkan koneksi HTTP biasa ke Private Registry.
* **Aktivasi Layanan:** Memastikan service Docker langsung aktif dan otomatis menyala saat server *reboot*.

#### Tahap 2 & 3: Manajemen Image (Frontend & Backend)
* **Kompilasi Dockerfile:** Masuk ke direktori source code dan merakit image lokal dengan format nama registry.
* **Upload Image:** Melakukan *push* hasil kompilasi image lokal ke server Private Registry.

#### Tahap 4: Peluncuran Container Aplikasi
* **Pembersihan Kontainer:** Menghapus kontainer lama untuk mengosongkan alokasi port jaringan.
* **Inisiasi Frontend:** Menjalankan kontainer frontend baru (Port `3000:80`) dengan *restart policy: always*.
* **Inisiasi Backend:** Menjalankan kontainer backend baru (Port `5000:5000`) dengan *restart policy: always*.

---

### Fungsi Handlers (Pemicu)
* **Restart Docker:** Mengeksekusi *restart* pada engine Docker hanya jika terjadi perubahan pada file konfigurasi `daemon.json`.


## 2 Create file Dockerfile to Appserver fe-dumbmerch

<p align="center"><img width="960" height="425" alt="image" src="https://github.com/user-attachments/assets/986c3140-db85-44fa-853c-d6f4656a01bb" /></p>

Dockerfile ini pakai trik **Multi-stage Build**.

## Tahap 1: Proses Perakitan Aplikasi (Build)

## Dokumentasi Teknis: Dockerfile Frontend (Multi-Stage Build)

Konfigurasi ini menerapkan metode **Multi-Stage Build** untuk memisahkan tahap kompilasi (*build*) dengan tahap distribusi (*runtime*), sehingga menghasilkan ukuran *image* produksi yang sangat efisien dan minimalis.

---

### Tahap 1: Kompilasi Aplikasi (Build Stage)

```dockerfile
FROM node:18-alpine AS builder
```
*   **Fungsi:** Menggunakan *base image* Node.js 18 berbasis Alpine Linux untuk proses kompilasi. Label `AS builder` digunakan sebagai referensi ekstraksi aset pada tahap berikutnya.

```dockerfile
WORKDIR /app
```
*   **Fungsi:** Menetapkan direktori `/app` sebagai ruang kerja utama seluruh instruksi di dalam container.

```dockerfile
COPY package*.json ./
RUN npm install
```
*   **Fungsi:** Menyalin manifes dependensi dan menginstal *library*. Pemisahan langkah ini berfungsi untuk mengoptimalkan fitur *layer caching* pada Docker.

```dockerfile
COPY . .
```
*   **Fungsi:** Mentransfer seluruh kode sumber aplikasi dari mesin lokal ke dalam direktori kerja container.

```dockerfile
ENV NODE_OPTIONS=--openssl-legacy-provider
```
*   **Fungsi:** Menyuntikkan variabel lingkungan untuk mengatasi isu kompatibilitas algoritma enkripsi (OpenSSL) antara React versi lama dan Node.js versi baru.

```dockerfile
RUN npm run build
```
*   **Fungsi:** Mengeksekusi kompilasi produksi guna menghasilkan aset statis (HTML, CSS, JS) di dalam direktori `/app/build`.

---

### Tahap 2: Server Produksi (Serve Stage)

```dockerfile
FROM nginx:alpine
```
*   **Fungsi:** Menginisiasi *runtime* baru menggunakan web server Nginx berbasis Alpine yang efisien untuk lingkungan produksi.

```dockerfile
COPY --from=builder /app/build /usr/share/nginx/html
```
*   **Fungsi:** Menyalin hasil kompilasi statis dari tahap pertama (`--from=builder`) ke direktori publik standar milik Nginx.

```dockerfile
COPY nginx.conf /etc/nginx/conf.d/default.conf
```
*   **Fungsi:** Menerapkan konfigurasi kustom Nginx untuk mendukung *routing* aplikasi SPA (*Single Page Application*), guna mencegah error HTTP 404 saat halaman di-*refresh*.

```dockerfile
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
*   **Fungsi:** Membuka port HTTP standar (Port 80) dan menjalankan servis Nginx di latar depan (*foreground*) agar kontainer tetap aktif berjalan secara konstan.

--- 

## 2 Create file Dockerfile to Appserver be-dumbmerch

<p align="center"><img width="953" height="327" alt="image" src="https://github.com/user-attachments/assets/16c1d750-c5ba-44e4-8470-2c274afeb99c" /></p>

## Dokumentasi Teknis: Dockerfile Frontend (Multi-Stage Build)

Konfigurasi ini menerapkan metode **Multi-Stage Build** untuk memisahkan tahap kompilasi kode (*build*) dengan tahap eksekusi (*runtime*). Hasil akhirnya adalah *image* produksi yang sangat kecil karena hanya berisi file biner matang tanpa membawa *compiler* Go.

---

### Tahap 1: Kompilasi Aplikasi (Build Stage)

```dockerfile
FROM golang:1.21-alpine AS builder
```
*   **Fungsi:** Menggunakan *base image* Go versi 1.21 berbasis Alpine Linux untuk proses kompilasi. Label `AS builder` digunakan sebagai referensi ekstraksi aset pada tahap kedua.

```dockerfile
WORKDIR /app
```
*   **Fungsi:** Menetapkan direktori `/app` sebagai ruang kerja utama untuk seluruh instruksi di dalam container.

```dockerfile
COPY go.mod go.sum ./
RUN go mod download
```
*   **Fungsi:** Menyalin file manajemen modul Go dan mengunduh (*download*) seluruh *library* dependensi. Pemisahan langkah ini mengoptimalkan fitur *layer caching* pada Docker.

```dockerfile
COPY . .
```
*   **Fungsi:** Mentransfer seluruh kode sumber aplikasi backend dari mesin lokal ke direktori kerja container.

```dockerfile
RUN CGO_ENABLED=0 GOOS=linux go build -o main .
```
*   **Fungsi:** Mengompilasi kode Go menjadi file biner tunggal bernama `main`. Parameter `CGO_ENABLED=0` memastikan file biner bersifat statis (tidak bergantung pada library OS), dan `GOOS=linux` memastikan target sistem operasinya adalah Linux.

---

### Tahap 2: Lingkungan Produksi (Run Stage)

```dockerfile
FROM alpine:latest
```
*   **Fungsi:** Menginisiasi *runtime* baru menggunakan OS Alpine Linux versi terbaru yang sangat ringan (hanya sekitar 5MB) untuk menjalankan aplikasi di produksi.

```dockerfile
WORKDIR /app
COPY --from=builder /app/main .
COPY --from=builder /app/.env .
```
*   **Fungsi:** Menetapkan folder kerja, lalu menyalin file biner (`main`) dan file konfigurasi lingkungan (`.env`) dari tahap pertama (`--from=builder`) ke container baru.

```dockerfile
EXPOSE 5000
CMD ["./main"]
```
*   **Fungsi:** Menginformasikan bahwa aplikasi berjalan di port internal 5000, serta mengeksekusi file biner `./main` secara langsung untuk menyalakan server backend.

---

## Variabel all ansible

```yaml

---
# Konfigurasi SSH & Koneksi Default
ansible_port: 3333
ansible_user: finaltask-adi
ansible_ssh_private_key_file: /home/adi/.ssh/jay-key.pem

# Konfigurasi Ansibel
target_user: "finaltask-adi"
user_password_hash: "$6$JVoSl9OOUFYIc1XO$hhDqhKB............"
ssh_public_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQAB............"
ssh_port: 3333

# Konfigurasi Github
ssh_key_path: "/home/finaltask-adi/.ssh/authorized_keys"

# Variabel Docker Registry private
registry_port: 5000
registry_storage_path: "/var/lib/registry"

# Konfigurasi Database & Deployment Application
db_user: "dumbmerch"
db_password: "neonjago"
db_name: "dumbmerch_db"
db_storage_path: "/home/finaltask-adi/storage"

registry_url: "15.232.124.118:5000"
project_root: "/home/finaltask-adi"
app_version: "latest"
tag_docker: "staging"
tag_docker_prod: "production"

```

## Run ansible-playbook deploy-app.yaml

```bash

ansible-playbook deploy-app.yaml

```

<p align="center"><img width="956" height="969" alt="image" src="https://github.com/user-attachments/assets/d3b643be-5d6e-4ed8-ae4f-8b01295575a8" /></p>
<p align="center"><img width="962" height="181" alt="image" src="https://github.com/user-attachments/assets/8fb5d256-727e-4689-98f4-881c60118b6c" /></p>

---

# Deploy database 

## Create Taks ansible-Playbook file name deploy-db.yaml

```yaml

---
- hosts: database
  become: true
  tasks:

    - name: Install dependencies for Docker repository
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-pip
          - python3-docker
        state: present
        update_cache: true

    - name: Add Docker official GPG key
      ansible.builtin.shell: |
        install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        chmod a+r /etc/apt/keyrings/docker.gpg
      args:
        creates: /etc/apt/keyrings/docker.gpg

    - name: Set up the stable Docker repository
      ansible.builtin.shell: |
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
          $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
      args:
        creates: /etc/apt/sources.list.d/docker.list

    - name: Install Docker Engine and Containerd
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-buildx-plugin
          - docker-compose-plugin
        state: present
        update_cache: true

    - name: Ensure Docker service is running and enabled
      ansible.builtin.service:
        name: docker
        state: started
        enabled: true

    - name: Add remote user to docker group
      ansible.builtin.user:
        name: "{{ ansible_user }}"
        groups: docker
        append: true

    - name: Ensure PostgreSQL storage directory exists
      ansible.builtin.file:
        path: "{{ db_storage_path }}"
        state: directory
        mode: "0700"

    - name: Run PostgreSQL Docker container
      community.docker.docker_container:
        name: postgres-db
        image: postgres:15-alpine
        state: started
        restart_policy: always
        published_ports:
          - "5432:5432"
        env:
          POSTGRES_USER: "{{ db_user }}"
          POSTGRES_PASSWORD: "{{ db_password }}"
          POSTGRES_DB: "{{ db_name }}"
        volumes:
          - "{{ db_storage_path }}:/var/lib/postgresql/data"

    - name: Wait for PostgreSQL container to be fully ready
      ansible.builtin.command:
        cmd: "docker exec postgres-db pg_isready -U {{ db_user }}"
      register: pg_ready
      until: pg_ready.rc == 0
      retries: 10
      delay: 3
      changed_when: false

    - name: Configure PostgreSQL to listen on all interfaces (remote access)
      ansible.builtin.command:
        cmd: 'docker exec postgres-db psql -U {{ db_user }} -d {{ db_name }} -c "ALTER SYSTEM SET listen_addresses = ''*'';"'
      changed_when: true

    - name: Check if remote access rule already exists in pg_hba.conf
      ansible.builtin.command:
        cmd: "docker exec postgres-db grep -q '0.0.0.0/0' /var/lib/postgresql/data/pg_hba.conf"
      register: hba_check
      failed_when: false
      changed_when: false

    - name: Append remote access rule to pg_hba.conf if not exists
      ansible.builtin.command:
        cmd: 'docker exec postgres-db bash -c "echo ''host all all 0.0.0.0/0 md5'' >> /var/lib/postgresql/data/pg_hba.conf"'
      when: hba_check.rc != 0
      changed_when: true

    - name: Restart PostgreSQL container to apply configuration changes
      community.docker.docker_container:
        name: postgres-db
        state: started
        restart: true


```

## Penjelasan Otomatisasi Deployment Database (PostgreSQL)

Skrip ini mengotomatiskan instalasi Docker Engine, deployment container PostgreSQL, serta konfigurasi hak akses jaringan agar database dapat menerima koneksi dari server luar (*remote access*).

---

### Variabel Utama
* `db_storage_path` : Lokasi folder di server fisik untuk menyimpan data database secara permanen.
* `db_user`         : Username untuk administrator database PostgreSQL.
* `db_password`     : Password untuk user database.
* `db_name`         : Nama database awal yang otomatis dibuat di dalam container.

---

### Langkah Kerja Skrip (Tasks)

#### Tahap 1: Setup Environment Docker
* **Instalasi Paket Dependensi:** Mengunduh modul inti Ubuntu dan Python SDK (`python3-docker`) agar Ansible dapat mengendalikan container Docker secara langsung.
* **Import GPG Key & Repositori:** Mengunduh kunci enkripsi resmi dan mendaftarkan repositori stabil Docker ke dalam sistem APT Ubuntu.
* **Instalasi Docker Engine:** Memasang engine runtime utama (`docker-ce`, `docker-cli`, `containerd.io`, dan plugin compose).
* **Aktivasi Layanan & Hak Akses:** Memastikan service Docker langsung aktif dan mendaftarkan user remote ke dalam grup docker agar bisa eksekusi command tanpa `sudo`.

#### Tahap 2: Deployment Container PostgreSQL
* **Folder Storage Fisik:** Membuat direktori penyimpanan data di server lokal dengan hak akses ketat (`0700`) agar data tidak hilang saat container di-restart.
* **Inisiasi Container (`postgres-db`):** Menjalankan container PostgreSQL versi `15-alpine` pada port standar `5432:5432` dengan kebijakan otomatis menyala kembali jika server mati (*restart policy: always*).
* **Pengecekan Kesiapan (Healthcheck):** Menjalankan perintah `pg_isready` secara berkala (maksimal 10 kali percobaan) untuk memastikan service database sudah siap menerima koneksi sebelum skrip lanjut ke tahap berikutnya.

#### Tahap 3: Konfigurasi Akses Jaringan (Remote Access)
* **Listen Addresses:** Mengubah sistem internal PostgreSQL (`ALTER SYSTEM`) agar database bersedia mendengarkan trafik masuk dari semua kartu jaringan (`listen_addresses = '*'`).
* **Verifikasi & Injeksi `pg_hba.conf`:** Memeriksa aturan akses pada file konfigurasi database. Jika belum ada, sistem akan menyuntikkan baris `host all all 0.0.0.0/0 md5` agar database menerima koneksi dari IP luar menggunakan enkripsi password.
* **Penerapan Konfigurasi (Reload):** Melakukan *restart* container `postgres-db` secara instan untuk menerapkan seluruh konfigurasi jaringan yang baru saja diubah.

## .env file database

<p align="center"><img width="475" height="362" alt="image" src="https://github.com/user-attachments/assets/5a3548c1-dc67-4515-84ce-48d7262d2669" /></p>

### 1. Konfigurasi Aplikasi Backend

*   `PORT=5000`
    *   **Fungsi:** Menetapkan port jaringan internal tempat aplikasi backend berjalan dan mendengarkan lalu lintas data (*traffic*).
*   `APP_ENV=staging`
    *   **Fungsi:** Menentukan mode operasi aplikasi. Status `staging` menandakan aplikasi berjalan di lingkungan uji coba

---

### 2. Konfigurasi Koneksi Database (PostgreSQL)

Bagian ini berisi kredensial yang dibutuhkan oleh backend untuk melakukan autentikasi dan komunikasi dengan server database.

*   `DB_HOST=15.232.4.76`
    *   **Fungsi:** Alamat IP publik atau domain tempat server database eksternal berada.
*   `DB_PORT=5432`
    *   **Fungsi:** Port standar yang digunakan untuk masuk ke layanan database PostgreSQL.
*   `DB_USER=dumbmerch`
    *   **Fungsi:** Username administrator atau pengguna khusus yang memiliki hak akses ke skema database.
*   `DB_PASSWORD=neonjago`
    *   **Fungsi:** Kata sandi rahasia untuk autentikasi user database yang bersangkutan.
*   `DB_NAME=dumbmerch_db`
    *   **Fungsi:** Nama database spesifik di dalam PostgreSQL yang akan dibaca dan dimanipulasi oleh aplikasi.
*   `DB_SSLMODE=disable`
    *   **Fungsi:** Mematikan fitur enkripsi SSL/TLS selama proses pengiriman data antara backend dan database. Opsi ini digunakan untuk menyederhanakan koneksi pada jaringan yang sudah dianggap aman.

---

## Run ansible-playbook db_deploy.yaml

```bash

ansible-playbook db_deploy.yaml

```

<p><img width="959" height="829" alt="image" src="https://github.com/user-attachments/assets/9516b7d0-a15a-4842-aaa7-7a78ae30d63f" />
</p>
<p><img width="960" height="126" alt="image" src="https://github.com/user-attachments/assets/593e0d99-805d-4190-b19f-ce2e02ab9cca" />
</p>

## Test database

<p><img width="957" height="373" alt="image" src="https://github.com/user-attachments/assets/b452fc5c-2a86-4cb6-855c-869edddfa8f4" /></p>

* *Catatan* untuk Create load balancing for frontend and backend ada di tahap Webserver (gatway)* *
