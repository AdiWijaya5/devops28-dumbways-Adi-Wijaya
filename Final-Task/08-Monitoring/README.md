## Monitoring build with ansible

Build Monitoring mengunakan anible dan memantau server all lewat Grafana Dasboard

### Install docker ke all server

```yaml

---
- name: Setup Docker
  hosts: all
  become: true

  tasks:
    - name: Update apt cache
      ansible.builtin.apt:
        update_cache: true

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

    - name: Set up Docker repository
      ansible.builtin.shell: |
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
          $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

    - name: Update apt cache after adding Docker repository
      ansible.builtin.apt:
        update_cache: true

    - name: Install Docker Engine and Python SDK
      ansible.builtin.apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
          - python3-docker
        state: present

    - name: Add user finaltask-adi to docker group
      ansible.builtin.user:
        name: "finaltask-adi"
        groups: docker
        append: true

    - name: Ensure Docker service is started and enabled
      ansible.builtin.systemd:
        name: docker
        state: started
        enabled: true


```

### Buat file deploy-monitoring.yaml

```yaml

---
- name: Setup Central Monitoring Server
  hosts: monitoring
  become: true
  vars:
    app_dir: /opt/monitoring

  tasks:
    - name: Create working directory for monitoring
      ansible.builtin.file:
        path: "{{ app_dir }}"
        state: directory
        mode: "0755"

    - name: Copy separate Prometheus configuration file
      ansible.builtin.copy:
        src: prometheus.yml
        dest: "{{ app_dir }}/prometheus.yml"
        mode: "0644"

    - name: Create Docker Compose file for Prometheus and Grafana
      ansible.builtin.copy:
        dest: "{{ app_dir }}/docker-compose.yml"
        content: |
          version: '3.8'

          volumes:
            prometheus_data:
            grafana_data:

          services:
            prometheus:
              image: prom/prometheus:latest
              container_name: prometheus
              restart: unless-stopped
              ports:
                - "9090:9090"
              volumes:
                - ./prometheus.yml:/etc/prometheus/prometheus.yml
                - prometheus_data:/prometheus
              command:
                - '--config.file=/etc/prometheus/prometheus.yml'
                - '--storage.tsdb.path=/prometheus'
                - '--storage.tsdb.retention.time=7d'

            grafana:
              image: grafana/grafana:latest
              container_name: grafana
              restart: unless-stopped
              ports:
                - "3000:3000"
              volumes:
                - grafana_data:/var/lib/grafana
              environment:
                - GF_SECURITY_ADMIN_USER=admin
                - GF_SECURITY_ADMIN_PASSWORD=admin

    - name: Run Monitoring Stack
      ansible.builtin.shell: docker compose up -d --force-recreate
      args:
        chdir: "{{ app_dir }}"

```
### Buat file prometheus.yml buat target job yang mau di monitoring


```yaml

global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - 16.79.183.142:9090

  # Memantau server monitoring
  - job_name: "monitoring-server"
    static_configs:
      - targets:
          - 16.79.183.142:9100

  # Memantau server app staging
  - job_name: "app-server"
    static_configs:
      - targets:
          - 15.232.21.109:9100

  # Memantau server database
  - job_name: "database-server"
    static_configs:
      - targets:
          - 15.232.4.76:9100

    # Memantau server gateway
  - job_name: "gateway-server"
    static_configs:
      - targets:
          - 15.232.124.118:9100

    # Memantau server app Deploy
  - job_name: "app-server-deploy"
    static_configs:
      - targets:
          - 15.232.150.220:9100

    # Memantau server database-deploy
  - job_name: "database-server-deploy"
    static_configs:
      - targets:
          - 16.78.167.23:9100

    # Memantau server gateway-deploy
  - job_name: "gateway-server-deploy"
    static_configs:
      - targets:
          - 15.232.150.220:9100


```

### Buat file deploy-node-exporter.yaml buat target data yang mau di ambil

```yaml

---
- name: Install Node Exporter on Target Servers
  hosts: all
  become: true

  tasks:
    - name: Run Node Exporter container
      ansible.builtin.docker_container:
        name: node-exporter
        image: prom/node-exporter:latest
        restart_policy: unless-stopped
        ports:
          - "9100:9100"
        volumes:
          - /proc:/host/proc:ro
          - /sys:/host/sys:ro
          - /:/rootfs:ro
        command:
          - "--path.procfs=/host/proc"
          - "--path.rootfs=/rootfs"
          - "--path.sysfs=/host/sys"
          - "--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)"

```

### Buat tebel View dengan nama CPU Usage(%)

```bash

- node_cpu_seconds_total{...mode="idle"}[5m]   => Mengambil data waktu CPU yang sedang menganggur (idle) pada instance selama 5 menit terakhir.
- rate(...)                                    => Menghitung kecepatan kenaikan data per detik
- avg(...)                                     => Merata-rata nilai idle dari seluruh core CPU yang ada
- * 100                                        => Mengubah nilai desimal menjadi persentase (0% - 100%)   
- 100 - (...)                                  => Membalikkan nilai idle menjadi tingkat penggunaan CPU (CPU Utilization) secara keseluruhan

```



<p align="center"><img width="1669" height="984" alt="image" src="https://github.com/user-attachments/assets/fd32de6f-e4b5-4b3d-b1d5-4ffb5dd9d43a" /></p>

---

### Buat tebel View dengan nama RAM Usage(%)

```bash

- node_memory_MemAvailable_bytes & node_memory_MemTotal_bytes   => Mengambil data kapasitas memori yang tersedia (available) dan total memori keseluruhan (total) pada instance 54.251.210.57:9100
- /                                                             => Menghitung rasio antara memori yang masih tersedia dengan total kapasitas memori
- 100 - (...)                                                   => Membalikkan rasio ketersediaan tersebut menjadi rasio memori yang sedang terpakai (used memory)   
- * 100                                                         => Mengonversi nilai desimal tersebut menjadi bentuk persentase (0% - 100%)

```

<p align="center"><img width="1679" height="943" alt="image" src="https://github.com/user-attachments/assets/0f404331-e16a-4385-9039-06d076adef6d" /></p>

---

### Buat tebel View dengan nama Disk Usage (%)

```bash

- node_filesystem_free_bytes{...}      => Metrik bawaan Node Exporter yang menunjukkan jumlah sisa byte disk yang kosong.
- node_filesystem_size_bytes{...}      => Metrik bawaan Node Exporter yang menunjukkan total kapasitas ukuran byte disk.
- instance=""                          => Membatasi pengambilan data hanya pada server dengan alamat IP dan port tersebut.
- job=""                               => Menyaring berdasarkan nama job yang didefinisikan di konfigurasi Prometheus.
- mountpoint="/"                       => Hanya memantau direktori utama sistem (root), sehingga mengabaikan partisi lain yang tidak relevan.
- fstype!="rootfs"                     => Mengecualikan tipe filesystem pseudo *rootfs* agar data tidak tercatat dua kali (duplikat).
 
```

<p align="center"><img width="1683" height="935" alt="image" src="https://github.com/user-attachments/assets/92d9dab6-b1a8-4386-afdb-7d69df5542da" /></p>

---

### Buat tebel View dengan nama Network Traffic In

```bash

- node_network_receive_bytes_total     => Mengukur total akumulasi data masuk sejak server menyala.
- device!~"lo|veth.*"                  => Mengabaikan jaringan internal/virtual (seperti Docker) agar yang dihitung hanya jaringan fisik asli.
- rate(...)[5m]                        => Menghitung rata-rata kecepatan kenaikan data per detik dalam rentang waktu 5 menit.
- / 1024                               => Mengubah satuan hasil dari Byte menjadi Kilobyte (KB/s) agar mudah dibaca pada grafik Grafana.

```

<p align="center"><img width="1678" height="943" alt="image" src="https://github.com/user-attachments/assets/9a8c7b96-3375-4157-95c6-a17d5c583bc4" /></p>

---

### Buat tebel View dengan nama Network Traffic Out

```bash

- node_network_transmit_bytes_total    => Mengukur total akumulasi data keluar (yang dikirim oleh server) sejak server menyala.
- device!~"lo|veth.*"                  => Mengabaikan jaringan internal atau virtual (seperti Docker) agar yang dihitung murni jaringan fisik asli server.
- rate(...)[5m]                        => Menghitung rata-rata kecepatan kenaikan data keluar per detik dalam rentang waktu 5 menit terakhir.
- / 1024                               => Mengubah satuan hasil dari Byte menjadi Kilobyte (KB/s) agar lebih mudah dibaca pada grafik Grafana.

```

<p align="center"><img width="1680" height="946" alt="image" src="https://github.com/user-attachments/assets/14072178-e2d4-432b-837d-c7de8894190e" /></p>

---

### Di Dasboard Grafana Kita dapat memantau semua server kita dan ada notifikasi telegram ketika kita tidak stay di dasboard ini 

<p align="center"><img width="1915" height="942" alt="image" src="https://github.com/user-attachments/assets/d89dda8c-433b-4db8-bfb6-71d4b29b0adc" /></p>


### Buat Alert ke Telegram

<p align="center"><img width="1909" height="981" alt="image" src="https://github.com/user-attachments/assets/e72dce38-2206-47d2-ab32-48f499e62f72" /></p>

---

### Buat notification policy di sini namanya server-notif01

<p align="center"><img width="1915" height="947" alt="image" src="https://github.com/user-attachments/assets/34bab8f4-a8df-47ab-b689-1e5a631f099b" />
<p align="center"><img width="949" height="582" alt="image" src="https://github.com/user-attachments/assets/af4bce3b-2af1-410c-bace-74f3695c8853" /></p>
<p align="center"><img width="1639" height="253" alt="image" src="https://github.com/user-attachments/assets/47df8d35-12dd-4c36-8004-1d877e1b6040" /></p>
</p>

--

### Buat Alert Rules CPU Terlalu Tinggi (> 85%)

```yaml

Kondisi (Condition): IS ABOVE 85 (Lebih dari 85)
Evaluasi setiap: 1m | Durasi (For): 1m
Ringkasan (Summary): Penggunaan CPU melebihi ambang batas normal!
Deskripsi (Description): Penggunaan CPU pada server telah mencapai di atas 85% (saat ini tinggi)
```

<p align="center"><img width="916" height="928" alt="image" src="https://github.com/user-attachments/assets/8378c90d-0e95-4323-bf36-4f02718f60c7" /></p>
<p align="center"><img width="921" height="682" alt="image" src="https://github.com/user-attachments/assets/82e917e7-ecfb-488e-bb04-6c9b5aaf5b70" /></p>

---

### Buat Alert Rules Penggunaan RAM Tinggi (> 90%)

```yaml

Kondisi (Condition): IS ABOVE 90 (Lebih dari 90)
Evaluasi setiap: 1m | Durasi (For): 1m
Ringkasan (Summary): Penggunaan Ram melebihi ambang batas normal!
Deskripsi (Description): Penggunaan RAM pada server telah melewati 90% dari total kapasitas

```

<p align="center"><img width="922" height="951" alt="image" src="https://github.com/user-attachments/assets/48465d72-fe80-4a05-bd03-adfc8d2459a1" /></p>
<p align="center"><img width="924" height="693" alt="image" src="https://github.com/user-attachments/assets/a4996cf5-f039-4453-89a5-5966f370eca9" /></p>

### Buat Alert Rules Sisa Disk Kosong (< 10 GB)

```yaml

Kondisi (Condition): IS BELOW 10 (kurang dari 10)
Evaluasi setiap: 1m | Durasi (For): 1m
Ringkasan (Summary): Sistem mendeteksi ruang penyimpanan pada server telah mencapai batas minimum.
Deskripsi (Description): Segera lakukan pembersihan file cache atau log yang tidak diperlukan agar server tidak mengalami kegagalan sistem (disk full).
```

<p align="center"><img width="922" height="911" alt="image" src="https://github.com/user-attachments/assets/8081143b-37b8-419a-9761-e55d97db7cd4" /></p>
<p align="center"><img width="925" height="672" alt="image" src="https://github.com/user-attachments/assets/b996def6-ef2c-4aca-9777-86b934a61f63" /></p>

### Buat Alert Rules Jaringan Masuk Terlalu Tinggi (Traffic full Anomali)

```yaml

Kondisi (Condition): IS ABOVE 50000000 (Lebih dari 50000000)
Evaluasi setiap: 1m | Durasi (For): 1m
Ringkasan (Summary): Trafik Jaringan Masuk (Inbound) Tinggi pada Server
Deskripsi (Description): Sistem mendeteksi lonjakan anomali pada lalu lintas jaringan masuk (inbound traffic) yang melebihi batas normal.
```

<p align="center"><img width="938" height="892" alt="image" src="https://github.com/user-attachments/assets/83a035b8-c1f5-4de2-a8c9-585b0be025d8" /></p>
<p align="center"><img width="918" height="701" alt="image" src="https://github.com/user-attachments/assets/bd7f96f6-7697-45bd-baa8-700f3aab0c4d" /></p>

---

<p align="center"><img width="1645" height="558" alt="image" src="https://github.com/user-attachments/assets/538814b3-cb1b-48be-ba4e-4a2a6ae16980" />
</p>

---

### View Notification to Telegram

<p align="center"><img width="1165" height="928" alt="image" src="https://github.com/user-attachments/assets/c65637b9-32e6-49dd-b31c-be24949458a2" /></p>



