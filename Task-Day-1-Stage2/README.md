Create new user for all of your server

  1. buat new user server
    <img width="949" height="405" alt="image" src="https://github.com/user-attachments/assets/fd3fac03-6e45-40b6-b6e7-e7d6bff4cc51" />
  
  2. Berikan Akses Sudo ke new user
    <img width="959" height="178" alt="image" src="https://github.com/user-attachments/assets/8451b5b9-f0b8-4200-9fa6-cdbb6734da92" />
  
  3. buau file authorized_keys dan Atur permission file
    <img width="958" height="395" alt="image" src="https://github.com/user-attachments/assets/8d537aac-d54f-439e-8d83-36edd3392d60" />
    
  4. Uji Coba Login
     <img width="951" height="665" alt="image" src="https://github.com/user-attachments/assets/6dd2b160-29fc-4f22-ab01-2bc4baebe416" />

Deploy database MySQ

  1. ubah MySQL bind address on /etc/mysql/mysql.conf.d/mysqld.cnf
     <img width="951" height="499" alt="image" src="https://github.com/user-attachments/assets/96837a71-6e56-490e-aa5f-bfc658ac000a" />
     
  2. masuk ke user root database
     <img width="949" height="427" alt="image" src="https://github.com/user-attachments/assets/1c1d5ff7-b94f-4782-a880-e2ca9699d64f" />

  3. buat database baru demo dan buat table dummy transaction
     <img width="889" height="398" alt="image" src="https://github.com/user-attachments/assets/c9d1a035-e7a4-4785-b4be-1896e13f5f2c" />
 
  5. lihat data base dengan koment "show DATABASES" yang sudah di buat 
     <img width="915" height="609" alt="image" src="https://github.com/user-attachments/assets/9417841f-e3ec-4e15-928b-94e9d5a6d7e8" />
     
  6. buat tabel transaction sebagai berikut
     <img width="946" height="206" alt="image" src="https://github.com/user-attachments/assets/a5dfd2d3-84f9-469a-9cfc-2e2259665441" />
     
  7. setelah itu buat data dummy pada tabel transaction
     <img width="927" height="98" alt="image" src="https://github.com/user-attachments/assets/1b435533-f034-40df-a003-3647a6be8c18" />

  8. membuat role dan hak akses, masuk ke data base demo dengan perintah "USE demo;"
     lalau buat role baru dengan perintah "CREATE ROLE 'admin', 'guest';
     <img width="476" height="88" alt="image" src="https://github.com/user-attachments/assets/8b73edb4-464b-4fbe-8a50-e782aff0180a" />

  9. buat user baru untuk admin dan masukan user ke role admin
    <img width="945" height="289" alt="image" src="https://github.com/user-attachments/assets/38b0e725-a621-4359-8aa0-56e1a0b92306" />

  10. Buat user baru untuk Tamu
    <img width="946" height="206" alt="image" src="https://github.com/user-attachments/assets/76a56472-5946-44e9-bad7-825ed44e873d" />

  11. lalu aktifkan role secara otomatis saat user login dan simpan perubahan dengan komen "FLUSH PRICILEGES"
    <img width="946" height="193" alt="image" src="https://github.com/user-attachments/assets/8d586df1-3c56-44f5-8cb5-b2920efa73d8" />

  12. lalu login ke role admin
    <img width="924" height="403" alt="image" src="https://github.com/user-attachments/assets/8c82e154-fd24-49dc-a568-79f91958be3f" />

  13. lalu masuk ke database demo
    <img width="936" height="355" alt="image" src="https://github.com/user-attachments/assets/26b15568-60f0-4fa6-a8e6-67a696c6040c" />

  14. lalu tambahkan data dengan komen "INSERT INTO transaction (customer_name, amount, status) VALUES
      ('Admin Test', 50000.00, 'success');" dan untuk mengeceknya "SELEC * FROM transaction;"
    <img width="939" height="373" alt="image" src="https://github.com/user-attachments/assets/0d2eca58-9057-4723-a3d6-650d6e8c0878" />

  15. lalu coba untuk UPDATE data dengan komen "UPDATE transaction SET amount =2000.00 WHERE customer_name ='Admin Adi test';"
      dan ini hasilnya
    <img width="926" height="290" alt="image" src="https://github.com/user-attachments/assets/67c683c2-a410-4311-90ae-18fa83d9c066" />

  16. lalu coba untuk UPDATE data dengan komen "DELETE FROM transaction WHERE customer_name = 'Dewi Lestari';"
      dan ini hasilnya
    <img width="930" height="301" alt="image" src="https://github.com/user-attachments/assets/5d2bd38c-f027-41a8-984b-e381a75b534a" />

  17. kita coba di akun tamu/guest dengan nama tono, di sini bisa melihat databasesnya dan isi tabelnya
    <img width="937" height="857" alt="image" src="https://github.com/user-attachments/assets/40a1b111-cc55-4d28-b7e4-2ea5d8dc96b0" />

  18.ketika tono/guest mau menambahkan data ke tabel transaction dengan komen yang sama seperti admin. 
     akan tetapi erorr karena tidak ada akses buat tono/guest untuk INSERT (menambahkan), UPDATE (edit) dan DELETE (menghapus) hanya bisa bisa mengakses untuk melihat saja (SELECT)
    <img width="979" height="452" alt="image" src="https://github.com/user-attachments/assets/ef3308e1-8fc4-49f6-a786-c9e9b9b4c039" />

  

    

