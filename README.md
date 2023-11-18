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

# Pengaturan Config

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

Semua Client Configuration
auto eth0
iface eth0 inet dhcp
```


