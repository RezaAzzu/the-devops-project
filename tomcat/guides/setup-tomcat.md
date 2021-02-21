# Rangkuman Setup Tomcat dan Deploy Webapp ke Tomcat Menggunakan Jenkins
Panduan ini berisi tutorial ringkas mengenai cara me-setup server Tomcat setelah [me-setup server Jenkins untuk pertama kalinya](https://razzubair.medium.com/membangun-server-jenkins-dan-meng-install-plugin-maven-pada-jenkins-912a0aecb329).
Dikarenakan tutorial ini ringkas, tutorial hanya memberikan step-by-step yang singkat dan padat saja.
Panduan ini berisi tentang:
1. Menambahkan rule baru pada security group Jenkins_Server.
2. Cara me-setup Tomcat pada Jenkins_Server. Jadi, Tomcat di-install pada server yang sama dengan Jenkins.
3. Konfigurasi Jenkins agar dapat terintegrasi dengan Tomcat.
4. Konfigurasi job Jenkins untuk men-deploy file .WAR (web app) ke Tomcat.

Sebelum memulai, pastikan:
1. Paham cara menambahkan rule baru pada security group dari instance EC2 AWS.
2. Jenkins_Server sudah ada (sudah mengikuti [tutorial](https://razzubair.medium.com/membangun-server-jenkins-dan-meng-install-plugin-maven-pada-jenkins-912a0aecb329) tentang setup Jenkins pada Jenkins_Server).
3. Jenkins sudah ter-install pada Jenkins_Server (sudah mengikuti [tutorial](https://razzubair.medium.com/membangun-server-jenkins-dan-meng-install-plugin-maven-pada-jenkins-912a0aecb329) tentang setup Jenkins pada Jenkins_Server).
4. Sudah meng-clone dan/atau mem-pull repo java-maven-helloworld (https://github.com/RezaAzzu/java-maven-helloworld) ke repo online pribadi Anda.
5. Paham mengoperasikan Linux secara dasar.

## 1. Menambahkan Port pada Security Group Jenkins_Server
1. Tambahkan rule baru pada security group Jenkins_Server dengan ketentuan sebagai berikut
    > Type: Anywhere

    > Port range: 8090

    > Source: Anywhere

## 2. Me-Setup Tomcat pada Jenkins_Server
### Meng-install Tomcat
1. Buka [tomcat.apache.org](http://tomcat.apache.org/download-80.cgi). Salin link dari Tomcat versi terbaru yang berformat ".tar.gz" di bawah "Binary Distributions" dan "Core"
2. Akses Jenkins_Server via SSH
3. Masuk ke root
```sudo su -```
4. Masuk ke directory `/opt` (dengan perintah `cd /opt`). Lalu, jalankan perintah-perintah berikut.
    ```
    cd /opt/
    wget [link-tomcat] #[link-tomcat] adalah link yang telah di-copy
    tar -xvzf [file-yang-terdownload]
    mv [folder-hasil-decompress] tomcat
    cd tomcat/
    ```
5. Buka `conf/server.xml`. Lalu, rubah port pada '<Connector port=...' menjadi '8090' (nilai port yang sama dengan rule yang ditambahkan pada security group Jenkins_Server)
    ```
    ...
    <Connector port="8090" ... >
    ...
    ```
    Jangan lupa untuk meng-save file tersebut.
6. Nyalakan service tomcat dengan menjalankan perintah
    ```
    cd /opt/tomcat/bin
    ./startup.sh
    ```
7. Buka alamat IP publik Jenkins_Server pada port 8090 via browser. Cek apakah Tomcat sudah dapat diakses.

## 3. Meng-Update User Info
1. Buka context.xml. Lalu, beri komen baris yang ada 'Valve className=...'-nya. Dari
    ```
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
    ```
    menjadi
    ```
    <!--<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />-->
    ```
2. Buka file `conf/tomcat-user.xml`. Lalu, di bawah tag 
`<tomcat-users ... version="1.0">` masukkan kode berikut
    ```
    <role rolename="manager-gui"/>
    <role rolename="manager-script"/>
    <role rolename="manager-jmx"/>
    <role rolename="manager-status"/>
    <user username="!admin!" password="!admin!" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
    <user username="udeploy" password="udeploy" roles="manager-script"/>
    <user username="tomcat" password="r4has15" roles="manager-gui"/>
    ```
	Kode ini berfungsi untuk memberi role dan user yang dapat dikenali Tomcat.
3. Restart Tomcat dengan menjalankan `shutdown.sh` lalu `startup.sh`. 
Catatan: Tomcat butuh waktu paling lama 5 menit sebelum dapat diakses via browser, jadi harus bersabar.

4. Akses Tomcat via browser. Klik "Manager App". Lalu, masukkan username `tomcat` dan password `r4has15`. 
Catatan: username dan password ini sama dengan salah satu user yang ditambahkan pada langkah nomor 2.
Bila berhasil, proses penambahan user ke Tomcat sudah selesai. Sekarang, lanjut ke Jenkins.

## 4. Deploy Webapp dari Jenkins ke Tomcat
1. Buka Jenkins via browser, dan login. 
Catatan: gunakan credentials (username dan password) yang telah dibuat pada tutorial sebelumnya, yakni username `admin` dan password `jenkins2021`.
2. Klik Manage Jenkins > Manage Plugins > Available.
3. Di search bar, ketik `Deploy to Container`.
4. Pilih `Deploy to Container`. Klik `Install without restart`. Tunggu hingga selesai.
5. Buat job baru. Kembali ke home (klik logo Jenkins di kiri atas halaman), lalu klik `New item`.
6. Masukkan nama `Coba_Deploy_ke_Tomcat`.
7. Klik `Maven Project`. Klik `OK`.
8. Di bawah `Source Code Management`, klik `Git`.
9. Clone dan/atau pull repo java-maven-helloworld (https://github.com/RezaAzzu/java-maven-helloworld) ke repo pribadi Anda. Lalu, masukkan URL repo pribadi Anda ke `Repository URL`. 
Catatan: Jadi, URL yang dimasukkan merupakan URL repo pribadi Anda. Clone dan/atau pull repo ini dilakukan di luar Jenkins. 
Pastikan repo Anda dapat diakses secara publik (tidak privat) dan online (dapat diakses via Internet). Misal, taruh repo tadi di GitHub, GitLab, Bitbucket, dan sebagainya. 
10. Di bawah `Build`, sebelah `Goals and options`, ketikkan `clean install package`.
11. Klik `Add-post build action`, klik `Deploy WAR/EAR to a container`
12. Di sebelah `WAR/EAR files`, masukkan `**/*.war`
13. Di sebelah `Containers`, klik `Add Container`. Lalu, pilih `Tomcat 8.x Remote`
14. Di sebelah `Credentials`, klik `Add`. Masukkan username `udeploy` dan password `udeploy` (sesuai yang sudah ditambahkan di `conf/tomcat-user.xml`). Masukkan ID `deployer_user`.
15. Klik `Add`.
16. Pilih credential yang barusan diisi (`udeploy/***** ... `).
17. Di sebelah `Tomcat URL`, masukkan alamat dari Tomcat (alamat IP publik dan port dari Tomcat). Misal: http://52.77.239.100:8090.
Catatan: karena pada panduan ini Tomcat dan Jenkins terdapat dalam satu server yakni Jenkins_Server, maka alamat Tomcat dan Jenkins_Server pasti sama. Namun, yang berbeda adalah di port-nya.
18. Klik `Apply`, lalu `Save`.
19. Klik `Build Now`. Tunggu hingga build selesai.

## 5. Membuka Webapp Tomcat
1. Di Jenkins_Server (masuk via SSH), di directory `/opt/tomcat/webapps`, akan ada directory dan file baru, yakni `webapp/` dan `webapp.war`. Artinya, web application telah ter-deploy ke Tomcat.
2. Sekarang, buka alamat Tomcat ditambah dengan `/webapp` di belakang alamatnya. Contoh: http://52.77.239.100:8090/webapp
3. Webapp akan terbuka.

Catatan: Setelah Webapp berhasil di-deploy tanpa error, bila deploy/panduan ini diulangi atau dicoba sekali lagi, akan menghasilkan error. Untuk itu, jangan ikuti panduan ini lebih dari satu kali setelah error. Error ini dikarenakan Webapp yang sudah di-deploy masih ada dalam Jenkins_Server dan harus dihapus sebelum dapat menerima versi Webapp baru.  
Panduan-panduan berikutnya akan mengatasi masalah ini.
