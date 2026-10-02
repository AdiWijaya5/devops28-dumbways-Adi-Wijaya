# Repository
## Menambahkan Key ke GitHub

### - generated SSH key new to terminal EC2 :

  ```bash

  ssh-keygen -t rsa -b 4096 -C "adiwijaya.jy@gmail.com"
  ```
<p align="center"><img width="960" height="186" alt="image" src="https://github.com/user-attachments/assets/2d546c80-87da-47b5-806a-8709d3469235" /></p>
<p align="center"><img width="956" height="231" alt="image" src="https://github.com/user-attachments/assets/5728e797-1498-4285-b383-babe44e94f68" /></p>

  (Salin/copy seluruh teks yang muncul, biasanya berawalan ```ssh-rsa ...``` atau ```ssh-ed25519 ...``` pindahken ke fail authorized_keys di ~/finaltask-adi/.ssh).
  
<p align="center"><img width="957" height="361" alt="image" src="https://github.com/user-attachments/assets/ddaf472e-9508-4366-82a0-577b81f1579e" /></p>

### - Daftarkan ke GitHub:
  + Buka GitHub di browser.
  + Masuk ke Settings (Pengaturan akun Anda, bukan settings repository).
  + Di menu samping kiri, klik SSH and GPG keys.
  + Klik tombol hijau New SSH key.
  + create name ```finaltask-adi``` lalu Paste isi public key yang sudah disalin tadi ke kolom Key.
  + Klik Add SSH key.

  <p align="center"><img width="1919" height="988" alt="Screenshot 2026-10-02 162149" src="https://github.com/user-attachments/assets/48641ffa-0c7a-4faa-8322-9b3d8d3bdcd7" /></p>
   <p align="center"><img width="1919" height="991" alt="image" src="https://github.com/user-attachments/assets/2b9b63fd-fee1-4d64-ab03-0724b626e800" /></p>

### - Create new repository 
  + Repository name ```fe-dumbmerch```
  + Configuration. Choose visibility ```private``` .
    
 <p align="center"><img width="1919" height="1036" alt="image" src="https://github.com/user-attachments/assets/d6ed5b3b-bec8-4962-bba5-19ff7e33a414" /></p>


  + Repository name ```be-dumbmerch```
  + Configuration. Choose visibility ```private```.
    
 <p align="center"><img width="1919" height="1037" alt="image" src="https://github.com/user-attachments/assets/9e04fb58-32da-4f1c-8cca-8a4fa0557d1f" /></p>


### Quick setup fe-dumbmerch 
  - Frontend github URL repository whit ssh. Adiwijaya5/fe-dumbmerch
  
```bash

 git@github.com:AdiWijaya5/fe-dumbmerch.git
```
 <p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/1e5a925b-5361-4e9c-a67a-17643f5e327e" /></p>

 ### Quick setup fe-dumbmerch 
  - Frontend github URL repository whit ssh. Adiwijaya5/be-dumbmerch
  
```bash

 git@github.com:AdiWijaya5/be-dumbmerch.git
```
 <p align="center"><img width="1919" height="1042" alt="image" src="https://github.com/user-attachments/assets/1847c5db-949a-45d9-bbde-88c0839fec8d" /></p>
  
## Git Clone Frontend and Backend 

- Frontend github URL
  
```bash

 git clone https://github.com/demo-dumbways/fe-dumbmerch.git
```

- Backend github URl

```bash

 git clone https://github.com/demo-dumbways/be-dumbmerch.git
```
## Create ansible-playbook name setup-repo-stag.yaml
```yaml

---
- hosts: appserver
  become: true
  become_user: finaltask-adi
  vars:
    repos:
      - name: "fe-dumbmerch"
        demo_url: "https://github.com/demo-dumbways/fe-dumbmerch.git"
        private_url: "git@github.com:AdiWijaya5/fe-dumbmerch.git"
      - name: "be-dumbmerch"
        demo_url: "https://github.com/demo-dumbways/be-dumbmerch.git"
        private_url: "git@github.com:AdiWijaya5/be-dumbmerch.git"

  tasks:
    - name: Pastikan direktori .ssh ada
      ansible.builtin.file:
        path: "/home/finaltask-adi/.ssh"
        state: directory
        mode: "0700"

    - name: Pastikan authorized_keys memiliki permission 600
      ansible.builtin.file:
        path: "/home/finaltask-adi/.ssh/authorized_keys"
        mode: '0600'

    - name: Verifikasi koneksi / Login GitHub via SSH menggunakan authorized_keys
      ansible.builtin.command:
        cmd: "ssh -T git@github.com"
      register: ssh_test
      failed_when: false
      changed_when: false

    - name: Konfigurasi global Git user
      ansible.builtin.command: "{{ item }}"
      loop:
        - "git config --global user.name 'Adiwijaya5'"
        - "git config --global user.email 'adiwijaya5699@gmail.com'"
      changed_when: false

    # --- PERBAIKAN: Loop dipasang ke setiap task menggunakan variabel item ---

    - name: Clone dari repo demo
      ansible.builtin.git:
        repo: "{{ item.demo_url }}"
        dest: "/home/finaltask-adi/{{ item.name }}"
        accept_hostkey: yes
        force: yes
        version: "master" # Pastikan clone dari branch main/master bawaan repo demo
      loop: "{{ repos }}"

    - name: Ubah remote URL ke private repository AdiWijaya5
      ansible.builtin.command:
        cmd: "git remote set-url origin {{ item.private_url }}"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true


    - name: Buat dan aktifkan branch staging secara lokal
      ansible.builtin.command:
        cmd: "git checkout -b staging || git checkout staging"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      register: checkout_staging
      failed_when: false
      changed_when: "'Switched to a new branch' in checkout_staging.stdout"


    - name: Push branch staging ke private repo menggunakan authorized_keys
      ansible.builtin.command:
        cmd: "git push -u origin staging"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true

```
## Repo baru telah berhasil dibuat.

<p align="center"><img width="1919" height="1039" alt="image" src="https://github.com/user-attachments/assets/4de74add-b226-48d8-8ab3-c974cb9dfdfa" /></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
