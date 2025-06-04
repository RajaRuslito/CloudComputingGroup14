# Instalasi dan Konfigurasi Apache Cloudstack Private Cloud
![Departemen Teknik Elektro](https://hackmd.io/_uploads/BJNH1Yoblg.png)
(Departemen Teknik Elektro)

**Kelompok 14**

Anggota Kelompok: 
- Aria Bima Sakti
- Bintang Marsyuma
- Raja Yonandro Ruslito
- Zikri Zulfa Azim

## Pendahuluan

### Apa itu Cloudstack

Apache CloudStack adalah perangkat lunak komputasi awan sumber terbuka yang dirancang untuk menyebarkan dan mengelola jaringan besar mesin virtual (VM). Sebagai platform Infrastructure-as-a-Service (IaaS), CloudStack menyediakan seperangkat fitur yang lengkap untuk membangun dan mengelola lingkungan cloud yang dapat diskalakan. Platform ini mendukung berbagai hypervisor, kemampuan API yang luas, serta seperangkat alat yang kaya untuk administrator cloud.

### Tujuan Penggunaan Apache CloudStack

Tujuan dari penggunaan Apache CloudStack dalam proyek ini adalah untuk membangun dan mengelola infrastruktur cloud yang scalable, fleksibel, dan mudah dikontrol melalui antarmuka grafis maupun command line. Secara spesifik, penggunaan CloudStack bertujuan untuk:

* Menyediakan platform Infrastructure as a Service (IaaS) yang stabil dan handal.
* Mempermudah proses provisioning dan manajemen mesin virtual secara terpusat.
* Mendukung kebutuhan simulasi, pengujian, atau deployment layanan secara terisolasi melalui virtualisasi.
* Meningkatkan efisiensi dalam pengelolaan resource seperti jaringan, storage, dan compute secara otomatis.
* Memberikan pengalaman langsung dalam mengelola cloud environment berbasis open-source yang digunakan secara luas di industri.

### Referensi Arsitektur

![Arsitektur Cloudstack](https://hackmd.io/_uploads/B1s5yFiWle.png)
(Arsitektur Cloudstack)

### Dependencies yang Digunakan

#### MySQL

MySQL adalah sistem manajemen basis data relasional (RDBMS) sumber terbuka yang menggunakan Structured Query Language (SQL) untuk mengakses, menambahkan, dan mengelola data dalam basis data. Dikembangkan oleh Oracle Corporation, MySQL banyak digunakan untuk aplikasi web dan solusi basis data tertanam karena keandalannya, skalabilitasnya, dan kemudahan penggunaannya.

Apache CloudStack adalah perangkat lunak komputasi awan sumber terbuka untuk membuat, mengelola, dan menyebarkan layanan infrastruktur cloud. MySQL memainkan peran penting dalam fungsi CloudStack dengan berfungsi sebagai basis data utama untuk menyimpan informasi penting seperti data pengguna, informasi node komputasi, dan informasi array penyimpanan.

#### KVM

KVM (Kernel-based Virtual Machine) adalah teknologi virtualisasi sumber terbuka yang dibangun ke dalam kernel Linux. Teknologi ini memungkinkan kernel Linux berfungsi sebagai hypervisor, yang memungkinkan pembuatan dan pengelolaan mesin virtual (VM). KVM banyak digunakan karena efisiensi, skalabilitas, dan dukungannya terhadap berbagai sistem operasi tamu. CloudStack membutuhkan KVM sebagai lapisan abstraksi untuk mencegah akses langsung yang tidak aman ke perangkat keras dasar. KVM menyediakan metode yang aman untuk mengelola dan mengalokasikan sumber daya perangkat keras seperti CPU, penyimpanan, dan jaringan.

## Pengaturan Lingkungan Komputasi

### Konfigurasi Hardware yang Digunakan

```
CPU : Intel Core i5 gen 10
RAM : 32 GB
Storage : 150GB
Network : Ethernet 100GB/s
Operating System : Ubuntu Server 24.04
```

### Network Address

```
Network Address : 192.168.0.0/24
Host IP address : 192.168.0.172/24
Gateway : 192.168.0.1
```

## Konfigurasi Network 

### Mengubah Isi dari File Konfigurasi Jaringan pada Direktori /netplan

```
cd /etc/netplan
sudo nano ./0*.yaml
```

Isi dari file tersebut adalah seperti ini:
```
# This is the network config written by 'subiquity'
network:    
  version: 2
  ethernets:
    eno1:                             # Interface Name
      addresses: [192.168.0.172/24]         # Ip Address
      gateway4: 192.168.0.1                # Gateway
      nameservers:
        addresses: [10.1.1.1,8.8.8.8]   # DNS Server
```

Ubah isi file:

```
# This is the network config written by 'subiquity'
network:
  version: 2
  renderer: networkd
  ethernets:
    eno1:
      dhcp4: false
      dhcp6: false
      optional: true
  bridges:
    cloudbr0:
      addresses: [192.168.0.172/24]  #Your host IP address
      routes:
        - to: default
          via: 192.168.0.1
      nameservers:
        addresses: [1.1.1.1,8.8.8.8]
      interfaces: [eno1]
      dhcp4: false
      dhcp6: false
      parameters:
        stp: false
        forward-delay: 0
```

### Terapkan Konfigurasi Jaringan

```
sudo -i  #open new shell with root privileges
netplan generate #generate config file for the renderer
netplan apply  #applies network configuration to the system
reboot #reboot the system
```

> Catatan: Anda mungkin mengalami beberapa kesalahan selama langkah ini, pastikan Anda menggunakan "spasi" bukan "tab" saat memodifikasi file konfigurasi jaringan
> Catatan: Untuk memeriksa apakah konfigurasi jaringan Anda sudah diterapkan atau belum, gunakan perintah ifconfig dan cari antarmuka "br0", pastikan alamatnya sama dengan yang Anda konfigurasikan sebelumnya



### Uji Penggunaan Jaringan untuk Memastikan Apakah Konfigurasi telah Diterapkan dengan Baik

```
ifconfig     #check the ip address and existing interface
ping -c 20 google.com  #make sure you could connect to the internet
```

> Catatan: jika Anda tidak dapat melakukan ping ke google.com, coba ping gateway Anda dan 8.8.8.8
> Langkah ini untuk memastikan koneksi komputer dan internet Anda berhasil
### Masuk Sebagai Root User

```
sudo -i
```

### Menginstal Alat Pemantauan Hardware Resource

```
apt update -y
apt upgrade -y
apt install htop lynx duf -y
apt install bridge-utils
```

* htop adalah alat pemantauan penggunaan CPU
* duf adalah alat pemantauan penggunaan Disk
* lynx adalah peramban web berbasis CLI

### Aktifkan Login Root SSH

```
sed -i '/#PermitRootLogin prohibit-password/a PermitRootLogin yes' /etc/ssh/sshd_config
#restart ssh service
service ssh restart
#or
systemctl restart sshd.service
```

### Periksa Konfigurasi SSH
```
nano /etc/ssh/sshd_config
```
Temukan baris 'PermitRootLogin' pastikan diatur ke 'yes'

![image](https://hackmd.io/_uploads/ByJdavoble.png)


## Instalasi Cloudstack (Controller and Compute Node di Host yang Sama)


### Mengimpor Cloudstack Repositories Key

```
sudo -i
mkdir -p /etc/apt/keyrings 
wget -O- http://packages.shapeblue.com/release.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/cloudstack.gpg > /dev/null
echo deb [signed-by=/etc/apt/keyrings/cloudstack.gpg] http://packages.shapeblue.com/cloudstack/upstream/debian/4.18 / > /etc/apt/sources.list.d/cloudstack.list
```

* Baris pertama adalah untuk membuat direktori untuk menyimpan kunci publik cloudstack
* wget -O untuk mengunduh URL yang diberikan dan mengalihkan output ke perintah 'gpg --dearmor'
* Perintah 'gpg --dearmor' akan mengonversi dari ASCII armored ke format biner
* Perintah 'sudo tee' akan mengalihkan perintah 'gpg --dearmor' ke file /etc/apt/keyrings/cloudstack.gpg

### Periksa Repositori yang Ditambahkan
```
nano /etc/apt/sources.list.d/cloudstack.list
```
Pastikan ini ada pada file tersebut:

![image](https://hackmd.io/_uploads/rkqfCDsZle.png)


### Menginstal Cloudstack dan Server MySQL

```
apt-get update -y
apt-get install cloudstack-management mysql-server
```

> Catatan: Proses download akan memakan waktu lama

### Konfigurasi MySQL

#### Buka File Konfigurasi MySQL

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

#### Salin Baris Ini di Bawah Bagian [mysqld]

```
server-id = 1
sql-mode="STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION,ERROR_FOR_DIVISION_BY_ZERO,NO_ZERO_DATE,NO_ZERO_IN_DATE,NO_ENGINE_SUBSTITUTION"
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=1000
log-bin=mysql-bin
binlog-format = 'ROW'
```

#### Mulai Ulang Layanan MySQL
```
systemctl restart mysql
```

#### Periksa Status Layanan MySQL
```
systemctl status mysql
```
mysql diharapkan berjalan aktif sebagaimana output berikut:
![image](https://hackmd.io/_uploads/Bk1d0Pi-lx.png)


### Deploy Database sebagai Root dan Kemudian Buat Pengguna "cloud" dengan Kata Sandi "cloud" juga

```
cloudstack-setup-databases cloud:cloud@localhost --deploy-as=root:Pa$$w0rd -i 192.168.0.172
```

> pastikan alamat di akhir sesuai dengan alamat server

### Konfigurasikan Penyimpanan Primer dan Sekunder

```
apt-get install nfs-kernel-server quota
echo "/export  *(rw,async,no_root_squash,no_subtree_check)" > /etc/exports
mkdir -p /export/primary /export/secondary
exportfs -a
```

### Konfigurasikan Server NFS

```
sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server
sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common
echo "NEED_STATD=yes" >> /etc/default/nfs-common
sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota
service nfs-kernel-server restart
```
Penjelasan

```
`sed -i -e 's/^RPCMOUNTDOPTS="--manage-gids"$/RPCMOUNTDOPTS="-p 892 --manage-gids"/g' /etc/default/nfs-kernel-server`
# Perintah ini menggunakan perintah sed (stream editor) untuk mencari baris RPCMOUNTDOPTS="--manage-gids" dalam file /etc/default/nfs-kernel-server dan menggantinya dengan RPCMOUNTDOPTS="-p 892 --manage-gids". Opsi -i memberi tahu sed untuk mengedit file di tempatnya (misalnya, menyimpan perubahan ke file asli).

`sed -i -e 's/^STATDOPTS=$/STATDOPTS="--port 662 --outgoing-port 2020"/g' /etc/default/nfs-common`
# Perintah ini menggunakan sed untuk mencari baris STATDOPTS= dalam file /etc/default/nfs-common dan menggantinya dengan STATDOPTS="--port 662 --outgoing-port 2020".

`echo "NEED_STATD=yes" >> /etc/default/nfs-common`
# Perintah ini menambahkan baris NEED_STATD=yes di akhir file /etc/default/nfs-common. Operator >> di bash digunakan untuk menambahkan output ke sebuah file.

`sed -i -e 's/^RPCRQUOTADOPTS=$/RPCRQUOTADOPTS="-p 875"/g' /etc/default/quota`
# Perintah ini menggunakan sed untuk mencari baris RPCRQUOTADOPTS= dalam file /etc/default/quota dan menggantinya dengan RPCRQUOTADOPTS="-p 875".

`service nfs-kernel-server restart`
# Perintah ini akan memulai kembali Layanan NFS
```

## Konfigurasikan Host Cloudstack dengan KVM Hypervisor

### Instal KVM dan Cloudstack Agent

```
apt-get install qemu-kvm cloudstack-agent -y
```

### Konfigurasikan Manajemen Virtualisasi KVM

#### Ubah Beberapa Baris

```
sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf

# On Ubuntu 22.04, add LIBVIRTD_ARGS="--listen" to /etc/default/libvirtd instead.

sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd
```

Penjelasan
```
`sed -i -e 's/\#vnc_listen.*$/vnc_listen = "0.0.0.0"/g' /etc/libvirt/qemu.conf`
# Perintah ini menggunakan perintah sed (stream editor) untuk mencari baris dalam file /etc/libvirt/qemu.conf yang dimulai dengan #vnc_listen dan menggantinya dengan vnc_listen = "0.0.0.0". Opsi -i memberi tahu sed untuk mengedit file di tempatnya (yaitu, menyimpan perubahan ke file asli). Alamat 0.0.0.0 adalah alamat IP khusus yang digunakan dalam pemrograman jaringan untuk menentukan semua alamat IP pada mesin lokal.

`sed -i.bak 's/^\(LIBVIRTD_ARGS=\).*/\1"--listen"/' /etc/default/libvirtd`
# Perintah ini menggunakan sed untuk mencari baris dalam file /etc/default/libvirtd yang dimulai dengan LIBVIRTD_ARGS= dan menggantinya dengan LIBVIRTD_ARGS="--listen". Opsi -i.bak memberi tahu sed untuk mengedit file di tempatnya dan membuat cadangan file asli dengan ekstensi .bak.
```

#### Tambahkan Beberapa Baris Ini

```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
```
Penjelasan:
```
echo 'listen_tls=0' >> /etc/libvirt/libvirtd.conf
# nonaktifkan mendengarkan TLS pada daemon libvirt

echo 'listen_tcp=1' >> /etc/libvirt/libvirtd.conf
# aktifkan mendengarkan TCP pada daemon libvirt

echo 'tcp_port = "16509"' >> /etc/libvirt/libvirtd.conf
# tentukan port tempat daemon akan mendengarkan koneksi TCP

echo 'mdns_adv = 0' >> /etc/libvirt/libvirtd.conf
# nonaktifkan DNS multicast, layanan libvirtd tidak akan ditemukan melalui nDNS

echo 'auth_tcp = "none"' >> /etc/libvirt/libvirtd.conf
# nonaktifkan autentikasi untuk koneksi TCP ke daemon libvirt
```

#### Mulai Ulang libvirtd

```
systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket
systemctl restart libvirtd
```

Penjelasan
```
`systemctl mask libvirtd.socket libvirtd-ro.socket libvirtd-admin.socket libvirtd-tls.socket libvirtd-tcp.socket`
# Perintah ini menggunakan systemctl, pengelola sistem dan layanan untuk Linux, untuk menutupi beberapa unit soket yang terkait dengan layanan libvirtd. Menutupi unit di systemd secara efektif menonaktifkannya dan membuatnya tidak mungkin untuk memulainya secara manual atau mengizinkan layanan lain untuk memulainya. Dalam kasus ini, perintah tersebut menutupi beberapa soket yang digunakan libvirtd untuk berkomunikasi dengan proses lain. Ini mungkin dilakukan untuk mencegah libvirtd menerima koneksi melalui soket ini.

`systemctl restart libvirtd`
# Perintah ini menggunakan systemctl untuk memulai ulang layanan libvirtd. Hal ini sering kali diperlukan setelah membuat perubahan pada konfigurasi layanan atau unit terkaitnya (seperti soket), untuk memastikan perubahan tersebut berlaku.
```

libvirtd adalah daemon yang menyediakan manajemen mesin virtual (VM), jaringan virtual, dan penyimpanan untuk berbagai teknologi virtualisasi, seperti KVM, QEMU, Xen, dan lainnya. Ini adalah bagian dari proyek libvirt, yang menawarkan perangkat untuk mengelola platform virtualisasi. Tujuan utama libvirtd adalah untuk menyediakan API yang konsisten dan aman untuk mengelola VM dan sumber daya terkait, terlepas dari teknologi virtualisasi yang mendasarinya.


### Konfigurasi untuk Mendukung Docker dan Layanan Lainnya

```
echo "net.bridge.bridge-nf-call-arptables = 0" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 0" >> /etc/sysctl.conf
sysctl -p
```
Paket ARP dan IP tidak akan diproses oleh arptable dan iptable

### Generate Unique Host ID

```
apt-get install uuid -y
UUID=$(uuid)
echo host_uuid = \"$UUID\" >> /etc/libvirt/libvirtd.conf
systemctl restart libvirtd
```

### Konfigurasikan Firewall Iptables dan Jadikan Persisten

```
NETWORK=192.168.0.0/24
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 111 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 2049 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 32803 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 32769 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 892 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 875 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 662 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8250 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8080 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 8443 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 9090 -j ACCEPT
iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 16514 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p udp --dport 3128 -j ACCEPT
#iptables -A INPUT -s $NETWORK -m state --state NEW -p tcp --dport 3128 -j ACCEPT

apt-get install iptables-persistent
#jawab dengan: yes-yes
```

Langkah ini memastikan semua port layanan yang digunakan oleh cloudstack tidak diblokir oleh firewall dan dapat diakses oleh jaringan

* '-A input' akan menambahkan aturan ke rantai aturan INPUT
* '-s $NETWORK' yang menentukan sumber paket, dalam kasus ini sumbernya adalah jaringan 192.168.101.0/24
* '-m state' menggunakan modul state untuk mencocokkan status paket, '--state NEW' berarti aturan hanya diterapkan untuk paket yang memulai koneksi baru
* '-p udp/tcp --dport [PORT NUMBER]' menerapkan aturan untuk protokol tertentu dan nomor port tujuan
* '-j ACCEPT' menerima paket yang cocok dengan aturan  



### Nonaktifkan apparmor di libvirt

```
ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/
ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/
apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd
apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper
```

Penjelasan

```
`ln -s /etc/apparmor.d/usr.sbin.libvirtd /etc/apparmor.d/disable/`
# Perintah ini membuat tautan simbolik (jenis berkas yang mengarah ke berkas atau direktori lain) dari /etc/apparmor.d/usr.sbin.libvirtd ke /etc/apparmor.d/disable/. Ini secara efektif menonaktifkan profil AppArmor untuk usr.sbin.libvirtd.

`ln -s /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper /etc/apparmor.d/disable/`
# Perintah ini membuat tautan simbolis dari /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper ke /etc/apparmor.d/disable/, menonaktifkan profil AppArmor untuk usr.lib.libvirt.virt-aa-helper.

`apparmor_parser -R /etc/apparmor.d/usr.sbin.libvirtd`
# Perintah ini menggunakan utilitas apparmor_parser untuk menghapus profil AppArmor untuk usr.sbin.libvirtd dari kernel. Opsi -R memberi tahu apparmor_parser untuk menghapus profil.

`apparmor_parser -R /etc/apparmor.d/usr.lib.libvirt.virt-aa-helper`
# Perintah ini menghapus profil AppArmor untuk usr.lib.libvirt.virt-aa-helper dari kernel.

```

### Jalankan Management Server

```
cloudstack-setup-management
systemctl status cloudstack-management
tail -f /var/log/cloudstack/management/management-server.log #if you want to troubleshoot wait until all services (components) running successfully
```

Penjelasan
```
`cloudstack-setup-management`
# Perintah ini digunakan untuk menyiapkan server manajemen untuk Apache CloudStack, perangkat lunak komputasi awan sumber terbuka untuk membuat, mengelola, dan menyebarkan layanan infrastruktur awan. Perintah ini mengonfigurasi koneksi basis data, menyiapkan alamat IP server manajemen, dan memulai server manajemen.

`systemctl status cloudstack-management`
# Perintah ini menggunakan systemctl, pengelola sistem dan layanan untuk Linux, untuk menampilkan status layanan cloudstack-management. Perintah ini menunjukkan apakah layanan sedang berjalan atau tidak, dan menampilkan entri log terbaru. Anda dapat menggunakan perintah ini untuk memeriksa apakah server manajemen CloudStack berjalan dengan benar.

`tail -f /var/log/cloudstack/management/management-server.log #if you want to troubleshoot`
# Perintah ini menampilkan akhir berkas log server manajemen CloudStack dan kemudian menampilkan data tambahan saat berkas bertambah. Ini sering digunakan untuk memantau berkas log secara real time. Ini dapat berguna untuk mengatasi masalah jika Anda mengalami masalah dengan server manajemen CloudStack. Opsi -f memberi tahu tail untuk tetap membuka berkas dan menampilkan baris baru saat ditambahkan.

```

### Buka Browser dan Ketik

```
http://<YOUR_IP_ADDRESS>:8080
```

Contoh

```
http://192.168.0.172:8080
```

### Anda Seharusnya Dapat Melihat Dashboard Cloudstack

![image](https://hackmd.io/_uploads/H1bDkujbxx.png)

## Konfigurasi Dashboard Cloudstack
### Login ke Dashboard Cloudstack 
![Screenshot 2025-04-30 230208](https://hackmd.io/_uploads/ByfexujWex.png)

### Buat Zone baru
1. Memilih core untuk pilihan zone
![Screenshot 2025-04-30 231239](https://hackmd.io/_uploads/HykBlOjWgl.png)
2. Pilih Basic di bagian ini agar network dikonfigurasi secara otomatis
![image](https://hackmd.io/_uploads/BJkaxuo-xe.png)
3. Isi detail zone dengan nama IP DNS (gunakan IP 8.8.8.8), IP internal DNS (IP subnet jaringan server), dan hypervisor (pilih KVM). 
![Screenshot 2025-04-30 231255](https://hackmd.io/_uploads/rJI1WusZgl.png)
![Screenshot 2025-04-30 231301](https://hackmd.io/_uploads/B11D-diZll.png)
4. Lanjutkan ke step berikutnya.
![image](https://hackmd.io/_uploads/ryvtWds-el.png)
5. Isi detail pod yang akan digunakan, mulai dari nama, IP gateway, subnet mask, dan alamat IP yang akan digunakan untuk manajemen internal cloudstack.
![Screenshot 2025-04-30 231317](https://hackmd.io/_uploads/By5jbdibll.png)
6. Isi detail jaringan tamu yang nantinya digunakan untuk jaringan pada mesin VM. 
![image](https://hackmd.io/_uploads/SkwE7uobex.png)
7. Buat cluster baru di zone ini
![image](https://hackmd.io/_uploads/BkRj7_oWxl.png)
8. Isi detail host yang akan mengisi cluster ini dengan alamat IP server, username (root), dan password (Pa$$w0rd).
![Screenshot 2025-04-30 231533](https://hackmd.io/_uploads/BJTkEOj-eg.png)
9. Isi detail primary storage seperti berikut.
![image](https://hackmd.io/_uploads/B1u3V_iZxg.png)
10. Isi juga detail untuk secondary storage sebagai berikut.
![image](https://hackmd.io/_uploads/S1JlBdoZge.png)
11. Selesaikan konfigurasi sampai berhasil seperti gambar berikut.
![Screenshot 2025-04-30 232555](https://hackmd.io/_uploads/Hk3Gruo-xl.png)

### Menambahkan ISO VM ke Cloudstack
1. Buka tab Images dan masuk ke tab ISOs. Klik Register ISO. 
![image](https://hackmd.io/_uploads/rkzevuo-xe.png)
2. Isi detail dari ISO yang akan digunakan.
![image](https://hackmd.io/_uploads/ByZnDuiWgl.png)
> Jika terjadi error, gunakan link ISO yang digunakan diperoleh dari halaman download chrome dan salin link dari file ISO yang sedang didownload tersebut.
3. Masuk ke detail ISO yang telah dibuat dan lihat progress download ISO-nya.
![image](https://hackmd.io/_uploads/HkfVOOsblg.png)
4. Tunggu hingga proses download ISO selesai.
![image](https://hackmd.io/_uploads/rJ1Pu_ibex.png)

### Membuat Instance atau VM di Cloudstack
1. Buka tab Compute dan masuk ke tab Instances. Klik Add Instance.
![image](https://hackmd.io/_uploads/ByTfT_oWxx.png)
2. Masukkan detail zone, pod, cluster dan host yang digunakan untuk deploy VM.
![image](https://hackmd.io/_uploads/BJKu6doZee.png)
3. Pilih ISO yang telah di-download sebelumnya.
![image](https://hackmd.io/_uploads/SkSq6do-gg.png)
4. Pilih konfigurasi mesin untuk VM dan ukuran penyimpanannya.
![image](https://hackmd.io/_uploads/BybkA_ibex.png)
5. Pilih default pada security group.
![image](https://hackmd.io/_uploads/HyWTytoZex.png)
6. Untuk bagian detail bisa diisi atau dibiarkan saja dan luncurkan instance-nya.
![image](https://hackmd.io/_uploads/H1xleKs-xg.png)

## Mengakses Server dari Jaringan yang Berbeda
### Download Tailscale di Perangkat yang Berbeda
**Client**

![image](https://github.com/user-attachments/assets/e08e2777-29be-499e-b0ce-bf706bebf9be)

**Owner**

![image](https://github.com/user-attachments/assets/a954412e-b23d-466d-bd01-411d866f085f)

**Owner**

![image](https://github.com/user-attachments/assets/b1a41866-5566-453d-a251-909970ebb0ca)

**Owner**

![image](https://github.com/user-attachments/assets/5d7c7e25-0ed9-4d1f-93dd-dc6badf9d9ec)

**Owner**

![image](https://github.com/user-attachments/assets/1f247c8a-7b91-429c-ab04-b4ecd5f77308)

**Owner**

![image](https://github.com/user-attachments/assets/35b731a8-1da1-4e33-b97e-928ac350b5dd)

**Client menjadi Admin**

![image](https://github.com/user-attachments/assets/fad71365-a12f-421c-b2a2-3880bda405b0)



----
## Kesimpulan
Dengan menggunakan Apache CloudStack, proses manajemen infrastruktur virtual menjadi lebih mudah, terpusat, dan efisien. Meskipun terdapat beberapa tantangan teknis saat instalasi, solusi dapat ditemukan dengan dokumentasi dan troubleshooting yang tepat.

## Link Video Demo

[https://youtu.be/Gbz_mSAZZ6U](https://www.youtube.com/watch?v=Gbz_mSAZZ6U)

----
## Referensi
- [1] A. Rifqi, “CloudStack Install and Configure,” GitHub, [Online]. Available: https://github.com/AhmadRifqi86/cloudstack-install-and-configure/tree/main/cloudstack-install. [Accessed: 4-Jun-2025].
- [2] Learn Techno, “Install Cloudstack Management Server on Ubuntu 20.04 | Apache Cloudstack | Cloudstack Installation,” YouTube, 13-May-2022. [Online Video]. Available: https://www.youtube.com/watch?v=DlJg3LYvIIs&t=569s. [Accessed: 4-Jun-2025].
