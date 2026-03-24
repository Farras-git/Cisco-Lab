# Lab 02 – Observe STP Topology Changes and Implement RSTP

##  Tujuan Lab
Lab ini bertujuan untuk memahami bagaimana **Spanning Tree Protocol (STP)** dan **Rapid Spanning Tree Protocol (RSTP)** bekerja dalam mencegah loop di jaringan layer 2. Fokus utama:
- Proses pemilihan **Root Bridge**.
- Peran port (Root Port, Designated Port, Alternate).
- Perbedaan waktu konvergensi antara STP klasik dan RSTP.

---

##  Environment Details
- **Simulator:** GNS3  
- **IOS Image:** Cisco IOSvL2 (Layer 2 Switch)  
- **Jumlah perangkat:** 3 switch (D1, D2, A1)  

---

##  Topologi Jaringan

<img width="631" height="471" alt="image" src="https://github.com/user-attachments/assets/13c300aa-6bfb-4784-9ad2-ab7619698e43" />

Topologi terdiri dari 3 switch dengan koneksi redundant:
- **D1 ↔ D2** (backbone link)  
- **D1 ↔ A1** (dua jalur redundant)  
- **D2 ↔ A1** (dua jalur redundant)  

### Interface Mapping
| Perangkat | Interface | Terhubung ke | Interface Lawan |
|-----------|-----------|--------------|-----------------|
| D1        | Gi0/0     | D2           | Gi0/0           |
| D1        | Gi0/1     | A1           | Gi0/0           |
| D1        | Gi0/2     | A1           | Gi0/1           |
| D2        | Gi0/2     | A1           | Gi0/3           |
| D2        | Gi0/1     | A1           | Gi0/2           |


<img width="277" height="518" alt="image" src="https://github.com/user-attachments/assets/bd38899f-0e13-4efc-ae08-4ab7da196a7e" />



## Addressing Table
| Device | Interface | IPv4 Address | Subnet Mask |
|--------|-----------|--------------|-------------|
| D1     | VLAN1     | 10.0.0.1     | 255.0.0.0   |
| D2     | VLAN1     | 10.0.0.2     | 255.0.0.0   |
| A1     | VLAN1     | 10.0.0.3     | 255.0.0.0   |

---

## Part 1 - Konfigurasi Perangkat

#### 🔹 D1
```bash
hostname D1
spanning-tree mode pvst
banner motd # D1, STP and RSTP Lab #
line con 0
 exec-timeout 0 0
 logging synchronous
exit
interface range g0/3, g1/0-3, g2/0-3, g3/0-3
 shutdown
exit
interface range g0/0-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
vlan 2
 name SecondVLAN
exit
interface vlan 1
 ip address 10.0.0.1 255.0.0.0
 no shutdown
exit
copy running-config startup-config
```

#### 🔹 D2
```bash
hostname D2
spanning-tree mode pvst
banner motd # D2, STP and RSTP Lab #
line con 0
 exec-timeout 0 0
 logging synchronous
exit
interface range g0/3, g1/0-3, g2/0-3, g3/0-3
 shutdown
exit
interface range g0/0-2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
vlan 2
 name SecondVLAN
exit
interface vlan 1
 ip address 10.0.0.2 255.0.0.0
 no shutdown
exit
copy running-config startup-config
```
#### 🔹 A1
```bash
hostname A1
spanning-tree mode pvst
banner motd # A1, STP and RSTP Lab #
line con 0
 exec-timeout 0 0
 logging synchronous
exit
interface range g1/0-3, g2/0-3, g3/0-3
 shutdown
exit
interface range g0/0-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 no shutdown
exit
vlan 2
 name SecondVLAN
exit
interface vlan 1
 ip address 10.0.0.3 255.0.0.0
 no shutdown
exit
copy running-config startup-config
```

## Part 2 – Discover the Default Spanning Tree

### Tujuan
Bagian ini bertujuan untuk melihat bagaimana **PVST+ (Per-VLAN Spanning Tree Plus)** secara otomatis memilih root bridge dan menentukan port role pada jaringan yang sudah dikonfigurasi. Karena ada dua VLAN (VLAN 1 dan VLAN 2), maka akan ada dua proses pemilihan root bridge.

---

### Hasil Perintah `show spanning-tree root`

#### 🔹 D1
```bash
D1# show spanning-tree root
```

<img width="544" height="104" alt="image" src="https://github.com/user-attachments/assets/bfdcb013-10b0-4253-9d24-f0dc66c93c04" />

➡ D1 sendiri adalah root bridge untuk VLAN 1 dan VLAN 2 (cost = 0, tidak ada root port karena dia root).

#### 🔹 D2
```bash
D2# show spanning-tree root
```

<img width="535" height="108" alt="image" src="https://github.com/user-attachments/assets/adfa82fe-e318-400a-80aa-a08a1b34782d" />

➡ D2 mengenali D1 sebagai root bridge. Port Gi0/0 menjadi root port dengan cost 4 (default untuk GigabitEthernet).

#### 🔹 A1
```bash
A1# show spanning-tree root
```

<img width="553" height="109" alt="image" src="https://github.com/user-attachments/assets/2835971d-fe0f-4555-9075-871d64a5a487" />

➡ Sama seperti D2, A1 juga menunjuk D1 sebagai root bridge. Port Gi0/0 menjadi root port dengan cost 4.

Analisis
Root Bridge: D1 terpilih sebagai root bridge untuk VLAN 1 dan VLAN 2.

Alasan: Semua switch menggunakan priority default (32768 + VLAN ID → 32769 untuk VLAN 1, 32770 untuk VLAN 2). Karena priority sama, pemilihan ditentukan oleh MAC address terendah. D1 memiliki MAC paling kecil, sehingga dia menang.

Root Ports:

D2 → Gi0/0

A1 → Gi0/0

Designated Ports: Port yang mengarah keluar dari root bridge (D1) akan menjadi designated port.

Non-Designated Ports: Port lain di D2 dan A1 yang tidak dipilih akan masuk blocking/alternate untuk mencegah loop.

## Part 3 – Identify Designated Ports

###  Tujuan
Menentukan port mana yang berperan sebagai **Designated Port**. Designated Port adalah port yang tetap aktif forwarding di setiap segmen, mengirim BPDU, dan belajar MAC address. Semua port di root bridge otomatis menjadi Designated Port, dan di setiap segmen non-root akan ada satu Designated Port.

---

### Hasil Observasi

#### 🔹 D1 (Root Bridge)
```bash
D1# show spanning-tree
```
<img width="557" height="596" alt="image" src="https://github.com/user-attachments/assets/12001347-a61e-4037-9bfb-eb7b1aa69401" />

➡ Semua port aktif di D1 otomatis menjadi Designated Port karena D1 adalah root bridge.


#### 🔹 D2
```bash
D1# show spanning-tree
```
<img width="593" height="621" alt="image" src="https://github.com/user-attachments/assets/592b9512-b200-4620-b1dc-ed1c2d16bef2" />


#### 🔹 A1
```bash
A1# show spanning-tree
```
<img width="596" height="626" alt="image" src="https://github.com/user-attachments/assets/25e1304a-c5fc-42ff-8d2b-af37f3b54ff9" />

➡ A1 hanya memiliki satu Root Port (Gi0/0) menuju D1.

➡ Port lain (Gi0/1, Gi0/2, Gi0/3) masuk Alternate/Blocking karena hanya satu root port yang dipakai. Alternate port akan aktif jika root port gagal.

#### Perbedaan Root Port, Designated Port, dan Alternate Port dalam STP/RSTP:

🔹 Root Port (RP)
- Lokasi: hanya ada di switch non-root.

- Fungsi: port yang dipilih sebagai jalur utama menuju root bridge.

- Ciri: selalu Forwarding, hanya satu root port per switch.

Contoh Di Lab ini:

A1 → Gi0/0

D2 → Gi0/0

🔹 Designated Port (DP)
- Lokasi: ada di setiap segmen jaringan.

- Fungsi: port yang dipilih untuk mengirim BPDU dan menjaga segmen tetap terhubung ke root.

- Ciri: semua port di root bridge otomatis jadi designated port. Di segmen non-root, port dengan path cost lebih rendah ke root akan jadi designated.

Contoh Di Lab ini:

Semua port aktif di D1 (karena D1 adalah root bridge).

Port Gi0/1 dan Gi0/2 di D2 (karena D2 punya cost lebih rendah dibanding A1).

🔹 Alternate Port (Altn)
- Lokasi: di switch non-root, pada link redundant yang tidak dipakai.

- Fungsi: standby, akan aktif jika root port gagal.

- Ciri: status Blocking (tidak forwarding traffic, hanya dengar BPDU).

Contoh Di Lab ini:

A1 → Gi0/1, Gi0/2, Gi0/3 (semua masuk blocking karena hanya Gi0/0 yang dipakai sebagai root port).


## Part 4 – Implement and Observe Rapid Spanning Tree Protocol (RSTP)

###  Tujuan
Mengaktifkan **Rapid Spanning Tree Protocol (RSTP)** pada semua switch dan mengamati bagaimana RSTP mempercepat proses konvergensi dibanding STP klasik.

---

###  Langkah Implementasi

#### 🔹 D2 – Uji Topology Change dengan Debug
```bash
D2# debug spanning-tree events
D2(config)# interface g0/0
D2(config-if)# shutdown
```

Observasi:

<img width="788" height="271" alt="image" src="https://github.com/user-attachments/assets/41a79eac-ad98-4c9b-89b2-9c0ff8b29304" />


➡ Dari output terlihat:

Root berpindah ke port Gi0/1 dengan path cost 8.

Port Gi0/2 masuk blocking.

Waktu recovery sekitar 15 detik (masih mengikuti timer STP klasik).

#### 🔹 D1 – Aktifkan RSTP
```bash
D1(config)# spanning-tree mode rapid-pvst
```
Observasi:

Setelah mode diubah ke RSTP, interface VLAN1 sempat down ±30 detik sebelum kembali up.

Hal ini terjadi karena RSTP masih kompatibel dengan STP, sehingga jika hanya satu switch yang diubah, tetap mengikuti timer STP klasik.

#### 🔹 Verifikasi di D1
```bash
D1# show spanning-tree
```
Output:

<img width="589" height="601" alt="image" src="https://github.com/user-attachments/assets/7f0e1eff-b618-4640-863f-3e2a6a0b541f" />


Protokol berubah menjadi rstp.

D1 tetap menjadi root bridge untuk VLAN 1 dan VLAN 2.

Semua port aktif di D1 berstatus Designated Forwarding.

#### 🔹 D2 dan A1 – Aktifkan RSTP
```bash
D2(config)# spanning-tree mode rapid-pvst
A1(config)# spanning-tree mode rapid-pvst
```
Observasi:

Setelah semua switch menggunakan RSTP, konvergensi menjadi jauh lebih cepat.

Contoh: interface VLAN1 di A1 hanya down selama <0.05 detik sebelum kembali up.

Inilah “rapid” dalam RSTP: recovery hampir instan dibanding STP klasik.

Analisis

STP klasik: butuh 15–30 detik untuk recovery setelah link berubah.

RSTP: recovery bisa <1 detik jika semua switch mendukung.

Backward compatibility: jika hanya sebagian switch menggunakan RSTP, maka tetap mengikuti timer STP.

Kesimpulan: RSTP mempercepat konvergensi, sangat penting untuk jaringan modern dengan banyak link redundant.
