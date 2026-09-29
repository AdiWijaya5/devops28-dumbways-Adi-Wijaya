1. Akses server menggunakan terminal
   
   - Generate SSH Key di komputer lokal
   <img width="907" height="781" alt="image" src="https://github.com/user-attachments/assets/c6bfff4f-a6d6-4fbb-acbe-f8c87e23995b" />

  - lalu Salin Public Key ke server:
    <img width="1032" height="247" alt="image" src="https://github.com/user-attachments/assets/965ca3b2-0447-4be9-af14-d5087c5f1537" />

  - lalu masuk ke Teminal/cmd
    <img width="963" height="556" alt="image" src="https://github.com/user-attachments/assets/7a94e130-54fa-412e-b55d-f3266fb02f06" />
2. Konfigurasi ssh kalian agar bisa di akses hanya menggunakan publickey (password opsional, bisa dimatikan)

  - masuk ke direktori /etc/ssh lalu masuk ketik perintah "sudo nano sshd_config" passwordAuthentication jadi "no" untuk menonaktifkan login dengan password
    <img width="871" height="788" alt="image" src="https://github.com/user-attachments/assets/088e4527-333f-4063-a732-75501445f1cd" />

 - lalu commit sudo systemctl restart ssh
  <img width="706" height="875" alt="image" src="https://github.com/user-attachments/assets/af7f431c-3e96-4e5f-a73d-6e465639b9bd" />

3. Step-by-Step Text Manipulation
   1. Membuat file teks baru (echo): "echo -e "DevOps Day 3\nServer Configuration\nApp Port: 3000\nApp Port: 5000" > testfile.txt. lalu kita cek dengan comment ls
    <img width="1115" height="314" alt="image" src="https://github.com/user-attachments/assets/52e61182-35d2-40ef-b210-9a4840603d3a" />

  2. Menampilkan isi file (cat):
    <img width="978" height="449" alt="image" src="https://github.com/user-attachments/assets/2dea27ec-8c0d-4679-bee7-d2de2fc92015" />

  3. Mencari baris dengan kata kunci tertentu (grep):
     <img width="420" height="159" alt="image" src="https://github.com/user-attachments/assets/64d2f3ee-eee4-4633-9b0d-78961af0eee3" />

  4. Mengubah/mengganti teks dalam file (sed): contoh engganti kata 'Server' menjadi 'System' dan menampilkan hasilnya
    <img width="458" height="243" alt="image" src="https://github.com/user-attachments/assets/ee6b7c4e-b7a4-4160-a621-65a50804294f" />

3. 1. Konfigurasi dan Aktifkan UFW Firewall dengan comment sudo ufw allow port/tcp
   <img width="695" height="426" alt="Screenshot 2026-08-19 144155" src="https://github.com/user-attachments/assets/54523562-e963-48d4-9859-1eae7e028963" />

  2. lalu aktifkan UFW nya dengan cara comment "sudo ufw enable"
  <img width="753" height="246" alt="image" src="https://github.com/user-attachments/assets/7e58dc25-9806-4103-8c06-337affe2a3b2" />

  4. Lalu Cek status UFW untuk memastikan aturan port sudah aktif: dengan comment "sudo ufw status verbose" di sini sudah aktif semua
  <img width="669" height="484" alt="image" src="https://github.com/user-attachments/assets/7c619af7-888d-41a7-b062-f4d48fd3ec5b" />

    
      

    
    

