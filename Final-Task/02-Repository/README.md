# Repository
## 1. Menambahkan Key ke GitHub

### 1. generated SSH key new to terminal EC2 :

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

### 2. Create new repository 
  + Repository name ```fe-dumbmerch```
  + Configuration. Choose visibility ```private``` .
    
 <p align="center"><img width="1919" height="1036" alt="image" src="https://github.com/user-attachments/assets/d6ed5b3b-bec8-4962-bba5-19ff7e33a414" /></p>


  + Repository name ```be-dumbmerch```
  + Configuration. Choose visibility ```private```.
    
 <p align="center"><img width="1919" height="1037" alt="image" src="https://github.com/user-attachments/assets/9e04fb58-32da-4f1c-8cca-8a4fa0557d1f" /></p>


### 3. Quick setup fe-dumbmerch 
  - Frontend github URL repository whit ssh. Adiwijaya5/fe-dumbmerch
  
```bash

 git@github.com:AdiWijaya5/fe-dumbmerch.git
```
 <p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/1e5a925b-5361-4e9c-a67a-17643f5e327e" /></p>

 ## 2. Quick setup fe-dumbmerch 
  - Frontend github URL repository whit ssh. Adiwijaya5/be-dumbmerch
  
```bash

 git@github.com:AdiWijaya5/be-dumbmerch.git
```
 <p align="center"><img width="1919" height="1042" alt="image" src="https://github.com/user-attachments/assets/1847c5db-949a-45d9-bbde-88c0839fec8d" /></p>
  
### 4. Git Clone Frontend and Backend 

- Frontend github URL
  
```bash

 git clone https://github.com/demo-dumbways/fe-dumbmerch.git

```

- Backend github URl

```bash

 git clone https://github.com/demo-dumbways/be-dumbmerch.git

```

## 3. Create ansible-playbook name setup-repo-stag.yaml
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
        mode: "0600"

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

    - name: Buat file .env khusus untuk frontend (fe-dumbmerch)
      ansible.builtin.copy:
        content: "REACT_APP_BASEURL=https://api.staging.adi.studentdumbways.my.id/api/v1\n"
        dest: "/home/finaltask-adi/fe-dumbmerch/.env"
        mode: "0644"
      when: item.name == "fe-dumbmerch"
      loop: "{{ repos }}"

    - name: Stage semua file (git add)
      ansible.builtin.command:
        cmd: "git add ."
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true
	
    - name: Commit perubahan ke repository lokal
      ansible.builtin.command:
        cmd: "git commit -m 'Initial commit for staging'"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      register: git_commit
      failed_when: false
      changed_when: "'nothing to commit' not in git_commit.stdout"

    - name: Push branch staging ke private repo menggunakan authorized_keys
      ansible.builtin.command:
        cmd: "git push -u origin staging"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true

```

## 4. Create ansible-playbook name setup-repo-prod.yaml

```yaml

---
- hosts: appserver
  become: true
  become_user: finaltask-adi
  vars:
    repos:
      - name: "fe-dumbmerch"
        private_url: "git@github.com:Adiwijaya5/fe-dumbmerch.git"
      - name: "be-dumbmerch"
        private_url: "git@github.com:Adiwijaya5/be-dumbmerch.git"

  tasks:
    - name: Buat dan aktifkan branch production
      ansible.builtin.command:
          cmd: "git checkout -b production"
          chdir: "/home/finaltask-adi/{{ repo.name }}"
      register: checkout_prod
      failed_when: false
      changed_when: "'Switched to a new branch' in checkout_prod.stdout"

    - name: Buat file .env khusus untuk frontend (fe-dumbmerch)
      ansible.builtin.copy:
        content: "REACT_APP_BASEURL=https://api.adi.studentdumbways.my.id/api/v1\n"
        dest: "/home/finaltask-adi/fe-dumbmerch/.env"
      mode: "0644"
      when: item.name == "fe-dumbmerch"
      loop: "{{ repos }}"

    - name: Stage semua file (git add)
      ansible.builtin.command:
        cmd: "git add ."
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true
      
    - name: Commit perubahan ke repository lokal
      ansible.builtin.command:
        cmd: "git commit -m 'Initial commit for staging'"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      register: git_commit
      failed_when: false
      changed_when: "'nothing to commit' not in git_commit.stdout"

    - name: Push branch production ke private repo menggunakan authorized_keys
      ansible.builtin.command:
        cmd: "git push -u origin production"
        chdir: "/home/finaltask-adi/{{ item.name }}"
      loop: "{{ repos }}"
      changed_when: true


```

## 5. Repo frontend branch staging telah berhasil dibuat.

<p align="center"><img width="1919" height="1042" alt="image" src="https://github.com/user-attachments/assets/70fe7363-8c60-4ac4-b2ba-0c174a79656f" /></p>

### 1. berhasil menambahkan .env branch staging

<p align="center"><img width="1919" height="997" alt="image" src="https://github.com/user-attachments/assets/41fa97df-8446-47a9-b468-3f9425dc8aa1" /></p>

### 2. Repo branch production telah berhasil dibuat.

<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/849bab89-0aa0-444f-a961-2f0e7c754ec3" /></p>

### 3. berhasil menambahkan .env branch production

<p align="center"><img width="1919" height="397" alt="image" src="https://github.com/user-attachments/assets/3bf79b89-cc7b-40c9-bd5b-d3a757d3e2d4" /></p>

### 4. Frontend Branch staging dan production
Memilikin dua branch utama:

```bash
# staging
# production
```
| Branch| Environment |
| :--- | :--- |
| staging | Staging |
| production | Production |

---

<p align="center"><img width="956" height="800" alt="image" src="https://github.com/user-attachments/assets/f7c94334-0ec5-4209-b43d-efb88aba4fa1" />
</p>

---

## 6. Repo backend branch staging telah berhasil dibuat.

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/1940767d-b045-4951-b88d-d00a073a4f95" /></p>

### 1. menambahkan .env lewat remot terminal dan push branch staging

<p align="center"><img width="1919" height="687" alt="image" src="https://github.com/user-attachments/assets/a8447845-a4b2-4752-ba9f-20530a6f90f8" /></p>

### 2. Repo branch production telah berhasil dibuat.

<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/0b9d3999-f656-44fa-ba91-097c344cd441" /></p>

### 3. menambahkan .env lewat remot terminal dan push .env branch production

<p align="center"><img width="1911" height="642" alt="image" src="https://github.com/user-attachments/assets/60275189-f73e-4193-97bb-d9740541fae2" /></p>

### 4. backend Branch staging dan production
Memilikin dua branch utama:

```bash
# staging
# production
```

| Branch| Environment |
| :--- | :--- |
| staging | Staging |
| production | Production |

---

<p align="center"><img width="1915" height="691" alt="image" src="https://github.com/user-attachments/assets/2a16b9ff-a55c-414d-8751-77572eacfc7d" /></p>

## 7. Frontend Node.js dan NPM

```bash

node -v   => v22.23.3
npm -v    => 10.9.9

```

<p align="center"><img width="957" height="123" alt="image" src="https://github.com/user-attachments/assets/04f81307-4800-4dbf-a482-552a3f44859a" />
</p>

### 6. backend go 

```bash

go version    => Go 1.26.0


```


<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
