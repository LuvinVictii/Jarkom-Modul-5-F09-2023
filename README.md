# Jarkom-Modul-5-F09-2023

| Nama                   | NRP        |
|------------------------|------------|
| Muhammad Daffa Harits  | 5025211005 |
| Muhammad Naufal Baihaqi| 5025211103 |

## Intro
Setelah pandai mengatur jalur-jalur khusus, kalian diminta untuk membantu North Area menjaga wilayah mereka dan kalian dengan senang hati membantunya karena ini merupakan tugas terakhir.

1. Tugas pertama, buatlah peta wilayah sesuai berikut ini:

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/1abfcb43-82df-4974-afad-03920e44acaf" width="600">
</p>
  Keterangan:
  
  - Richter adalah DNS Server
  - Revolte adalah DHCP Server
  - Sein dan Stark adalah Web Server
  - Jumlah Host pada SchwerMountain adalah 64
  - Jumlah Host pada LaubHills adalah 255
  - Jumlah Host pada TurkRegion adalah 1022
  - Jumlah Host pada GrobeForest adalah 512

2. Untuk menghitung rute-rute yang diperlukan, gunakan perhitungan dengan metode VLSM. Buat juga pohonnya, dan lingkari subnet yang dilewati.
3. Kemudian buatlah rute sesuai dengan pembagian IP yang kalian lakukan.
4. Tugas berikutnya adalah memberikan ip pada subnet SchwerMountain, LaubHills, TurkRegion, dan GrobeForest menggunakan bantuan DHCP.

## Soal 1
> Agar topologi yang kalian buat dapat mengakses keluar, kalian diminta untuk mengkonfigurasi Aura menggunakan iptables, tetapi tidak ingin menggunakan MASQUERADE.

jalankan command berikut di aura :
```
ETH0_IP=$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

iptables -t nat -A POSTROUTING -o eth0 -j SNAT --to-source $ETH0_IP
```

lalu test dengan melakukan ping ke luar seperti google.com. berikut merupakan hasilnya:
<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/318dff40-545f-4030-b643-0fc985b07132" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/67fb952b-e21b-482f-8f67-c110be56a76c" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/d3f28c93-4173-4c3f-9ba6-7add3869183b" width="600">
</p>

## Soal 2
> Kalian diminta untuk melakukan drop semua TCP dan UDP kecuali port 8080 pada TCP.

gunakan aturan berikut di heiter (yang penting tidak di client) 

```
iptables -F

iptables -A INPUT -p icmp -j ACCEPT

iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

iptables -A INPUT -p tcp -j DROP

iptables -A INPUT -p udp -j DROP
```
lalu lakukan testing menggunakan client dengan salah satu client membuka port 8080 (yang kami pakai => SchwerMountain dan satu client yang lain menunjuk ke ip dhcp SchwerMountain). Berikut hasilnya :

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/857f9f19-e797-426c-82be-fd77f487116a" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/1189729a-f38f-4544-a6cb-6881f2ee5d2f" width="600">
</p>

## Soal 3
> Kepala Suku North Area meminta kalian untuk membatasi DHCP dan DNS Server hanya dapat dilakukan ping oleh maksimal 3 device secara bersamaan, selebihnya akan di drop.

gunakan aturan berikut pada dns server (richter) dan dhcp server (revolte) :
```
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

iptables -A INPUT -p icmp -m connlimit --connlimit-above 3 --connlimit-mask 0 -j DROP
```
lalu lakukan ping menuju revolte yang merupakan dhcp server (jangan di stop dulu hingga semua client telah mengeklik ping tersebut).
<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/7f949503-ac7c-4d28-a32b-652b0216d719" width="900">
</p>

terlihat hanya 3 client yang dapat terhubung

## Soal 4
> Lakukan pembatasan sehingga koneksi SSH pada Web Server hanya dapat dilakukan oleh masyarakat yang berada pada GrobeForest.

gunakan aturan berikut pada web server :
```
iptables -A INPUT -p tcp --dport 22 -s 10.56.16.0/22 -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -j DROP
```

lalu buka port 22 menggunakan : `nc -l -p 22` di web server (contoh : stark). <br>

testing => gunakan nmap (ip stark) 22 => `nmap 10.56.31.2 22` . lakukan di semua client dan akan terlihat hanya di GrobeForest saja yang port 22 nya open, sisanya filtered.

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/bf32c6b8-56ad-4029-a6cd-ac768c12540f" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/5f2a5519-1d37-4157-8931-0fa72707a0ae" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/23251d4b-6d2c-4dab-9be3-fb5de0a753f7" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/b138040e-c2b3-4e17-9214-789bfdb1ea06" width="600">
</p>

## Soal 5
> Selain itu, akses menuju WebServer hanya diperbolehkan saat jam kerja yaitu Senin-Jumat pada pukul 08.00-16.00.

akan lebih tenang jika rules yang ada sebelumnya pada web server dihapus terlebih dahulu menggunakan `iptables -F INPUT`. <br>

gunakan rules berikut pada web server :
```
iptables -A INPUT -p tcp --dport 22 -s 10.56.16.0/22 -m time --timestart 08:00 --timestop 16:00 --weekdays Mon,Tue,Wed,Thu,Fri -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -j DROP

```

karena saat pengecekan belum masih masuk di range jam tersebut, makan untuk pengecekan saja digunakan timestop `14:00`. <br>
Didapatkan output sebagai berikut :

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/1b375abf-850b-411f-85a3-974748587e7a" width="600">
</p>

terlihat di GrobeForest sudah filtered untuk port 22 karena diluar jamnya (digunakan timestop berbeda dengan soal untuk testing)


## Soal 6
> Lalu, karena ternyata terdapat beberapa waktu di mana network administrator dari WebServer tidak bisa stand by, sehingga perlu ditambahkan rule bahwa akses pada hari Senin - Kamis pada jam 12.00 - 13.00 dilarang (istirahat maksi cuy) dan akses di hari Jumat pada jam 11.00 - 13.00 juga dilarang (maklum, Jumatan rek).

gunakan aturan berikut pada web server :

```
iptables -A INPUT -p tcp --dport 22 -s 10.56.16.0/22 -m time --timestart 12:00 --timestop 13:00 --weekdays Mon,Tue,Wed,Thu -j DROP

iptables -A INPUT -p tcp --dport 22 -s 10.56.16.0/22 -m time --timestart 11:00 --timestop 13:00 --weekdays Fri -j DROP
```

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/4862fbf2-6e0b-446f-919d-4227560bbd6c" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/46d84a3e-252d-4ca8-b53e-dac2021ee07b" width="600">
</p>

## Soal 7
> Karena terdapat 2 WebServer, kalian diminta agar setiap client yang mengakses Sein dengan Port 80 akan didistribusikan secara bergantian pada Sein dan Stark secara berurutan dan request dari client yang mengakses Stark dengan port 443 akan didistribusikan secara bergantian pada Sein dan Stark secara berurutan.

gunakan rules berikut pada `heiter` :

```
iptables -A PREROUTING -t nat -p tcp --dport 80 -d 10.56.16.3 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.56.16.3

iptables -A PREROUTING -t nat -p tcp --dport 80 -d 10.56.16.3 -j DNAT --to-destination 10.56.31.2

iptables -A PREROUTING -t nat -p tcp --dport 443 -d 10.56.31.2 -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 10.56.31.2

iptables -A PREROUTING -t nat -p tcp --dport 443 -d 10.56.31.2 -j DNAT --to-destination 10.56.16.3
```
test menggunakan dapat 2 opsi :
1. command di sein : `while true; do nc -l -p 80 -c 'echo "ini sein"'; done` . sambil command di stark : `while true; do nc -l -p 80 -c 'echo "ini stark"'; done` <br>
lalu test di client dengan `nc 10.56.31.2 80` ber kali kali.

2. command di sein : `while true; do nc -l -p 443 -c 'echo "ini sein"'; done` . sambil command di stark : `while true; do nc -l -p 443 -c 'echo "ini stark"'; done` <br>
lalu test di client (cth : TurkRegion) dengan `nc 10.56.31.2 443` ber kali kali. <br><br>

contoh menggunakan opsi 2: 

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/4a5241d4-9fb4-4fba-a4e5-f066dd4569bd" width="600">
</p>

## Soal 8
> Karena berbeda koalisi politik, maka subnet dengan masyarakat yang berada pada Revolte dilarang keras mengakses WebServer hingga masa pencoblosan pemilu kepala suku 2024 berakhir. Masa pemilu (hingga pemungutan dan penghitungan suara selesai) kepala suku bersamaan dengan masa pemilu Presiden dan Wakil Presiden Indonesia 2024.

buat file bash yang berisi aturan pada soal, seperti sebagai berikut:

```
Revolte_Subnet="10.56.29.0/30"

Pemilu_Start=$(date -d "2023-10-19T00:00" +"%Y-%m-%dT%H:%M")

Pemilu_End=$(date -d "2024-02-15T00:00" +"%Y-%m-%dT%H:%M")

iptables -A INPUT -p tcp -s $Revolte_Subnet --dport 80 -m time --datestart "$Pemilu_Start" --datestop "$Pemilu_End" -j DROP
```

lalu jalankan file bash tersebut. lalu test di `revolte` dan di `grobeforest` seperti sebagai berikut :

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/e8f03a92-c893-45df-b154-d3c18cbc3114" width="600">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/20346944-bc15-4cbb-8aeb-c6c35c3c1e0c" width="600">
</p>

<br>
terlihat pada revolte port 22 sudah filtered


## Soal 9
> Sadar akan adanya potensial saling serang antar kubu politik, maka WebServer harus dapat secara otomatis memblokir  alamat IP yang melakukan scanning port dalam jumlah banyak (maksimal 20 scan port) di dalam selang waktu 10 menit.
> (clue: test dengan nmap)

gunakan aturan berikut pada web server :
```
iptables -N portscan

iptables -A INPUT -m recent --name portscan --update --seconds 600 --hitcount 20 -j DROP
iptables -A FORWARD -m recent --name portscan --update --seconds 600 --hitcount 20 -j DROP

iptables -A INPUT -m recent --name portscan --set -j ACCEPT
iptables -A FORWARD -m recent --name portscan --set -j ACCEPT
```

test di client bisa memakai dua cara:
1. ping ip web server (contoh stark : 10.56.31.2)
ping akan macet setelah send req ke 20 diterima oleh stark
2. menggunakan bash test9.sh pada grobeforest yang sudah ada syntax looping menggunakan nmap juga ke stark <br>
    ```
    for i in {1..25}; do
      echo $i
      nmap -p 80 -T2 -sS 10.56.31.2
      sleep 2
    done
    ```
agar lebih cepat, digunakan testing cara 1 :

<p align="center">
  <img src="https://github.com/LuvinVictii/Jarkom-Modul-5-F09-2023/assets/78089862/f0880e94-17c3-43d2-afb3-90ad0d31165d" width="600">
</p>

ping diberhentikan karena saat `stark` ,sebagai web server yang digunakan, sudah menerima 20 req. maka ping akan diam saja tidak mengeluarkan apapun.

## Soal 10
> Karena kepala suku ingin tau paket apa saja yang di-drop, maka di setiap node server dan router ditambahkan logging paket yang di-drop dengan standard syslog level. 

gunakan aturan berikut di semua node yang merupakan server dan semua node yang merupakan router :
```
iptables -A INPUT  -j LOG --log-level debug --log-prefix 'Dropped Packet' -m limit --limit 1/second --limit-burst 10
```

penjelasan singkat: <br>
1. `-A INPUT`: Menambahkan aturan ke chain INPUT. Ini berarti aturan ini akan berlaku untuk paket-paket yang menuju ke sistem lokal.

2. `-j LOG`: Menentukan tindakan yang akan diambil jika paket cocok dengan aturan ini, yaitu mencatat pesan log.

3. `--log-level debug`: Menentukan tingkat log yang akan digunakan. Dalam hal ini, tingkat lognya adalah "debug". Anda dapat mengganti "debug" dengan tingkat log yang sesuai dengan kebutuhan Anda (info, warning, debug, dll.).

4. `--log-prefix 'Dropped Packet'`: Menentukan awalan (prefix) untuk pesan log yang akan dicatat. Dalam hal ini, pesan log akan dimulai dengan teks "Dropped Packet".

5. `-m limit --limit 1/second` --limit-burst 10: Menggunakan modul limit untuk mengontrol frekuensi pencatatan log.
   * -m limit: Menyertakan modul limit.
   * --limit 1/second: Menentukan batasan frekuensi pencatatan log, dalam hal ini setiap detik satu kali.
   * --limit-burst 10: Menentukan jumlah burst atau "limit-burst". Ini adalah jumlah log yang dapat dicatat dalam satu waktu burst sebelum batasan frekuensi berlaku.


<hr>

# Kendala
1. Telat memahami konsep iptables yang merupakan cara untuk memberikan aturan antar node
2. Waktunya bersamaan dengan masa-masa akhir Final Project tugas lain
