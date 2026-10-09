## Testing App

### Install SonarQube Scanner With Ansible

```yaml

---
- name: Setup and Run SonarQube 
  hosts: sonarqube
  become: true
  tasks:
    - name: Update apt cache
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install dependencies for Docker repository
      ansible.builtin.apt:
        name:
          - apt-transport-https
          - ca-certificates
          - curl
          - gnupg
          - lsb-release
          - python3-pip
          - python3-docker
        state: present
        update_cache: true

    - name: Add Docker official GPG key
      ansible.builtin.shell: |
        install -m 0755 -d /etc/apt/keyrings
        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
        chmod a+r /etc/apt/keyrings/docker.gpg
      args:
        creates: /etc/apt/keyrings/docker.gpg

    - name: Set up the stable Docker repository
      ansible.builtin.shell: |
        echo \
          "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
          $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
      args:
        creates: /etc/apt/sources.list.d/docker.list

    - name: Install Docker Engine and Containerd
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

    - name: Add remote user to docker group
      ansible.builtin.user:
        name: "{{ ansible_user }}"
        groups: docker
        append: true

    - name: Set vm.max_map_count for Elasticsearch (Minimum requirement)
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: true

    - name: Run Lightweight SonarQube container with Memory Limits
      shell: |
        # Hapus container lama jika ada agar bersih
        docker rm -f sonarqube 2>/dev/null || true

        docker run -d \
          --name sonarqube \
          --restart always \
          -p 9000:9000 \
          -m 2g \
          --memory-swap 3g \
          -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
          -e SONAR_SEARCH_JAVAADDITIONALOPTS="-Xms512m -Xmx512m -Ddiscovery.type=single-node" \
          -e SONAR_WEB_JAVAADDITIONALOPTS="-Xms256m -Xmx512m" \
          -v sonarqube_data:/opt/sonarqube/data \
          -v sonarqube_extensions:/opt/sonarqube/extensions \
          -v sonarqube_logs:/opt/sonarqube/logs \
          sonarqube:lts-community
      args:
        executable: /bin/bash

```


<p align="center"><img width="958" height="1014" alt="image" src="https://github.com/user-attachments/assets/52da0b71-6658-4f02-a642-b2627a4be768" /></p>


### add Plugin Stage SonarQube Scanner

<p align="center"><img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/5a408789-7cc2-478b-931a-fbecc6b6a0f4" />
</p>


### - login SonarQube

---
<p align="center"><img width="1919" height="1040" alt="Screenshot 2026-10-07 024423" src="https://github.com/user-attachments/assets/1ca0d290-2d0b-4a4a-95c8-38ce4f73b334" /></p>

- generate token SonarQube

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/fe6631df-8711-4701-9425-9ed46d93ff33" /></p>

- buat secarlet sonarqube-token di jenkins

<p align="center"><img width="669" height="549" alt="image" src="https://github.com/user-attachments/assets/5988438e-6646-4ab7-bbc2-2de5a94ab54e" />
</p>

- tambahkan SonarQube installations
  + Buka Manage Jenkins > System.
  + Cari bagian SonarQube installations
  + Pastikan kolom Name diisi persis: ```SonarQubeServer```
  + Pastikan kolom url Server URL isi dengan url server testing ```http://localhost:9000```
  + Pastikan kolom Server authentication token sudah memilih: ```sonarqube-token``` yang sudah di buat tadi
       

<p align="center"><img width="1671" height="769" alt="image" src="https://github.com/user-attachments/assets/7362f899-3600-4a58-8ca7-7dfaffe11c0d" /></p>


### CI/CD Pipeline fe-dumbmerch-testing

```yaml


def secret = 'ssh_credentials_id'      
def app_env = 'staging'      
def iamge_tag = "staging"
def app_server_ip = '15.232.21.109'    
def app_server_user = 'finaltask-adi'
def registry = 'registry.adi.studentdumbways.my.id'
def image ='registry.adi.studentdumbways.my.id/fe-dumbmerch:staging'
def app_url = 'https://staging.adi.studentdumbways.my.id'

pipeline {
    agent any

    stages {
        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${app_env}..."
                checkout scm
            }
        }
       
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=fe-dumbmerch \
                        -Dsonar.projectName=fe-dumbmerch \
                        -Dsonar.sources=. "
                    }
                }
            }
        }

        stage('Pull & Test Existing Docker Image') {
            steps {
                echo "Pulling existing image from registry for local testing..."
                sh "docker pull ${image}"
                
                echo "Running smoke test on the pulled image..."
                sh """
                    docker run -d -p 3001:80 --name test-frontend-container ${image}
                    sleep 3
                    curl --fail http://localhost:3001 || exit 1
                    docker rm -f test-frontend-container
                """
            }
        }

        stage('Deploy') {
            steps {
                echo "Connecting to server via SSH Deploy apps (Staging)..."
                sshagent(["${secret}"]) {
                    sh """
                        ssh -p 3333 -o StrictHostKeyChecking=no ${app_server_user}@${app_server_ip} "\
                        docker login ${registry} && \
                        docker pull ${registry}/fe-dumbmerch:${iamge_tag} && \
                        docker stop fe-dumbmerch-${app_env} || true && \
                        docker rm -f fe-dumbmerch-${app_env} || true && \
                        docker run -d --name fe-dumbmerch-${app_env} -p 5000:5000 ${registry}/fe-dumbmerch:${iamge_tag}"
                    """
                }
            }
        }

        stage('Test Running App with Wget Spider') {
            steps {
                echo "Testing backend server responsiveness...."
                sh "wget --spider --no-verbose ${app_url} || true"
            }
        }
    }

    post {
        success {
            echo "SUCCESS: All stages including SonarQube analysis, local image testing, Deploy, and Wget Spider test passed successfully!"
        }
        failure {
            echo "FAILURE: Pipeline failed during testing or deployment. Please check the console logs for details."
        }
    }
}


```

---

### CI/CD Pipeline be-dumbmerch-testing

```yaml


def secret = 'ssh_credentials_id'      
def app_env = 'staging'      
def iamge_tag = "staging"
def app_server_ip = '15.232.21.109'    
def app_server_user = 'finaltask-adi'
def registry = 'registry.adi.studentdumbways.my.id'
def image ='registry.adi.studentdumbways.my.id/be-dumbmerch:staging'
def app_url = 'https://api.staging.adi.studentdumbways.my.id/api/v1/products'

pipeline {
    agent any

    stages {
        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${app_env}..."
                checkout scm
            }
        }
       
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=be-dumbmerch \
                        -Dsonar.projectName=be-dumbmerch \
                        -Dsonar.sources=. "
                    }
                }
            }
        }

        stage('Pull & Test Existing Docker Image') {
            steps {
                echo "Pulling existing image from registry for local testing..."
                sh "docker pull ${image}"
                
                echo "Running smoke test on the pulled image..."
                sh """
                    docker run -d -p 5001:5000 --name test-backend-container ${image}
                    sleep 3
                    curl --fail http://localhost:5001 || exit 1
                    docker rm -f test-backend-container
                """
            }
        }

        stage('Deploy') {
            steps {
                echo "Connecting to server via SSH Deploy apps (Staging)..."
                sshagent(["${secret}"]) {
                    sh """
                        ssh -p 3333 -o StrictHostKeyChecking=no ${app_server_user}@${app_server_ip} "\
                        docker login ${registry} && \
                        docker pull ${registry}/be-dumbmerch:${iamge_tag} && \
                        docker stop be-dumbmerch-${app_env} || true && \
                        docker rm -f be-dumbmerch-${app_env} || true && \
                        docker run -d --name be-dumbmerch-${app_env} -p 5000:5000 ${registry}/be-dumbmerch:${iamge_tag}"
                    """
                }
            }
        }

        stage('Test Running App with Wget Spider') {
            steps {
                echo "Testing backend server responsiveness...."
                sh "wget --spider --no-verbose ${app_url} || true"
            }
        }
    }

    post {
        success {
            echo "SUCCESS: All stages including SonarQube analysis, local image testing, Deploy, and Wget Spider test passed successfully!"
        }
        failure {
            echo "FAILURE: Pipeline failed during testing or deployment. Please check the console logs for details."
        }
    }
}


```


### - Berhasil CICD Testing fe-dumbmerch 

<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/484697eb-c165-4583-bd03-c3ccad19de2e" /></p>

### - Berhasil menjalankan SonarQube Analysi dengan waktu 21s
  + Hasil dari jenkins
    
<p align="center"><img width="1552" height="292" alt="image" src="https://github.com/user-attachments/assets/ca152978-9ac5-4346-9db1-07b98f702e14" /></p>

---

### - Berhasil dari Testing application accessibility via domain using Wget Spide
<p align="center"><img width="1555" height="513" alt="image" src="https://github.com/user-attachments/assets/9b886f09-1e12-4d58-8461-d17a1d30e8af" /></p>


### - Berhasil CICD Testing be-dumbmerch 

<p align="center"><img width="1919" height="1037" alt="image" src="https://github.com/user-attachments/assets/fed73f64-7607-430b-9f7c-d58b7c05a747" /></p>

### - Berhasil menjalankan SonarQube Analysi dengan waktu 21s
  + Hasil dari jenkins
    
<p align="center"><img width="1651" height="674" alt="image" src="https://github.com/user-attachments/assets/7adf4ccb-ca1c-402b-b70f-94e68c3e1c59" /></p>

---
+ Hasil dari dasboard SonarQube
<p align="center"><img width="1919" height="836" alt="image" src="https://github.com/user-attachments/assets/d51051a8-05f3-4a60-9e2b-d220897fcce4" /></p>

---

<p align="center"></p>

---
<p align="center"><img width="1919" height="340" alt="image" src="https://github.com/user-attachments/assets/f6e3b830-22b4-434f-9905-b734c0277b04" /></p>















