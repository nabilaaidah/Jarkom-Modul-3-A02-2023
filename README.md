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
### Setelah mengalahkan Demon King, perjalanan berlanjut. Kali ini, kalian diminta untuk melakukan register domain berupa riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP (0) mengarah pada worker yang memiliki IP [prefix IP].x.1.

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
### Lakukan konfigurasi sesuai dengan peta yang sudah diberikan.

#### Jawaban
Konfigurasi dilakukan dengan membentuk topologi seperti yang dicontohkan dan mengatur config per nodenya. Jawaban dari nomor ini telah ditulis pada awal lapres.


## Soal 2 & 3 & 4 & 5
### Kemudian, karena masih banyak spell yang harus dikumpulkan, bantulah para petualang untuk memenuhi kriteria berikut.:
#### a. Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.
#### b. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80
#### c. Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168 (3)
#### d. Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut (4)
#### e. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit (5)

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
### Pada masing-masing worker PHP, lakukan konfigurasi virtual host untuk website berikut dengan menggunakan php 7.3.

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
