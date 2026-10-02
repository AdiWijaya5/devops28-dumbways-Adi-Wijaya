# SERVER 
## AWS Infrastructure Automation & Configuration (Terraform & Ansible)


## 1. Provision Infrastructure with Terraform
  - Initialize and apply the Terraform configurations to spin up the VPC, subnets, security groups, and EC2 instances:
    
  ```bash

  terraform init
  terrafrom plan
  terraform apply
  ```

  - Extract the public IP outputs and the generated public key:

  ```bash

  terraform output generated_public_key
  ```
<p align="center"><img width="960" height="425" alt="image" src="https://github.com/user-attachments/assets/2ffa5530-f158-4f12-8d81-07301644011b" /></p>

  
## 2. Config Ansible
### - add Konfigurasi Ansibel file all.yaml
  ```yaml
  ---
# Konfigurasi SSH & Koneksi Default
ansible_port: 22
ansible_user: ubuntu

# Konfigurasi Ansibel 
target_user: "finaltask-adi"
user_password_hash: "$6$JVoSl9OOUFYIc1XO$hhDqhKBs2uxqXlLaXkdf7WEHxbb.NstWxATa3DnksM1tPEgvTe..."
ssh_public_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC2IxTcUnfQSfdTla00OGWg67jlRN2SlGkCBr..."
ssh_port: 3333
  
  ```

### - Create server.yaml to sad
  ```yaml

  ---
- hosts: all
  become: true

  tasks:
    - name: Update apt repo and cache, and upgrade packages
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 3600
        upgrade: safe

    - name: Create new user finaltask-adi
      ansible.builtin.user:
        name: "{{ target_user }}"
        password: "{{ user_password_hash }}"
        shell: /bin/bash
        create_home: true
        groups: sudo
        append: true

    - name: Create .ssh directory for the user
      ansible.builtin.file:
        path: /home/{{ target_user }}/.ssh
        state: directory
        owner: "{{ target_user }}"
        group: "{{ target_user }}"
        mode: '0700'

    - name: Add SSH Public Key for secure login (.pem key)
      ansible.builtin.authorized_key:
        user: "{{ target_user }}"
        state: present
        key: "{{ ssh_public_key }}"


    - name: Change SSH port from 22 to 3333
      ansible.builtin.lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^#?Port"
        line: "Port {{ ssh_port }}"
        state: present
      notify: Restart SSH

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

    - name: Setup hostname for appserver
      ansible.builtin.hostname:
        name: appserver 
      when: " 'appserver' in group_names "

    - name: Setup hostname for database server
      ansible.builtin.hostname:
        name: database 
      when: " 'database' in group_names "

    - name: Setup hostname for gateway server
      ansible.builtin.hostname:
        name: gateway 
      when: " 'gateway' in group_names "

    - name: Setup hostname for k8s-master server
      ansible.builtin.hostname:
        name: master 
      when: " 'master' in group_names "

    - name: Setup hostname for k8s-worker-1 server
      ansible.builtin.hostname:
        name: worker-1 
      when: " 'worker-1' in group_names " 

    - name: Setup hostname for worker-2 server
      ansible.builtin.hostname:
        name: worker-2 
      when: " 'worker-2' in group_names "   

    - name: Enable UFW Firewall
      community.general.ufw:
        state: enabled
        policy: deny

    - name: Allow only required ports in UFW
      community.general.ufw:
        rule: allow
        port: "{{ item }}"
        proto: tcp
      loop:
        - '{{ ssh_port }}'
        - '80'
        - '443'
        - '5000'
        - '5432'
        - '6443'

  handlers:
    - name: Validate SSH configuration
      ansible.builtin.command: sshd -t
      changed_when: false
      listen: Restart SSH

    - name: Restart SSH
      ansible.builtin.service:
        name: ssh
        state: restarted
  ```
#
### - Running a Playbook
  ```bash

  ansible-playbook server.yaml
  ```

<p align="center"><img width="960" height="531" alt="image" src="https://github.com/user-attachments/assets/faad92c3-1f02-4ae6-bdea-fc1e6e966307" /></p>

### - Successfully ubuntu Update apt repo and cache, and upgrade packages

<p align="center"><img width="962" height="146" alt="image" src="https://github.com/user-attachments/assets/89dd0ae9-0373-4073-8ca6-34eb7d04a6bd" /></p>

### - Successfully Create new user finaltask-adi

<p align="center"><img width="959" height="149" alt="image" src="https://github.com/user-attachments/assets/a056265a-4220-4d28-9e23-37b8d96a639d" /></p>

### - Successfully Create a dedicated .ssh configuration directory inside the home folder of your new user.

<p align="center"><img width="961" height="137" alt="image" src="https://github.com/user-attachments/assets/8a873119-8a7e-4ff8-b1e8-228c906cd810" />
</p>

### - Successfully Registering the public key into the server so that the finaltask-adi user can log in securely using the .pem private key from your local computer. 

<p align="center"><img width="961" height="140" alt="image" src="https://github.com/user-attachments/assets/4ef8c632-6895-4ada-bec7-fb3fcf14f3c0" /></p>

### - Successfully changed the SSH port from the default port 22 to 3333 on all servers in the inventory list. 

<p align="center"><img width="960" height="141" alt="image" src="https://github.com/user-attachments/assets/0f2fb05c-bfa4-48c6-8519-c7568dbf39ff" /></p>

### - Successfully enabled password and keyboard-interactive authentication across all servers in the inventory list.  

<p align="center"><img width="960" height="279" alt="image" src="https://github.com/user-attachments/assets/48357d98-1c70-4470-822d-75f9e7a9915b" /></p>

### - Successfully setup hostname 

<p align="center"><img width="962" height="827" alt="image" src="https://github.com/user-attachments/assets/59965a98-0381-4742-9944-5c030d1955e1" /></p>

### - Successfully Enable UFW Firewall

<p align="center"><img width="960" height="141" alt="image" src="https://github.com/user-attachments/assets/cf97a117-f5c0-43e5-90d7-456fa0d0d683" /></p>

### - Successfully Allow only required ports in UFW port 80, 443, 5000, 5432, 6443

<p align="center"><img width="961" height="652" alt="image" src="https://github.com/user-attachments/assets/e52d953a-d791-414a-ab6e-a718e8b9016b" /></p>

### - Successfully Validate SSH configuration and Restart SSH

<p align="center"><img width="963" height="415" alt="image" src="https://github.com/user-attachments/assets/0031c63c-1ba6-4996-ba2d-4be3ba516252" />
</p>


## 3. Configure Local SSH Access
  - Update your local SSH configuration file (~/.ssh/config) to easily access the instances using the custom port 3333:
    
  ```yaml

  Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Port 3333
  ```

#

## 4. Edit terrafrom main.tf security group

### - Configure security group
```hcl

# =======================================================
# --- SECURITY GROUP ---
# =======================================================

resource "aws_security_group" "sg" {
  name        = "security-group-"  # penggunaan name baru agar namanya otomatis unik
  description = "Security group allowing SSH 3333, HTTP, HTTPS, PostgreSQL" # <-- Diubah dari 22 ke 3333
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "SSH Custom Port" 
    from_port   = 3333              # <-- UBAH DARI 22 MENJADI 3333
    to_port     = 3333              # <-- UBAH DARI 22 MENJADI 3333
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "HTTPS"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "PostgreSQL"
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
lifecycle { 
    create_before_destroy = true  # <--- untuk membuat Security Group baru terlebih dahulu sebelum menghapus yang lama.
  }
}
  
```

### - Successfully Terrafrom apply 
<p align="center"><img width="960" height="675" alt="image" src="https://github.com/user-attachments/assets/352495a2-ca6b-476a-adea-0f0fb0aec89e" /></p>


## 5. Test Login With Password/SSH Key port 3333
> [!NOTE]
> Setelah provisioning Ansible selesai, Kita dapat menguji login ke server menggunakan kunci privat `.pem` atau menggunakan metode *password* untuk user `finaltask-adi` pada port `3333`.
> Contoh perintah login dengan port kustom:
> ```bash
> ~/.ssh ssh -i jay-key.pem finaltask-adi@<SERVER_IP>
> ```
>
> > Contoh perintah login dengan *password*:
> ```bash
> ~/.ssh ssh finaltask-adi@<SERVER_IP>
> ```

### Contoh App-Server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
  <p align="center"><img width="960" height="657" alt="image" src="https://github.com/user-attachments/assets/ec2a08cd-e45a-4c51-91f5-4c3315cc167f" />
</p>

### - Test Login With *password*
#
  <p align="center"><img width="959" height="650" alt="image" src="https://github.com/user-attachments/assets/de6c678c-2c05-486b-9dd1-7399f20b93b3" /></p>
  

### Contoh Gateway-server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
<p align="center"><img width="951" height="857" alt="image" src="https://github.com/user-attachments/assets/e16211da-a4ea-46a8-a4ad-e2a56a632995" /></p>

### - Test Login With *password*
# 
<p align="center"><img width="956" height="733" alt="image" src="https://github.com/user-attachments/assets/bcb8e0f6-c47e-48b0-a17a-7d6fa7b11214" /></p>


### Contoh Database-server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
<p align="center"><img width="955" height="838" alt="image" src="https://github.com/user-attachments/assets/e18a3c74-5b23-4211-b110-16c59a7bc91c" />
</p>

### - Test Login With *password*
# 
<p align="center"><img width="957" height="728" alt="image" src="https://github.com/user-attachments/assets/3f7aa311-5317-4e2c-9c5c-51e923d52d56" />
</p>


### Contoh Master-server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
<p align="center"><img width="955" height="787" alt="image" src="https://github.com/user-attachments/assets/a032eeea-5d3e-4070-9dc9-edabe11a511c" /></p>

### - Test Login With *password*
# 
<p align="center"><img width="959" height="680" alt="image" src="https://github.com/user-attachments/assets/67931d55-fb87-43d4-a737-cd32000af4b1" />
</p>


### Contoh Worker-1-server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
<p align="center"><img width="960" height="654" alt="image" src="https://github.com/user-attachments/assets/d0a5e586-f049-47f1-9563-0d040e8eb87e" />
</p>

### - Test Login With *password*
# 
<p align="center"><img width="956" height="701" alt="image" src="https://github.com/user-attachments/assets/19a2d4ce-30e5-4dd4-9954-1d012a02cc69" />
</p>


### Contoh Worker-2-server di sini sudah mengunakan hostname sesuai dengan Inventory dan user `finaltask-adi`
### - Test Login With SSH Key
#
<p align="center"><img width="956" height="883" alt="image" src="https://github.com/user-attachments/assets/8d334f06-e321-4ee3-a157-f022dd73f391" />
</p>

### - Test Login With *password*
# 
<p align="center"><img width="950" height="759" alt="image" src="https://github.com/user-attachments/assets/fb3c600d-3c8b-4ad2-a738-5ce4e2b1b573" />

</p>



