# Private Docker Registry Deployment

**Private Docker Registry** adalah server penyimpanan internal (milik sendiri) yang berfungsi untuk menyimpan, mengelola, dan mendistribusikan *Docker Images* secara mandiri, terpisah dari Docker Hub publik. 

Dalam arsitektur Anda, Registry ini dideploy pada server **gateway** menggunakan skrip otomatisasi Ansible.

---

##  add variabels to file all 

```yml 

# Variabel Docker Registry private
registry_port: 5000
registry_storage_path: "/var/lib/registry"

```

--- 

##  Create file ansible setup-docker-registry.yaml

```yaml

---
- hosts: gateway
  become: true

  tasks:
    - name: Ensure prerequisite packages are installed
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
        state: present
        update_cache: yes

    # --- 1. DOCKER INSTALLATION ---
    - name: Create Docker keyring directory
      ansible.builtin.file:
        path: /etc/apt/keyrings
        state: directory
        mode: "0755"

    - name: Add Docker's official GPG key
      ansible.builtin.shell: |
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        chmod a+r /etc/apt/keyrings/docker.gpg
      args:
        creates: /etc/apt/keyrings/docker.gpg

    - name: Set up the Docker repository using deb822
      ansible.builtin.deb822_repository:
        name: docker
        uris: https://download.docker.com/linux/ubuntu
        suites: "{{ ansible_distribution_release }}"
        components: [stable]
        signed_by: /etc/apt/keyrings/docker.gpg
        state: present

    - name: Install Docker Engine and Plugin
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

    # --- 2. PYTHON DOCKER SDK  ---
    - name: Install python3-pip and Docker SDK for Python
      ansible.builtin.apt:
        name:
          - python3-pip
          - python3-docker
        state: present

    # --- 3. DEPLOY PRIVATE DOCKER REGISTRY CONTAINER ---
    - name: Create local directory for Docker Registry storage
      ansible.builtin.file:
        path: "{{ registry_storage_path }}"
        state: directory
        mode: "0755"

    - name: Run official Docker Registry container
      community.docker.docker_container:
        name: registry
        image: registry:2
        state: started
        restart_policy: always
        published_ports:
          - "{{ registry_port }}:5000"
        volumes:
          - "{{ registry_storage_path }}:/var/lib/registry"


```


## Penjelasan
## 1. Prasyarat Sistem & Konfigurasi Repositori

* **Dependensi Paket (`apt`)**: Menginstal paket utilitas esensial (`apt-transport-https`, `curl`, `gnupg`, `lsb-release`) untuk mengaktifkan komunikasi HTTPS yang aman dan pengelolaan repositori pihak ketiga pada *host* target Ubuntu.
* **Pengelolaan GPG Key Docker**: Membuat direktori penyimpanan kunci di `/etc/apt/keyrings`, mengambil kunci kriptografi resmi Docker melalui `curl`, dan menyimpannya ke `/etc/apt/keyrings/docker.gpg` guna memverifikasi keaslian paket.
* **Pendaftaran Repositori (`deb822_repository`)**: Mengonfigurasi sumber repositori resmi Docker (`download.docker.com`) menggunakan format modern `deb822` yang ditautkan dengan GPG key terverifikasi serta dipetakan secara dinamis ke rilis distribusi sistem target.

---

## 2. Instalasi Docker Engine

* **Deployment Paket Inti (`apt`)**: Memasang rangkaian utama kontainerisasi:
  * `docker-ce` (Runtime inti Docker Engine)
  * `docker-ce-cli` (Antarmuka manajemen *command-line*)
  * `containerd.io` (Runtime manajemen siklus hidup kontainer)
  * `docker-buildx-plugin` & `docker-compose-plugin` (Ekstensi lanjutan untuk *build* dan orkestrasi)
* **Pengelolaan Status Layanan**: Memastikan layanan *daemon* Docker aktif berjalan (`started`) dan dikonfigurasi untuk menyala otomatis saat sistem *booting* (`enabled`).

---

## 3. Konfigurasi Python Docker SDK

* **Dependensi (`python3-pip` & `python3-docker`)**: Menyediakan pustaka *binding* Python pada *host* remote. Hal ini merupakan syarat wajib agar modul bawaan Ansible untuk Docker (`community.docker.docker_container`) dapat berinteraksi secara terprogram dengan *daemon* Docker lokal.

---

## 4. Deployment Private Docker Registry

* **Penyediaan Penyimpanan Persisten**: Membuat direktori lokal pada sistem berkas *host* berdasarkan variabel yang ditentukan (`{{ registry_storage_path }}` atau `/var/lib/registry`) dengan izin akses `0755`. Langkah ini memastikan *image* kontainer yang di-*push* tetap persisten di tengah siklus hidup kontainer.
* **Eksekusi Siklus Hidup Kontainer (`community.docker.docker_container`)**:
  * **`name: registry`**: Menetapkan pengidentifikasi kontainer.
  * **`image: registry:2`**: Mengambil dan menjalankan *image* resmi sumber terbuka Docker Registry.
  * **`state: started` & `restart_policy: always`**: Menjamin ketersediaan tinggi melalui kebijakan *restart* otomatis.
  * **`published_ports`**: Mengekspos pemetaan port aplikasi (`{{ registry_port }}:5000`) untuk integrasi *upstream proxy*.
  * **`volumes`**: Melakukan *mount* jalur penyimpanan lokal ke dalam direktori data internal *registry* kontainer.

---

## 4. Run ansible setup-docker-registry.yaml

  ```bash

  ansible-playbook setup-docker-registry.yaml 
  ```

<p><img width="1065" height="973" alt="image" src="https://github.com/user-attachments/assets/662ba370-e210-49eb-a004-a1e457448416" /></p>


## 5. Test Search browser

<p align="center"><img width="955" height="578" alt="image" src="https://github.com/user-attachments/assets/d1570a31-59d8-471f-8569-aa942297f585" />
</p>
