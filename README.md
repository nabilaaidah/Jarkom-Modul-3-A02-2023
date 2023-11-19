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

![image](https://github.com/nabilaaidah/Jarkom-Modul-3-A02-2023/assets/110476969/f04f13fd-906a-4279-aa71-e7065cdb168d)

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
### a. Semua CLIENT harus menggunakan konfigurasi dari DHCP Server.
### b. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.16 - [prefix IP].3.32 dan [prefix IP].3.64 - [prefix IP].3.80
### c. Client yang melalui Switch4 mendapatkan range IP dari [prefix IP].4.12 - [prefix IP].4.20 dan [prefix IP].4.160 - [prefix IP].4.168 (3)
### d. Client mendapatkan DNS dari Heiter dan dapat terhubung dengan internet melalui DNS tersebut (4)
### e. Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch3 selama 3 menit sedangkan pada client yang melalui Switch4 selama 12 menit. Dengan waktu maksimal dialokasikan untuk peminjaman alamat IP selama 96 menit (5)

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
Keterangan: Dikarenakan terdapat dua range IP yang diminta, sehingga range ditulis dua kali.

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
