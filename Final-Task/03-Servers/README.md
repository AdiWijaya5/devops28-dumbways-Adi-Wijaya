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

#
## 2. Configure Local SSH Access
  - Update your local SSH configuration file (~/.ssh/config) to easily access the instances using the custom port 3333 and your private key (jay-key.pem):
    
  ```yaml

  Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
  ```
#  
## 3. Config Ansible
### - all vars
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
