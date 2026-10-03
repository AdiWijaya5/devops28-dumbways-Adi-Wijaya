# Web Server

## Web server ini buat config domain setiap server/app


### 1. Create Taks ansible-Playbook file name setup-gateway-ssl.yaml -> staging

```yaml

---
- hosts: gateway
  become: true

  tasks:
    - name: Install Certbot and Nginx plugin
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-dns-cloudflare
          - python3-certbot-nginx
        state: present
        update_cache: yes

    - name: Verify service nginx is running
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes

    - name: Ensure secure directory for Cloudflare credentials exists
      ansible.builtin.file:
        path: /etc/letsencrypt
        state: directory
        mode: "0700"

    - name: Deploy Cloudflare API credentials file
      ansible.builtin.copy:
        content: |
          dns_cloudflare_api_token = {{ cloudflare_api_token }}
        dest: /etc/letsencrypt/cloudflare.ini
        mode: "0600"

    # --- GENERATE WILDCARD SSL OTOMATIS VIA CLOUDFLARE DNS ---
    - name: Generate Wildcard SSL certificate using Certbot Cloudflare DNS plugin
      ansible.builtin.command:
        cmd: "certbot certonly --dns-cloudflare --dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini -d {{ domain_name }} -d {{ base_domain }} --agree-tos --email {{ admin_email }} --non-interactive"
      args:
        creates: "/etc/letsencrypt/live/{{ domain_name }}/fullchain.pem"

    # --- 2. KONFIGURASI NGINX REVERSE PROXY UNTUK KETIGA DOMAIN ---
    - name: Create configuration Nginx using templates
      ansible.builtin.template:
        src: "{{ item.src_file }}"
        dest: "/etc/nginx/sites-available/{{ item.dest_name }}"
        mode: "0644"
      loop:
        - { src_file: "registry.conf.j2", dest_name: "registry" }
        - { src_file: "staging-app.conf.j2", dest_name: "staging-app" }
        - { src_file: "api-staging.conf.j2", dest_name: "api-staging" }
      notify: Restart Nginx

    - name: Enable all Nginx sites by creating symlinks
      ansible.builtin.file:
        src: "/etc/nginx/sites-available/{{ item }}"
        dest: "/etc/nginx/sites-enabled/{{ item }}"
        state: link
      loop:
        - registry
        - staging-app
        - api-staging
      notify: Restart Nginx

    - name: Remove default Nginx site
      ansible.builtin.file:
        path: /etc/nginx/sites-enabled/default
        state: absent
      notify: Restart Nginx

    # --- 4. OTOMATISASI PERPANJANGAN SSL (AUTO-RENEWAL CRONJOB) ---
    - name: Add Cronjob for automatic Certbot renewal
      ansible.builtin.cron:
        name: "Certbot Automatic Renewal"
        minute: "0"
        hour: "3"
        job: "certbot renew --quiet --post-hook 'systemctl reload nginx'"

  handlers:
    - name: Restart Nginx
      ansible.builtin.service:
        name: nginx
        state: restarted



```

# Penjelasan Ansible Playbook Otomatisasi SSL (Certbot) & Nginx Proxy

Skrip ini mengotomatiskan penerbitan **Sertifikat SSL Wildcard gratis (HTTPS)** via Certbot Cloudflare DNS, serta melakukan konfigurasi **Nginx Reverse Proxy** untuk membagi lalu lintas data ke tiga subdomain berbeda secara aman.

---

### Variabel Utama
* `cloudflare_api_token` : Token rahasia dari akun Cloudflare Anda untuk memverifikasi kepemilikan domain.
* `domain_name`          : Nama domain utama (`adi.studentdumbways.my.id`).
* `base_domain`          : Format wildcard domain (misal: `*.dumbmerch.my.id`) untuk mencakup semua subdomain.
* `admin_email`          : Email admin untuk kebutuhan notifikasi masa aktif dari Let's Encrypt.

---

### Langkah Kerja Skrip (Tasks)

#### Tahap 1: Pengamanan Kredensial & Integrasi Cloudflare
* **Instalasi Certbot & Plugin:** Memasang *tool* `certbot` beserta plugin Cloudflare DNS dan Nginx agar sistem bisa berkomunikasi dengan Cloudflare dan Nginx secara otomatis.
* **Folder Keamanan Let's Encrypt:** Membuat direktori `/etc/letsencrypt` dengan hak akses super ketat (`0700`) agar berkas sertifikat tidak bisa diintip user lain.
* **Token Cloudflare (`cloudflare.ini`):** Menyalin token API Cloudflare Anda ke dalam file konfigurasi dengan hak akses terisolasi (`0600`) khusus untuk dibaca oleh Certbot.

#### Tahap 2: Penerbitan Sertifikat SSL Wildcard
* **Pembuatan SSL Otomatis:** Menjalankan perintah Certbot menggunakan metode tantangan DNS (*DNS Challenge*). Certbot akan otomatis membuat *TXT record* sementara di Cloudflare untuk memvalidasi domain, lalu menerbitkan sertifikat HTTPS resmi.

#### Tahap 3: Konfigurasi Nginx Reverse Proxy (Alokasi 3 Subdomain)
* **Penyusunan Template Konfigurasi:** Menyalin file template kustom `.j2` ke folder `sites-available` untuk mendaftarkan tiga subdomain utama:
  1. `registry` (untuk Private Docker Registry)
  2. `staging-app` (untuk aplikasi Frontend)
  3. `api-staging` (untuk server API Backend)
* **Aktivasi Situs & Pembersihan:** Membuat tautan aktif (*symlink*) ke folder `sites-enabled` untuk mengaktifkan ketiga situs tersebut, sekaligus menghapus konfigurasi halaman bawaan (*default*) Nginx agar tidak bentrok.

#### Tahap 4: Otomatisasi Perpanjangan SSL (Cronjob)
* **Auto-Renewal:** Menambahkan jadwal otomatis (*cronjob*) di sistem Linux yang akan berjalan setiap hari jam 03.00 subuh. Skrip ini akan mengecek masa aktif SSL, memperpanjangnya secara otomatis jika hampir kedaluwarsa, dan melakukan *reload* pada Nginx.

---

### Fungsi Handlers (Pemicu)
* **Restart Nginx:** Memastikan service Nginx di-restart untuk menerapkan file konfigurasi subdomain baru hanya jika ada perubahan pada tahap pengisian template.

## 2. foder template/file registry.conf.j2, staging-app.conf.j2, api-staging.conf.j2 dan di sini juga mengimplemtasikan *Create load balancing for frontend and backend di taks no 5

<p align="center"><img width="1155" height="521" alt="image" src="https://github.com/user-attachments/assets/d8206a90-d18a-4ec9-a15d-bcd549a50b9d" /></p>

---

<p align="center"><img width="1206" height="536" alt="image" src="https://github.com/user-attachments/assets/2fba9d79-5f89-4873-b04b-cdb8e7c4e35d" /></p>

---

<p align="center"><img width="1207" height="569" alt="image" src="https://github.com/user-attachments/assets/d634c693-feb2-4899-8b86-3ba68b1a9152" /></p>

---

## Penjelasan Teknis: Nginx Proxy Frontend Staging

Konfigurasi ini berfungsi sebagai **Reverse Proxy** dan **SSL Terminator** untuk mengarahkan trafik HTTPS eksternal ke server aplikasi internal.

---

### 1. Definisi Upstream (Target)
* Mengarahkan trafik ke kluster `frontend_cluster` pada <IP-Server> (Lokasi kontainer aplikasi Frontend berjalan).

### 2. HTTP Redirection (Port 80)
* Otomatis mengalihkan (`HTTP 301`) seluruh akses HTTP biasa pada subdomain `staging.{{ domain_name }}` ke jalur aman HTTPS.

### 3. HTTPS Server & SSL (Port 443)
* Mengaktifkan koneksi aman pada Port 443 menggunakan sertifikat SSL Wildcard Let's Encrypt (`fullchain.pem` & `privkey.pem`).

### 4. Proxy Headers (Location `/`)
* **`proxy_pass`:** Meneruskan permintaan dari browser luar langsung ke `frontend_cluster`.
* **`proxy_set_header`:** Mengirimkan metadata asli pengunjung (seperti IP Asli dan Protokol HTTPS) ke kontainer aplikasi agar sistem log membaca data secara akurat.

## 3. Configurasi DNS.cloudflare

<p><img width="1917" height="986" alt="image" src="https://github.com/user-attachments/assets/c37fb59d-fdc6-4be1-8e3e-1042aef00af8" /></p>
<p><img width="1919" height="666" alt="image" src="https://github.com/user-attachments/assets/25fba30d-38ca-4927-83d8-a8b5a87312fc" />

<p><img width="924" height="412" alt="image" src="https://github.com/user-attachments/assets/6b1c0ad3-07fa-40fc-91a9-9ba00fb6a6e0" />
</p>

# Penjelasan Teknis: Konfigurasi Cloudflare API Token (`Create Custom Token`)

Gambar di atas menampilkan proses pembuatan **Cloudflare API Token** kustom. Token ini berfungsi sebagai kredensial autentikasi aman bagi **Certbot** di server gateway untuk melakukan *DNS Challenge* (validasi kepemilikan domain) saat menerbitkan sertifikat SSL Let's Encrypt Wildcard secara otomatis.

---

### Komponen Konfigurasi Utama

#### 1. Token Name (Identitas)
* **Nama:** `finaltask-adi`
* **Fungsi:** Label pengenal internal di dasbor Cloudflare untuk mengidentifikasi tujuan penggunaan token otomatisasi ini.

#### 2. Permissions (Hak Akses Kontrol)
* **`Zone - Zone - Read`** : Izin bagi skrip luar untuk membaca data dan konfigurasi dasar pada zona domain.
* **`Zone - DNS - Edit`** : Izin krusial yang memberikan hak untuk menambah atau mengubah *DNS Records*. Hak akses ini wajib ada agar Certbot dapat menyuntikkan *TXT Record* verifikasi SSL Let's Encrypt secara otomatis.

#### 3. Zone Resources (Cakupan Wilayah)
* **Setelan:** `Include - Specific zone - [Domain Anda]`
* **Fungsi:** Menerapkan prinsip *least privilege* (hak akses minimal), yang membatasi agar token ini hanya memiliki kuasa pada satu domain spesifik yang dipilih dan tidak dapat memanipulasi domain lain di akun Cloudflare Anda.

#### 4. Pengamanan Tambahan (Opsional)
* **Client IP Address Filtering** : Opsi pembatasan hak eksekusi token agar hanya valid jika dipanggil dari alamat IP server gateway Anda.
* **TTL (Masa Berlaku)** : Menentukan batas waktu aktif token sebelum kedaluwarsa secara otomatis demi alasan keamanan.

## Daftar DNS Records Cloudflare

<p align="center"><img width="959" height="790" alt="image" src="https://github.com/user-attachments/assets/837c5bfc-d3fb-4877-91dd-c5efac2de6d2" /></p>

----

<p align="center"><img width="959" height="841" alt="image" src="https://github.com/user-attachments/assets/7710e5c9-db10-46b2-8c54-b089ae90c726" /></p>

---

<p align="center"><img width="958" height="796" alt="image" src="https://github.com/user-attachments/assets/1bc8f931-11a3-40e6-a96d-1f4bc7d664c8" /></p>

## Penjelasan Teknis: Konfigurasi DNS Cloudflare (`studentdumbways.my.id`)

Gambar tersebut menampilkan 2 **DNS Records tipe A** yang diarahkan ke IP Server Gateway Anda (**`15.232.124.118`**).

---

### Pemetaan Subdomain Aktif

*   **API Backend:** `api.staging.adi` → Diarahkan ke IP `15.232.124.118` dengan status **DNS Only Off**.
*   **Web Frontend:** `staging.adi` → Diarahkan ke IP `15.232.124.118` dengan status **DNS Only Off**
*   **Docker Registry** `registry.adi` → Diarahkan ke IP `15.232.124.118` dengan status **DNS Only Off**
---

## Run ansible-playbook setup-gateway-ssl.yaml

```bash

ansible-playbook setup-gateway-ssl.yaml

```

<p><img width="890" height="388" alt="image" src="https://github.com/user-attachments/assets/0c8db74b-a273-4266-99ea-b4af18db13f6" />
</p>
<p><img width="909" height="61" alt="image" src="https://github.com/user-attachments/assets/48f6528b-8f4b-4fa2-af89-7bb7a5fc8dd3" />
</p>
<p><img width="904" height="274" alt="image" src="https://github.com/user-attachments/assets/373d0ad5-f9e4-4a6e-86e7-3ca7a62aa9cb" />
</p>

## Success integration SSL 

<p align="center"><img width="1919" height="1035" alt="image" src="https://github.com/user-attachments/assets/209d88df-05de-4210-865d-bdb4d46012c0" /></p>
---

<p align="center"><img width="1919" height="1037" alt="image" src="https://github.com/user-attachments/assets/02ed6d50-5f0e-4c27-8f38-74bc31dbbce5" /></p>
---

<p align="center"><img width="1919" height="1037" alt="image" src="https://github.com/user-attachments/assets/88d8160b-84c9-417d-a265-3b0cf2ef9f5e" /></p>

<p align="center"></p>
<p align="center"></p>
<p align="center"></p>

