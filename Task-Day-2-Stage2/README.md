Instalasi docker to ubuntu server aws
1. pertama lakukan update dan upgarade server
   
   <img width="955" height="463" alt="image" src="https://github.com/user-attachments/assets/d78be2ea-361b-49d4-a39f-5ee55ebb5453" />
   
2. ketik berapah ubuntu version
   
   <img width="950" height="309" alt="image" src="https://github.com/user-attachments/assets/14cc2601-a625-479e-9065-1a5c09479924" />

3. Jalankan perintah berikut untuk menghapus (uninstall) semua paket yang bentrok/berkonflik

   <img width="950" height="251" alt="image" src="https://github.com/user-attachments/assets/183fa25c-d1f7-4174-bed3-d1f4dc177c8b" />

4. Set up Docker's apt repository

   <img width="960" height="892" alt="image" src="https://github.com/user-attachments/assets/657e146f-b913-40a3-ab15-965968841974" />

6. instal Docker packages latest vesion

   <img width="954" height="908" alt="image" src="https://github.com/user-attachments/assets/e3fb16bd-f6ee-4ad3-b1a1-c2fa0d294399" />

7. lihat apakah docker sudah berjalan

   <img width="956" height="503" alt="image" src="https://github.com/user-attachments/assets/895bc6bc-f4ef-40e4-ba4f-55143fa6a9af" />

8. lihat version docker kita
   
   <img width="957" height="969" alt="image" src="https://github.com/user-attachments/assets/6e930194-e47d-4cf0-926b-516ca1fad078" />
   
9. LOGIN DOCKER masuk ke terminal lalu ketik "docker login"

    <img width="954" height="651" alt="image" src="https://github.com/user-attachments/assets/730fbb99-c908-4b45-afcb-31c43d24093e" />

10. berikan akses docker ke user adi

    <img width="942" height="254" alt="image" src="https://github.com/user-attachments/assets/3fbc208d-1c10-4221-8b46-7da0e51fc005" />

11. lalu kita uplode ke docker hub dengan cara  docker push contoh username/riopstory:tag = adiwijayajy/dumways:prod di sini ada tag yang menginisialkan tanda buid images ini ketika sudah di push akan ada dua id images yang sama

    <img width="1693" height="288" alt="image" src="https://github.com/user-attachments/assets/81c85d51-8aa3-46fa-8d56-9ab56248dede" />
    <img width="1307" height="370" alt="image" src="https://github.com/user-attachments/assets/b0ebfc5f-72ab-4031-b955-6968cbcb9431" />

 12. lalu kita uplode ke docker hub dengan cara  docker push contoh username/riopstory:tag = adiwijayajy/dumways:prod di sini ada tag yang menginisialkan tanda buid images ini. ketika sudah di push akan ada dua id images yang sama

     <img width="894" height="107" alt="image" src="https://github.com/user-attachments/assets/a7bdf335-4baa-4ac6-b14c-6e17403df57e" />

 13. dan di docker hub pun sudah teruplode

     <img width="949" height="966" alt="image" src="https://github.com/user-attachments/assets/f7fed56c-c6ac-4d5d-8847-4252f45c2536" />

 14. ikuti langkah langkah intalasi  dari https://certbot.eff.org/instructions?ws=nginx&os=snap&tab=wildcard. kita disini menggunakan cloudflare, lalu generet token api di cloudflare

     <img width="1913" height="1041" alt="image" src="https://github.com/user-attachments/assets/56f125bc-4384-426f-98e7-d87f5e15c76b" />
 
 15. setelah step ini selesai kita generate setificate

     <img width="1479" height="673" alt="image" src="https://github.com/user-attachments/assets/74cd9d15-de59-4389-bb9e-bc66d6478b8e" />

 16. setelah itu buat docker-compose.yml, di volumes nginx itu yang akan mengarah ke config domain dan sertifikat certbot. sedangkan di cartbot sama ada tambahan config ipa cloudflare kita

     <img width="957" height="725" alt="image" src="https://github.com/user-attachments/assets/8cee4c31-caf1-4cfa-800a-6ee7500d67f3" />

 17. setelah kita instal docker di server kita tahap selanjutnya kita clone project lalu buat file baru bernama Dockerfile. file ini akan membuat / generate project kita jadi images yang berjalan di docker ini contoh Dockerfile

     <img width="702" height="504" alt="image" src="https://github.com/user-attachments/assets/5147ed77-ea6b-484c-b628-00f96e5812bf" />

 18. disini kita bisa memberikan perintah "docker build -t name_image:tag" seperti contoh di gambar

     <img width="952" height="340" alt="image" src="https://github.com/user-attachments/assets/64b0aca3-6a3e-435c-92a8-2944e4f51bc5" />

 19. lalu untuk mengecek images sudah terbuat bisa comment "docker images"

     <img width="953" height="96" alt="image" src="https://github.com/user-attachments/assets/d4c89f34-1807-4976-9007-b92cc8eda076" />

 20. setelah itu membuat docker-compose.yaml, fungsi "build: ." untuk membuat container, "images" nama images yang telah dibuat tadi lalu masukan, "container_name" nama container yang mau di buat, "restart: always" ketika container ini di matikan container ini akan terhapus dan me restart, "ports" kenapah di sini ada 2 port yang sama karena yang pertama sebagai opsi ketika port yang utama atau kedua sudah terpakai/ error

     <img width="957" height="312" alt="image" src="https://github.com/user-attachments/assets/0531ec4d-30c6-4cb7-895a-04e03cb62e00" />

 21. back end dan database, setelah git clone langkah selanjutnya merubah config.json backend di sini dirubah  database dan user password

     <img width="959" height="590" alt="image" src="https://github.com/user-attachments/assets/0a94324b-737c-496b-833c-2fbefea4f85f" />

 22. lalu edit .env yang isinya semua config privat database

     <img width="726" height="169" alt="image" src="https://github.com/user-attachments/assets/18dd785e-1ec1-42e2-9bfd-9b5da32b670a" />

 23. di bakend menggunakan node 1, dan pm2 ecosyistem.config.js dan untuk config nya sama seperti frontend

     <img width="735" height="343" alt="image" src="https://github.com/user-attachments/assets/8126a552-b039-41d2-b170-6429cc5bcfda" />
     <img width="699" height="227" alt="image" src="https://github.com/user-attachments/assets/69608ec6-d57b-4128-b84d-107c02f548f1" />

 24. lalu untuk docker-compose.yaml
     
     <img width="951" height="1036" alt="image" src="https://github.com/user-attachments/assets/1ae8f35f-1805-472a-96e4-633ae777c8b5" />
