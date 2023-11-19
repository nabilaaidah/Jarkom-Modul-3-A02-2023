# Jarkom-Modul-3-A02-2023

<table>
    <tr>
        <th colspan=2> Kelompok A02 </th>
    </tr>
    <tr>
        <th>NRP</th>
        <th>Nama Anggota</th>
    </tr>
    <tr>
        <td>5025211032</td>
        <td>Nabila A'idah Diani</td>
    </tr>
</table>

Link Grimoire: https://its.id/m/A02_Grimoire_Jarkom3_2023

Link GNS3: https://its.id/m/A02_GNS3_Jarkom3_2023

## Topologi

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/47142ec0-68d7-48fa-aa5d-436f0600a92e)


## Pengaturan Config

Dikarenakan node yang akan digunakan mempunyai aturan seperti berikut:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/e1af9b57-60d5-4a29-b4f4-82efb98cc8cf)

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/bb079e98-9547-4822-a276-29e37e92bc3b)

Maka, pengaturan config setiap node adalah seperti berikut:

```
AURA
auto eth0
iface eth0 inet dhcp
up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.0.0.0/16

auto eth1
iface eth1 inet static
	address 10.0.1.9
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.0.2.9
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 10.0.3.9
	netmask 255.255.255.0

auto eth4
iface eth4 inet static
	address 10.0.4.9
	netmask 255.255.255.0


HIMMEL
auto eth0
iface eth0 inet static
	address 10.0.1.1
	netmask 255.255.255.0
	gateway 10.0.1.9


HEITER
auto eth0
iface eth0 inet static
	address 10.0.1.2
	netmask 255.255.255.0
	gateway 10.0.1.9


DENKEN
auto eth0
iface eth0 inet static
	address 10.0.2.2
	netmask 255.255.255.0
	gateway 10.0.2.9


EISEN // LB
auto eth0
iface eth0 inet static
	address 10.0.2.1
	netmask 255.255.255.0
	gateway 10.0.2.9


Configuration Node Lain
auto eth0
iface eth0 inet dhcp
```

Dikarenakan terdaapat petulisan bahwa worker memiliki IP yang static, maka semua worker diberikan fixed address
```
echo ‘
host Lawine{
	hardware ethernet 7e:58:d8:55:f1:8e;
	fixed-address 10.0.3.1;
}
host Linie{
	hardware ethernet 6a:a0:56:8d:35:a8;
	fixed-address 10.0.3.2;
}
host Lugner{
	hardware ethernet b2:8e:86:ed:a6:94;
	fixed-address 10.0.3.3;
}
host Frieren{
	hardware ethernet 6e:16:c8:77:a5:6c;
	fixed-address 10.0.4.1;
}
host Flamme{
	hardware ethernet 5a:90:d0:d2:35:d5;
	fixed-address 10.0.4.2;
}
host Fern{
	hardware ethernet 56:17:50:e0:92:fc;
	fixed-address 10.0.4.3;
}’ >> /etc/dhcp/dhcpd.conf
```

## Soal 0
> Setelah mengalahkan Demon King, perjalanan berlanjut. Kali ini, kalian diminta untuk melakukan register domain berupa riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP (0) mengarah pada worker yang memiliki IP [prefix IP].x.1.

#### Jawaban
Pada soal ini, diperintahkan membuat domain riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP, sehingga yang dilakukan adalah melakukan instalasi dependencies pada .bashrc dan menginisiasi domain dan mengarahkan pada node yang diinginkan.

1. Instalasi dependencies yang dibutuhkan
```
apt update
apt install bind9 -y
```

2. Mendefinisikan domain
```
echo ‘
zone “canyon.a02.com” {
	type master;
	file “/etc/bind/jarkom/canyon.a02.com”;
};

zone “channel.a02.com”{
	type master;
	file “/etc/bind/jarkom/channel.a02.com”;
};’  > /etc/bind/named.conf.local
```

3. Membentuk directory
```
mkdir -p /etc/bind/jarkom
```

4. Mendefinisikan IP ke dalam domain
```
echo ‘
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA    canyon.a02.com. root.canyon.a02.com. (
                              2         ; Serial
                         604800    ; Refresh
                          86400     ; Retry
                        2419200   ; Expire
                         604800 )    ; Negative Cache TTL
;
@       IN      NS     canyon.a02.com.
@       IN      A       10.0.2.3; IP Load Balancer
riegel  IN      A       10.0.4.1’ > /etc/bind/jarkom/canyon.a02.com
echo ‘
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA    channel.a02.com. root.channel.a02.com. (
                              2         ; Serial
                         604800    ; Refresh
                          86400     ; Retry
                        2419200   ; Expire
                         604800 )    ; Negative Cache TTL
;
@       IN      NS     channel.a02.com.
@       IN      A       10.0.2.3; IP Load Balancer
granz  IN      A       10.0.3.1’ > /etc/bind/jarkom/channel.a02.com
```

5. Konfigurasi DNS menggunakan BIND9
```
echo 'options {
        directory "/var/cache/bind";

        forwarders {
                192.168.122.1;
        };

        // dnssec-validation auto;
        allow-query{any;};
        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
}; ' >/etc/bind/named.conf.options
```

6. Melakukan BIND9 start
```
service bind9 start
```

## Soal 1
> Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.

#### Jawaban
Konfigurasi dilakukan dengan membentuk topologi seperti yang dicontohkan dan mengatur config per nodenya. Jawaban dari nomor ini telah ditulis pada awal lapres.


## Soal 2 & 3 & 4 & 5
> Kemudian, karena masih banyak spell yang harus dikumpulkan, bantulah para petualang untuk memenuhi kriteria berikut:
> 
>  a. Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.
>
>  b. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80
>
>  c. Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168 (3)
>
>  d. Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut (4)
>
>  e. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit (5)

#### Jawaban
a. Semua client menggunakan konfigurasi dari DHCP Server dengan cara konfigurasi per nodenya didefinisikan dengan berikut:
```
auto eth0
iface eth0 inet dhcp
```

b. Client yang terhubung dengan switch 3 akan memiliki range IP dari yang telah ditentukan pada soal, sehingga diperlukan subnet untuk mengatur konfigurasi ini:
```
subnet 10.0.3.0 netmask 255.255.255.0 {
	range 10.0.3.16 10.0.3.32;
	range 10.0.3.64 10.0.3.80;
	option routers 10.0.3.9;
	option broadcast-address 10.0.3.255;
	option domain-name-servers 10.0.1.2;
	default-lease-time 180;
	max-lease-time 5760;	
}
```
Keterangan: Dikarenakan terdapat dua range IP yang diminta, sehingga range ditulis dua kali.

c. Client yang terhubung dengan switch 4 akan memiliki range IP dari yang telah ditentukan pada soal, sehingga diperlukan subnet untuk mengatur konfigurasi ini:
```
subnet 10.0.4.0 netmask 255.255.255.0 {
	range 10.0.4.12 10.0.4.20;
	range 10.0.4.160 10.0.4.168;
	option routers 10.0.4.9;
	option broadcast-address 10.0.4.255;
	option domain-name-servers 10.0.1.2;
	default-lease-time 720;
	max-lease-time 5760;
}
```
Keterangan: Dikarenakan terdapat dua range IP yang diminta, sehingga range ditulis dua kali. Subnet-subnet ini ditulis ke dalam file `/etc/dhcp/dhcpd.conf`

d. Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut
```
option domain-name-servers 10.0.1.2;
```
Keterangan: dikarenakan client dhcp akan terhubung dengan internet, maka perlu didefinisikan `option domain-name-servers` dengan IP Heiter (DNS Server)

e. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit

Untuk client pada switch 3, mereka memiliki waktu selama 3 menit sebagai `default-lease-time`, sehingga akan didefinisikan sebagai berikut:
```
default-lease-time 180;
```
Dan pada switch 4, mereka memiliki waktu selama 12 menit sebagai `default-lease-time`, sehingga akan didefinisikan sebagai berikut:
```
default-lease-time 720;
```
Pada kedua switch, mereka sama-sama memiliki waktu selama 96 menit untuk waktu maksimal alokasi, sehingga akan didefinisikan sebagai berikut:
```
max-lease-time 5760;
```

Keterangan: semua waktu dideifinisikan menggunakan detik

Untuk melihat apakah ip dhcp terimplementasikan pada client, maka perlu dilakukan konfigurasi pada dhcp relay:
```
apt update
apt install isc-dhcp-relay -y
```
Dengan konfigurasi tambahan sebagai berikut:
```
SERVERS="10.0.1.1" # IP Dhcp sever
INTERFACES="eth1 eth2 eth3 eth4”
```
Dan melakukan enable ip4 forwarding pada `/etc/sysctl.conf`
```
net.ipv4.ip_forward=1
```
Setelah itu melakukan start service dengan command berikut:
```
service isc-dhcp-relay start
```

Hasil:

1. IP dhcp pada client dengan switch 3
   
![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/5560ce74-4d7a-47be-b45d-8beb2687eefa)

2. IP dhcp pada client dengan switch 4
   
![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/74ee465b-aa89-44ce-af6b-76fea822095b)



## Soal 6
> Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3.

#### Jawaban
1. Melakukan instalasi dependencies yang dibutuhkan pada PHP worker
```
apt update
apt install nginx -y
service nginx start
apt-get install php php-fpm -y
```

2. Melakukan instalasi dependencies yang dibutuhkan pada client
```
apt update
apt install lynx -y
apt install htop -y
apt install apache2-utils -y
apt install nginx -y
```

3. Pada PHP worker, buat file di dalam root yang akan menginstalasi hal yang dibutuhkan sesuai yang telah didefinisikan dalam soal. Lalu, pindahkan ke file directory yang diinginkan
```
git clone -b granz https://github.com/nabilaaidah/JarkomPrak3-Dependencies
mv JarkomPrak3-Dependencies /var/www/html/granz.channel.a02.com
```

4. Lakukan sedikit editorial pada file `/etc/nginx/sites-enabled/default` yang ada pada PHP worker

a. Jadikan root `/var/www/html` menjadi `/var/www/html/granz.channel.a02.com`

b. Tambahkan `index.php` dalam index

c. Ganti `server-name _` menjadi `server-name granz.channel.a02.com`

d. Lakukan uncomment pada beberapa code:
```
location ~ \.php$ {
    include snippets/fastcgi-php.conf;
#
#   # With php-fpm (or other unix sockets):
    fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
#   # With php-cgi (or other tcp sockets):
#   fastcgi_pass 127.0.0.1:9000;
}
```
e. Lakukan restart nginx dan start php
```
service nginx restart
service php7.3-fpm start
```

5. Jalankan command berikut pada client untuk melihat hasilnya
```
lynx granz.channel.a02.com
```

Hasil:
![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/765eb2ac-f5a0-49ca-a9cb-9baa05184956)


## Soal 7
> Kepala suku dari Bredt Region memberikan resource server sebagai berikut:
> 
>  a. Lawine, 4GB, 2vCPU, dan 80 GB SSD.
>
>  b. Linie, 2GB, 2vCPU, dan 50 GB SSD.
>
>  c. Lugner 1GB, 1vCPU, dan 25 GB SSD.
>
>  aturlah agar Eisen dapat bekerja dengan maksimal, lalu lakukan testing dengan 1000 request dan 100 request/second

#### Jawaban
Dikarenakan pada soal ini, terdapat informasi mengenai memori ataupun cpu pada setiap workernya. Maka, algoritma yang baik dalam melakukan request testing adalah round robin weighted, dengan setiap nodenya akan memiliki weight yang berbeda tergantung dengan informasi perfomance featurenya. Dikarenakan untuk menjalankan algoritma round robin memerlukan load balancer, maka diperlukan instalasi dependencies pada load balancer
```
apt update
apt install nginx -y
service nginx start
```

Setelah itu, membuat algoritma round robin weighted dalam load balancer
```
echo ‘upstream weighted {
	server 10.0.3.1 weight=4;
	server 10.0.3.2 weight=2;
	server 10.0.3.3 weight=1;
}

server {
	listen 80;
	server_name granz.channel.A02.com;
		location / {
                proxy_pass http://weighted;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
        }
	error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}’ > /etc/nginx/sites-available/lb-jarkom
```
Keterangan: pada algoritma tersebut dapat dilihat bahwa pada setiap ip workernya, terdapat penulisan weight masing-masing.

Dikarenakan algo tersebut disimpan dalam file `/etc/nginx/sites-available/lb-jarkom`, maka diperlukan symlink dengan file `/etc/nginx/sites-enabled/default`. Command yang diperlukan adalah sebagai berikut:
```
unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled/default
service nginx restart
```

Untuk melakukan request testing, kita dapat berpindah ke node client dan menjalankan command apache benchmark ke load balancer:
```
ab -n 1000 -c 100 http://10.0.2.3/
```

Setelah itu, kita dapat melihat hasil dari testing request yang dikerjakan oleh setiap node worker dengan menjalankan command berikut:
```
cat /var/log/nginx/access.log | grep “GET” | wc -l
```

Berikut merupakan hasil dari jumlah request yang diterima oleh setiap worker:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/51092999-00c5-424a-9c1e-e0f5ffb5fc01)


## Soal 8
> Karena diminta untuk menuliskan grimoire, buatlah analisis hasil testing dengan 200 request dan 10 request/second masing-masing algoritma Load Balancer dengan ketentuan sebagai berikut:
>
> a. Nama Algoritma Load Balancer
>
> b. Report hasil testing pada Apache Benchmark
>
> c. Grafik request per second untuk masing masing algoritma.
>
> d. Analisis 

#### Jawaban
a. Round Robin
```
upstream round{
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```

b. Weighted Round Robin
```
upstream round{
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```

c. Least Connection
```
upstream least{
	least_conn;
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```

d. IP Hash
```
upstream least{
	ip_hash;
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```

e. Generic Hash
```
upstream least{
	hash $request_uri consistent;
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```
Untuk lanjutan code di setiap fiturnya adalah sebagai berikut:
```
server {
	listen 80;
	server_name granz.channel.A02.com;
		location / {
                proxy_pass http://robin;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
        }
	error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}
```

Dikarenakan screenshot benchmark dan htop terlalu banyak, maka dapat diakses pada link ini https://its.id/m/A02_Grimoire_Jarkom3_2023

Berikut merupakan grafik perbandingan dari 5 algoritma tersebut:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/aa135d26-a988-4125-896f-d29063619dc6)



## Soal 9
> Dengan menggunakan algoritma Round Robin, lakukan testing dengan menggunakan 3 worker, 2 worker, dan 1 worker sebanyak 100 request dengan 10 request/second, kemudian tambahkan grafiknya pada grimoire.

#### Jawaban
Berikut merupakan code algoritma round robin:
a. 3 Worker
```
upstream robin {
	server 10.0.3.1;
	server 10.0.3.2;
	server 10.0.3.3;
}
```

b. 2 Worker
```
upstream robin {
	server 10.0.3.1;
	server 10.0.3.2;
}
```

c. 1 worker
```
upstream robin {
	server 10.0.3.1;
}
```

Untuk lanjutan code di setiap fiturnya adalah sebagai berikut:
```
server {
	listen 80;
	server_name granz.channel.A02.com;
		location / {
                proxy_pass http://robin;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
        }
	error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}
```

Dikarenakan algo tersebut disimpan dalam file, maka diperlukan symlink dengan file `/etc/nginx/sites-enabled/default` dan melakukan service nginx. Command yang diperlukan adalah sebagai berikut:
```
unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/[nama-file] /etc/nginx/sites-enabled/default
service nginx restart
```

Dikarenakan screenshot benchmark dan htop terlalu banyak, maka dapat diakses pada link ini https://its.id/m/A02_Grimoire_Jarkom3_2023

Berikut merupakan grafik perbandingan dari semua code dengan jumlah workernya masing-masing:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/0baca76f-e713-4f8f-93b4-2948c4765bcf)


## Soal 10
> Selanjutnya coba tambahkan konfigurasi autentikasi di LB dengan dengan kombinasi username: “netics” dan password: “ajkyyy”, dengan yyy merupakan kode kelompok. Terakhir simpan file “htpasswd” nya di /etc/nginx/rahasisakita/

#### Jawaban
Untuk menambahkan konfigurasi autentikasi, maka perlu dijalankan command berikut dalam load balancer
```
mkdir /etc/nginx/rahasiakita
htpasswd -c /etc/nginx/rahasiakita/.htpasswd netics
Pass: ajka02
```

Pada soal ini, digunakan algoritma round robin weighted dengan command berikut:
```
echo 'upstream weighted {
    server 10.0.3.1 weight=4;
    server 10.0.3.2 weight=2;
    server 10.0.3.3 weight=1;
}

server {
    listen 80;
    server_name granz.channel.A02.com;

    location / {
        proxy_pass http://weighted;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;

        auth_basic "Administrator'\''s Area";
        auth_basic_user_file /etc/nginx/etc/nginx/rahasiakita/.htpasswd;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/lb_error.log;
    access_log /var/log/nginx/lb_access.log;
}' > /etc/nginx/sites-available/lb-jarkom

unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled/default
service nginx restart
```

Pada node client, dijalankan command berikut untuk mengirimkan request dengan autentikasi
```
ab -A netics:ajka02 -n 100 -c 100 http://granz.channel.a02.com/
```

Hasilnya adalah sebagai berikut:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/d5aac31d-74d4-405d-90dd-4dc46786d87d)


## Soal 11
> Lalu buat untuk setiap request yang mengandung /its akan di proxy passing menuju halaman https://www.its.ac.id.

#### Jawaban
Tambahkan code berikut pada soal no. 10:
```
location /its {
        proxy_pass https://www.its.ac.id;
    
	auth_basic "Administrator'\''s Area";
        auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
}
```
Keterangan: Jika IP LB diakses dengan tambahan endpoint `/its`, maka akan menghubungkan ke `https://www.its.ac.id` dengan autentikasi.

Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/12e1a6d5-20b0-4e21-ad75-60be46f87183)

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/7e8f3db0-6bc3-4e64-a600-41d4c2c9f90e)

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/df2d2f16-ef92-4e7d-adef-c8d87a2cc467)


## Soal 12
> Selanjutnya LB ini hanya boleh diakses oleh client dengan IP [Prefix IP].3.69, [Prefix IP].3.70, [Prefix IP].4.167, dan [Prefix IP].4.168.

#### Jawaban
Code berikut digunakan untuk memberikan batasan dengan IP apa saja client dapat mengakses
```
 location / {
allow 10.0.3.69;
        allow 10.0.3.70;
        allow 10.0.4.167;
        allow 10.0.4.168;
	deny all;

        proxy_pass http://weighted;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;

        auth_basic "Administrator'\''s Area";
        auth_basic_user_file /etc/nginx/rahasiakita/.htpasswd;
    }
```

Hasil jika diakses oleh IP yang tidak didefinisikan dalam soal:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/b0767954-25fd-4135-a27a-413ac63dc6bf)

## Soal 13
> Semua data yang diperlukan, diatur pada Denken dan harus dapat diakses oleh Frieren, Flamme, dan Fern

#### Jawaban
##### Pada Denken
Install dependensi dan atur database pada node database dengan command berikut:
```
apt-get install mariadb-server -y
service mysql start
echo ‘
[mysqld]
skip-networking=0
skip-bind-address
‘ > /etc/mysql/my.cnf

mysql -u root -p

CREATE USER 'kelompoka02'@'10.0.4.1' IDENTIFIED BY 'passworda02';
CREATE USER 'kelompoka02'@'10.0.4.2' IDENTIFIED BY 'passworda02';
CREATE USER 'kelompoka02'@'10.0.4.3' IDENTIFIED BY 'passworda02';
CREATE USER 'kelompoka02'@'localhost' IDENTIFIED BY 'passworda02';
CREATE DATABASE dbkelompoka02;

GRANT ALL PRIVILEGES ON *.* TO 'kelompoka02'@'10.0.4.1';
GRANT ALL PRIVILEGES ON *.* TO 'kelompoka02'@'10.0.4.2';
GRANT ALL PRIVILEGES ON *.* TO 'kelompoka02'@'10.0.4.3';
GRANT ALL PRIVILEGES ON *.* TO 'kelompoka02'@'localhost';

FLUSH PRIVILEGES;
```

Lalu, atur `bind address` pada `/etc/mysql/mariadb.conf.d/50-server.cnf` menjadi
```
bind addres = 0.0.0.0
```

##### Pada Worker
Install dependensi pada node worker dengan command sebagai berikut:
```
apt update
apt-get install mariadb-client -y
```

Jalankan command berikut untuk menghubungkan dengan database
```
mariadb --host=10.0.2.2 --port=3306 --user=kelompoka02 --password
```
Keterangan
- IP yang terdapat pada host merupakan IP Denken
- Port 3306 merupakan port untuk Mariadb
- User berisikan nama user yang terdaftar dalam sql

Hasil jika diakses oleh IP yang tidak diberikan akses:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/52a653d7-3040-4e51-8b4c-f26bdf2a381c)

Hasil jika diakses oleh IP yang diberikan akses:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/403acbae-2615-4de6-97f8-2878e8e67ac5)


## Soal 14
> Frieren, Flamme, dan Fern memiliki Riegel Channel sesuai dengan quest guide berikut. Jangan lupa melakukan instalasi PHP8.0 dan Composer

#### Jawaban

Lakukan instalasi dependensi berikut pada semua node worker:
```
apt-get update
apt-get install -y lsb-release ca-certificates apt-transport-https
software-properties-common gnupg2
curl -sSLo /usr/share/keyrings/deb.sury.org-php.gpg https://packages.sury.org/php/apt.gpg
sh -c 'echo "deb [signed-by=/usr/share/keyrings/deb.sury.org-php.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list'
apt-get update
apt-get install php8.0-mbstring php8.0-xml php8.0-cli php8.0-common php8.0-intl php8.0-opcache php8.0-readline php8.0-mysql php8.0-fpm php8.0-curl unzip wget -y
apt-get install nginx -y
wget https://getcomposer.org/download/2.0.13/composer.phar
chmod +x composer.phar
mv composer.phar /usr/bin/composer
apt-get install git -y
git clone https://github.com/martuafernando/laravel-praktikum-jarkom
mv laravel-praktikum-jarkom /var/www/
```

Jalankan command berikut pada file laravel:
```
Composer update
composer install
cp .env.example .env
php artisan migrate:fresh
php artisan db:seed --class=AiringsTableSeeder
php artisan key:generate
php artisan jwt:secret
```

Konfigurasi file .env yang baru:
```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=10.0.2.2
DB_PORT=3306
DB_DATABASE=dbkelompoka02
DB_USERNAME=kelompoka02
DB_PASSWORD=passworda02

BROADCAST_DRIVER=log
CACHE_DRIVER=file
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

MEMCACHED_HOST=127.0.0.1

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=
AWS_USE_PATH_STYLE_ENDPOINT=false

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME=https
PUSHER_APP_CLUSTER=mt1

VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```


## Soal 15 & 16 & 17
> Riegel Channel memiliki beberapa endpoint yang harus ditesting sebanyak 100 request dengan 10 request/second. Tambahkan response dan hasil testing pada grimoire.
>  a. POST /auth/register (15)
>
>  b. POST /auth/login (16)
>
>  c. GET /me (17)

#### Jawaban

##### Worker
Jalankan command berikut pada node worker:
```
echo ‘
server {

    listen 80;

    root /var/www/laravel-praktikum-jarkom/public;

    index index.php index.html index.htm;
    server_name riegel.canyon.a02.com;

    location / {
            try_files $uri $uri/ /index.php?$query_string;
    }

    # pass PHP scripts to FastCGI server
    location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.0-fpm.sock;
    }

location ~ /\.ht {
            deny all;
    }

    error_log /var/log/nginx/implementasi_error.log;
    access_log /var/log/nginx/implementasi_access.log;
}’ > /etc/nginx/sites-available/implementasi
unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/implementasi /etc/nginx/sites-enabled/default
chown -R www-data.www-data /var/www/laravel-praktikum-jarkom/storage
service php8.0-fpm start
```

##### Load Balancer
Jalankan command berikut pada node load balancer:
```
echo ‘upstream laravel {
	server 10.0.4.1;
	server 10.0.4.2;
	server 10.0.4.3;
}

server {
	listen 80;
	server_name riegel.canyon.a02.com;
		location / {
                proxy_pass http://laravel;
                proxy_set_header    X-Real-IP $remote_addr;
                proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header    Host $http_host;
        }
	error_log /var/log/nginx/lb_error.log;
access_log /var/log/nginx/lb_access.log;
}’ > /etc/nginx/sites-available/lb-jarkom

unlink /etc/nginx/sites-enabled/default
ln -s /etc/nginx/sites-available/lb-jarkom /etc/nginx/sites-enabled/default
service nginx restart
```

##### Client
Jalankan command berikut untuk melihat hasilnya:

a. Jalankan dengan curl
```
curl -X POST -d '{"username": "username", "password": "password"}' -H "Content-Type: application/json" http://riegel.canyon.a02.com/api/auth/register
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/f7966d3f-8ddc-410c-ba5e-e7a16a9861a7)


```
curl -X POST -d '{"username": "username", "password": "password"}' -H "Content-Type: application/json" http://riegel.canyon.a02.com/api/auth/login
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/e9d0296f-4056-4c6f-a4e7-c23a30ea1367)


```
curl -H "Authorization: Bearer [token]" http://riegel.canyon.a02.com/api/me
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/532ae131-c816-4058-8855-198ca3a7c17c)


b. Jalankan dengan apache benchmark
```
ab -n 100 -c 10 -p <(curl -X POST -d '{"username": "username", "password": "password"}' -H "Content-Type: application/json" http://riegel.canyon.a02.com/api/auth/register
) -T "application/json" http://riegel.canyon.a02.com/api/auth/register
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/fbb21665-8c4d-47a7-bbf2-f19546782b34)


```
ab -n 100 -c 10 -p <(curl -X POST -d '{"username": "username", "password": "password"}' -H "Content-Type: application/json" http://riegel.canyon.a02.com/api/auth/login
) -T "application/json" http://riegel.canyon.a02.com/api/auth/login
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/7ffa7f32-4b41-49be-832f-9bbba838002e)


```
ab -n 100 -c 10 -p <(curl -H "Authorization: Bearer [token]" http://riegel.canyon.a02.com/api/me
) -T "application/json" http://riegel.canyon.a02.com/api/me
```
Hasil:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/492c6c91-079b-4e86-aff7-813137a7dd7e)


## Soal 18
> Untuk memastikan ketiganya bekerja sama secara adil untuk mengatur Riegel Channel maka implementasikan Proxy Bind pada Eisen untuk mengaitkan IP dari Frieren, Flamme, dan Fern.

#### Jawaban
Pada soal ini, diperintahkan untuk menggunakan Proxy Bind pada Eisen dalam mengaitkan IP Worker Laravel dengan IP Load Balancer, sehingga ditambahkan code ini:
```
 location /frieren {
        proxy_bind 10.0.2.3;	
        proxy_pass http://10.0.4.1/index.php;
 
    }

    location /flamme {
	proxy_bind 10.0.2.3;
        proxy_pass http://10.0.4.2/index.php;
       
    }

    location /fern {
	proxy_bind 10.0.2.3;
        proxy_pass http://10.0.4.3/index.php;
	}
```

Lalu, lakukan `service nginx restart`

Hasil dapat dilihat pada client dengan command `lynx http://[ip lb]/[nama node worker laravel]/`. Berikut merupakan hasilnya:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/e55c8bd5-c040-4b72-bf08-60b00b689745)


## Soal 19
> Untuk meningkatkan performa dari Worker, coba implementasikan PHP-FPM pada Frieren, Flamme, dan Fern. Untuk testing kinerja naikkan
>  - pm.max_children
>  - pm.start_servers
>  - pm.min_spare_servers
>  - pm.max_spare_servers
> sebanyak tiga percobaan dan lakukan testing sebanyak 100 request dengan 10 request/second kemudian berikan hasil analisisnya pada Grimoire.

#### Jawaban
Jalankan command berikut pada node worker laravel:
```
echo ‘
user = eisen_user
group = eisen_user
listen = /var/run/php8.0-fpm-eisen-site.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off

pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.process_idle_timeout = 10s’ > /etc/php/8.0/fpm/pool.d/eisen.conf

groupadd eisen_user
useradd -g eisen_user eisen_user
```

Lalu, ubah konfigurasi socket php-fpm menjadi seperti berikut:
```
location ~ \.php$ {
                include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (or other unix sockets):
                fastcgi_pass unix:/var/run/php8.0-fpm-eisen-site.sock;
        #       # With php-cgi (or other tcp sockets):
        #       fastcgi_pass 127.0.0.1:9000;
}
```

Setelah itu, jalankan command service berikut:
```
service php8.0-fpm start
service nginx restart
```

Dikarenakan terdapat perintah untuk membuat 3 konfigurasi berbeda, maka terdapat 3 hasil untuk nomor ini:

1. Hasil 1
```
pm = dynamic
pm.max_children = 75
pm.start_servers = 10
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.process_idle_timeout = 10s
```

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/0a41bf5b-678d-4e6c-a41b-fa1977185bba)


2. Hasil 2
```
pm = dynamic
pm.max_children = 35
pm.start_servers = 5
pm.min_spare_servers = 1
pm.max_spare_servers = 5
pm.process_idle_timeout = 6s
```

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/6ef2619a-9252-4851-be89-8a9b8d5c761f)


3. Hasil 3
```
pm = dynamic
pm.max_children = 45
pm.start_servers = 2
pm.min_spare_servers = 3
pm.max_spare_servers = 15
pm.process_idle_timeout = 8s
```

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/99a17d70-c37b-4be4-a8af-7fbc10115a72)


## Soal 20
> Nampaknya hanya menggunakan PHP-FPM tidak cukup untuk meningkatkan performa dari worker maka implementasikan Least-Conn pada Eisen. Untuk testing kinerja dari worker tersebut dilakukan sebanyak 100 request dengan 10 request/second.

#### Jawaban
Untuk mengerjakan nomor ini, maka konfigurasi pada load balancer dapat diubah sebagai berikut:
```
upstream laravel {
    least_conn;
    server 10.0.4.1;
    server 10.0.4.2;
    server 10.0.4.3;
}
```

Berikut merupakan hasilnya:

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/59519210-aede-426a-916c-a5fe426617db)


## Kendala:
- Ada beberapa soal yang direvisi sehingga menghambat pengerjaan
