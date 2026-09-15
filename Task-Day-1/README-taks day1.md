# devops26-dumbways-Adi-Wijaya
task day 1

1. pastikan sudah menginstall virtualbox lalu masukan file iso ubuntu versi 22.4.5


<img width="964" height="753" alt="image" src="https://github.com/user-attachments/assets/12df5de1-04bf-42b4-a20a-14d91a431abc" />


2. lalu ada konfigurasi username dan password


<img width="967" height="747" alt="image" src="https://github.com/user-attachments/assets/03ad95c7-5f63-46ca-bc75-04331fa16957" />


3. sesuaikan kapasitas processor, memory, dan disk


 <img width="970" height="756" alt="Screenshot 2026-08-13 231848" src="https://github.com/user-attachments/assets/7f854435-551b-4ef6-88e4-3d5867907434" />


4. pilih bahasa sesuai dengan kebutuhan saya menggunakan bahasa english


 <img width="1184" height="885" alt="Screenshot 2026-08-13 234341" src="https://github.com/user-attachments/assets/8da9136d-3ebe-41fd-be97-7027c7fff03e" />


5. ada pilihan install versi terbaru ubuntunya kita skip aja

   
 <img width="1109" height="892" alt="image" src="https://github.com/user-attachments/assets/b4539a31-0ab6-4ae2-a3b5-d54f9498f671" />


6. untuk default kayboardnya seerti ini kita done kan aja


 <img width="1150" height="896" alt="Screenshot 2026-08-14 003326" src="https://github.com/user-attachments/assets/9d3fcf0a-84b4-4f19-ad9d-88693492411b" />


7. kita pilih yang ubuntu server 


 <img width="1168" height="893" alt="image" src="https://github.com/user-attachments/assets/c2d6f278-2698-4b09-89fc-08250b4550f8" />


8. lalu kita konfigurasi internet ipv4 dengan internet wifi kita


 <img width="1115" height="880" alt="image" src="https://github.com/user-attachments/assets/eb998598-668c-4716-b5ca-044b64005fec" />


9. lalu kita maksimalkan penyimanan server kita dengan tambahkan partition penyimpanan


 <img width="1075" height="827" alt="image" src="https://github.com/user-attachments/assets/5437da34-b446-4814-8602-33abfd39e154" />


10. lalu kita continue setelah itu kita isikan data server kita


<img width="1178" height="853" alt="image" src="https://github.com/user-attachments/assets/01e50d0b-b3b6-4503-93c8-0ef18d963160" />


11. setelah itu kita langsung skip aja untuk versi pro nya kita menggunakan ubuntu versi biasa


<img width="1111" height="882" alt="image" src="https://github.com/user-attachments/assets/9c7a8478-02d7-42ac-861a-a36df1c2c68b" />


12. kita juga skip buat install openSSH nya dan featured server kita langsung ilih done 


<img width="1172" height="878" alt="image" src="https://github.com/user-attachments/assets/324dc26e-7e26-4d70-a0c1-50755e9fb87b" />

<img width="1191" height="871" alt="image" src="https://github.com/user-attachments/assets/5dfecf49-e22f-4c08-b233-b6316184f419" />


13. tunggu proses installation dan setelah itu reboot now


<img width="1205" height="871" alt="image" src="https://github.com/user-attachments/assets/6086a98a-ff3a-4250-961c-0e70a22af633" />


15. di sini saya ada kesalahan di konfigurasinya jadi saya konfigurasi ulang dengan cara ketik di server saya "cd etc/netplan" lalu saya ketikan komen lagi "sudo nano /etc/netplan/50-cloud-init.yaml" setelah itu saya update konfigurasi


<img width="823" height="861" alt="image" src="https://github.com/user-attachments/assets/948faf51-3ddf-4112-8acd-9ec64338e003" />


16. setelah selesai konfigurasi tidak lupa untuk di coment kembali seperti ini "sudo netlan --debug apply", lalu saya jalankan test ping, dan selesai sudah terinstal server ubuntu dan sudah terkoneksi jaringan internet


<img width="755" height="611" alt="image" src="https://github.com/user-attachments/assets/2c1b0f0a-f7ab-4604-b471-084860493204" />












