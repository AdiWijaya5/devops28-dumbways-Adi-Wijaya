# Final Task Bootcamp DumbWays DevOps - Adi Wijaya

Repositori ini berisi kumpulan Final Taks

# Dumbmerch Project - Deployment & Architecture Guide

Dokumentasi ini berisi panduan arsitektur, spesifikasi lingkungan, dan tata cara deployment aplikasi **Dumbmerch** yang terbagi ke dalam 3 server terpisah (Gateway, Appserver, dan Database).

---

## 🏗️ Topologi & Pembagian Server

Sistem ini dibagi menjadi tiga server utama agar lebih aman, terstruktur, dan terisolasi:

1. **Server Gateway (`gateway`)**
   * **Komponen**: Web Server (Nginx/Reverse Proxy) & **Docker Private Registry** (Ber-SSL HTTPS / Tanpa Password).
   * **Fungsi**: Sebagai pintu masuk trafik utama pengguna dan pusat penyimpanan *image* Docker.
2. **Server Appserver (`appserver`)**
   * **Komponen**: Node.js (v18) & Golang (v1.21).
   * **Fungsi**: Tempat melakukan *build* Docker image untuk Frontend (`fe-dumbmerch`) dan Backend (`be-dumbmerch`) serta menjalankan container aplikasi.
3. **Server Database (`database`)**
   * **Komponen**: PostgreSQL.
   * **Fungsi**: Dikhususkan khusus untuk penyimpanan data aplikasi.

---


## 📂 Daftar Final Task & Navigasi 

| Project | Tahapan | Status |
| :--- | :--- | :--- |
| **[Project-1](./01-Provisioning)** | Provisioning | Selesai |
| **[Project-2](./02-Repository)** | Repository  | Selesai |
| **[Project-3](./03-Servers)** | Servers | Selesai |
| **[Project-4](./04-Docker-Registry-Private)** | Docker Registry Private  | Selesai |
| **[Project-5](./05-Deployment-Apps)** | Deployment Apps | Selesai |
| **[Project-6](./06-CICD)** | CICD  | Selesai |
| **[Project-7](./07-Testing)** | Testing | Selesai |
| **[Project-8](./08-Monitoring)** | Monitoring | Selesai |
| **[Project-9](./09-Web-Server)** | Web Server | Selesai |
| **[Project-10](./10-Kubernetes)** | Kubernetes | Selesai |


---
