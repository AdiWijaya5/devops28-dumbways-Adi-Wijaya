# Automation Terraform, Ansible dan Monitoring

## 1. instal Package manager Ubuntu
  ### Masukan  perintah untuk mengunduh dan mendaftarkan kunci keamanan digital resmi (GPG Key)
  
  ```
  wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
  ```
  
  <p align="center"><img width="1388" height="354" alt="image" src="https://github.com/user-attachments/assets/af78a8ea-c05a-482c-a2ce-ca082c025b70" /></p>

  ### Masukan perintah ini berfungsi untuk mendaftarkan alamat server unduhan (repositori) resmi HashiCorp ke dalam sistem Linux Ubuntu </a>

  ```
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
  ```

  ### lalu masukan perintah ini untuk mengunduh dan memasang (menginstal) aplikasi Terraform ke dalam sistem Linux secara otomatis dari internet.
  
  ```
  sudo apt update && sudo apt install terraform
  ```

  <p align="center"><img width="1042" height="690" alt="image" src="https://github.com/user-attachments/assets/cb8dbfe7-86f2-4d8c-8bd9-59da734fe087" /></p>

  ### lihat apakah terrafrom sudah terinstall 

  ```
  terraform version
  ```
  
  <p align="center"><img width="938" height="179" alt="image" src="https://github.com/user-attachments/assets/e98bbb94-d0ed-45cd-a770-7f47a1db2f01" /></p>

## 2. Buat IAM user dan requirement is to store the access key dengan nama ec2-terrafrom

  <p aligin="center"><img width="953" height="979" alt="image" src="https://github.com/user-attachments/assets/bb3a3dee-95f5-4cde-87cb-d898ea3809b2" />
</p>

  ### lalu setup set permission yang boleh adatau role apa aja yang bisa di lakukan IAM user ec2-terrafrom tersebut.

  <p align="center"><img width="942" height="981" alt="image" src="https://github.com/user-attachments/assets/35679589-c943-4cfc-8173-d301594817ed" /></p>

  ### di sini permissions summarynya lalu creat users
  
  <p align="center"><img width="942" height="980" alt="image" src="https://github.com/user-attachments/assets/06eed232-469e-450e-b73c-ec53ce8bd8f6" /></p>

  ### setelah itu hasil dari Retrieve access keys keluar

  <p align="center"><img width="1919" height="994" alt="image" src="https://github.com/user-attachments/assets/4db6a3f5-af84-46cd-b20b-af79335c8761" /></p>

## 3. Setelah itu buat struktur folder dan file sesuai tugas yang di berikan
 
  ```
  mkdir -p Automation/Terraform/aws
  touch Automation/Terraform/aws/main.tf
  touch Automation/Terraform/aws/provider.tf
  ```

  ## 📁 Struktur Direktori Project 
 
  ```text
Automation
   └── Terraform
       └── aws
          ├── main.tf
          ├── provider.tf
          ├── data.tf
          └── variables.tf
  ```
  ### Instal aws cli

  <p align="center"><img width="953" height="136" alt="image" src="https://github.com/user-attachments/assets/762bf1e6-6c39-4b8d-9e08-ff6863116e1b" /></p>

  ### Setelah terinstal 

  <p align="center"><img width="953" height="136" alt="image" src="https://github.com/user-attachments/assets/802b2421-945f-4efb-b307-c89443959f7c" /></p>
  
  ### Lalu buat code provider.tf (Buat jalan perintah server berjalan di region aws singapura)

  ```
  "ap-southeast-1"
  ```

  <p align="center"><img width="613" height="320" alt="image" src="https://github.com/user-attachments/assets/93ed5426-472d-47de-a1f7-ec9eb466bc72" /> </p>


  ### Buat code main.tf (Buat jalankan perintah buat server aws dengan spec OS ubuntu 24 dan debian 11)

  ada juga configuration VPC, Gateway, dan subner, serta ada security grupp yang saya setung untuk inbound port 1 - 1000 jadi 
  allow Tcp dan utuk outbound nya juga, buat 2 block storage lalu attach block storage.

  <p align="center"><img width="677" height="805" alt="image" src="https://github.com/user-attachments/assets/c8d1adbc-375b-4f2e-b257-55632a193644" /></p>
  <p align="center"><img width="677" height="805" alt="image" src="https://github.com/user-attachments/assets/6f7e5a31-7886-4064-a310-c8ab3a30be47" /></p>
  <p align="center"><img width="677" height="805" alt="image" src="https://github.com/user-attachments/assets/dcd701fa-e7b9-4192-a5bc-6e8f7fbfac50" /></p>
  <p align="center"><img width="677" height="805" alt="image" src="https://github.com/user-attachments/assets/56c308d3-7d89-420d-825d-69c0dd8cb4f9" /></p>
  <p align="center"><img width="677" height="805" alt="image" src="https://github.com/user-attachments/assets/f359e357-81c1-4ccd-8708-f1a0fd45afa2" /></p>
 
  ### buat file baru data.tf yang berisikan data dari aws ubuntu versi dan debian versi yang akan di panggil di main.tf

  <p align="center"><img width="822" height="577" alt="image" src="https://github.com/user-attachments/assets/6e8c6753-0020-4d8f-b581-fa3f1ff2b93c" /></p>

  ### buat file baru variables.tf (membuat variabel cidr_bloc, subnet, instance_type dan key server yang sudah ada sebelumnya)

  <p align="center"><img width="822" height="364" alt="image" src="https://github.com/user-attachments/assets/aa140c69-34a4-4d37-82cf-2e35cac634e3" /></p>

  ### buat file output untuk menampilkan hasil dari pembuatan elastic ip

  <p align="center"><img width="822" height="152" alt="image" src="https://github.com/user-attachments/assets/4f3d5b59-df37-4e7c-a77c-1c99dc4e2d44" /></p>

  ### Langkah Selanjutnya configure IAM credentials to authenticate 

  <p align="center"><img width="663" height="201" alt="image" src="https://github.com/user-attachments/assets/8f698ba0-d7b5-445d-a5d0-8440686922ad" /></p>

  ### Kita cek kembali configure IAM credentials kemali dengan comment

  ```
  aws configure list
  ```

  ### akan menampilkan configure aws IAM credentials 

  <p align="center"><img width="661" height="173" alt="image" src="https://github.com/user-attachments/assets/d93ba4a8-be26-41ef-a5b0-93b18fbe74d4" /></p>

  ### Setelah setup semua selesai lakukan terrafrom init

  <p align="center"><img width="657" height="200" alt="image" src="https://github.com/user-attachments/assets/5cd24a90-36a4-4529-a458-bd6866f40776" /></p>
  <p align="center"><img width="657" height="184" alt="image" src="https://github.com/user-attachments/assets/ce411ceb-c637-4179-95b7-900de61200f3" /></p>

  ###  Setelah itu lalukan terrafrom plan untuk informasi apa aja yang sudah di update terrafrom 

  <p align="center"><img width="658" height="261" alt="image" src="https://github.com/user-attachments/assets/4ccbfd72-3d4f-431d-880f-7ee8708dee12" /></p>

  ### Setelah itu lakukan terrafrom apply

  <p align="center"><img width="671" height="510" alt="image" src="https://github.com/user-attachments/assets/1780a204-3b53-4f29-ae2d-870769b44367" /></p>

  ### Setelah terrafrom apply server akan terbuat 

  <p align="center"><img width="1915" height="285" alt="image" src="https://github.com/user-attachments/assets/526ea62a-8b54-4be3-9329-67d173f90d4a" /></p>

  ### Setelah berhasil di buat kita tes SSH/Remot di terminal local PC/Leptop kita untuk ubuntu dan debian

  <p align="center"><img width="826" height="631" alt="image" src="https://github.com/user-attachments/assets/e0efadd4-0fff-42f2-9487-9cffbdad3bf9" /></p>
  <p align="center"><img width="955" height="291" alt="image" src="https://github.com/user-attachments/assets/639b2167-d1cf-4e63-a739-83117500b154" /></p>

## 4. Ansible instruksi untuk mengotomatisasi server

  ### Instalasi Ansible pada Control Node:
  + Untuk pengguna Ubuntu 
    
    ```
    sudo apt update
    sudo apt install software-properties-common
    sudo add-apt-repository --yes --update ppa:ansible/ansible
    sudo apt install ansible -y
    ```
    <p align="center"><img width="956" height="931" alt="image" src="https://github.com/user-attachments/assets/c381e489-2c4c-4941-9f30-3b668b158034" /></p>
    <p align="center"><img width="953" height="464" alt="image" src="https://github.com/user-attachments/assets/f69f9cdc-ade5-479a-83b4-96c2b19fbc65" /></p

  + Verifikasi Instalasi Ansible

    <p align="center"><img width="949" height="315" alt="image" src="https://github.com/user-attachments/assets/18e3e18c-f318-4364-b76d-10e7dc27f4b9" /></p>

  ### Setelah itu buat Folder grup_var dan di dalamya ada file all.yaml dan webservers.yaml

  + Di dalam file all.yaml berisikan konfigurasi SSH, Variabel Global Aplikasi & User
  + Variabel konfigurasi SSL certificate, dan Variabel Konfigurasi Docker & Ap

  ```
  all.yaml
  ---
# Konfigurasi SSH & Koneksi Default
ansible_port: 22
ansible_user: ubuntu

# Variabel Global Aplikasi & User
app_user: "ady"
app_user_password_hash: "$6$JVoSl9OOU*****"
ssh_public_key: "ssh-rsa AAAAB3NzaC1y*****"

domain_config: "domain.conf"
domain_name: "adiwijaya.studentdumbways.my.id"
admin_email: "adiwijaya.jy@ygmail.com"

# Konfigurasi Docker & App
docker_image: "adiwijayajy/wayshub-frontend:prod"
container_name: "wayshub-fe"
app_port: 3000
nodejs_port: 3000

  ```

  + dan untuk webvariabel.yaml khusus untuk menimpa variabel default untuk debian_server

  ```
  webvariabel.yaml
  ---
  host_flavors:
    debian_server:
      ansible_user: admin
      ansible_ssh_private_key_file: /home/adi/.ssh/adi-key.pem
  ```

  ### buat file Inventory sebagai daftar host yang akan dikelola dan diautomasi oleh Ansible

  + Mengenali Target Server: Memberitahu Ansible alamat IP atau nama host dari mesin-mesin yang ingin di akses
  + Pengelompokan [Grouping]: Mengorganisir host ke dalam grup tertent agar playbook bisa dijalankan secara spesifik
  + Penyimpanan Variabel: Menyimpan konfigurasi khusus atau variabel untuk host/grup
    
  ```
  Inventory
  ---
  [webservers]
  13.251.123.17 
  54.251.210.57
  
  [gateway]
  47.130.163.228
  
  [monitoring]
  13.213.92.132
  
  [all:vars]
  ansible_user=ady
  ansible_python_interpreter=/usr/bin/python3
  ansible_config=ansible.cfg
  ```

  ### buat file ansible.cfg. untuk konfigurasi utama Ansible dalam mengatur perilaku dan setelan default saat playbook dijalankan

  ```
  [defaults]
  inventory = Inventory
  private_key_file = ~/.ssh/adi-key.pem
  host_key_checking = False
  remote_user = ubuntu
  ansible_python_interpreter = /usr/bin/python3
  ```

  
  ### buat file creat_user.yaml dan bisa login dengan ssh key & password buat server yang sudah di buat di terrafrom

  ```
  create-user.yaml
  ---
- name: Create New User and Setup Access
  hosts: all
  become: true

  tasks:
    - name: Create user new with password enkripsi and group
      ansible.builtin.user:
        name: "{{ app_user }}"
        password: "{{ app_user_password_hash }}"
        shell: /bin/bash
        create_home: true
        groups: sudo
        append: true

    - name: Ensure .ssh directory exists with correct permissions
      ansible.builtin.file:
        path: "/home/{{ app_user }}/.ssh"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: "0700"

    - name: Add SSH Public Key for secure login (.pem key)
      ansible.builtin.authorized_key:
        user: "{{ app_user }}"
        state: present
        key: "{{ ssh_public_key }}"

    - name: Enable password authentication in SSH configuration
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^PasswordAuthentication"
        line: "PasswordAuthentication yes"
        state: present
      notify: Restart SSH

    - name: Enable SSH kayboard-interactive authentication in SSH configuration
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^KbdInteractiveAuthentication"
        line: "KbdInteractiveAuthentication yes"
        state: present
      notify: Restart SSH

    - name: Setup hostname
      ansible.builtin.hostname:
        name: "webserver"
      when: "'webservers' in group_names"

    - name: Setup hostname for gateway server
      ansible.builtin.hostname:
        name: "gateway"
      when: "'gateway' in group_names"

    - name: Setup hostname for monitoring server
      ansible.builtin.hostname:
        name: monitoring
      when: "'monitoring' in group_names"

  handlers:
    - name: Validate SSH configuration
      ansible.builtin.command: sshd -t
      changed_when: false
      listen: Restart SSH

    - name: Restart SSH
      ansible.builtin.service:
        name: ssh
        state: restarted
      become: true

  ```
  ### untuk memeriksa konektivitas dan memastikan Ansible dapat terhubung serta melakukan autentikasi ke seluruh host yang terdaftar di dalam file Inventory mengunakan code

  ```
  ansible all -m ping  
  ```

  <p align="center"><img width="792" height="349" alt="image" src="https://github.com/user-attachments/assets/8a3d19fb-c301-429a-8314-511e8d960c9c" /></p>

  ### Setelah itu jalan kan perintah seperti dibawah ini intuk membuat user baru yang bisa masuk ke server dengan password dan SSH Keys serta Setup hostname sesuai Host Inventory

  +  ansible-playbook => Perintah dasar untuk menjalankan modul Ansible
  +  create-user.yaml => Nama file skrip berformat YAML yang berisi definisi konfigurasi  (tasks) yang ingin dijalankan 
  
  ```
  ansible-playbook create-user.yaml
  ```

  dan setelah di Jalankan Hasilnya

  <p align="center"><img width="981" height="139" alt="image" src="https://github.com/user-attachments/assets/0d9d7e03-d25d-4686-83bb-ca0d985f50b9" /></p>
  
  ### Buat file instalasi-docker.yaml. untuk menginstal docker ke seluruh server dan Frontend App

  ```
  instalasi-docker.yaml
  ---
  - name: Install Docker and Deploy Frontend Application
  hosts: all
  become: true

  tasks:
    - name: Update apt and install prerequisites
      apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-pip
        state: present
        update_cache: yes

    - name: Set up Docker repository & Add Docker official GPG key
      deb822_repository:
        name: docker
        uris: "https://download.docker.com/linux/ubuntu"
        suites: "{{ ansible_facts['distribution_release'] }}"
        components: ["stable"]
        signed_by: "https://download.docker.com/linux/ubuntu/gpg"
        state: present

    - name: Install Docker Engine
      apt:
        name:
          - docker-ce
          - docker-ce-cli
          - containerd.io
          - docker-compose-plugin
        state: present
        update_cache: true

    - name: Ensure Docker service is running and enabled
      service:
        name: docker
        state: started
        enabled: true

    - name: use app_user to group docker
      user:
        name: "{{ app_user }}"
        groups: docker
        append: true

    - name: Pull frontend image from Docker Hub
      docker_image:
        name: "{{ docker_image }}"
        source: pull

    - name: Restart Docker service to clear cache/locks
      ansible.builtin.systemd:
        name: docker
        state: restarted

    - name: Run frontend container
      docker_container:
        name: "{{ container_name }}"
        image: "{{ docker_image }}"
        state: started
        restart_policy: always
        published_ports:
          - "{{ app_port }}:3000"


  ```

  ### Setelah Terinstal Docker lalu buat file Instal-nginx.yaml untuk server gateway sebagai reverse-proxy

  ```
  instal-nginx.yaml
  ---
- name: Install NGINX
  hosts: gateway
  become: true

  tasks:
    - name: Instaling Nginx
      ansible.builtin.apt:
        name: nginx
        state: latest
        update_cache: true

    - name: Copying Reverse Proxy Configuration to Remote Server
      ansible.builtin.copy:
        src: /home/adi/rproxy/domain.conf
        dest: /etc/nginx/sites-available/rproxy.conf
        owner: ady
        group: ady
        mode: "0644"

    - name: Enable site by creating symlink
      ansible.builtin.file:
        src: /etc/nginx/sites-available/rproxy.conf
        dest: /etc/nginx/sites-enabled/rproxy.conf
        state: link

    - name: Restart Nginx service
      ansible.builtin.systemd_service:
        state: restarted
        daemon_reload: true
        name: nginx

  ```

  ### Membuat file intall-certbot.yaml buat Generated SSL certificate

  ```
  install-certbot.yaml
  ---
- name: Install and Configure Certbot SSL
  hosts: gateway
  become: true

  tasks:
    - name: Install Certbot and python3-certbot-nginx
      ansible.builtin.apt:
        name:
          - certbot
          - python3-certbot-nginx
        state: present
        update_cache: true

    - name: Generate SSL Certificates with Certbot
      ansible.builtin.command: >
        certbot --nginx --non-interactive --agree-tos -m adiwiajaya.jy@gmail.com
        -d adiwijaya.studentdumbways.my.id
      args:
        creates: /etc/letsencrypt/live/adiwijaya.studentdumbways.my.id/fullchain.pem

  ```

  ### konfigurasi rproxy

  <p align="center"><img width="738" height="233" alt="image" src="https://github.com/user-attachments/assets/b71280a0-d189-47b8-977b-e3c52958ea8b" /></p>
  
  ## 5. Membuat folder dockermonitoringyeng berisikan compose-monitoring.yml dan node-exporter.yml

  + compose-monitoring.yml. Membuat buat docker images grafana dan prometheus dengan coustem volumes
    
    ```
    compose-monitoring.yml
    ---
    services:
    grafana:
      container_name: grafana
      restart: always
      expose:
        - "3000"
      image: grafana/grafana
      ports:
        - "3001:3000"
      stdin_open: true
      volumes:
        - "grafana-data:/var/lib/grafana"
  
    prometheus:
      container_name: prometheus
      restart: always
      expose:
        - "9090"
      image: prom/prometheus
      ports:
        - "9090:9090"
      stdin_open: true
      volumes:
        - "/home/ady/monitoring/prometheus.yml:/etc/prometheus/prometheus.yml"
        - "prometheus-data:/prometheus"
      command:
        - "--config.file=/etc/prometheus/prometheus.yml"
  
    volumes:
      prometheus-data:
      grafana-data:

    ```

+ dan node-exporter.yml. untuk membuat docker images node exporter
   
    ```
    version: "3.8"

    services:
      node-exporter:
        command:
          - "--web.listen-address=:9100"
          - "--path.procfs=/host/proc"
          - "--path.sysfs=/host/sys"
          - "--path.rootfs=/host"
          - "--collector.filesystem.ignored-mount-points=^/(sys|proc|dev|host|etc|var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)'"
        expose:
          - "9100"
        image: prom/node-exporter
        restart: always
        ports:
          - "9100:9100"
        volumes:
          - "/proc:/host/proc:ro"
          - "/sys:/host/sys:ro"
          - "/:/host:ro"

    ```

  ### Setelah membuat docker images buat monitoring kita buat config prometheus buat target dan job manah yang ingin kita lihat

  ```
  scrape_configs:
  - job_name: "wayshubs"
    scrape_interval: 5s
    static_configs:
      - targets:
          - 54.251.210.57:9100
          - 13.213.92.132:9100
  - job_name: ady
    scrape_interval: 5s
    static_configs:
      - targets:
          - 54.251.210.57:9100
          - 13.213.92.132:9100

  ```
      
  ### membuat taks (ansible-playbook) nginx-monitoring.yaml karena tadi udah bikin khusus buat frontend app sekarang tambahkan buat monitoring dan Generated SSL certificate

  ```
  ---
  - name: Configure Nginx Reverse Proxy monitoring
    hosts: gateway
    become: true
  
    tasks:
      - name: Copying Reverse Proxy Configuration to Remote Server
        ansible.builtin.copy:
          src: /home/adi/monitoring/domain.conf
          dest: /etc/nginx/sites-available/monitoring.conf
          owner: ady
          group: ady
          mode: "0644"
  
      - name: Enable site by creating symlink
        ansible.builtin.file:
          src: /etc/nginx/sites-available/monitoring.conf
          dest: /etc/nginx/sites-enabled/monitoring.conf
          state: link
  
      - name: Install Certbot and python3-certbot-nginx
        ansible.builtin.apt:
          name:
            - certbot
            - python3-certbot-nginx
          state: present
          update_cache: true
  
      - name: Generate SSL Certificates with Certbot
        ansible.builtin.command: >
          certbot --nginx --non-interactive --agree-tos -m adiwiajaya.jy@gmail.com
          -d exporter-adiwijaya.studentdumbways.my.id
          -d prom-adiwijaya.studentdumbways.my.id
          -d monitoring-adiwijaya.studentdumbways.my.id
        args:
          creates: /etc/letsencrypt/live/exporter-adiwijaya.studentdumbways.my.id/fullchain.pem
  
      - name: Restart Nginx service
        ansible.builtin.systemd_service:
          state: restarted
          daemon_reload: true
          name: nginx

  ```

  ### dan konfigurasi proxy dan generate ssl certbot

  <p align="center"><img width="615" height="616" alt="image" src="https://github.com/user-attachments/assets/0015a401-9f9a-46b4-8d79-8a593fa1a552" /></p>

  ### buat file (taks) start-exporter.yaml untuk menjalankan/mendeploy aplikasi exporter "penarik metrik sistem/aplikasi" agar data performanya bisa dikumpulkan dan dibaca oleh server monitoring

  ```
  start-exporter.yaml
  ---
  - name: "Installing Exporter"
    hosts: webservers, monitoring
    become: true
    vars:
      remote: "/home/ady/monitoring"
      local: "/home/adi/Automation/Ansible"
  
    tasks:
      - name: Create monitoring directory
        ansible.builtin.file:
          path: "{{ remote }}"
          state: directory
          owner: "ady"
          group: "ady"
          mode: "0755"
  
      - name: Sending compose config
        ansible.builtin.copy:
          src: "{{ local }}/dockermonitoring/node-exporter.yml"
          dest: "{{ remote }}/node-exporter.yaml"
          owner: "ady"
          group: "ady"
          mode: "0644"
  
      - name: Restart Docker service
        ansible.builtin.systemd:
          name: docker
          state: restarted
  
      - name: "Running docker-compose"
        community.docker.docker_compose_v2:
          files:
            - node-exporter.yaml
          project_name: monitoring
          project_src: "{{ remote }}"
          state: present

  ```

  ### Buat file (taks) start-monitoring.yaml untuk mengotomatiskan instalasi dan menjalankan layanan pemonitoran melewati Grafana & Prometheus di server menggunakan Docker Compose.

  ```
  ---
  - name: Installing Server Monitoring
    hosts: monitoring
    become: true
    vars:
      remote: "/home/ady/monitoring"
      local: "/home/adi/Automation/Ansible"
  
    tasks:
      - name: Sending prometheus config
        ansible.builtin.copy:
          src: "{{ local }}/config/prometheus.yml"
          dest: "{{remote}}/prometheus.yml"
          owner: "ady"
          group: "ady"
          mode: "0644"
  
      - name: Send compose-monitoring.yml
        ansible.builtin.copy:
          src: "{{ local }}/dockermonitoring/compose-monitoring.yml"
          dest: "{{remote}}/compose-monitoring.yml"
          owner: "ady"
          group: "ady"
          mode: "0644"
  
      - name: Restart Docker service to clear cache/locks
        ansible.builtin.systemd:
          name: docker
          state: restarted
  
      - name: Start monitoring containers (Docker Up)
        community.docker.docker_compose_v2:
          files:
            - compose-monitoring.yml
          project_name: monitoring
          project_src: "{{ remote }}"
          enabled: true
          state: present

  ```

## 6.  Monitoring dengan Grafana

  ### Setelah berhasil automasi ansible start-monitoring.yaml

  <p align="center"><img width="1565" height="458" alt="image" src="https://github.com/user-attachments/assets/5543ce64-073d-4d31-821f-e1b252f184f2" /></p>

  ### Masuk ke domain yang sudah berjalan

  <p align="center"><img width="1916" height="1034" alt="image" src="https://github.com/user-attachments/assets/7e421741-c592-44df-b20d-3964d174d166" /></p>
  
  ### lalu masuk setelah itu add user dan password dan berikan permissions

  <p align="center"><img width="1566" height="954" alt="image" src="https://github.com/user-attachments/assets/9197fc26-a17a-4318-89ea-11a0890be457" /></p>

  <p align="center"><img width="1560" height="938" alt="image" src="https://github.com/user-attachments/assets/48d088f7-ec4b-4d80-a469-341ea0936b35" /></p>

  ### Langkah selanjutnya sebelum create Dasboard koneksikan terlebih dahulu untuk prometheus nya

  <p align="center"><img width="1560" height="951" alt="image" src="https://github.com/user-attachments/assets/3145181b-4a3c-42cf-99ae-4e4bf163bb8d" /></p>
  <p align="center"><img width="922" height="848" alt="image" src="https://github.com/user-attachments/assets/77811b63-6da4-4fa6-b78b-3370d0b4b285" /></p>
  
  ### Buat tebel View dengan nama CPU Usage(%)
  
  + Dokumentasi Query
    
    ```
    Persentase Penggunaan CPU (CPU Usage(%))
    -----
    100 - (avg(rate(node_cpu_seconds_total{instance="54.251.210.57:9100",job="wayshubs",mode="idle"}[5m])) * 100)
    -----
    - node_cpu_seconds_total{...mode="idle"}[5m] => Mengambil data waktu CPU yang sedang menganggur (idle) pada instance 54.251.210.57:9100 selama 5 menit terakhir.
    - rate(...)                                  => Menghitung kecepatan kenaikan data per detik
    - avg(...)                                   => Merata-rata nilai idle dari seluruh core CPU yang ada
    - * 100                                      => Mengubah nilai desimal menjadi persentase (0% - 100%)   
    - 100 - (...)                                => Membalikkan nilai idle menjadi tingkat penggunaan CPU (CPU Utilization) secara keseluruhan
    ```

  <p><img width="852" height="711" alt="image" src="https://github.com/user-attachments/assets/5414f931-67ce-491c-a34b-26501d284c2a" /></p>
    
  ### Buat table View dengan nama Disk Usage (%)

  + Dokumentasi Query
    
  ```
  Persentase Penggunaan Disk (Disk Usage)
  -----
  (1 - (node_filesystem_free_bytes{instance="54.251.210.57:9100",job="wayshubs",mountpoint="/",fstype!="rootfs"} / node_filesystem_size_bytes{instance="54.251.210.57:9100",job="wayshubs",mountpoint="/",fstype!="rootfs"})) * 100
  -----
  - node_filesystem_free_bytes & node_filesystem_size_bytes => Mengambil data sisa kapasitas disk yang kosong (free) dan total kapasitas disk (size) pada instance 54.251.210.57:9100 dengan titik mount (mountpoint) di /
  - fstype!="rootfs"                                        => digunakan untuk mengecualikan tipe filesystem rootfs duplikat agar data yang dihitung tidak ganda
  -  /                                                      => Membagi sisa ruang kosong dengan total kapasitas untuk menghasilkan rasio desimal sisa ruang penyimpanan
  - 1 - (...)                                               => Membalikkan nilai sisa ruang kosong menjadi rasio ruang disk yang terpakai (used space)
  - * 100                                                   => Mengonversi nilai desimal tersebut menjadi bentuk persentase (0% - 100%)
  ```

  <p align="center"><img width="1570" height="959" alt="image" src="https://github.com/user-attachments/assets/4b0b6fa8-f154-466f-ad32-9ede559bb4b2" /></p>

  ### Buat tebel View dengan nama RAM Usage(%)
  
  + Dokumentasi Query
    
    ```
    Persentase Penggunaan Memori (RAM Usage)
    -----
    (1 - (node_memory_MemAvailable_bytes{instance="54.251.210.57:9100",job="wayshubs"} / node_memory_MemTotal_bytes{instance="54.251.210.57:9100",job="wayshubs"})) * 100
    -----
    - node_memory_MemAvailable_bytes & node_memory_MemTotal_bytes => Mengambil data kapasitas memori yang tersedia (available) dan total memori keseluruhan (total) pada instance 54.251.210.57:9100
    - /                                                           => Menghitung rasio antara memori yang masih tersedia dengan total kapasitas memori
    - 100 - (...)                                                 => Membalikkan rasio ketersediaan tersebut menjadi rasio memori yang sedang terpakai (used memory)   
    - * 100                                                       => Mengonversi nilai desimal tersebut menjadi bentuk persentase (0% - 100%)
    ```
    <p align="center"><img width="1564" height="957" alt="image" src="https://github.com/user-attachments/assets/89cfed49-cf43-4f6c-8af3-1ffa26a3a95b" /></p>
    
  ### Selanjutnya Setup Contact Point Discord dengan coustom massage
  + buka menu Alerting > Contact points > Add contact point
  + beri nama "Notif-monitoring-fe"
  + Pada Integration, pilih Discord
  + Masukkan URL Webhook
  + Klik pada bagian Optional settings untuk membuka pengaturan kustom pesan,
    lalu sesuaikan kolom Message dengan format kustom:
    
  <p align="center"><img width="1563" height="955" alt="image" src="https://github.com/user-attachments/assets/cf5e74ee-0575-4fce-a2b8-840a7dd4e95c" /></p>  

  ### Buat alert rule CPU Usage over 20%

  ```
  Dengan rule name CPU Usage over 20%
  100 - (avg(rate(node_cpu_seconds_total{instance="54.251.210.57:9100",job="wayshubs",mode="idle"}[5m])) * 100)
  Kondisi (Condition): IS ABOVE 20 (Lebih dari 20)
  Evaluasi setiap: 1m | Durasi (For): 1m
  Ringkasan (Summary): Penggunaan CPU melebihi ambang batas normal!
  Deskripsi (Description): Penggunaan CPU pada server 13.213.92.132 telah mencapai di atas 20% (saat ini tinggi)
  ```
  
  <p align="center"><img width="1389" height="831" alt="image" src="https://github.com/user-attachments/assets/4144aa9c-cd40-4624-a0b6-2512ae14ceba" /></p>
  <p align="center"><img width="1374" height="715" alt="image" src="https://github.com/user-attachments/assets/f0c02a52-99fe-4c44-ad6f-4289ddb55677" /></p>
  <p align="center"><img width="1383" height="902" alt="image" src="https://github.com/user-attachments/assets/88a650b2-9e5c-4278-9557-3e6485a24513" /></p>

  ### Buat alert rule RAM Usage over 75%

  ```
  Dengan rule name RAM Usage over 75%
  (1 - (node_memory_MemAvailable_bytes{instance="13.213.92.132:9100",job="wayshubs"} / node_memory_MemTotal_bytes{instance="13.213.92.132:9100",job="wayshubs"})) * 100
  Kondisi (Condition): IS ABOVE 75 (Lebih dari 75)
  Evaluasi setiap: 1m | Kapasitas RAM server kritis!
  Ringkasan (Summary): Penggunaan CPU melebihi ambang batas normal!
  Deskripsi (Description): Penggunaan RAM pada server 13.213.92.132 telah melewati 75% dari total kapasitas
  ```

  <p align="center"><img width="1455" height="956" alt="image" src="https://github.com/user-attachments/assets/b26bcf43-c944-49fa-ad29-be0dbbf71477" /></p>
  <p align="center"><img width="1565" height="910" alt="image" src="https://github.com/user-attachments/assets/97ae7c86-8d6d-4e4e-9691-97e3a04384e7" /></p>
  <p align="center"><img width="1570" height="913" alt="image" src="https://github.com/user-attachments/assets/1ec1dff8-34f2-41fb-b95f-785c841dcd5c" /></p>

  ### Dan sudah terkoneksi Discord

  <p align="center"><img width="677" height="405" alt="image" src="https://github.com/user-attachments/assets/8e46bb41-e005-4774-adc8-5c7459c3c36d" />
</p>
  

  
