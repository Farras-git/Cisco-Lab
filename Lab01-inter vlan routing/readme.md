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

## 🎯 Tujuan Lab
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

#### f. Konfigurasi PC2
IPv4: 10.2.60.50/24

IPv6: 2001:db8:acad:1060::50/64

Default Gateway: 10.2.60.1 / 2001:db8:acad:1060::d1

#### g. Verifikasi konektivitas
Dari PC1 lakukan ping ke PC2:

```bash
PC1> ping 10.2.60.50
PC1> ping 2001:db8:acad:1060::50
```
Jika berhasil, berarti SW-1 sudah melakukan Inter-VLAN Routing dengan benar.

#### h. Periksa MAC Address Table
Gunakan perintah berikut di SW-1:

```bash
SW-1# show mac address-table dynamic
```
Contoh output:

Code
Mac Address Table
-------------------------------------------
Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
50      0050.56b3.8137    DYNAMIC     Gi1/0/23
60      0050.56b3.994b    DYNAMIC     Gi1/0/24

Total Mac Addresses for this criterion: 2
