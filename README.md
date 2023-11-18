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
Jadi di soal ini disuruh bikin domain riegel.canyon.yyy.com untuk worker Laravel dan granz.channel.yyy.com untuk worker PHP, sehingga yang dilakukan adalah melakukan instalasi dependencies pada .bashrc dan menginisiasi domain dan mengarahkan pada node yang diinginkan.

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
