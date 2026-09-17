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

7. Chisa memutuskan mendirikan FTP Server pada node miliknya dengan shared folder di /var/wired/data. Terapkan kebijakan akses: user alice (hak akses read & write), user mika (dibatasi read-only), dan user eiri (dibatasi tanpa izin akses / blacklist). Buktikan konfigurasi dengan membuat file signal_alice.txt dari user alice, dan buktikan penolakan akses saat user eiri mencoba login.

disini kami melakukan set up ftp server dengan config sebagai berikut:

```
#!/bin/sh

# Atur DNS agar apk update bisa jalan
echo "nameserver 192.168.122.1" > /etc/resolv.conf

apk update
apk add vsftpd shadow

# Buat direktori shared
SHARED="/var/wired/data"
mkdir -p $SHARED
chmod 777 $SHARED

# Tambahkan nologin ke shells jika belum ada
grep -q "/sbin/nologin" /etc/shells || echo "/sbin/nologin" >> /etc/shells

# Buat user (Alice, Mika, Eiri)
id alice &>/dev/null || (useradd -d $SHARED -s /sbin/nologin alice && echo "alice:alice123" | chpasswd)
id mika &>/dev/null || (useradd -d $SHARED -s /sbin/nologin mika && echo "mika:mika123" | chpasswd)
id eiri &>/dev/null || (useradd -d $SHARED -s /sbin/nologin eiri && echo "eiri:eiri123" | chpasswd)

# Konfigurasi Utama vsftpd
cat <<EOF > /etc/vsftpd.conf
listen=YES
listen_address=0.0.0.0
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
allow_writeable_chroot=YES
seccomp_sandbox=NO
user_config_dir=/etc/vsftpd_users
EOF

# Konfigurasi Folder Berbasis User (Per-User Config)
mkdir -p /etc/vsftpd_users

# Hak akses penuh untuk Alice
cat <<EOF > /etc/vsftpd_users/alice
write_enable=YES
EOF

# Read-Only untuk Mika
cat <<EOF > /etc/vsftpd_users/mika
write_enable=NO
EOF

# Blacklist total untuk Eiri
cat <<EOF > /etc/vsftpd_users/eiri
write_enable=NO
download_enable=NO
dirlist_enable=NO
EOF

# Jalankan layanan vsftpd
pkill vsftpd
vsftpd /etc/vsftpd.conf &
```

kemudian kami menyimpan config file tersebut di file `ftp.sh` agar ketika nodenya direset config file tidak hilang dan juga mempermudah kami agar tidak set up ulang setiap kami reboot nodenya.

Kami login pada akun alice pada node Alice. Disini kami membuktikan bahwa akun alice memiliki akses untuk read & write dengan melakukan command `echo "hi ini alice" > signal_alice.txt`. Lakukan command `put signal_alice.txt` untuk mengirim file ke server ftp.
![alt text](assets/Read&Write.png)

Selanjutnya kami login pada akun mika di node Mika untuk membuktikan akun mika hanya read-only. Kami menggunakan command `ls` dan `put test.txt` untuk membuktikan kalau akun mika hanya read-only. Pada gambar dibawah bisa dilihat kalau melakukan command `put` muncul tulisan "permission denied".
![alt text](assets/Read-only.png)

Selanjutnya kami login pada akun eiri di node Eiri untuk membuktikan kalau Eiri diblacklist dari server ftp. Kami menggunakan command `ls` dan `put tes_eiri.txt`. Bisa dilihat pada gambar dibawah ketika melakukan command tersebut maka muncul tulisan "permission denied"
![alt text](assets/Blacklist.png)

8. Kelompok rahasia Knights perlu mengirimkan dokumen laporan intelijen ke FTP Server Chisa. Lakukan koneksi FTP client dari node Knights ke FTP Server Chisa menggunakan akun alice. Upload file berikut (link file). Analisis sesi Wireshark dan sebutkan: perintah FTP untuk upload (STOR), kode status sukses server (226), dan port data TCP yang dinegosiasikan pada mode PASV.

Pertama kita pergi ke node Knights untuk login pake akun alice. kemudian kita menggunakan command
```
get https://drive.google.com/drive/folders/1tvZpueSH9E3GWwXM6KNnM64Y5wNoIAYP?usp=sharing
```
untuk meng-upload file dari google drive.
![alt text](assets/Knights%20login%20to%20Alice.png)

Kemudian kami mengecek di wireshark untuk setiap ip yang tercapture pada wireshark.
![alt](assets/wireshark%20knights%20no%208.png)
