### Install With Ansible

<p align="center"><img width="958" height="1014" alt="image" src="https://github.com/user-attachments/assets/52da0b71-6658-4f02-a642-b2627a4be768" /></p>


### add Plugin Stage SonarQube Scanner

<p align="center"><img width="957" height="565" alt="image" src="https://github.com/user-attachments/assets/5a408789-7cc2-478b-931a-fbecc6b6a0f4" />
</p>




- login
---
<p align="center"><img width="1919" height="1040" alt="Screenshot 2026-10-07 024423" src="https://github.com/user-attachments/assets/1ca0d290-2d0b-4a4a-95c8-38ce4f73b334" /></p>


<p align="center"><img width="1919" height="1034" alt="image" src="https://github.com/user-attachments/assets/484315bc-bbcf-415b-85d0-629bfd53ee23" />
</p>

- generate token SonarQube

<p align="center"><img width="1919" height="1040" alt="image" src="https://github.com/user-attachments/assets/45e31b36-4f38-4f74-b134-5e52a3a643d1" />
</p>

- buat secarlet sonarqube-token di jenkins

<p align="center"><img width="1919" height="1044" alt="Screenshot 2026-10-07 051117" src="https://github.com/user-attachments/assets/1da08c6a-4715-4f42-8574-461dfa1a0f1c" />
</p>

- tambahkan SonarQube installations
  + Buka Manage Jenkins > System.
  + Cari bagian SonarQube installations
  + Pastikan kolom Name diisi persis: ```SonarQubeServer```
  + Pastikan kolom url Server URL isi dengan url server testing ```http://localhost:9000```
  + Pastikan kolom Server authentication token sudah memilih: ```sonarqube-token``` yang sudah di buat tadi
       

<p align="center"><img width="1671" height="769" alt="image" src="https://github.com/user-attachments/assets/7362f899-3600-4a58-8ca7-7dfaffe11c0d" /></p>


### CI/CD Pipeline fe staging

```yaml


def secret = 'ssh_credentials_id'      
def app_env = 'staging'      
def iamge_tag = "staging"     
def container = 'be-dumbmerch'
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



         stage('Testing Code (SonarQubeScanner)') {
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
    

        stage('Smoke Test') {
            steps {
                echo 'Running application smoke test...'
                sh """
                    docker run -d -p 5001:5000 --name test-frontend-container ${image}
                    sleep 3
                    curl --fail http://15.232.21.109:5001 || exit 1
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
                        docker rm -f be-dumbmerch-${app_env} || true && \
                        docker run -d --name fe-dumbmerch-${app_env} -p 5000:5000 ${registry}/be-dumbmerch:${iamge_tag}"
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
