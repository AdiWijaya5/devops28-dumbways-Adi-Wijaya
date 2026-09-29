1. Didalam arsitektur Reverse Proxy, klien (user) tidak berhubungan langsung dengan server aplikasi (backend). Klien mengirimkan permintaan (request) ke reverse proxy (seperti Nginx atau HAProxy),
   lalu reverse proxy memfasilitasi dan meneruskan request tersebut ke server aplikasi di bagian dalam.

   <img width="400" height="415" alt="Struktur Web Server dengan Reverse Proxy drawio" src="https://github.com/user-attachments/assets/c2b8630a-4ebc-4fce-95d7-866738ab445a" />

2.- menambahkan ip server di host

   <img width="721" height="747" alt="image" src="https://github.com/user-attachments/assets/240fa59d-02bb-4c68-9e0e-bdfd2759b016" />
   
   - Buat File Konfigurasi Server Block Buka terminal, lalu buat file konfigurasi Nginx baru di dalam /etc/nginx/sites-enabled dengan nama "adiwijaya.conf", serta tambahkan konfigurasi seperti di bawah ini :
   
   <img width="604" height="179" alt="image" src="https://github.com/user-attachments/assets/a5d693e3-8cda-4400-b1e0-f625591d8541" />

   - Uji apakah sintaks konfigurasi sudah benar "sudo nginx -t".
  
  <img width="614" height="58" alt="image" src="https://github.com/user-attachments/assets/e0548e83-1628-4a59-be52-94ef2a183f1c" />

  - Uji apakah sintaks konfigurasi sudah benar dan Jika 'successful', reload layanan Nginx

  <img width="1092" height="451" alt="image" src="https://github.com/user-attachments/assets/2cd46e4e-7269-44f2-87f8-4bd741ca1af3" />

  - lalu lihat domain nya sudah berhasil di akses

 <img width="1919" height="1032" alt="image" src="https://github.com/user-attachments/assets/68043f69-a191-48cf-93af-cc0253236393" />
