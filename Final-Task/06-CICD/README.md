<img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/26689852-de85-428a-9702-3b2ee62b937d" />## CICD

### Penjelasan Teknis: Alur Jenkins CI/CD Pipeline (Staging Environment)

Skrip **Jenkins Pipeline (Declarative)** ini mengotomatiskan seluruh alur kerja CI/CD untuk lingkungan *staging*, mulai dari penarikan kode sumber, pengujian kualitas dan keamanan, pembuatan *image* Docker, hingga *deployment* otomatis ke server target melalui SSH.

---

### CI/CD Pipeline staging

```yaml


def secret = 'ssh_credentials_id'      
def directory = 'fe-dumbmerch'
def app_env = 'staging'      
def iamge_tag = "staging"     
def container = 'fe-dumbmerch'
def registry = 'registry.adi.studentdumbways.my.id'
def app_server_ip = '15.232.21.109'
def app_server_user = 'finaltask-adi'
def image ='registry.adi.studentdumbways.my.id/fe-dumbmerch:staging'

pipeline {
    agent any

    stages {
        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${app_env}..."
                checkout scm
            }
        }

        
        stage('Image Build') {
            steps {
                echo "Building Docker image : ${iamge_tag}..."
                sh "docker build -t ${registry}/fe-dumbmerch:${iamge_tag} -f Dockerfile ."
            }
        }    

        stage('Smoke Test') {
            steps {
                echo 'Running application smoke test...'
                sh """
                    docker run -d -p 3001:80 --name test-frontend-container ${image}
                    sleep 3
                    curl --fail http://15.232.21.109:3001 || exit 1
                    docker rm -f test-frontend-container
                """
            }
        }

        stage('Push Image into Private Registry (No Auth)') {
            steps {
                echo "Pushing image to private Docker registry without password..."
                sh "docker push ${registry}/fe-dumbmerch:${iamge_tag}"
            }
        }

        stage('SSH & Redeploy') {
            steps {
                echo "Connecting to server via SSH to pull and redeploy apps (Staging)..."
                sshagent(["${secret}"]) {
                    sh """
                        ssh -p 3333 -o StrictHostKeyChecking=no ${app_server_user}@${app_server_ip} "\
                        docker login ${registry} && \
                        docker pull ${registry}/fe-dumbmerch:${iamge_tag} && \
                        docker stop fe-dumbmerch-${app_env} || true && \
                        docker rm fe-dumbmerch-${app_env} || true && \
                        docker run -d --name fe-dumbmerch-${app_env} -p 80:80 ${registry}/fe-dumbmerch:${iamge_tag}"
                    """
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                echo "Cleaning up local build assets and workspace..."
                sh "docker rmi ${image} || true"
                cleanWs()
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


def secret = 'ssh_credentials_id'      
def directory = 'be-dumbmerch'
def app_env = 'staging'      
def iamge_tag = "staging"     
def container = 'be-dumbmerch'
def registry = 'registry.adi.studentdumbways.my.id'
def app_server_ip = '15.232.21.109'
def app_server_user = 'finaltask-adi'
def image ='registry.adi.studentdumbways.my.id/be-dumbmerch:staging'

pipeline {
    agent any

    stages {
        stage('Repository Pull') {
            steps {
                echo "Pulling code from repository branch: ${app_env}..."
                checkout scm
            }
        }

        
        stage('Image Build') {
            steps {
                echo "Building Docker image : ${iamge_tag}..."
                sh "docker build -t ${registry}/be-dumbmerch:${iamge_tag} -f Dockerfile ."
            }
        }

        // stage('Testing Code (SonarQubeScanner)') {
        //     steps {
        //         echo "Running SonarQube analysis for code quality..."
        //         script {
        //             def scannerHome = tool 'SonarQubeScanner'
        //             withSonarQubeEnv('SonarQubeServer') {
        //                 sh "${scannerHome}/bin/sonar-scanner \
        //                     -Dsonar.projectKey=fe-dumbmerch-staging \
        //                     -Dsonar.sources=."
        //             }
        //         }
        //     }
        // }
    

        stage('Smoke Test') {
            steps {
                echo 'Running application smoke test...'
                sh """
                    docker run -d -p 5001:80 --name test-frontend-container ${image}
                    sleep 3
                    curl --fail http://15.232.21.109:5001 || exit 1
                    docker rm -f test-frontend-container
                """
            }
        }

        stage('Push Image into Private Registry (No Auth)') {
            steps {
                echo "Pushing image to private Docker registry without password..."
                sh "docker push ${registry}/be-dumbmerch:${iamge_tag}"
            }
        }

        stage('SSH & Redeploy') {
            steps {
                echo "Connecting to server via SSH to pull and redeploy apps (Staging)..."
                sshagent(["${secret}"]) {
                    sh """
                        ssh -p 3333 -o StrictHostKeyChecking=no ${app_server_user}@${app_server_ip} "\
                        docker login ${registry} && \
                        docker pull ${registry}/be-dumbmerch:${iamge_tag} && \
                        docker stop be-dumbmerch-${app_env} || true && \
                        docker rm be-dumbmerch-${app_env} || true && \
                        docker run -d --name be-dumbmerch-${app_env} -p 5000:5000 ${registry}/be-dumbmerch:${iamge_tag}"
                    """
                }
            }
        }

        stage('Cleanup Workspace') {
            steps {
                echo "Cleaning up local build assets and workspace..."
                sh "docker rmi ${image} || true"
                cleanWs()
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
*   `registry` dan `IMAGE_TAG`: Menunjuk ke Private Docker Registry Anda dengan label versi `staging`.
*   `app_server_ip` dan `app_server_user`: Alamat IP dan kredensial user untuk akses remote server aplikasi.
*   `ssh_credentials_id`: Mengambil ID rahasia dari sistem Jenkins Credentials untuk mengamankan kunci SSH.

---

### 2. Alur Kerja Otomatisasi (Stages)

#### Tahap 1 & 2: Pengambilan & Pengujian Kode Sumber
*   **Repository Pull:** Jenkins menarik (*download*) kode program terbaru dari branch aktif repositori Git Anda.
*   **Testing Code (SonarQube):** Menjalankan pemindaian statis (*Static Application Security Testing - SAST*) menggunakan SonarQube Scanner untuk memeriksa kualitas kode program, mendeteksi *bug*, serta celah keamanan pada kode sebelum dirakit.

#### Tahap 3 & 4: Pembuatan & Pemindaian Image Docker
*   **Image Build:** Merakit kode program menjadi sebuah *image* Docker kustom dengan nama penanda `fe-dumbmerch:staging` berdasarkan berkas Dockerfile.
*   **Image Testing (Trivy Scan):** Memindai paket OS dan dependensi di dalam *image* Docker yang baru dibuat menggunakan Trivy untuk mendeteksi adanya celah keamanan kritis (*HIGH, CRITICAL vulnerabilities*).

#### Tahap 5: Pengunggahan Aset (Push Image)
*   **Push Image into Private Registry:** Melakukan autentikasi aman ke server Private Docker Registry Anda (`registry.adi.studentdumbways.my.id`), lalu mengunggah (*push*) *image* tersebut agar bisa digunakan oleh server lain.



---

### 3. Notifikasi Akhir (Post Actions)
*   **Success:** Mencetak log sukses ke konsol Jenkins jika seluruh tahapan dari awal hingga akhir berhasil dilewati tanpa eror.
---



### Membuat GitHub PAT (Classic)

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/8ecf6159-bec2-47eb-8015-01cf524689f0" /></p>



<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/44f9d342-ca37-4a40-a831-128f74687894" />
</p>


### Install With Ansible
<p align="center"><img width="907" height="555" alt="image" src="https://github.com/user-attachments/assets/39da3108-6e09-4f55-a9c3-1543271379c6" /></p>



### add Plugin Stage SonarQube Scanner

<p align="center"><img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/5a408789-7cc2-478b-931a-fbecc6b6a0f4" />
</p>

### daftar Tool Scanner-nya

- Klik tombol + Add SonarQube Scanner yang ada di bagian bawah gambar tersebut.

  + Isi Name dengan tulisan SonarQubeScanner (tanpa spasi, persis seperti di Jenkinsfile).
  + Centang opsi Install automatically.
  + Pilih versi scanner pada menu dropdown yang muncul.
  + Klik tombol Save di bagian paling bawah halaman konfigurasi Jenkins.

<p align="center"><img width="957" height="1035" alt="image" src="https://github.com/user-attachments/assets/34f39dae-2d6a-481c-872f-a6146d456d47" />
</p>


<p align="center"><img width="956" height="1040" alt="image" src="https://github.com/user-attachments/assets/d26d5afc-134d-4845-859e-b964e6847c79" />
</p>

## jalankan asnible test code servber

<p align="center"><img width="958" height="1014" alt="image" src="https://github.com/user-attachments/assets/52da0b71-6658-4f02-a642-b2627a4be768" /></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>












