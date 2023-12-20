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
![](https://private-user-images.githubusercontent.com/115441787/291972229-b9932eff-c63e-4aad-887e-0f55fb04805b.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQxMTAsIm5iZiI6MTcwMzA5MzgxMCwicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTcyMjI5LWI5OTMyZWZmLWM2M2UtNGFhZC04ODdlLTBmNTVmYjA0ODA1Yi5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzM2NTBaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT1iMTNlYTgxY2NhMjE5NTllMmMwZDIzNGJiNDE1OGYzYmM5NGU1ZTgwZTU4OWQwYzg4ZTU1YjkwZmU5OGRhZDI2JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.hdnvgJpB91aq3uiiBq9IHDyH6q-4fL-w7WLu61c9XrA)

![](https://private-user-images.githubusercontent.com/115441787/291972633-2f098f7f-c0d1-4bb9-b8f5-25b620d4d6d8.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQyMjIsIm5iZiI6MTcwMzA5MzkyMiwicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTcyNjMzLTJmMDk4ZjdmLWMwZDEtNGJiOS1iOGY1LTI1YjYyMGQ0ZDZkOC5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzM4NDJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT05MThmNDZhYTRiMDQ5YWM1MWZiNzZjMWU0MWRiM2YzYTI1N2JlNGY1ZDJiYzcxMTcyMzg0M2M2MjIwZjgxODgwJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.prnPMc06O37m5iQMtY3jWezshz6VzQfAvuxrrEhkWbs)

![](https://private-user-images.githubusercontent.com/115441787/291973081-ce753815-5447-40db-b096-62377d2f5ea3.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQzNDYsIm5iZiI6MTcwMzA5NDA0NiwicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTczMDgxLWNlNzUzODE1LTU0NDctNDBkYi1iMDk2LTYyMzc3ZDJmNWVhMy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzQwNDZaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0xNzJkZDIzNzc2ZTQ5ODQ2NDJjNGY3YTEyMjVjNTIxN2JjZTczMGRhNWJiZTJjNWNlNjgyM2EwNjlhZTVmOTZmJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.ltNdrHCZqd_t-G8UzwKPRWRhxwpD86dcclSR1w2Q9FA)


![](https://private-user-images.githubusercontent.com/115441787/291973602-ee8aca6f-ef51-4324-838f-9222f6e54fc3.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQ0NzIsIm5iZiI6MTcwMzA5NDE3MiwicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTczNjAyLWVlOGFjYTZmLWVmNTEtNDMyNC04MzhmLTkyMjJmNmU1NGZjMy5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzQyNTJaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT01ZTFjMThhNTVjMTU2NmUxYjk2YTUzMTllMTA3MzEyZTMxNzk4YjBhZDAyNzI2MDc1NjJhOWQ5MzAzMzVhZTU2JlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.qIpxhRptHBQf0zOPA80cGFdtZSH4lU_gKZY4mdvC5zI)

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
![](https://private-user-images.githubusercontent.com/115441787/291974788-5ab33986-abdd-4e25-99ff-11712fb757e5.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQ3OTUsIm5iZiI6MTcwMzA5NDQ5NSwicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTc0Nzg4LTVhYjMzOTg2LWFiZGQtNGUyNS05OWZmLTExNzEyZmI3NTdlNS5qcGc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzQ4MTVaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT01OWUzM2FlMDFmNDZkMzg4ZjAwNWY3MGVmMWY2ZGU1NzE2NGJlMWZkOWY5NTQ2YWU1YjUzODI1NjRiYTI4YWQzJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.d2RtSoURSglYIVxaDUKE7eXYkTINkZid-yJ2pRjgAkM)

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

![](https://private-user-images.githubusercontent.com/115441787/291975340-0b2603e4-8d76-4611-be2b-117eab23d1be.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDMwOTQ5MzcsIm5iZiI6MTcwMzA5NDYzNywicGF0aCI6Ii8xMTU0NDE3ODcvMjkxOTc1MzQwLTBiMjYwM2U0LThkNzYtNDYxMS1iZTJiLTExN2VhYjIzZDFiZS5qcGc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjIwJTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIyMFQxNzUwMzdaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT0xMzk4YzJjNmNkNDhhODMwOGEyZWI1MThlOTRiMGI4ZjM0NDA1MjdkZDUwMjM3ZWZmOWZlYmE2NGZhYTJjMmMxJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.LUrsu9MYVLqDBJEsk-ZohcGb0F2GfpdGYEOjZFyYuIw)

## Soal 8
> Karena berbeda koalisi politik, maka subnet dengan masyarakat yang berada pada Revolte dilarang keras mengakses WebServer hingga masa pencoblosan pemilu kepala suku 2024 berakhir. Masa pemilu (hingga pemungutan dan penghitungan suara selesai) kepala suku bersamaan dengan masa pemilu Presiden dan Wakil Presiden Indonesia 2024.

## Soal 9
> Sadar akan adanya potensial saling serang antar kubu politik, maka WebServer harus dapat secara otomatis memblokir  alamat IP yang melakukan scanning port dalam jumlah banyak (maksimal 20 scan port) di dalam selang waktu 10 menit.
> (clue: test dengan nmap)

## Soal 10
> Karena kepala suku ingin tau paket apa saja yang di-drop, maka di setiap node server dan router ditambahkan logging paket yang di-drop dengan standard syslog level. 
