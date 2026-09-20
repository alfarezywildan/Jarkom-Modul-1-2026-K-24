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

```sh
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


11. pertama kita melakukan setup di node chisa dengan membuat `setup_telnet.sh` yang isinya:

```sh
#!/bin/sh

echo "[*] Memperbarui repository dan menginstal busybox-extras (telnetd)..."
apk update
apk add busybox-extras

echo "[*] Membuat user 'phantom_user' dengan password 'wired_ghost'..."
adduser -h /home/phantom_user -s /bin/sh -D phantom_user
echo "phantom_user:wired_ghost" | chpasswd

killall telnetd 2>/dev/null
telnetd -l /bin/login

echo "[+] Setup selesai! Layanan Telnet aktif dan siap diuji dari node Eiri."
```

setelah itu kita beri izin dan menjalankannya:

```sh
chmod +x setup_telnet.sh
./setup_telnet.sh
```

Selesai setup kita lanjut ke node Eiri untuk melakukan telnet

```sh
telnet 192.223.2.2
```

setelah itu tangkap sesi itu menggunakan wireshark



Jelaskan Mengapa Setiap Karakter Terkirim dalam Paket TCP Terpisah
 **Jawaban:** 
  Setiap karakter dikirimkan dalam paket TCP yang terpisah karena protokol seperti Telnet menggunakan mode interaktif terminal berbasis karakter (*character-at-a-time*). Dalam mode ini, sistem tidak menunggu kalimat atau satu baris teks selesai diketik dalam *buffer*, melainkan langsung memproses dan mengirimkan setiap penekanan tombol (*keystroke*) seketika itu juga ke server agar karakter tersebut dapat langsung di-*echo* dan tampil secara *real-time* di layar pengguna tanpa adanya jeda atau *lag*.

12. Pertama kita ke node nya knights untuk melakukan setup menyambungkan port nya dengan membuat `setup_listener.sh` isinya:

```sh
#!/bin/sh

nohup sh -c "nc -lvkp 22 & nc -lvkp 80 &" > /tmp/port_test.out 2>&1 &

echo "setup selesai"
```

setelah itu kita beri izin dan menjalankannya:

```sh
chmod +x setup_telnet.sh
./setup_listener.sh
```

Selanjutnya kita ke node Alice untuk melakukan Netcat `nc` ke port 22, 80, dan port tertutup 7777.

```sh
nc -vz 192.223.3.2 22
nc -vz 192.223.3.2 80
nc -vz 192.223.3.2 7777
```

Setelah itu kita buka wireshark untuk menganalisisnya:


## Analisis Perbedaan TCP Flag (Port Terbuka vs Port Tertutup)

### 1. Analisis Port Terbuka (Open Port)
* **Respons Server:** Ketika klien mengirimkan paket *SYN* ke port yang sedang aktif/terbuka (misalnya port layanan SSH di port `22` atau HTTP di port `80`), server merespons dengan mengembalikan kombinasi flag **SYN-ACK** (`0x0012`).
* **Kesimpulan:** Keberadaan flag *SYN-ACK* menandakan bahwa server siap menerima koneksi dan proses *3-way handshake* TCP dapat dilanjutkan.

### 2. Analisis Port Tertutup (Closed Port)
* **Respons Server:** Ketika klien mengirimkan paket *SYN* ke port yang tidak aktif atau tertutup (misalnya port uji coba pada port `7777`), server merespons secara langsung dengan mengembalikan flag **RST-ACK** (`0x0014`).
* **Kesimpulan:** Flag *Reset (RST)* yang dikombinasikan dengan *ACK* menunjukkan bahwa port tersebut tertutup dan menolak koneksi secara tegas, sehingga sesi komunikasi langsung dihentikan oleh sistem.

13. Pertama kita melakukan setup pada node knights untuk menginstall openssh server dengan `openssh.sh`

```sh
#!/bin/sh

apk update
apk add openssh

ssh-keygen -A

id mika_admin >/dev/null 2>&1 || adduser -h /home/mika_admin -s /bin/sh -D mika>
echo "mika_admin:secure_pass" | chpasswd

sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd>
sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_>
sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_co>

# Jalankan sshd langsung tanpa service manager
/usr/sbin/sshd

echo "[+] Setup SSH server di Alpine selesai dan aktif!"
```

Setelah melakukan setup kita lakukan pembersihan port dan SSH lama

```sh
killall nc
killall dropbear
killall sshd 
```

Jalankan OpenSSH di latar belakang
```sh
/usr/sbin/sshd
```

Selanjutnya kita ke node Mika untuk mengcopy public key dengan langkah dibawah ini:

```sh
ssh-keygen -R 192.223.3.2
```
Setelah itu kita kembali lagi ke node knights untuk menempel public key milik Mika

```sh
mkdir -p /home/mika_admin/.ssh

# Tempelkan kunci publik Mika di antara tanda kutip di bawah ini
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDGZbEAzkjFkbUG1/6Y7FCXWpiFKNSu+UjcbnFFpgsYV/5EHBcZz3sfICGS0D6xkWpIGUdU6imFHyWZ6y58FZbYk4EKS5LIxKvtMYjBdzqZ2AaIKSWSa49bniRxy1dvT9Ne3rgldXjhxguudjGbQEUuzJVbIJQAaMmpRqhc5DjUv7OH5tlbCboUVDWt5vPIdCy3S3Aa92dC/vPlLaDJ8fUJkANBvh/sHP0mh0i+E7dZ/SooRzW29tV/YRttZpNXtWhxW06/dPTfC2dC+zu26eNPMMk0gCf94qBW8/jsNAGyTofZuj38TQtqv44Lgs0RKgfpSO999ke9js5ozP82/yTJ root@Mika" > /home/mika_admin/.ssh/authorized_keys

chmod 700 /home/mika_admin/.ssh
chmod 600 /home/mika_admin/.ssh/authorized_keys
chown -R mika_admin:mika_admin /home/mika_admin/.ssh
```

Lalu kembali laki ke node Mika untuk menjalankan koneksi SSH ke node knights

```sh
ssh mika_admin@192.223.3.2
```

dilanjutkan dengan menganalisis menggunakan wireshark

## Analisis Protokol SSH (Protocol Version Exchange, Key Exchange, & Enskripsi)

### 1. Identifikasi Paket Protocol Version Exchange
* **Analisis:** Berdasarkan hasil tangkapan paket, tahap awal komunikasi SSH dimulai dengan **Protocol Version Exchange**. Klien dan server saling bertukar string versi protokol yang digunakan secara terbuka (*plain text*) sebelum enkripsi aktif.
* **Contoh Paket:** Terlihat pada tangkapan paket, klien mengirimkan string versi dengan payload seperti `SSH-2.0-OpenSSH_10.2`.

### 2. Identifikasi Paket Key Exchange (KEX)
* **Analisis:** Setelah pertukaran versi, proses berlanjut ke **Key Exchange (KEX)**. Pada tahap ini, kedua belah pihak melakukan negosiasi algoritma kriptografi, metode pertukaran kunci (seperti `mlkem768x25519-sha256`), serta verifikasi identitas host menggunakan algoritma seperti `ssh-ed25519`. Paket ini berisi parameter negosiasi algoritma dan kunci publik sementara.

### 3. Penjelasan Mengapa Kredensial Tidak Terlihat dalam Bentuk Teks Terbuka
* **Analisis:** Berbeda dengan Telnet yang mengirimkan seluruh data dan kredensial secara mentah (*plain text*), SSH langsung mengaktifkan lapisan enkripsi yang kuat (seperti `chacha20-poly1305` atau AES) segera setelah proses *Key Exchange* selesai. Oleh karena itu, seluruh data interaktif berikutnya—termasuk *username*, *password*, atau perintah yang diketik—berubah menjadi *ciphertext* yang dienkripsi secara penuh, sehingga tidak dapat dibaca secara langsung di Wireshark.