# Provisioning

## Karena keterbatasan kombinasi alokasi *resource* pada tipe instans bawaan AWS EC2, spesifikasi perangkat keras disesuaikan dengan menggunakan tipe instans **`t3.micro`** (yang memiliki profil standar **2 vCPU dan 1 GiB RAM**).
---

### Realisasi Spesifikasi Infrastruktur

Otomatisasi dikelola penuh dari **Local Machine** menggunakan kolaborasi **Terraform** dan **Ansible**. Berikut adalah profil server yang digunakan di AWS:

*   **Gateway Server (`t3.micro` - 2 CPU, 1GB RAM):**  
    Bertindak sebagai gerbang utama lalu lintas data, menjalankan Nginx Reverse Proxy, otomasi SSL Certbot, serta meng-host Private Docker Registry.
*   **Database Server (`t3.micro` - 2 CPU, 1GB RAM):**  
    Server terisolasi khusus untuk menjalankan container database PostgreSQL 15 secara aman.
*   **Appserver (`t3.small` - 2 CPU, 2GB RAM):**  
    Server utama dengan RAM lebih besar yang didedikasikan untuk menangani proses kompilasi (*build time*) serta menjalankan container Frontend dan Backend aplikasi.

---

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

#
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
  tags                 = { Name = "VPC" }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  tags                    = { Name = "Subnet" }
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
# --- SECURITY GROUP ---
# =======================================================

resource "aws_security_group" "sg" {
  name        = "security-group"
  description = "Security group allowing SSH 22, HTTP, HTTPS, PostgreSQL"
  vpc_id      = aws_vpc.main.id

  ingress {
    description = "TCP"
    from_port   = 22
    to_port     = 22
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
  filename        = "/home/adi/.ssh/jay-key.pem"
  file_permission = "0400"
}

# =======================================================
# --- DYNAMIC SERVERS PROVISIONING ---
# =======================================================

locals {
  servers = {
    "gateway"    = { type = var.gateway_instance_type, name = "Gateway-Server" }
    "appserver"  = { type = var.app_instance_type, name = "App-Server" }
    "database"   = { type = var.db_instance_type, name = "Database-Server" }
    "master"     = { type = var.k8s_master_instance_type, name = "Master" }
    "worker-1"   = { type = var.k8s_worker_instance_type, name = "Worker-1" }
    "worker-2"   = { type = var.k8s_worker_instance_type, name = "Worker-2" }
  }
}

resource "aws_instance" "server" {
  for_each               = local.servers
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = each.value.type
  subnet_id              = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.sg.id]
  key_name               = aws_key_pair.generated.key_name

  credit_specification {
      cpu_credits = "standard"
    }

  tags = {
    Name = each.value.name
    OS   = "Ubuntu 22.04 LTS"
  }
}

# =======================================================
# --- ELASTIC IP ---
# =======================================================

locals {
  # Batasi server yang diberi EIP agar tidak melewati limit AWS (maksimal 5)
  servers_with_eip = {
    "gateway" = local.servers["gateway"]
    "master"  = local.servers["master"]
  }
}

resource "aws_eip" "eip_server" {
  for_each   = local.servers_with_eip
  instance   = aws_instance.server[each.key].id
  domain     = "vpc"
  depends_on = [aws_internet_gateway.gw]
  tags       = { Name = "EIP-${each.value.name}" }
}

# =======================================================
# --- EBS VOLUMES & ATTACHMENTS ---
# =======================================================

resource "aws_ebs_volume" "storage_server" {
  for_each          = local.servers
  availability_zone = aws_instance.server[each.key].availability_zone
  size              = 15

  tags = { Name = "Volume-${each.value.name}" }
}

resource "aws_volume_attachment" "attach_server" {
  for_each    = local.servers
  device_name = "/dev/xvdf"
  volume_id   = aws_ebs_volume.storage_server[each.key].id
  instance_id = aws_instance.server[each.key].id
}
```

### Penjelasan Main.tf
- Berikut adalah penjelasan per bagian dari: DATA SOURCE: DARI AMI UBUNTU 22.04 LTS
  + ```most_recent = true``` -> Memilih AMI dengan versi yang paling baru.
  + ```owners = ["099720109477"]``` -> Menentukan pemilik resmi dari AMI tersebut, yaitu Canonical (pembuat Ubuntu).
  + ```filter "name"``` -> Mencari template sistem operasi Ubuntu versi Jammy Jellyfish 22.04 LTS (64-bit x86).
  + ```filter "virtualization-type"``` -> Memastikan tipe virtualisasi menggunakan jenis ```vim``` (Hardware Virtual Machine) yang didukung standar AWS.

#
- Berikut adalah penjelasan per bagian dari: NETWORKING (VPC & Subnet) 
  + ```aws_vpc.main``` -> Membuat jaringan virtual privat (VPC) dengan blok CIDR ```10.0.0.0/16``` mengaktifkan fitur DNS hostnames, dan memberikan tag nama berdasarkan variabel (var.environment)
  + ```aws_subnet.public``` -> Membuat subnet publik di dalam VPC utama dengan blok CIDR  ```10.0.1.0/24``` . Dan fitur ```map_public_ip_on_launch = true``` memastikan setiap instance yang dibuat di subnet ini akan otomatis mendapatkan IP publik.
  + ```aws_internet_gateway.gw``` -> Membuat Internet Gateway (IGW) dan menghubungkannya ke VPC agar jaringan di dalam VPC dapat berkomunikasi dengan internet luar.
  + ```aws_route_table.rt``` -> Membuat tabel rute (route table) di dalam VPC. Di dalamnya terdapat aturan rute  ```(0.0.0.0/0)```  yang mengarahkan seluruh lalu lintas keluar (traffic) menuju Internet Gateway.
  + ```aws_route_table_association.rta``` -> Menghubungkan (associate) tabel rute publik tersebut ke subnet publik  ```(aws_subnet.public)``` agar subnet tersebut resmi menjadi public subnet yang bisa mengakses internet.

 #
- Berikut adalah penjelasan per bagian dari: SECURITY GROUP
  + ```aws_security_group.sg``` -> Membuat firewall virtual baru untuk melindungi server di dalam VPC utama (aws_vpc.main.id).
  + ```name``` & ```description``` -> Memberikan nama ```"security-group"``` dan catatan deskripsi agar mudah dikenali di AWS Console.
  + ```ingress``` (Port 22 - SSH) -> Membuka akses remote server dari internet.
  + ```ingress``` (Port 80 - HTTP) -> Membuka akses website standar.
  + ```ingress``` (Port 443 - HTTPS) -> Membuka akses website aman ber-SSL.
  + ```ingress``` (Port 5432 - PostgreSQL) -> Membuka akses database. 
  +  ```egress``` (Protocol -1) -> Mengizinkan server bebas keluar ke internet (untuk update, install aplikasi, dll).

#
- Berikut adalah penjelasan per bagian dari:Generate private key dengan algoritma RSA
  + ```tls_private_key.this``` -> Membuat Kunci Privat (RSA). Berfungsi untuk menghasilkan sepasang kunci kriptografi (publik dan privat) menggunakan ```algoritma RSA``` dengan ukuran ```rsa_bits  = 4096``` yang sangat aman.
  + ```aws_key_pair.generated``` -> Mendaftarkan Kunci Publik ke AWS. Mengambil bagian public key dari kunci yang baru dibuat, lalu mendaftarkannya ke AWS dengan nama ```jay-key``` agar bisa dipasang pada server EC2.
  + ```local_file.private_key``` -> Menyimpan Kunci Privat Secara Lokal. lalu otomatis menyimpan bagian private key ```(private_key_pem)``` ke dalam komputer lokal dengan nama file ```jay-key.pem``` dan mengatur izin akses file menjadi 0400 (hanya bisa dibaca) demi keamanan.

 #
- Berikut adalah penjelasan per bagian dari: DYNAMIC SERVERS PROVISIONING (Staging & Production) 
  + ```locals { servers = { ... } }``` -> Untuk mendefinisikan kumpulan peta (map) data server yang berisi konfigurasi tipe instance dan nama tag untuk keenam node secara terpusat (Gateway, App Server, Database, Master, Worker-1, dan Worker-2).
  + ```resource "aws_instance" "server"``` -> Membuat server virtual (EC2) secara dinamis menggunakan perulangan ```for_each``` berdasarkan data lokal ```servers```. Menggunakan AMI Ubuntu 22.04 LTS, spesifikasi tipe dari variabel masing-masing node, ditaruh di subnet publik, dihubungkan dengan security group dan key pair ```jay-key``` , serta diberi tag nama server secara otomatis.
  + ```aws_instance.database``` -> Membuat server EC2 ketiga yang dikhususkan untuk menjalankan database, menggunakan spesifikasi tipe database dari variabel serta terhubung ke jaringan dan kunci akses yang sama.
  + ```credit_specification``` -> Mengatur mode kredit CPU (diset ke ```"standard"```) untuk instance tipe burstable (seperti ```t3.micro ```atau ```t3.small```) guna memastikan performa kredit CPU terkontrol dengan baik.
  + ```locals { servers_with_eip = { ... } }``` -> Untuk menyaring server yang dipasangi IP tetap (hanya gateway dan master) agar tidak melanggar batas maksimal 5 Elastic IP dari AWS.
  + ```resource "aws_eip" "eip_server"``` -> Membuat dan mengalokasikan Elastic IP (IP publik statis) secara dinamis untuk setiap server yang dibuat, serta memastikan prosesnya bergantung pada ketersediaan internet gateway ```(depends_on)```.
  + ```instance = aws_instance.server[each.key].id``` -> Menyambungkan IP statis yang sudah dibuat langsung ke instance server yang bersesuaian.
  + ``` domain = "vpc"``` -> Menentukan bahwa IP dibuat khusus untuk jaringan cloud VPC modern.
  + ```tags = { Name = "EIP-${each.value.name}" }``` -> Memberikan nama label pada IP di dashboard AWS agar mudah dikenali.
  + ```resource "aws_ebs_volume" "storage_server"``` -> Membuat volume penyimpanan tambahan (EBS) berukuran 15 GB secara dinamis untuk masing-masing server di Availability Zone yang sama persis dengan tempat server tersebut berada.
  + ```resource "aws_volume_attachment" "attach_server"``` -> Menghubungkan (attach) volume EBS yang telah dibuat ke setiap instance server pada jalur perangkat blok (device name) /dev/xvdf

  
# 

### - Create file variabels.tf

```hcl

# =======================================================
# --- Variabel staging ---
# =======================================================

variable "aws_region" {
  type        = string
  default     = "ap-southeast-3"
  description = "AWS Region Jakarta"
}

# =======================================================
# --- Variabel staging ---
# =======================================================

variable "gateway_instance_type" {
  type    = string
  default = "t3.micro"
  description = "Tipe instance untuk Gateway / Nginx Reverse Proxy"
}

variable "app_instance_type" {
  type    = string
  default = "t3.small"
  description = "Tipe instance untuk App Server"
}

variable "db_instance_type" {
  type    = string
  default = "t3.micro"
  description = "Tipe instance untuk Database Server"
}

# =======================================================
# --- Variabel untuk Kubernetes (Production) ---
# =======================================================

variable "k8s_master_instance_type" {
  type        = string
  default     = "t3.small"
  description = "Tipe instance untuk Node Master Kubernetes"
}

variable "k8s_worker_instance_type" {
  type        = string
  default     = "t3.small"
  description = "Tipe instance untuk Node Worker Kubernetes"
}

```

### Penjelasan variabels.tf
- Berikut adalah penjelasan per bagian dari: Variabel region
  + ```aws_region``` -> Untuk menentukan wilayah (region) AWS default yang akan digunakan, yaitu ```ap-southeast-3``` (Jakarta).

- Berikut adalah penjelasan per bagian dari: Variabel staging
  + ```gateway_instance_type``` -> Menentukan spesifikasi ukuran server (tipe instance) untuk Gateway atau Nginx Reverse Proxy, yaitu type ```t3.micro```. harisnya t2.micro akan tetapi di aws sudah tidak ada saya menggunakan opsi t3.micro dengan cpu2 ram 1gb
  + ```app_instance_type``` -> Menentukan spesifikasi ukuran server untuk App Server (Server Aplikasi), yaitu type ```t3.small```.
  + ```db_instance_type``` -> Menentukan spesifikasi ukuran server untuk Database Server, yaitu type ```t3.micro```.
- 
  + ```k8s_master_instance_type``` -> Menentukan spesifikasi ukuran server untuk App Server (Server Aplikasi), yaitu type ```t3.small```.
  + ```k8s_worker_instance_type``` -> Menentukan spesifikasi ukuran server untuk App Server (Server Aplikasi), yaitu type ```t3.small```.
Berikut adalah penjelasan per bagian dari: Variabel Production    
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

output "EIP-gateway_ip" {
  description = "Public IP untuk Gateway Server"
  value       = aws_eip.eip_server["gateway"].public_ip
}

output "EIP-master_ip" {
  description = "Public IP untuk Kubernetes Master Server"
  value       = aws_eip.eip_server["master"].public_ip
}

# Untuk server yang tidak pakai EIP
output "appserver_ip" {
  description = "Public IP untuk App Server"
  value       = aws_instance.server["appserver"].public_ip
}

output "database_ip" {
  description = "Public IP untuk Database Server"
  value       = aws_instance.server["database"].public_ip
}

output "worker1_ip" {
  description = "Public IP untuk Kubernetes Worker-1 Server"
  value       = aws_instance.server["worker-1"].public_ip
}

output "worker2_ip" {
  description = "Public IP untuk Kubernetes Worker-2 Server"
  value       = aws_instance.server["worker-2"].public_ip
}


```
### Penjelasan outputs.tf
- Berikut adalah penjelasan per bagian dari: Output Terraform
  + ```output``` -> Menampilkan informasi IP publik ke layar terminal setelah proses terraform apply selesai
  + ```output "EIP-gateway_ip"``` & ```"EIP-master_ip"``` -> Mengambil IP dari resource Elastic IP ```(aws_eip.eip_server)```, karena kedua server ini (Gateway dan Master) menggunakan IP statis/permanen yang tidak akan berubah.
  + ```output "appserver_ip"```, ```"database_ip"```, dll. -> Mengambil IP langsung dari instance EC2 (aws_instance.server), karena server-server ini tidak menggunakan Elastic IP (untuk menghindari batas limit AWS), melainkan menggunakan IP publik dinamis bawaan AWS.
 
#


## 5 . Run terraform Inisialisasi

```bash

terraform init
```

<p align="center"><img width="961" height="619" alt="image" src="https://github.com/user-attachments/assets/c6659afd-d5cf-4507-9634-2f9f54c57199" /></p>

## 6. Execute terraform plan (Review Plan)

```bash

terraform plan
```
<p align="center"><img width="957" height="184" alt="image" src="https://github.com/user-attachments/assets/548fa2dd-3393-4591-bd81-a1c83577d4cd" /></p>

## 6. Run terraform apply 

```bash

terraform apply
```

<p align="center"><img width="905" height="231" alt="image" src="https://github.com/user-attachments/assets/677ab699-77d0-4cb3-bddd-2279a6d00ba3" /></p>

## 7. Success integration Server with terraform

<p align="center"><img width="1913" height="1036" alt="image" src="https://github.com/user-attachments/assets/3d3f862e-2830-42cc-a5db-6d230c6ae2c2" /></p>

### - Success key to save local computer
<p align="center"><img width="955" height="357" alt="image" src="https://github.com/user-attachments/assets/651881e9-49a9-4dd2-8c33-d7cbd0644553" /></p>

### - Secure your .pem file permissions
  ```bash

  chmod 400 /home/adi/.ssh/jay-key.pem
  ```
<p align="center"><img width="959" height="255" alt="image" src="https://github.com/user-attachments/assets/e75b7142-f88e-4118-90cf-f290fd5612cb" /></p>

#
## 8. Test Login With SSH keys
### - App-Server => Success login
<p align="center"><img width="956" height="783" alt="image" src="https://github.com/user-attachments/assets/1a43a27e-5b31-468a-8d60-611db13cd109" /></p>

#
### - Database-Server => Success login
<p align="center"><img width="959" height="789" alt="image" src="https://github.com/user-attachments/assets/c82f68f2-d2ce-4f9e-b92e-7b640d8485c1" /></p>

#
### - Gateway-Server => Success login
<p align="center"><img width="956" height="778" alt="image" src="https://github.com/user-attachments/assets/9cf78bd6-ef0a-43ee-a649-761aba537353" /></p>

#
### - Master => Success login
<p align="center"><img width="958" height="802" alt="image" src="https://github.com/user-attachments/assets/6ba089ff-6747-4282-aaa2-ba1d370924e7" /></p>

#
### - Worker-1 => Success login
<p align="center"><img width="959" height="804" alt="image" src="https://github.com/user-attachments/assets/9f04bb88-f1bd-4da5-bb9b-9c449e954e17" /></p>

#
### - Worker-2 => Success login
<p align="center"><img width="959" height="807" alt="image" src="https://github.com/user-attachments/assets/22314974-c7e2-4995-830f-a65676e7a866" />
</p>


## 9. Create folder Ansible
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
15.232.124.118

[appservers]
43.218.140.185

[databases]
16.79.57.75

[master]
15.232.150.220

[worker-1]
108.136.50.89

[worker-2]
16.78.50.175

```

#

### - Create folder group_vars and file all.yaml

```yaml

---
# Konfigurasi SSH & Koneksi Default
ansible_port: 22
ansible_user: ubuntu


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
#
### Penjelasan Provisioning.yaml
  + ```hosts: all``` -> Menjalankan perintah ini ke seluruh server yang ada di dalam inventory Ansible.
  + ```become: true``` -> Menjalankan perintah menggunakan hak akses superuser (sudo), setara dengan root.
  + ```ansible.builtin.apt```(Update cache) -> Melakukan pembaruan repository APT (```apt update```) pada sistem Ubuntu agar daftar paket aplikasi selalu yang terbaru, dengan batasan waktu cache ```3600``` detik (1 jam) agar prosesnya tidak berulang-ulang terlalu sering.
  + ```hosts: gateway``` -> Perintah di bagian ini hanya dikhususkan untuk server yang tergabung dalam grup ```gateway```.
  + ```Install Nginx``` -> Mengunduh dan menginstal web server ```Nginx``` menggunakan paket manajer ```apt``` (```state: present```).
  + ```Service Nginx``` -> Memastikan layanan Nginx langsung dijalankan (```state: started```) dan diset otomatis aktif (```enabled: true```) setiap kali server dinyalakan ulang.
  + ```hosts: databases``` -> Perintah di bagian ini hanya dikhususkan untuk server yang berada di dalam grup ```databases```.
  + ```Install PostgreSQL``` -> Menginstal server database PostgreSQL melalui ```apt```.
  + ```Service PostgreSQL``` -> Memastikan layanan database PostgreSQL dijalankan (```state: started```) dan otomatis aktif kembali (```enabled: true```) saat server reboot.

### - Success Run playbook
```bash

ansible-playbook provisioning.yaml
```

<p align="center"><img width="956" height="981" alt="image" src="https://github.com/user-attachments/assets/5747efbd-77b7-44bc-9de6-330c50d32c65" /></p>
<p align="center"><img width="964" height="257" alt="image" src="https://github.com/user-attachments/assets/c49dc848-44f8-4289-ba2e-e063cc4d161c" /></p>















