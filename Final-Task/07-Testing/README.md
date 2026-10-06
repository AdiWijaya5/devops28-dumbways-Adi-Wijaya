
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
