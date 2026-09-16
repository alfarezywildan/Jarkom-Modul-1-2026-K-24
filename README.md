# Jarkom-Modul-1-2026-K-24

## Member

| Nama                      | NRP        |
| ------------------------- | ---------- |
| Wildan Alfarezy       | 5027251088 |
| Ashkhabil Abror Budihardjo | 5027251049 |

## Laporan

1. Untuk pembangunan The Wired Pertama-tama kita kita membuat Router, Switch, entitas,serta NAT nya terlebih dahulu, lalu entitas dikonfigurasi sebagai client menggunakan prefix ip.

![alt text](assets/Topology.png)

2. Setelah itu kita melakukan konfigurasi dari router nya hingga pada entitasnya menjadi client menggunakan prefix ip.

Konfig pada Router:
```
auto eth0
iface eth0 inet dhcp
  up sysctl -w net.ipv4.ip_forward=1    
  up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

auto eth1
iface eth1 inet static
    address 192.223.1.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 192.223.2.1
    netmask 255.255.255.0

auto eth3
iface eth3 inet static
  address 192.223.3.1
  netmask 255.255.255.0
```

Konfig pada masing-masing entitas:
![alt text](assets/Alice.png)

![alt text](assets/Mika.png)

![alt text](assets/Chisa.png)

![alt text](assets/Knights.png)

![alt text](assets/Eiri.png)

3. memastikan seluruh Entitas (Client) di bawah Switch 1, Switch 2, dan Switch 3 dapat saling terhubung dan berkomunikasi satu sama lain.

![alt text](<assets/komunikasi alice.png>)

![alt text](<assets/komunikasi mika.png>)

4. Setelah itu kita melakukan pengecekan bahwa setiap Client dapat terhubung ke internet secara mandiri dengan melakukan konfigurasi pada setiap Client

```
echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

lalu kita cek apakah tersambung dengan penge cekan ke `google.com`:

![alt text](<assets/tes internet.png>)



5. Kita memastikan suluruh konfigurasi jaringan agar tidak hilang saat semua nya di node restart dengan membuat Buat script verifikasi di /root/cek_status.sh pada Router Lain.

```sh
ip -br a
echo" "
iptables -t nat -L -v -n  
```

Hasilnya:
![alt text](<assets/cek status.jpeg>)

6. Menjalankan file berikut (link file) lalu melakukan packet sniffing menggunakan Wireshark pada interface node mika, lalu menerapkan display filter khusus untuk enyaring paket yang berprotokol DNS atau ICMP.

Pertama - tama kita download file zip https://drive.google.com/drive/folders/1ZjFvWIjvAQAjE9pPthm7V_bGyaSt93lY?usp=sharing manual di google kemudian kita isi filenya dengan `nano traffic_protocol7.sh`

isi dari `traffic_protocol7.sh`:
```sh
#!/bin/bash
echo "============================================"
echo "  Protocol 7 Traffic Generator v2026"
echo "  Node: Mika Iwakura"
echo "============================================"
echo "[*] Generating DNS & ICMP traffic..."

# ICMP Traffic
ping -c 5 8.8.8.8 &
ping -c 5 1.1.1.1 &
ping -c 3 its.ac.id &

# DNS Queries
nslookup google.com 8.8.8.8 &
nslookup its.ac.id 8.8.8.8 &
nslookup github.com 1.1.1.1 &
dig @8.8.8.8 example.com A &
dig @1.1.1.1 cloudflare.com AAAA &

wait
echo "[*] Traffic generation complete."
echo "[*] Check Wireshark for captured packets."
```

kemudian kita menjalankan isi file tersebut dengan `bash traffic_protocol7.sh` selanjutnya kita click kanan pada kabel yang menghubungkan switch 1 dengan mika, lalu kita start capture.

![alt text](assets/protocol7.jpeg)

![alt text](assets/protocol7_2.jpeg)

hasilnya si Mika minta request ke ip its.ac.id yaitu 103.94.189.5 kemudian Mika juga melakukan request ke ip address milik Alice. Mika melakukan request ke ip 8.8.8.8 dan ip 1.1.1.1

