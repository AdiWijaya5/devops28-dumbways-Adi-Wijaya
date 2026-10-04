## CICD

### Penjelasan Teknis: Alur Jenkins CI/CD Pipeline (Staging Environment)

Skrip **Jenkins Pipeline (Declarative)** ini mengotomatiskan seluruh alur kerja CI/CD untuk lingkungan *staging*, mulai dari penarikan kode sumber, pengujian kualitas dan keamanan, pembuatan *image* Docker, hingga *deployment* otomatis ke server target melalui SSH.

---

### CI/CD Pipeline staging

```yaml


pipeline {
    agent any

    environment {
        REGISTRY = "registry.adi.studentdumbways.my.id"
        
        // Logika branch-aware untuk staging
        IMAGE_TAG = "staging"
        APP_ENV = "staging"
        
        APP_SERVER_IP = "15.232.4.76"
        APP_SERVER_USER = "finaltask-adi"
        SSH_CREDENTIALS_ID = "{{ secrets.SSH_CREDENTIALS_ID }}" // ID Credential Jenkins untuk SSH ke Server
    }

    stages {
        // 1. REPOSITORY PULL
        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${env.BRANCH_NAME}..."
                checkout scm
            }
        }

        // 2. TESTING CODE (SONARQUBE)
        stage('Testing Code (SonarQube)') {
            steps {
                echo "Running SonarQube analysis for code quality..."
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fe-dumbmerch-staging \
                            -Dsonar.sources=."
                    }
                }
            }
        }

        // 3. IMAGE BUILD
        stage('Image Build') {
            steps {
                echo "Building Docker image with tag: ${env.IMAGE_TAG}..."
                script {
                    appImage = docker.build("${env.REGISTRY}/fe-dumbmerch:${env.IMAGE_TAG}", "-f Dockerfile .")
                }
            }
        }

        // 4. ADDITIONAL TESTING (TRIVY VULNERABILITY SCAN)
        stage('Image Testing (Trivy Scan)') {
            steps {
                echo "Scanning Docker image for vulnerabilities using Trivy..."
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG}"
            }
        }

        // 5. PUSH IMAGE INTO PRIVATE REGISTRY
        stage('Push Image into Private Registry') {
            steps {
                echo "Pushing image to private Docker registry..."
                script {
                    docker.withRegistry("https://${env.REGISTRY}", env.CREDENTIALS_ID) {
                        appImage.push()
                    }
                }
            }
        }

        // 6. SSH INTO SERVER & PULL IMAGE & REDEPLOY
        stage('SSH & Redeploy') {
            steps {
                echo "Connecting to server via SSH to pull and redeploy apps (Staging)..."
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.APP_SERVER_USER}@${env.APP_SERVER_IP} "\
                        docker login ${env.REGISTRY} && \
                        docker pull ${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG} && \
                        docker stop fe-dumbmerch-${env.APP_ENV} || true && \
                        docker rm fe-dumbmerch-${env.APP_ENV} || true && \
                          ${env.REGISTRY}/fe-dumbmerch:${env.IMAGE_TAG}"
                    """
                }
            }
        }

    post {
        success {
            echo "Staging CI/CD Pipeline successfully finished all stages!"
        }
        failure {
            echo "Staging CI/CD Pipeline failed. Please check logs on each stage."
        }
    }
}


```

### CI/CD Pipeline be staging

```yaml


pipeline {
    agent any

    environment {
        REGISTRY = "registry.adi.studentdumbways.my.id"
        
        // Logika branch-aware untuk staging
        IMAGE_TAG = "staging"
        APP_ENV = "staging"
        
        APP_SERVER_IP = "15.232.4.76"
        APP_SERVER_USER = "finaltask-adi"
        SSH_CREDENTIALS_ID = "{{ secrets.SSH_CREDENTIALS_ID }}" // ID Credential Jenkins untuk SSH ke Server
    }

    stages {

        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${env.BRANCH_NAME}..."
                checkout scm
            }
        }


        stage('Testing Code (SonarQube)') {
            steps {
                echo "Running SonarQube analysis for code quality..."
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeServer') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=be-dumbmerch-staging \
                            -Dsonar.sources=."
                    }
                }
            }
        }


        stage('Image Build') {
            steps {
                echo "Building Docker image with tag: ${env.IMAGE_TAG}..."
                script {
                    appImage = docker.build("${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG}", "-f Dockerfile .")
                }
            }
        }


        stage('Image Testing (Trivy Scan)') {
            steps {
                echo "Scanning Docker image for vulnerabilities using Trivy..."
                sh "trivy image --exit-code 0 --severity HIGH,CRITICAL ${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG}"
            }
        }

        stage('Push Image into Private Registry') {
            steps {
                echo "Pushing image to private Docker registry..."
                script {
                    docker.withRegistry("https://${env.REGISTRY}", env.CREDENTIALS_ID) {
                        appImage.push()
                    }
                }
            }
        }

        stage('SSH & Redeploy') {
            steps {
                echo "Connecting to server via SSH to pull and redeploy apps (Staging)..."
                sshagent([env.SSH_CREDENTIALS_ID]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${env.APP_SERVER_USER}@${env.APP_SERVER_IP} "\
                        docker login ${env.REGISTRY} && \
                        docker pull ${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG} && \
                        docker stop be-dumbmerch-${env.APP_ENV} || true && \
                        docker rm be-dumbmerch-${env.APP_ENV} || true && \
                        docker run -d \
                          --name be-dumbmerch-${env.APP_ENV} \
                          -p 5001:5000 \
                          -e DB_HOST=15.232.4.76 \
                          -e DB_PORT=5432 \
                          -e DB_USER=dumbmerch \
                          -e DB_PASSWORD=neonjago \
                          -e DB_NAME=dumbmerch_db \
                          -e DB_SSLMODE=disable \
                          ${env.REGISTRY}/be-dumbmerch:${env.IMAGE_TAG}"
                    """
                }
            }
        }

    post {
        success {
            echo "Staging CI/CD Pipeline successfully finished all stages!"
        }
        failure {
            echo "Staging CI/CD Pipeline failed. Please check logs on each stage."
        }
    }
}


```



### 1. Blok Inisiasi Lingkungan (Environment Variables)
Mendefinisikan variabel global yang akan digunakan di seluruh tahapan otomatisasi:
*   `REGISTRY` dan `IMAGE_TAG`: Menunjuk ke Private Docker Registry Anda dengan label versi `staging`.
*   `APP_SERVER_IP` dan `USER`: Alamat IP dan kredensial user untuk akses remote server aplikasi (`15.232.4.76`).
*   `SSH_CREDENTIALS_ID`: Mengambil ID rahasia dari sistem Jenkins Credentials untuk mengamankan kunci SSH.

---

### 2. Alur Kerja Otomatisasi (Stages)

#### Tahap 1 & 2: Pengambilan & Pengujian Kode Sumber
*   **Repository Pull:** Jenkins menarik (*download*) kode program terbaru dari branch aktif repositori Git Anda.
*   **Testing Code (SonarQube):** Menjalankan pemindaian statis (*Static Application Security Testing - SAST*) menggunakan SonarQube Scanner untuk memeriksa kualitas kode program, mendeteksi *bug*, serta celah keamanan pada kode sebelum dirakit.

#### Tahap 3 & 4: Pembuatan & Pemindaian Image Docker
*   **Image Build:** Merakit kode program menjadi sebuah *image* Docker kustom dengan nama penanda `fe-dumbmerch:staging` berdasarkan berkas Dockerfile.
*   **Image Testing (Trivy Scan):** Memindai paket OS dan dependensi di dalam *image* Docker yang baru dibuat menggunakan Trivy untuk mendeteksi adanya celah keamanan kritis (*HIGH, CRITICAL vulnerabilities*).

#### Tahap 5: Pengunggahan Aset (Push Image)
*   **Push Image into Private Registry:** Melakukan autentikasi aman ke server Private Docker Registry Anda (`https://studentdumbways.my.id`), lalu mengunggah (*push*) *image* tersebut agar bisa digunakan oleh server lain.



---

### 3. Notifikasi Akhir (Post Actions)
*   **Success:** Mencetak log sukses ke konsol Jenkins jika seluruh tahapan dari awal hingga akhir berhasil dilewati tanpa eror.
*   **Failure:** Mencetak log peringatan gagal jika ada salah satu tahapan yang terhenti akibat eror (seperti gagal uji SonarQube, Trivy, atau aplikasi tidak merespons).
---


### bikin FE CI/CD Pipeline Trigger

- bikin file .github/workflows/ci-cd.yml lalu git push 

### workflows production
```yaml

name: FE CI/CD Pipeline Trigger

on:
  push:
    branches:
      - staging
jobs:
  trigger-fe-staging:
    if: github.ref == 'refs/heads/staging'
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Jenkins FE Staging
        uses: appleboy/jenkins-action@master
        with:
          url: "http://15.232.21.109:8080"
          user: "${{ secrets.JENKINS_USERNAME }}"
          token: "${{ secrets.JENKINS_API_TOKEN }}"
          job: "fe-dumbmerch-staging"

```

### workflows production

```yaml

name: CI/CD Pipeline Trigger

on:
  push:
    branches:
      - production

jobs:
  trigger-jenkins-production:
    if: github.ref == 'refs/heads/production'
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Jenkins Production
        uses: appleboy/jenkins-action@master
        with:
          url: "http://15.232.21.109:8080"
          user: "${{ secrets.JENKINS_USERNAME }}"
          token: "${{ secrets.JENKINS_API_TOKEN }}"
          job: "fe-dumbmerch-production"

```

## Penjelasan Teknis: GitHub Actions Webhook Trigger

Skrip **GitHub Actions** ini berfungsi sebagai **pemicu otomatis (Webhook)** untuk memerintahkan server **Jenkins** agar langsung memulai *pipeline* deployment setiap kali ada perubahan kode Frontend.

---

### 1. Kondisi Pemicu (Trigger)
*   **Event:** Otomatis aktif hanya ketika terjadi pengiriman kode (`git push`) atau penggabungan kode (*merge*) ke dalam branch **`staging/production`**.

### 2. Lingkungan Eksekusi (Jobs)
*   Berjalan di atas mesin virtual virtual (*Runner*) gratis berbasis **Ubuntu** milik GitHub.

### 3. Langkah Kerja Kontrol (Steps)
*   **Koneksi API Jenkins:** Menggunakan plugin `appleboy/jenkins-action` untuk menembak IP server Jenkins (`15.232.21.109:8080`).
*   **Keamanan Kredensial:** Mengambil *Username* dan *API Token* Jenkins secara aman melalui fitur **GitHub Secrets**
*   **Target Eksekusi:** Memerintahkan Jenkins untuk langsung menjalankan (*build*) proyek CI/CD dengan nama spesifik: **`fe-dumbmerch-staging/fe-dumbmerch-production`**.


## Actions secrets and variables

<p><img width="1918" height="994" alt="image" src="https://github.com/user-attachments/assets/480cee82-ead4-4134-a873-641858ccb3f1" /></p>


