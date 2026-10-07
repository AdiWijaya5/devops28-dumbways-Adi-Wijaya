## CICD

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
                        docker run -d --name fe-dumbmerch-${app_env} -p 3000:80 ${registry}/fe-dumbmerch:${iamge_tag}"
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

### - Tambahkan Credentials Globlal untuk Target_IP

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/b704587d-1c68-4c52-b8bb-2f677caa9d37" /></p>


### - Tambahkan Credentials Globlal untuk Target_User

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/006c1386-2f8f-4bfd-872c-b2decab33a5c" /></p>





### create CICD job deploy staging

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/f9dc0790-c8a1-47b6-a20a-9fcf5424283b" />
</p>

<p align="center"><img width="1914" height="1045" alt="image" src="https://github.com/user-attachments/assets/97dd19f9-2fe0-4596-89a8-51b37a9ede85" />
</p>


<p align="center"><img width="1912" height="1053" alt="image" src="https://github.com/user-attachments/assets/84816782-7247-424c-8ff3-9a296a62cd64" />
</p>

<p align="center"><img width="1919" height="1044" alt="image" src="https://github.com/user-attachments/assets/fb1daa4d-90d4-43c5-9de6-3c948f2db270" /></p>

### Job 1 Repository Pull Pulling code from repository branch: staging

<p align="center"><img width="1536" height="687" alt="image" src="https://github.com/user-attachments/assets/e5a8281c-8bff-49c4-8e34-6b6d856edb8f" /></p>

### Job 2 Image build on top Docker use Dockerfile

<p align="center"><img width="1533" height="650" alt="image" src="https://github.com/user-attachments/assets/c56fb922-ce0d-4ba5-894c-5c166adb6413" /></p>

### Job 3 Testing Code With Smoke Test

<p align="center"><img width="1538" height="551" alt="image" src="https://github.com/user-attachments/assets/69cd3f2c-8a5b-4474-ac3e-338d6646eac6" /></p>

### Job 4 Push Image into Docker Registry Private(No Auth)

<p align="center"><img width="1537" height="643" alt="image" src="https://github.com/user-attachments/assets/0b0ee7e8-36e4-4760-8484-f8471e9b33ca" /></p>

### Job 5 Connecting to Server via SSH to pull and Redeploy: staging

<p align="center"><img width="1551" height="525" alt="image" src="https://github.com/user-attachments/assets/0ac68c25-1d4c-40c7-9ec8-47a05112e696" /></p>

### Job 6 Cleanup Workspace

<p align="center"><img width="1548" height="517" alt="image" src="https://github.com/user-attachments/assets/1bb797f3-07c7-4264-bf1e-e6115559182b" /></p>

### Job 6 Notif Staging Success

<p align="center"><img width="1547" height="360" alt="image" src="https://github.com/user-attachments/assets/39c67ab2-7635-4d26-94e0-34ff022beaf2" /></p>


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









