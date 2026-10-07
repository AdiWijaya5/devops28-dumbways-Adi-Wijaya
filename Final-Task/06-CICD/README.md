# CICD

## Penjelasan Teknis: Alur Jenkins CI/CD Pipeline (Staging Environment)

Skrip **Jenkins Pipeline (Declarative)** ini mengotomatiskan seluruh alur kerja CI/CD untuk lingkungan *staging*, mulai dari penarikan kode sumber, pengujian kualitas dan keamanan, pembuatan *image* Docker, hingga *deployment* otomatis ke server target melalui SSH.

---

## CI/CD Pipeline staging

```yaml


def secret = 'ssh_credentials_id'      
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
                        docker rm -f fe-dumbmerch-${app_env} || true && \
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

---


## CI/CD Pipeline be staging

----

```yaml


def secret = 'ssh_credentials_id'      
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
 
        stage('Smoke Test API') {
            steps {
                sh """
                    docker rm -f test-backend-container || true
                    docker run -d -p 5001:5000 --name test-backend-container ${image}
                    sleep 5
                    
                    # Cek apakah container berjalan
                    [ "\$(docker inspect -f '{{\$.State.Running}}' test-backend-container)" = "true" ] || { docker logs test-backend-container; docker rm -f test-backend-container; exit 1; }
                    
                    # Bersihkan
                    docker rm -f test-backend-container
                    echo "Smoke test passed!"
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
## Konfigurasi Global

Sebelum menjalankan pipeline ini di Jenkins, pastikan environment berikut sudah siap:
* **Jenkins Credentials:** ID credential bertipe SSH dengan nama `ssh_credentials_id` sudah diset di Jenkins untuk akses remote server.
* **Private Registry:** Akses ke `registry.adi.studentdumbways.my.id` sudah dikonfigurasi dengan baik.

Berikut adalah variabel global yang digunakan di dalam Jenkinsfile:
* `secret = 'ssh_credentials_id'` (Credential Jenkins untuk SSH key)
* `app_env = 'staging'` (Environment target deployment)
* `image_tag = "staging"` *(Catatan: ada typo kecil pada variabel `iamge_tag` di skrip asli)*
* `container = 'fe-dumbmerch'` (Nama dasar container aplikasi)
* `registry = 'registry.adi.studentdumbways.my.id'` (Alamat Private Docker Registry)
* `app_server_ip = '15.232.21.109'` (Alamat IP Server Staging)
* `app_server_user = 'finaltask-adi'` (Username SSH Server Staging)
* `image = 'registry.adi.studentdumbways.my.id/fe-dumbmerch:staging'` (Kombinasi lengkap nama image dan tag)

## Penjelasan Alur Pipeline (Stages)

### 1. Repository Pull (`Repository Pull`)
* **Aksi:** `checkout scm`
* **Fungsi:** Mengambil source code terbaru dari repository Git sesuai dengan branch/environment yang sedang berjalan (`staging`).

### 2. Image Build (`Image Build`)
* **Aksi:** `docker build -t ${registry}/fe-dumbmerch:${iamge_tag} -f Dockerfile .`
* * **Aksi:** `docker build -t ${registry}/be-dumbmerch:${iamge_tag} -f Dockerfile .`
* **Fungsi:** Membangun Docker image secara lokal di server Jenkins menggunakan `Dockerfile` yang ada di root project, lengkap dengan penamaan tag registry.

### 3. Smoke Test (`Smoke Test`)
* **Aksi:** 
  1. Menjalankan container uji coba sementara (`docker run`) di port `3001`.
  2. Memberi jeda 3 detik (`sleep 3`) agar aplikasi siap.
  3. Melakukan pengecekan koneksi (`curl --fail`) ke endpoint server/container.
  4. Menghapus container uji tersebut (`docker rm -f`).
* **Fungsi:** Memastikan image yang baru dibuild benar-benar sehat dan bisa merespons *request* sebelum dikirim ke production/staging server.

### 4. Push ke Private Registry (`Push Image into Private Registry`)
* **Aksi:** `docker push ${registry}/fe-dumbmerch:${iamge_tag}`
* * **Aksi:** `docker push ${registry}/be-dumbmerch:${iamge_tag}`
* **Fungsi:** Mengunggah Docker image yang sudah lolos uji *smoke test* ke private Docker registry.

### 5. Deployment ke Server (`SSH & Redeploy`)
* **Aksi:** Menggunakan plugin `sshagent` untuk meremote server tujuan via SSH (port `3333`). Perintah yang dijalankan di server:
  1. `docker login`: Masuk ke private registry dari sisi server.
  2. `docker pull`: Mengunduh image terbaru dari registry.
  3. `docker stop` & `docker rm`: Menghentikan dan membersihkan container staging yang lama (mengabaikan error jika container belum ada pakai `|| true`).
  4. `docker run`: Menjalankan container baru dengan nama `fe-dumbmerch-staging` dan melayout port ke `3000:80`.
  5. `docker run`: Menjalankan container baru dengan nama `be-dumbmerch-staging` dan melayout port ke `5000:5000`.

### 6. Pembersihan Workspace (`Cleanup Workspace`)
* **Aksi:** `docker rmi` dan `cleanWs()`
* **Fungsi:** Menghapus sisa image Docker yang menumpuk di mesin Jenkins serta membersihkan workspace agar tidak membebani kapasitas penyimpanan server Jenkins.

---

## Penanganan Status (`Post Actions`)

* **`success`:** Menampilkan log sukses apabila seluruh *stage* dari awal sampai akhir berhasil dieksekusi tanpa kendala.
* **`failure`:** Memberikan informasi jika pipeline gagal, sehingga developer dapat segera memeriksa bagian *stage* mana yang mengalami error melalui Jenkins Console Output.

---

### Membuat GitHub PAT (Classic)

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/8ecf6159-bec2-47eb-8015-01cf524689f0" /></p>

<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/44f9d342-ca37-4a40-a831-128f74687894" />
</p>

---

### - Tambahkan Credentials Globlal untuk Target_IP

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/b704587d-1c68-4c52-b8bb-2f677caa9d37" /></p>

--

### - Tambahkan Credentials Globlal untuk Target_User

<p align="center"><img width="1919" height="1038" alt="image" src="https://github.com/user-attachments/assets/006c1386-2f8f-4bfd-872c-b2decab33a5c" /></p>

---

## Create CICD job deploy frontend:staging

<p align="center"><img width="1919" height="1041" alt="image" src="https://github.com/user-attachments/assets/f9dc0790-c8a1-47b6-a20a-9fcf5424283b" />
</p>

<p align="center"><img width="1914" height="1045" alt="image" src="https://github.com/user-attachments/assets/97dd19f9-2fe0-4596-89a8-51b37a9ede85" />
</p>

---

<p align="center"><img width="1912" height="1053" alt="image" src="https://github.com/user-attachments/assets/84816782-7247-424c-8ff3-9a296a62cd64" />
</p>

<p align="center"><img width="1919" height="790" alt="image" src="https://github.com/user-attachments/assets/eba67e96-0d23-453f-93b0-5d698f1fd6a7" /></p>

### Job 1 Repository Pull Pulling code from repository branch: staging

<p align="center"><img width="1536" height="687" alt="image" src="https://github.com/user-attachments/assets/e5a8281c-8bff-49c4-8e34-6b6d856edb8f" /></p>

---

### Job 2 Image build on top Docker use Dockerfile

<p align="center"><img width="1533" height="650" alt="image" src="https://github.com/user-attachments/assets/c56fb922-ce0d-4ba5-894c-5c166adb6413" /></p>

---

### Job 3 Testing Code With Smoke Test

<p align="center"><img width="1538" height="551" alt="image" src="https://github.com/user-attachments/assets/69cd3f2c-8a5b-4474-ac3e-338d6646eac6" /></p>

---

### Job 4 Push Image into Docker Registry Private(No Auth)

<p align="center"><img width="1537" height="643" alt="image" src="https://github.com/user-attachments/assets/0b0ee7e8-36e4-4760-8484-f8471e9b33ca" /></p>

---

### Job 5 Connecting to Server via SSH to pull and Redeploy: staging

<p align="center"><img width="1551" height="525" alt="image" src="https://github.com/user-attachments/assets/0ac68c25-1d4c-40c7-9ec8-47a05112e696" /></p>

---

### Job 6 Cleanup Workspace

<p align="center"><img width="1548" height="517" alt="image" src="https://github.com/user-attachments/assets/1bb797f3-07c7-4264-bf1e-e6115559182b" /></p>

---

### Job 6 Notif Staging Success

<p align="center"><img width="1547" height="360" alt="image" src="https://github.com/user-attachments/assets/39c67ab2-7635-4d26-94e0-34ff022beaf2" /></p>

---

## Create CICD job deploy bacend:staging

<p align="center"><img width="1919" height="1032" alt="image" src="https://github.com/user-attachments/assets/7beda121-0552-4983-8987-00094b302d1e" /></p>


<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>
<p align="center"></p>









