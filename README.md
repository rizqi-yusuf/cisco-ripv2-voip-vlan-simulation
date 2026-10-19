<div align="center">
  <h1>🚀 Simulasi Jaringan Enterprise Ring: RIPv2, VoIP Multi-Site, Wireless & ACL</h1>
  <p>
    <b>Simulasi Cisco Packet Tracer tingkat lanjut dengan Ring Backbone 4 node, dynamic routing RIPv2, VoIP (CME), segmentasi VLAN, dan Access Control List (ACL).</b>
  </p>

  [🇬🇧 English](README.md) | **🇮🇩 Bahasa Indonesia**
</div>

<br />

## 📸 Topologi Jaringan

<img width="716" height="376" alt="Topologi Jaringan" src="https://github.com/user-attachments/assets/8389b79d-ffea-403b-8cab-0edd8a65aa92" />

## 📖 Gambaran Proyek
Repositori ini berisi panduan konfigurasi dan dokumentasi lengkap untuk topologi jaringan Enterprise skala menengah hingga besar. Backbone-nya dirancang dengan **Topologi Ring** pada 4 router untuk menjamin ketersediaan tinggi (*high availability*) dan redundansi. Jaringan ini mengintegrasikan layanan data center terpusat, segmentasi departemen (VLAN), jaringan wireless dinamis (DHCP), serta telepon IP lintas-site (VoIP) menggunakan Cisco CallManager Express (CME).

---

## 🚀 Memulai (Cara Penggunaan)

### Prasyarat
Untuk membuka dan menjalankan simulasi ini, kamu membutuhkan:
* **Cisco Packet Tracer** (disarankan versi 8.0 atau lebih baru)
* Pemahaman dasar tentang CLI Cisco IOS

### Instalasi & Eksekusi
1. **Buka Simulasi:**
   Jalankan Cisco Packet Tracer lalu buka file `.pkt` yang ada di repositori ini.
2. **Tunggu Konvergensi:**
   Beri waktu beberapa detik agar Spanning Tree Protocol (STP) memindahkan port switch ke status *forwarding* (panah hijau) dan RIPv2 selesai bertukar tabel routing.
3. **Jelajahi Jaringan:**
   Arahkan kursor ke perangkat untuk melihat alokasi IP-nya, buka tab CLI router untuk memeriksa konfigurasi, atau gunakan terminal PC untuk menjalankan perintah `ping` dan `tracert`.

---

## 💡 Arsitektur & Pilihan Desain (Alasan di Baliknya)
Untuk mendemonstrasikan prinsip network engineering kelas enterprise, beberapa pilihan desain berikut diterapkan:

1. **Backbone Topologi Ring untuk Redundansi:** Dengan menghubungkan 4 router dalam satu lingkaran (`200.20...` -> `200.30...` -> `200.40...` -> `200.50...`), jaringan mencapai ketersediaan tinggi. Jika satu kabel WAN terputus, dynamic routing otomatis menghitung ulang dan mengalihkan trafik ke sisi lain dari ring.
2. **RIPv2 daripada RIPv1:** RIPv2 dipilih karena mendukung **VLSM (Variable Length Subnet Masking)**. Ini penting karena arsitekturnya memadukan subnet `/30` untuk link WAN point-to-point (menghemat ruang IP) dan subnet `/24` untuk LAN lokal.
3. **Router-on-a-Stick (802.1Q Trunking):** Alih-alih memakai banyak interface fisik router untuk jaringan Voice, Data, dan Wireless, VLAN di-trunk ke satu interface Gigabit memakai sub-interface. Cara ini memangkas biaya perangkat dan kabel yang berantakan.
4. **DHCP Option 150 untuk VoIP:** IP Phone membutuhkan server TFTP untuk mengunduh konfigurasi dan firmware. Option 150 sengaja dikonfigurasi di DHCP pool untuk mengarahkan telepon ke IP router yang menjalankan `telephony-service`.
5. **VLAN Voice Khusus:** Memisahkan trafik suara (VLAN 50) dari trafik data mencegah *broadcast storm* dan membuka peluang penerapan QoS (*Quality of Service*) di masa depan untuk memprioritaskan paket suara dan mencegah *jitter* audio.

---

## 🗺️ Tabel Alokasi IP & Segmentasi

| Site / Zona | VLAN / Layanan | Subnet / Network | Deskripsi & Tujuan |
| :--- | :--- | :--- | :--- |
| **Site 1 (Kiri Atas)** | OB Premium / ACL | `192.50.33.0/24` | LAN Data dengan implementasi Access Control List. |
| **Site 2 (Kanan Atas)** | Server Center | `192.70.33.0/24` | Server terpusat (`Qyv.com` di `192.70.33.2`). |
| **Site 2 (Kanan Atas)** | Jaringan Printer | `192.33.60.0/24` | VLAN khusus untuk printer jaringan. |
| **Site 3 (Kiri Bawah)** | LAN Data | `192.30.33.0/24` | Jaringan workstation / PC kabel. |
| **Site 3 (Kiri Bawah)** | Wireless (DHCP-2) | `192.40.33.0/24` | DHCP pool untuk perangkat mobile (tablet/smartphone). |
| **Site 4 (Kanan Bawah)** | Voice (IP Phone) | `192.10.33.0/24` | Cisco CallManager Express (CME) untuk layanan VoIP. |
| **Site 4 (Kanan Bawah)** | Wireless (DHCP-1) | `192.20.33.0/24` | DHCP pool untuk perangkat mobile tambahan. |
| **Backbone WAN** | Serial Antar-Router | `200.X.X.0/30` | `200.20.20.0`, `200.30.30.0`, `200.40.40.0`, `200.50.50.0` |

---

## 📝 Konfigurasi Perangkat Lengkap

### BAGIAN A: KONFIGURASI ROUTER (RIPv2 & Layanan)

#### 1. Router Kiri Atas (OB Premium)
```text
enable
configure terminal
hostname R_TopLeft

! --- Interface Serial WAN ---
interface Serial0/3/1
 ip address 200.20.20.1 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 200.50.50.2 255.255.255.252
 no shutdown

! --- Interface LAN ---
interface GigabitEthernet0/1
 ip address 192.50.33.1 255.255.255.0
 no shutdown

! --- Dynamic Routing RIPv2 ---
router rip
 version 2
 network 192.50.33.0
 network 200.20.20.0
 network 200.50.50.0
 no auto-summary
```

#### 2. Router Kanan Atas (Server & Printer)
```text
enable
configure terminal
hostname R_TopRight

! --- Interface Serial WAN ---
interface Serial0/3/1
 ip address 200.20.20.2 255.255.255.252
 no shutdown

interface Serial0/3/0
 ip address 200.30.30.1 255.255.255.252
 no shutdown

! --- Sub-interface Inter-VLAN (Router-on-a-Stick) ---
interface GigabitEthernet0/0
 no shutdown

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.70.33.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.33.60.1 255.255.255.0

! --- Dynamic Routing RIPv2 ---
router rip
 version 2
 network 192.70.33.0
 network 192.33.60.0
 network 200.20.20.0
 network 200.30.30.0
 no auto-summary
```

#### 3. Router Kiri Bawah (LAN Data & DHCP Wireless)
```text
enable
configure terminal
hostname R_BotLeft

! --- Interface Serial WAN ---
interface Serial0/3/0
 ip address 200.50.50.1 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 200.40.40.2 255.255.255.252
 no shutdown

! --- Sub-interface Inter-VLAN ---
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.30.33.1 255.255.255.0

interface FastEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.40.33.1 255.255.255.0

! --- DHCP Server untuk Wireless (DHCP-2) ---
ip dhcp pool WIRELESS_DHCP_2
 network 192.40.33.0 255.255.255.0
 default-router 192.40.33.1
 dns-server 192.70.33.2

! --- Dynamic Routing RIPv2 ---
router rip
 version 2
 network 192.30.33.0
 network 192.40.33.0
 network 200.40.40.0
 network 200.50.50.0
 no auto-summary
```

#### 4. Router Kanan Bawah (Voice VoIP & DHCP Wireless)
```text
enable
configure terminal
hostname R_BotRight

! --- Interface Serial WAN ---
interface Serial0/3/0
 ip address 200.30.30.2 255.255.255.252
 no shutdown

interface Serial0/3/1
 ip address 200.40.40.1 255.255.255.252
 no shutdown

! --- Sub-interface Inter-VLAN ---
interface FastEthernet0/0
 no shutdown

interface FastEthernet0/0.50
 encapsulation dot1Q 50
 ip address 192.10.33.1 255.255.255.0

interface FastEthernet0/0.60
 encapsulation dot1Q 60
 ip address 192.20.33.1 255.255.255.0

! --- DHCP Server untuk IP Phone & Wireless ---
ip dhcp pool VOICE_POOL
 network 192.10.33.0 255.255.255.0
 default-router 192.10.33.1
 option 150 ip 192.10.33.1

ip dhcp pool WIRELESS_DHCP_1
 network 192.20.33.0 255.255.255.0
 default-router 192.20.33.1
 dns-server 192.70.33.2

! --- Cisco CallManager Express (Telephony Service) ---
telephony-service
 max-ephones 5
 max-dn 5
 ip source-address 192.10.33.1 port 2000
 auto assign 1 to 5

ephone-dn 1
 number 1001
ephone-dn 2
 number 1002
ephone-dn 3
 number 1003

! --- Dynamic Routing RIPv2 ---
router rip
 version 2
 network 192.10.33.0
 network 192.20.33.0
 network 200.30.30.0
 network 200.40.40.0
 no auto-summary
```

---

### BAGIAN B: KONFIGURASI SWITCH (Trunking & Access)

#### 1. Switch Kiri Atas (OB Premium)
```text
enable
configure terminal
hostname SW_TopLeft

! Port access untuk PC
interface FastEthernet0/1
 switchport mode access
 no shutdown
```

#### 2. Switch Kanan Atas (Server & Printer)
```text
enable
configure terminal
hostname SW_TopRight

vlan 10
 name SERVER
vlan 20
 name PRINTER

! Port trunk ke Router
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown

! Port access ke Printer (VLAN 20)
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 20
 no shutdown

! Port access ke Server (VLAN 10)
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 no shutdown
```

#### 3. Switch Kiri Bawah (Data & Wireless)
```text
enable
configure terminal
hostname SW_BotLeft

vlan 30
 name DATA
vlan 40
 name WIRELESS

! Port trunk ke Router
interface GigabitEthernet0/2
 switchport mode trunk
 no shutdown

! Port access ke Access Point (VLAN 40)
interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 40
 no shutdown

! Port access ke PC (VLAN 30)
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport access vlan 30
 no shutdown
```

#### 4. Switch Kanan Bawah (Voice & Wireless)
```text
enable
configure terminal
hostname SW_BotRight

vlan 50
 name VOICE
vlan 60
 name WIRELESS

! Port trunk ke Router
interface GigabitEthernet0/1
 switchport mode trunk
 no shutdown

! Port access ke Access Point (VLAN 60)
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 60
 no shutdown

! Port access ke IP Phone (Voice VLAN 50)
interface range FastEthernet0/1 - 3
 switchport mode access
 switchport voice vlan 50
 no shutdown
```

---

## 🧠 Konsep Inti & Penjelasan Teknis

Berikut rincian konsep jaringan utama yang membuat topologi ini berjalan mulus:

### 1. Dynamic Routing Protocol (RIPv2)
Routing Information Protocol versi 2 (RIPv2) dipakai untuk membagikan tabel routing antar 4 router secara otomatis. Dengan mengiklankan jaringan yang terhubung langsung (`network 192...` dan `network 200...`), router mempelajari jalur terbaik menuju subnet yang jauh secara dinamis. Karena backbone-nya berbentuk ring, jika satu jalur putus, RIPv2 otomatis menghitung ulang dan mengarahkan trafik ke arah sebaliknya.

### 2. Inter-VLAN Routing (802.1Q)
Agar VLAN yang berbeda (misalnya VLAN Server 10 dan VLAN Printer 20) bisa saling berkomunikasi, dipakai metode "Router-on-a-Stick". Satu interface fisik pada router dibagi menjadi beberapa sub-interface virtual (misalnya `GigabitEthernet0/0.10`). Setiap sub-interface memakai **enkapsulasi 802.1Q** untuk memberi tag trafik dengan VLAN ID masing-masing, dan berfungsi sebagai default gateway untuk VLAN tersebut.

### 3. Cisco CallManager Express (CME)
Router kanan bawah berperan sebagai server PBX terpusat untuk sistem telepon IP. Perintah `telephony-service` memungkinkan router menetapkan directory number (ekstensi seperti 1001, 1002) dan mengelola panggilan aktif. DHCP pool-nya menyertakan **Option 150**, yang memberi IP Phone alamat IP router CME agar bisa mengunduh file konfigurasi saat booting.

---

## 🛠️ Troubleshooting & Verifikasi

Untuk memastikan setup berjalan dengan benar, lakukan pengujian berikut:
* **Cek Tabel Routing:** Jalankan `show ip route` di privileged EXEC mode pada router mana pun. Kamu akan melihat beberapa rute bertanda `R` (dipelajari lewat RIP).
* **Uji Redundansi Ring:** Jalankan ping terus-menerus dari PC di Site 1 ke Server di Site 2 (`ping 192.70.33.2 -t`). Hapus salah satu kabel Serial yang menghubungkan router. Ping mungkin sempat timeout sebentar, tetapi akan otomatis pulih setelah RIP memperbarui tabel routing-nya.
* **Uji VoIP:** Buka GUI dua IP Phone yang berbeda. Kamu akan melihat nomor ekstensi terdaftar di pojok kanan atas layar telepon (misalnya `1001`). Tekan ekstensi telepon lainnya untuk memulai panggilan.
