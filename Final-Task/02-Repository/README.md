# Repository
## Menambahkan Key ke GitHub

### - Extract the public IP outputs and the generated public key:

  ```bash

  terraform output generated_public_key
  ```
<p align="center"><img width="960" height="425" alt="image" src="https://github.com/user-attachments/assets/2ffa5530-f158-4f12-8d81-07301644011b" /></p>

  (Salin/copy seluruh teks yang muncul, biasanya berawalan ```ssh-rsa ...``` atau ```ssh-ed25519 ...```).

### - Daftarkan ke GitHub:
  + Buka GitHub di browser.
  + Masuk ke Settings (Pengaturan akun Anda, bukan settings repository).
  + Di menu samping kiri, klik SSH and GPG keys.
  + Klik tombol hijau New SSH key.
  + create name ```finaltask-adi``` lalu Paste isi public key yang sudah disalin tadi ke kolom Key.
  + Klik Add SSH key.

  <p align="center"><img width="1919" height="990" alt="image" src="https://github.com/user-attachments/assets/fe859c65-0e7f-4967-9b01-93b3eb689a05" /></p>
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
- hosts: appservers
  become: true
  become_user: finaltask-adi
  vars:
    # Path ke jay-key di dalam Ubuntu Server
    ssh_key_path: "/home/finaltask-adi/.ssh/jay-key"
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
        mode: '0700'

    - name: Pastikan jay-key memiliki permission 600
      ansible.builtin.file:
        path: "{{ ssh_key_path }}"
        mode: '0600'

    - name: Tambahkan github.com ke known_hosts untuk menghindari prompt SSH
      ansible.builtin.known_hosts:
        name: "github.com"
        key: "{{ lookup('pipe', 'ssh-keyscan github.com') }}"
        path: "/home/finaltask-adi/.ssh/known_hosts"

    - name: Verifikasi koneksi / Login GitHub via SSH menggunakan jay-key
      ansible.builtin.command:
        cmd: "ssh -T -i {{ ssh_key_path }} -o StrictHostKeyChecking=no git@github.com"
      register: ssh_test
      failed_when: false # GitHub mengembalikan exit code 1 saat berhasil auth (Hi username!), jadi diizinkan
      changed_when: false

    - name: Konfigurasi global Git user (opsional tapi disarankan untuk push)
      ansible.builtin.command: "{{ item }}"
      loop:
        - "git config --global user.name 'Adiwijaya5'"
        - "git config --global user.email 'adiwijaya5699@gmail.com'"
      changed_when: false

    - name: Proses setup repository di Ubuntu Server menggunakan jay-key
      loop: "{{ repos }}"
      loop_control:
        loop_var: repo
      block:
        - name: Clone dari repo demo ({{ repo.name }})
          ansible.builtin.git:
            repo: "{{ repo.demo_url }}"
            dest: "/home/finaltask-adi/{{ repo.name }}"
            accept_hostkey: yes
            force: yes

        - name: Ubah remote URL ke private repository Adiwijaya5
          ansible.builtin.command:
            cmd: "git remote set-url origin {{ repo.private_url }}"
            chdir: "/home/finaltask-adi/{{ repo.name }}"
          changed_when: true

        - name: Set SSH key server khusus untuk git command di repository ini
          ansible.builtin.command:
            cmd: "git config core.sshCommand \"ssh -i {{ ssh_key_path }} -o IdentitiesOnly=yes\""
            chdir: "/home/finaltask-adi/{{ repo.name }}"
          changed_when: true

        - name: Buat dan aktifkan branch staging
          ansible.builtin.command:
            cmd: "git checkout -b staging"
            chdir: "/home/finaltask-adi/{{ repo.name }}"
          register: checkout_staging
          failed_when: false
          changed_when: "'Switched to a new branch' in checkout_staging.stdout"

        - name: Push branch staging ke private repo menggunakan jay-key
          ansible.builtin.command:
            cmd: "git push -u origin staging"
            chdir: "/home/finaltask-adi/{{ repo.name }}"
          changed_when: true


```



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
<p align="center"></p>
