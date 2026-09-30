# Provisioning

## 1. Add IAM User
### Create IAM User aws  and cofigure aws to terrafrom
  + Open Dasboard Identity and Access Management (IAM)
  + Open menu Access Management and IAM users
  + Press Create user

<p align="center"><img width="1919" height="996" alt="image" src="https://github.com/user-attachments/assets/10e5a1b4-dfab-4a50-86ef-561ce10e43bb" /></p>

### Enter your username.
  + Skip Provide user access to the AWS Management Console - optional.  then click Next.
 
<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/fb15b7df-1d72-4bcd-bd71-a97481edd6e7" /></p>

### Permissions summary 
  + succes add AdministratorAccess
  + succes add AmazonEC2FullAccess
  + then click Create user
  + succes add user

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/44b88ee1-f4c2-4bad-9afd-60e5b74c02db" /></p>

### Create Access key
  + then click  user Adi-Wijaya
  + and click Create access key

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/bfd319d4-9c74-4b99-9a84-356736f6d397" /></p>

### Choose Use Case and Best Practices
  + Select a Use Case: Choose the option that describes how you plan to use the access key (for example, select Command Line Interface (CLI) if you plan to use it with the AWS CLI).
  + Confirm and Proceed: Check the box next to "I understand the above recommendation and want to proceed to create an access key", then click the Next button.

<p align="center"><img width="1919" height="1036" alt="image" src="https://github.com/user-attachments/assets/a88f2559-4210-4c86-8d3b-fd0920ddf46d" /></p>

### Set Description Tag (Optional)
  + Create Access Key: Click the Create access key button to generate your credentials.

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/0bdcca0d-e7b4-4640-94bd-0cf0b6c55eae" /></p>

### Retrieve Access Keys
  + Download or Save: Click Download .csv file to securely save your credentials file to your computer.
  + Finish: Click Done to complete the process and close the setup page

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/fd891aeb-ac23-4709-a9e1-f1fba19579c0" /></p>

## 2. Configure aws to provisioning
  + aws configure new profile
  + add AWS Access Key ID
  + add AWS Secret Key
  + add Default region: app-southeast-3 (server Jakarta)
  
  ```bash
  # Use aws configure
  
  aws configure --profile <name-profile>
  ```

<p align="center"><img width="1297" height="443" alt="image" src="https://github.com/user-attachments/assets/e81ca0a8-a26c-490a-b0ca-2dd639284dbb" /></p>

## 3. Managing and Verifying Your AWS Profile
  - Check All Available Profiles
    
    ```bash
    # To see a list of all AWS profiles saved on your compute
    
    aws configure list-profiles
    ```

  - Set the Default Profile for Your Session

    ```bash
    # To avoid typing --profile adi-wijaya every time you run a command
    
    export AWS_PROFILE=adi-wijaya
    ```
    
  - Verify Your Connection

    ```bash
    # To test if your terminal is successfully connected using the adi-wijaya profile
    
    aws sts get-caller-identity
    ```
    
<p align="center"><img width="1298" height="379" alt="image" src="https://github.com/user-attachments/assets/211b2180-6cc9-4bb2-80a1-28700e63000c" /></p>


<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>



## 4. Create folder Terrafrom
### - Create file main.tf

```hcl

# =======================================================
# --- DATA SOURCE: DARI AMI UBUNTU 22.04 LTS  ---
# =======================================================

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] 
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# =======================================================
# --- NETWORKING (VPC & Subnet) ---
# =======================================================

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  tags                 = { Name = "VPC-${var.environment}" }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags                    = { Name = "Subnet-${var.environment}" }
}

resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.main.id
}

resource "aws_route_table" "rt" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.gw.id
  }
}

resource "aws_route_table_association" "rta" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.rt.id
}

# =======================================================
# Generate private key dengan algoritma RSA
# =======================================================

resource "tls_private_key" "this" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

# Register the public key with AWS EC2
resource "aws_key_pair" "generated" {
  key_name   = "jay-key"
  public_key = tls_private_key.this.public_key_openssh
}

# =======================================================
# Save the private key (.pem) to a local directory
# =======================================================

resource "local_file" "private_key" {
  content         = tls_private_key.this.private_key_pem
  filename        = "${path.module}/jay-key.pem"
  file_permission = "0400"
}

# =======================================================
# --- SERVERS: GATEWAY, APP, AND DATABASE ---
# =======================================================

resource "aws_instance" "gateway" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.gateway_instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.sg.id]
  key_name               = aws_key_pair.generated.key_name
  tags                   = { Name = "Gateway-Server-${var.environment}" }
}

resource "aws_instance" "appserver" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.app_instance_type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.sg.id]
  key_name               = aws_key_pair.generated.key_name
  tags                   = { Name = "App-Server-${var.environment}" }
}

resource "aws_instance" "database" {
  ami                     = data.aws_ami.ubuntu.id
  instance_type           = var.db_instance_type
  subnet_id               = aws_subnet.public.id
  vpc_security_group_ids  = [aws_security_group.sg.id]
  key_name                = aws_key_pair.generated.key_name
  tags                    = { Name = "Database-Server-${var.environment}" }
}
```

### Penjelasan Main.tf
- Berikut adalah penjelasan per bagian dari: DATA SOURCE: DARI AMI UBUNTU 22.04 LTS
  + ```most_recent = true``` -> Memilih AMI dengan versi yang paling baru.
  + ```owners = ["099720109477"]``` -> Menentukan pemilik resmi dari AMI tersebut, yaitu Canonical (pembuat Ubuntu).
  + ```filter "name"``` -> Mencari template sistem operasi Ubuntu versi Jammy Jellyfish 22.04 LTS (64-bit x86).
  + ```filter "virtualization-type"``` -> Memastikan tipe virtualisasi menggunakan jenis ```vim``` (Hardware Virtual Machine) yang didukung standar AWS.

- Berikut adalah penjelasan per bagian dari: NETWORKING (VPC & Subnet) 
  + ```aws_vpc.main``` -> Membuat jaringan virtual privat (VPC) dengan blok CIDR ```10.0.0.0/16``` mengaktifkan fitur DNS hostnames, dan memberikan tag nama berdasarkan variabel (var.environment)
  + ```aws_subnet.public``` -> Membuat subnet publik di dalam VPC utama dengan blok CIDR  ```10.0.1.0/24``` . Dan fitur ```map_public_ip_on_launch = true``` memastikan setiap instance yang dibuat di subnet ini akan otomatis mendapatkan IP publik.
  + ```aws_internet_gateway.gw``` -> Membuat Internet Gateway (IGW) dan menghubungkannya ke VPC agar jaringan di dalam VPC dapat berkomunikasi dengan internet luar.
  + ```aws_route_table.rt``` -> Membuat tabel rute (route table) di dalam VPC. Di dalamnya terdapat aturan rute  ```(0.0.0.0/0)```  yang mengarahkan seluruh lalu lintas keluar (traffic) menuju Internet Gateway.
  + ```aws_route_table_association.rta``` -> Menghubungkan (associate) tabel rute publik tersebut ke subnet publik  ```(aws_subnet.public)``` agar subnet tersebut resmi menjadi public subnet yang bisa mengakses internet.

- Berikut adalah penjelasan per bagian dari:Generate private key dengan algoritma RSA
  + ```tls_private_key.this``` -> Membuat Kunci Privat (RSA). Berfungsi untuk menghasilkan sepasang kunci kriptografi (publik dan privat) menggunakan ```algoritma RSA``` dengan ukuran ```rsa_bits  = 4096``` yang sangat aman.
  + ```aws_key_pair.generated``` -> Mendaftarkan Kunci Publik ke AWS. Mengambil bagian public key dari kunci yang baru dibuat, lalu mendaftarkannya ke AWS dengan nama ```jay-key``` agar bisa dipasang pada server EC2.
  + ```local_file.private_key``` -> Menyimpan Kunci Privat Secara Lokal. lalu otomatis menyimpan bagian private key ```(private_key_pem)``` ke dalam komputer lokal dengan nama file ```jay-key.pem``` dan mengatur izin akses file menjadi 0400 (hanya bisa dibaca) demi keamanan.
 
- Berikut adalah penjelasan per bagian dari: SERVERS:GATEWAY, APP, AND DATABASE
  + ```aws_instance.gateway``` -> Untuk membuat server virtual (EC2) baru menggunakan sistem operasi Ubuntu, tipe spesifikasi dari variabel, ditempatkan di subnet publik, dihubungkan dengan security group dan kunci akses (key pair), serta diberi nama tag server gateway.
  + ```aws_instance.appserver``` -> Membuat server EC2 terpisah yang berfungsi sebagai tempat menjalankan aplikasi utama, dengan konfigurasi dasar (AMI, subnet, security group, dan key pair) yang serupa.
  + ```aws_instance.database``` -> Membuat server EC2 ketiga yang dikhususkan untuk menjalankan database, menggunakan spesifikasi tipe database dari variabel serta terhubung ke jaringan dan kunci akses yang sama.

# 

### - Create file variabels.tf

```hcl

# =======================================================
# --- Variabel region ---
# =======================================================

variable "aws_region" {
  type        = string
  default     = "ap-southeast-3"
  description = "AWS Region Jakarta"
}

# =======================================================
# --- Variabel staging ---
# =======================================================

variable "environment" {
  type    = string
  default = "staging"
  description = "Environment (staging atau production)"
}

variable "gateway_instance_type" {
  type    = string
  default = "t2.micro"
  description = "Tipe instance untuk Gateway / Nginx Reverse Proxy"
}

variable "app_instance_type" {
  type    = string
  default = "t3.small"
  description = "Tipe instance untuk App Server"
}

variable "db_instance_type" {
  type    = string
  default = "t2.micro"
  description = "Tipe instance untuk Database Server"
}

```

### Penjelasan variabels.tf
- Berikut adalah penjelasan per bagian dari: Variabel region
  + ```aws_region``` -> Untuk menentukan wilayah (region) AWS default yang akan digunakan, yaitu ```ap-southeast-3``` (Jakarta).

- Berikut adalah penjelasan per bagian dari: Variabel staging
  + ```environment``` -> Untuk menentukan nama lingkungan infrastruktur dengan nilai default staging (biasanya digunakan untuk memberi nama tag otomatis).
  + ```gateway_instance_type``` -> Menentukan spesifikasi ukuran server (tipe instance) untuk Gateway atau Nginx Reverse Proxy, yaitu type ```t2.micro```.
  + ```app_instance_type``` -> Menentukan spesifikasi ukuran server untuk App Server (Server Aplikasi), yaitu type ```t3.small```.
  + ```db_instance_type``` -> Menentukan spesifikasi ukuran server untuk Database Server, yaitu type ```t2.micro```.

#

### - Create file provider.tf

```hcl

terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

```

### Penjelasan provider.tf
- Berikut adalah penjelasan per bagian dari: Provider TErrafrom
  + ```terraform``` -> Untuk menentukan persyaratan minimum versi Terraform yang harus digunakan (yaitu minimal versi 1.0.0 atau yang lebih baru) serta mendefinisikan provider plugin yang dibutuhkan, yaitu AWS buatan HashiCorp ```version = "~> 5.0"``` versi 5.x.
  + ```provider "aws"``` -> Untuk mengonfigurasi hubungan Terraform dengan layanan AWS, di mana wilayah (region)-nya diatur di variabel ```var.aws_region``` (di dalam var ```ap-southeast-3``` / Jakarta).

#

### - Create outputs.tf

```hcl

# =======================================================
# Output IP Address for Ansible Staging
# =======================================================

output "gateway_ip" {
  value = aws_instance.gateway.public_ip
}

output "appserver_ip" {
  value = aws_instance.appserver.public_ip
}

output "database_ip" {
  value = aws_instance.database.public_ip
}


```
### Penjelasan outputs.tf
- Berikut adalah penjelasan per bagian dari: Output Terraform
  + ```gateway_ip``` -> Yang berfungsi untuk menampilkan atau mengeluarkan informasi alamat IP publik dari server Gateway setelah proses pembuatan infrastruktur selesai.
  + ```appserver_ip``` -> Yang berfungsi untuk menampilkan alamat IP publik dari server App Server (aplikasi).
  + ```database_ip``` -> Yang berfungsi untuk menampilkan alamat IP publik dari server Database.


<p align="center">Outputs IP</p>
 
#

## 5. Success integration Server with terraform

<p align="center">Server</p>



## 6. Create folder Ansible
### - Create file ansible.cfg

```hcl

[defaults]
inventory = Inventory
private_key_file = ~/.ssh/jay-key.pem
host_key_checking = False
remote_user = ubuntu
ansible_python_interpreter = /usr/bin/python3

```

#

### - Create file Inventory

```yaml

[gateway]
138.189.12.3

[appservers]
118.125.763.23

[databases]
172.87.23.78

```

#

### - Create folder group_vars and file all.yaml

```yaml

---
# Konfigurasi SSH & Koneksi Default
ansible_port: 22
ansible_user: ubuntu

# Konfigurasi Ansibel 
target_user: "finaltask-adi"
user_password_hash: "$6$JVoSl9OOUFYIc1XO$hhDqhKBs2uxqXlLaXkdf7WEHxbb.NstWxATa3DnksK..."
ssh_public_key: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDY23BJ48yFwQ2pF8kje6WD0r1U57..."

```

#

### - Provisioning.yaml

```yaml

---
- hosts: all
  become: true
  tasks:
    - name: Update apt repo and cache
      ansible.builtin.apt:
        update_cache: true
        cache_valid_time: 3600

- hosts: gateway
  become: true
  tasks:
    - name: Install Nginx web server
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Ensure Nginx is running and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

- hosts: databases
  become: true
  tasks:
    - name: Install PostgreSQL database server
      ansible.builtin.apt:
        name: postgresql
        state: present

    - name: Ensure PostgreSQL is running and enabled
      ansible.builtin.service:
        name: postgresql
        state: started
        enabled: true

```

### - Connect to the Server Using an SSH Key

<p align="center">login Server</p>

#













