# Lab – Implement Inter-VLAN Routing

## 🌐 Topology
<img width="748" height="654" alt="image" src="https://github.com/user-attachments/assets/de9425fd-c307-4837-8cb2-780ad072394d" />


## 📑 Addressing Table

| Device | Interface   | IPv4 Address     | IPv6 Address                  | IPv6 Link-Local |
|--------|-------------|------------------|-------------------------------|-----------------|
| R1     | G0/1        | 10.1.13.1/24     | 2001:db8:acad:10d1::1/64      | fe80::1:1       |
|        | G0/0        | 10.1.3.1/24      | 2001:db8:acad:1013::1/64      | fe80::1:2       |
| SW-1   | G0/0        | 10.1.13.13/24    | 2001:db8:acad:10d1::d1/64     | -               |
|        | VLAN50      | 10.2.50.1/24     | 2001:db8:acad:1050::d1/64     | fe80::d1:1      |
|        | VLAN60      | 10.2.60.1/24     | 2001:db8:acad:1060::d1/64     | fe80::d1:3      |
| R2     | G0/0        | 10.1.3.3/24      | 2001:db8:acad:1013::3/64      | fe80::3:1       |
|        | G0/1.75     | 10.3.75.1/24     | 2001:db8:acad:3075::1/64      | fe80::3:2       |
|        | G0/1.85     | 10.3.85.1/24     | 2001:db8:acad:3085::1/64      | fe80::3:3       |
|        | G0/1.999    | -                | -                             | -               |
| SW-2   | VLAN75      | 10.3.75.14/24    | 2001:db8:acad:3075::d2/64     | fe80::d2:1      |
| PC1    | NIC         | 10.2.50.50/24    | 2001:db8:acad:1050::50/64     | -               |
| PC2    | NIC         | 10.2.60.50/24    | 2001:db8:acad:1060::50/64     | -               |
| PC3    | NIC         | 10.3.75.50/24    | 2001:db8:acad:3075::50/64     | -               |
| PC4    | NIC         | 10.3.85.50/24    | 2001:db8:acad:3085::50/64     | -               |

---

## 🎯 Objectives
- **Part 1: Build the Network and Configure Basic Device Settings**  
  Membangun topologi jaringan sesuai desain lab, kemudian melakukan konfigurasi dasar pada perangkat (hostname, IP address, IPv6 address, dan mengaktifkan interface). Tujuannya agar semua perangkat siap digunakan untuk tahap konfigurasi berikutnya.  

- **Part 2: Configure and Verify Inter-VLAN Routing on a Layer 3 Switch**  
  Mengkonfigurasi switch Layer 3 (SW-1) agar dapat melakukan routing antar VLAN menggunakan SVI (Switch Virtual Interface). Verifikasi dilakukan dengan ping antar host untuk memastikan komunikasi antar VLAN berjalan.  

- **Part 3: Configure and Verify Router-based Inter-VLAN Routing**  
  Mengkonfigurasi router (R2) dengan metode Router-on-a-Stick, yaitu menggunakan subinterface dengan encapsulation dot1q untuk setiap VLAN. Router akan berfungsi sebagai gateway bagi host di VLAN berbeda.  

- **Part 4: Examine CAM and CEF Details**  
  Melihat tabel CAM (Content Addressable Memory) pada switch dan tabel CEF (Cisco Express Forwarding) pada router. Tujuannya untuk memahami bagaimana perangkat menyimpan informasi forwarding (FIB dan adjacency table) sehingga proses pengiriman paket lebih cepat dan efisien.


---

## 🖥️ Environment
- Simulator: GNS3  
- IOS Router: `vios-adventerprisek9-m.SPA.159-3.M6`  
- IOS Switch: `viosl2-adventerprisek9-m.ssa.high_iron_20200929`  

---

## 📚 Background / Scenario
Metode untuk memindahkan paket dan frame dari satu interface ke interface lain telah berkembang seiring waktu.  
Dalam lab ini, kita akan mengkonfigurasi **Inter-VLAN Routing** dalam beberapa bentuk (menggunakan Layer 3 Switch dan Router-on-a-Stick), kemudian memeriksa tabel forwarding yang digunakan perangkat untuk membuat keputusan pengiriman paket.  

> **Catatan:**  
> - Lab ini adalah latihan konfigurasi dan verifikasi berbagai metode Inter-VLAN Routing, bukan representasi praktik terbaik jaringan di dunia nyata.  
> - Perangkat fisik yang biasanya digunakan di modul CCNP adalah **Cisco 4221** dan **Cisco 3650** dengan **Cisco IOS XE Release 16.9.4 (universalk9 image)**.  
> - Karena lab ini dijalankan di **GNS3**, maka digunakan image:  
>   - Router: `vios-adventerprisek9-m.SPA.159-3.M6`  
>   - Switch: `viosl2-adventerprisek9-m.ssa.high_iron_20200929`  
> - Perintah dan output mungkin sedikit berbeda tergantung versi IOS yang digunakan.  
> - Pastikan router dan switch dalam kondisi bersih (startup configuration dihapus) sebelum memulai lab.  

---

## 🛠️ Required Resources
- **2 Router** (Cisco 4221 atau image setara di GNS3)  
- **2 Switch** (Cisco 3650 atau image setara di GNS3)  
- **4 PC** (dengan terminal emulator, misalnya VPCS atau Tera Term)  
- **Console cables** untuk konfigurasi perangkat Cisco IOS via port console  
- **Ethernet dan serial cables** sesuai topologi yang digunakan  

---

## Tujuan Lab
- Membangun topologi jaringan sesuai desain.  
- Mengkonfigurasi dan memverifikasi **Inter-VLAN Routing** pada Layer 3 Switch.  
- Mengkonfigurasi dan memverifikasi **Router-on-a-Stick** untuk VLAN tambahan.  
- Memeriksa tabel **CAM** (Content Addressable Memory) pada switch dan tabel **CEF** (Cisco Express Forwarding) pada router untuk memahami bagaimana perangkat melakukan forwarding paket dengan efisien.

## ⚙️ Part 1: Build the Network and Configure Basic Device Settings

### Step 1: Cable the network
Hubungkan perangkat sesuai diagram topologi. Pastikan setiap router, switch, dan PC terhubung dengan kabel Ethernet/serial sesuai desain lab.

### Step 2: Configure basic settings
Masuk ke console masing-masing perangkat, lalu lakukan konfigurasi dasar berikut:

#### Router R1
```bash
no ip domain-lookup
hostname R1
line con 0
 exec-timeout 0 0
 logging synchronous
exit
banner motd # This is R1, Inter-VLAN Routing Lab #
```

#### Router R2
```bash
no ip domain-lookup
hostname R2
line con 0
 exec-timeout 0 0
 logging synchronous
exit
banner motd # This is R2, Inter-VLAN Routing Lab #
```
#### Switch SW-1
```bash
no ip domain-lookup
hostname SW-1
line con 0
 exec-timeout 0 0
 logging synchronous
exit
banner motd # This is SW-1, Inter-VLAN Routing Lab #
interface range g1/0-3, g2/0-3, g3/0-3
 shutdown
```
#### Switch SW-2
```bash
no ip domain-lookup
hostname SW-2
line con 0
 exec-timeout 0 0
 logging synchronous
exit
banner motd # This is SW-2, Inter-VLAN Routing Lab #
interface range g1/0-3, g2/0-3, g3/0-3
 shutdown
```
#### Step 3: Set the clock
Atur waktu pada setiap perangkat ke UTC:

```bash
clock set 14:00:00 23 March 2026
```
#### Step 4: Save configuration
Simpan konfigurasi agar tidak hilang saat reboot:

```bash
copy running-config startup-config
```

## ⚙️ Part 2: Configure and Verify Inter-VLAN Routing on a Layer 3 Switch

Pada bagian ini, kita akan mengkonfigurasi dan memverifikasi **Inter-VLAN Routing** menggunakan **Layer 3 Switch (SW-1)** dan **Router R1**. Tujuannya adalah agar host yang berada di VLAN berbeda dapat saling berkomunikasi melalui switch yang sudah diaktifkan fungsi routing-nya.

> **Catatan:**  
> - Pada perangkat fisik seperti Catalyst 3650 dengan IOS XE, fitur **SDM template** sudah mendukung operasi dual-stack (IPv4 dan IPv6) secara default, sehingga tidak perlu konfigurasi tambahan.  
> - Jika menggunakan perangkat atau image IOS lain, pastikan template mendukung IPv6 dengan perintah:  
>   ```
>   show sdm prefer
>   ```
>   Jika jumlah IPv6 unicast routes = 0, maka ubah template dengan:  
>   ```
>   sdm prefer dual-ipv4-and-ipv6 default
>   ```
>   (nama template bisa berbeda tergantung versi IOS). Setelah itu perangkat perlu di-*reload*.  
> - Karena lab ini dijalankan di **GNS3** dengan image `viosl2-adventerprisek9-m.ssa.high_iron_20200929`, fitur IPv6 routing sudah tersedia sehingga tidak perlu mengubah SDM template.

### Step 1: Konfigurasi Inter-VLAN Routing di SW-1
Pada langkah ini, kita akan mengaktifkan fungsi routing di SW-1, membuat VLAN, meng-assign port ke VLAN, serta mengkonfigurasi SVI (Switch Virtual Interface) untuk mendukung komunikasi antar VLAN.

#### a. Aktifkan IP routing dan IPv6 unicast routing
```bash
SW-1(config)# ip routing
SW-1(config)# ipv6 unicast-routing
```
#### b. Buat VLAN dan beri nama sesuai topologi
```bash
SW-1(config)# vlan 50
SW-1(config-vlan)# name Group50
SW-1(config-vlan)# exit

SW-1(config)# vlan 60
SW-1(config-vlan)# name Group60
SW-1(config-vlan)# exit
```
#### c. Assign port ke VLAN
```bash
SW-1(config)# interface g0/1
SW-1(config-if)# switchport mode access
SW-1(config-if)# switchport access vlan 50
SW-1(config-if)# no shutdown
SW-1(config-if)# exit

SW-1(config)# interface g0/2
SW-1(config-if)# switchport mode access
SW-1(config-if)# switchport access vlan 60
SW-1(config-if)# no shutdown
SW-1(config-if)# exit
```
#### d. Konfigurasi SVI untuk VLAN 50 dan VLAN 60
```bash
SW-1(config)# interface vlan 50
SW-1(config-if)# ip address 10.2.50.1 255.255.255.0
SW-1(config-if)# ipv6 address fe80::d1:2 link-local
SW-1(config-if)# ipv6 address 2001:db8:acad:1050::d1/64
SW-1(config-if)# no shutdown
SW-1(config-if)# exit

SW-1(config)# interface vlan 60
SW-1(config-if)# ip address 10.2.60.1 255.255.255.0
SW-1(config-if)# ipv6 address fe80::d1:3 link-local
SW-1(config-if)# ipv6 address 2001:db8:acad:1060::d1/64
SW-1(config-if)# no shutdown
SW-1(config-if)# exit
```
#### e. Konfigurasi PC1
IPv4: 10.2.50.50/24

IPv6: 2001:db8:acad:1050::50/64

Default Gateway: 10.2.50.1 / 2001:db8:acad:1050::d1

Konfigurasi di VPCS 
```
ip 10.2.50.50/24 10.2.50.1
ip 2001:db8:acad:1050::50/64 2001:db8:acad:1050::d1
```
#### f. Konfigurasi PC2
IPv4: 10.2.60.50/24

IPv6: 2001:db8:acad:1060::50/64

Default Gateway: 10.2.60.1 / 2001:db8:acad:1060::d1

Konfigurasi di VPCS 
```
ip 10.2.60.50/24 10.2.60.1
ip 2001:db8:acad:1060::50/64 2001:db8:acad:1060::d1
```
#### g. Verifikasi konektivitas
Dari PC1 lakukan ping ke PC2:

```bash
PC1> ping 10.2.60.50
PC1> ping 2001:db8:acad:1060::50
```
Jika berhasil, berarti SW-1 sudah melakukan Inter-VLAN Routing dengan benar.

<img width="417" height="239" alt="image" src="https://github.com/user-attachments/assets/8cf52ca9-6914-4a6e-b7c6-59eebfeb889b" />

#### h. Periksa MAC Address Table
Gunakan perintah berikut di SW-1:

```bash
SW-1# show mac address-table dynamic
```
Contoh output:

<img width="328" height="137" alt="image" src="https://github.com/user-attachments/assets/e5deff11-0c0c-475e-97c6-4b15cff780aa" />


Total Mac Addresses for this criterion: 2

### Step 2: Konfigurasi Routed Port dan Default Route di SW-1

#### a. Konfigurasi interface G0/0 sebagai routed port
Interface ini digunakan untuk koneksi langsung ke R1. Karena harus berfungsi sebagai routed port, maka mode switchport dihapus.

```bash
SW-1(config)# interface g0/0
SW-1(config-if)# no switchport
SW-1(config-if)# ip address 10.1.13.13 255.255.255.0
SW-1(config-if)# ipv6 address fe80::d1:1 link-local
SW-1(config-if)# ipv6 address 2001:db8:acad:10d1::d1/64
SW-1(config-if)# no shutdown
SW-1(config-if)# exit
```
#### b. Verifikasi interface tidak lagi terkait VLAN
Gunakan perintah berikut:

```bash
SW-1# show vlan brief | i g0/0
```
Jika tidak ada output, berarti interface sudah benar-benar menjadi routed port.

#### c. Konfigurasi static default route
Tambahkan default route IPv4 dan IPv6 yang mengarah ke R1:

```bash
SW-1(config)# ip route 0.0.0.0 0.0.0.0 10.1.13.1
SW-1(config)# ipv6 route ::/0 2001:db8:acad:10d1::1
```
Catatan:  
Jika muncul pesan error seperti:

```Code
%ADJ-3-RESOLVE_REQ: Adj resolve request: Failed to resolve 10.1.13.1
```
Itu berarti switch sudah mengirim ARP untuk mencari MAC address dari 10.1.13.1, tetapi belum mendapat balasan. Hal ini normal karena R1 belum dikonfigurasi. Setelah R1 aktif dengan IP yang sesuai, error ini akan hilang.

### Step 3: Konfigurasi Interface dan Static Routing di R1

#### a. Aktifkan IPv6 unicast routing
```bash
R1(config)# ipv6 unicast-routing
```
#### b. Konfigurasi interface sesuai Addressing Table
Interface G0/1 (terhubung ke SW-1)

```bash
R1(config)# interface g0/1
R1(config-if)# ip address 10.1.13.1 255.255.255.0
R1(config-if)# ipv6 address fe80::1:1 link-local
R1(config-if)# ipv6 address 2001:db8:acad:10d1::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
```
Interface G0/0 (terhubung ke R2)

```bash
R1(config)# interface g0/0
R1(config-if)# ip address 10.1.3.1 255.255.255.0
R1(config-if)# ipv6 address fe80::1:2 link-local
R1(config-if)# ipv6 address 2001:db8:acad:1013::1/64
R1(config-if)# no shutdown
R1(config-if)# exit
```
#### c. Konfigurasi static routing
Tambahkan route menuju jaringan VLAN di SW-1, serta default route ke R2:

IPv4 static routes

```bash
R1(config)# ip route 10.2.0.0 255.255.0.0 10.1.13.13
R1(config)# ip route 0.0.0.0 0.0.0.0 10.1.3.3
```
IPv6 static routes

```bash
R1(config)# ipv6 route 2001:db8:acad:1050::/64 2001:db8:acad:10d1::d1
R1(config)# ipv6 route 2001:db8:acad:1060::/64 2001:db8:acad:10d1::d1
R1(config)# ipv6 route ::/0 2001:db8:acad:1013::3
```
#### d. Verifikasi konektivitas
Dari R1 lakukan ping ke PC2:

```bash
R1# ping 10.2.60.50
R1# ping 2001:db8:acad:1060::50
```
<img width="581" height="156" alt="image" src="https://github.com/user-attachments/assets/323c21aa-2db3-4d80-a04b-72da5de83335" />

## ⚙️ Part 3: Configure and Verify Router-based Inter-VLAN Routing

### Catatan
- Pada switch fisik Catalyst 3650 dengan IOS XE, biasanya perlu cek **SDM template** dengan `show sdm prefer` untuk memastikan dukungan IPv6.  
- Karena lab ini dijalankan di **GNS3 (viosl2-adventerprisek9)**, perintah tersebut tidak tersedia. Fitur IPv6 routing sudah aktif secara default, sehingga tidak perlu konfigurasi tambahan.

---

### Step 1: Konfigurasi VLAN di SW-2
```bash
SW-2(config)# vlan 75
SW-2(config-vlan)# name Group75
SW-2(config-vlan)# exit

SW-2(config)# vlan 85
SW-2(config-vlan)# name Group85
SW-2(config-vlan)# exit

SW-2(config)# vlan 999
SW-2(config-vlan)# name NativeVLAN
SW-2(config-vlan)# exit
```
#### a. Assign port ke VLAN
```bash
SW-2(config)# interface g0/1
SW-2(config-if)# switchport mode access
SW-2(config-if)# switchport access vlan 75
SW-2(config-if)# no shutdown
SW-2(config-if)# exit

SW-2(config)# interface g0/2
SW-2(config-if)# switchport mode access
SW-2(config-if)# switchport access vlan 85
SW-2(config-if)# no shutdown
SW-2(config-if)# exit
```
#### b. Konfigurasi SVI untuk VLAN 75
```bash
SW-2(config)# interface vlan 75
SW-2(config-if)# ip address 10.3.75.14 255.255.255.0
SW-2(config-if)# ipv6 address fe80::d2:1 link-local
SW-2(config-if)# ipv6 address 2001:db8:acad:3075::d2/64
SW-2(config-if)# no shutdown
SW-2(config-if)# exit
```
#### c. Konfigurasi trunk ke R2
```bash
SW-2(config)# interface g0/0
SW-2(config-if)# switchport trunk encapsulation dot1q
SW-2(config-if)# switchport mode trunk
SW-2(config-if)# switchport trunk native vlan 999
SW-2(config-if)# switchport trunk allowed vlan 75,85,999
SW-2(config-if)# no shutdown
SW-2(config-if)# exit
```
Dengan ini, SW-2 sudah siap untuk berkomunikasi dengan R2 melalui trunk. Selanjutnya di R2 kita akan buat subinterface (Router-on-a-Stick) untuk VLAN 75 dan VLAN 85 agar PC3 dan PC4 bisa saling berkomunikasi.

### Step 2: Konfigurasi Subinterface di R2 (Router-on-a-Stick)

Pada langkah ini, kita akan mengaktifkan **Router-on-a-Stick** di R2 untuk mendukung VLAN 75 dan VLAN 85. Router akan membuat subinterface pada port G0/1 dengan encapsulation dot1q.

#### a. Aktifkan IPv6 unicast routing
```bash
R2(config)# ipv6 unicast-routing
```
#### b. Konfigurasi interface fisik
```bash
R2(config)# interface g0/1
R2(config-if)# no shutdown
R2(config-if)# exit
```
#### c. Konfigurasi subinterface untuk VLAN 75
```bash
R2(config)# interface g0/1.75
R2(config-subif)# encapsulation dot1q 75
R2(config-subif)# ip address 10.3.75.1 255.255.255.0
R2(config-subif)# ipv6 address fe80::3:2 link-local
R2(config-subif)# ipv6 address 2001:db8:acad:3075::1/64
R2(config-subif)# no shutdown
R2(config-subif)# exit
```
#### d. Konfigurasi subinterface untuk VLAN 85
```bash
R2(config)# interface g0/1.85
R2(config-subif)# encapsulation dot1q 85
R2(config-subif)# ip address 10.3.85.1 255.255.255.0
R2(config-subif)# ipv6 address fe80::3:3 link-local
R2(config-subif)# ipv6 address 2001:db8:acad:3085::1/64
R2(config-subif)# no shutdown
R2(config-subif)# exit
```
#### e. Konfigurasi subinterface untuk Native VLAN 999
```bash
R2(config)# interface g0/1.999
R2(config-subif)# encapsulation dot1q 999 native
R2(config-subif)# no shutdown
R2(config-subif)# exit
```
#### Konfigurasi PC3 dan PC4
PC3

IPv4: 10.3.75.50/24

IPv6: 2001:db8:acad:3075::50/64

Default Gateway: 10.3.75.1 / 2001:db8:acad:3075::1
```
ip 10.3.75.50/24 10.3.75.1
ip 2001:db8:acad:3075::50/64  2001:db8:acad:3075::1
```
PC4

IPv4: 10.3.85.50/24

IPv6: 2001:db8:acad:3085::50/64

Default Gateway: 10.3.85.1 / 2001:db8:acad:3085::1
```
ip 10.3.85.50/24 10.3.85.1
ip 2001:db8:acad:3085::50/64  2001:db8:acad:3085::1
```
Step 4: Verifikasi konektivitas
Dari PC3 lakukan ping ke PC4:

```bash
PC3> ping 10.3.85.50
PC3> ping 2001:db8:acad:3085::50
```

<img width="427" height="234" alt="image" src="https://github.com/user-attachments/assets/6a4a5f04-44d0-4fe7-b7d7-d4e71cb45d81" />

Jika ping berhasil, berarti R2 sudah melakukan Inter-VLAN Routing dengan metode Router-on-a-Stick.

### Step 3: Konfigurasi Static Routing untuk End-to-End Reachability

#### a. Konfigurasi interface G0/0 di R2
Interface ini menghubungkan R2 dengan R1. Atur alamat sesuai Addressing Table:

```bash
R2(config)# interface g0/0
R2(config-if)# ip address 10.1.3.3 255.255.255.0
R2(config-if)# ipv6 address fe80::3:1 link-local
R2(config-if)# ipv6 address 2001:db8:acad:1013::3/64
R2(config-if)# no shutdown
R2(config-if)# exit
```
#### b. Konfigurasi static default route di R2
Tambahkan default route IPv4 dan IPv6 yang mengarah ke R1:

```bash
R2(config)# ip route 0.0.0.0 0.0.0.0 10.1.3.1
R2(config)# ipv6 route ::/0 2001:db8:acad:1013::1
```
Dengan konfigurasi ini, semua trafik yang tidak dikenali oleh R2 akan diarahkan ke R1.

#### c. Verifikasi konektivitas
Dari PC3 lakukan ping ke PC2:

```bash
PC3> ping 10.2.60.50
PC3> ping 2001:db8:acad:1060::50
```
<img width="409" height="234" alt="image" src="https://github.com/user-attachments/assets/77ec0297-2185-4bed-93de-5406231df93f" />

Jika ping berhasil, berarti routing sudah berfungsi dua arah:

Dari VLAN 75/85 melalui R2 → R1 → SW-1 → VLAN 50/60.

Dari VLAN 50/60 melalui SW-1 → R1 → R2 → VLAN 75/85.

Ini menandakan solusi routing end-to-end sudah berjalan dengan baik.

## ⚙️ Part 4: Examine CAM and CEF Details

Pada bagian ini, kita akan melihat bagaimana perangkat menyimpan informasi forwarding untuk mempercepat proses pengiriman paket.

### Step 1: Periksa tabel CAM pada switch
Gunakan perintah berikut di SW-1 atau SW-2:
```bash
SW-1# show mac address-table dynamic
SW-2# show mac address-table dynamic
```
Output akan menampilkan MAC address host (PC1, PC2, PC3, PC4) beserta port yang terhubung. Ini menunjukkan bagaimana switch melakukan forwarding berdasarkan MAC address.

### Step 2: Periksa tabel routing dan CEF pada router
Gunakan perintah berikut di R1 dan R2:

```bash
R1# show ip route
R1# show ipv6 route
R1# show ip cef
R1# show ipv6 cef

R2# show ip route
R2# show ipv6 route
R2# show ip cef
R2# show ipv6 cef
```
Routing Table: Menunjukkan jaringan yang dikenal oleh router.

CEF (Cisco Express Forwarding): Menunjukkan FIB (Forwarding Information Base) dan adjacency table yang digunakan untuk forwarding cepat.

### Step 3: Analisis
CAM Table di switch menyimpan MAC address dan port, sehingga frame bisa dikirim ke tujuan dengan efisien.

CEF Table di router menyimpan informasi routing yang sudah dioptimalkan, sehingga paket bisa diteruskan dengan cepat tanpa perlu lookup berulang di routing table.

## Kesimpulan Lab
Dalam lab ini kita berhasil:

- Membangun topologi jaringan sesuai desain.

- Mengkonfigurasi Inter-VLAN Routing pada Layer 3 Switch (SW-1) menggunakan SVI.

- Mengkonfigurasi Router-on-a-Stick pada R2 untuk VLAN 75 dan VLAN 85.

- Mengaktifkan static routing di R1 dan R2 untuk memastikan komunikasi end-to-end antar semua host.

- Memeriksa CAM dan CEF tables untuk memahami bagaimana perangkat melakukan forwarding frame dan paket dengan efisien.

📌 Hasil akhir: Semua PC (PC1, PC2, PC3, PC4) dapat saling berkomunikasi baik dengan IPv4 maupun IPv6. Lab ini menunjukkan bagaimana kombinasi Layer 3 Switch + Router-on-a-Stick + Static Routing dapat digunakan untuk mendukung komunikasi antar VLAN dalam sebuah jaringan kompleks.

