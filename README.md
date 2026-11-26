| Name                      | NRP        | Kelas    |
| ---                       | ---        | -------- |
| Arya Rangga Putra Pratama | 5025241072 | Jarkom A |

> [!IMPORTANT]
> Remember to use the IP prefix allocation provided earlier for this pre-lab assignment (pra-praktikan) and for future ones.
> Access it on [Linktree](https://linktr.ee/jarkom)
>
> **IP Prefix used:** `10.4` (jadi alokasi induk Bagian A adalah `10.4.20.0/24`, Tugas Mandiri `10.4.50.0/24`)

> [!TIP]
> **Due Date : `26th November 2026, 23.00 WIB`**

#### Put your GNS3 Project files here (gdrive or github upload)!

`Put file URL here`

---

## Bagian A - Perencanaan: VLSM, Interface, dan Routing Table

### Problem A.1 - Perhitungan VLSM untuk Seluruh Segmen

**Langkah Kerja:**

**Langkah 1 - Urutkan segmen dari kebutuhan host terbesar ke terkecil.**

| Urutan | Segmen | Kebutuhan |
| --- | --- | --- |
| 1 | IF-106 | ±50 |
| 2 | IF-105 | ±40 |
| 3 | Ruang Baca (cadangan) | ±35 |
| 4 | Ruang Dosen | ±10 |
| 5 | Backbone A | 6 router |
| 6 | Backbone B | 6 router |
| 7-9 | Link R1-R2, R2-R3, R3-R1 | 2 |

**Langkah 2 - Tentukan prefix minimum** dengan syarat host valid `2^(32-prefix) - 2 >= kebutuhan`.

| Segmen | Kebutuhan | Pilihan prefix | Host valid | Ukuran blok |
| --- | --- | --- | --- | --- |
| IF-106 | 50 | /26 (/27 hanya 30, kurang) | 2^6 - 2 = 62 | 64 |
| IF-105 | 40 | /26 | 62 | 64 |
| Ruang Baca | 35 | /26 (/27 hanya 30, kurang) | 62 | 64 |
| Ruang Dosen | 10 | /28 (/29 hanya 6, kurang) | 2^4 - 2 = 14 | 16 |
| Backbone A/B | 6 router | /29 | 2^3 - 2 = 6 | 8 |
| Link point-to-point | 2 | /30 | 2^2 - 2 = 2 | 4 |

**Langkah 3 - Alokasikan berurutan dari `10.4.20.0`, tanpa tumpang tindih.**

```
10.4.20.0/24
├─ 10.4.20.0/26    IF-106        (.0   - .63)
├─ 10.4.20.64/26   IF-105        (.64  - .127)
├─ 10.4.20.128/28  Ruang Dosen   (.128 - .143)
├─ 10.4.20.144/29  Backbone A    (.144 - .151)
├─ 10.4.20.152/29  Backbone B    (.152 - .159)
├─ 10.4.20.160/30  Link R1-R2    (.160 - .163)
├─ 10.4.20.164/30  Link R2-R3    (.164 - .167)
├─ 10.4.20.168/30  Link R3-R1    (.168 - .171)
├─ 10.4.20.172 - .191            sisa, tidak terpakai
└─ 10.4.20.192/26  Ruang Baca    (.192 - .255)  <- Langkah 4
```

**Langkah 4 - Blok Ruang Baca.** Setelah `.171`, ruang kosong mulai `.172`. Blok /26 berukuran 64, network address harus kelipatan 64 (0, 64, 128, 192). `.172` bukan kelipatan 64, jadi blok berikutnya yang valid adalah `10.4.20.192/26` (`.192 - .255`). Hasil ini cocok dengan Tabel Alokasi Acuan (R4 `.193`, Higashi `.200`).

**Answer:**

| No | Segmen | Kebutuhan Host | Prefix (/xx) | Network Address | Broadcast Address |
| --- | --- | --- | --- | --- | --- |
| 1 | IF-106 | ±50 | /26 | 10.4.20.0 | 10.4.20.63 |
| 2 | IF-105 | ±40 | /26 | 10.4.20.64 | 10.4.20.127 |
| 3 | Ruang Dosen | ±10 | /28 | 10.4.20.128 | 10.4.20.143 |
| 4 | Backbone A | hingga 6 router | /29 | 10.4.20.144 | 10.4.20.151 |
| 5 | Backbone B | hingga 6 router | /29 | 10.4.20.152 | 10.4.20.159 |
| 6 | Link R1–R2 | 2 | /30 | 10.4.20.160 | 10.4.20.163 |
| 7 | Link R2–R3 | 2 | /30 | 10.4.20.164 | 10.4.20.167 |
| 8 | Link R3–R1 | 2 | /30 | 10.4.20.168 | 10.4.20.171 |
| 9 | Ruang Baca (cadangan) | ±35 | /26 | 10.4.20.192 | 10.4.20.255 |

![Foto perhitungan VLSM tulisan tangan](img/a1-vlsm-tulisan-tangan.jpg)

### Problem A.2 - Rencana Interface dan Routing Table: Chain Topology

**Langkah Kerja:**

1. **ether1 = host pertama blok LAN** (menjadi default gateway host): R1 `10.4.20.129/28`, R2 `10.4.20.65/26`, R3 `10.4.20.1/26`.
2. **ether2 dan ether3 sesuai link**, host pertama diberikan ke router bernomor lebih kecil: Link R1-R2 (`.160/30`): R1 `.161`, R2 `.162`. Link R2-R3 (`.164/30`): R2 `.165`, R3 `.166`. R1 ether3 dan R3 ether3 belum dipakai (link R3-R1 baru dipasang di Bagian C).
3. **Directly connected** tiap router adalah subnet dari interface miliknya sendiri. Sisanya perlu static route dengan next hop router tetangga.
4. **Summarization:** pada R1, route IF-105 `10.4.20.64/26` dan IF-106 `10.4.20.0/26` keduanya lewat R2 (`.162`). Keduanya berurutan dan bisa digabung menjadi `10.4.20.0/25` (`.0 - .127`). Pada R3 tidak bisa, karena `10.4.20.64/26` dan `10.4.20.128/28` tidak berada dalam satu blok yang pas. Pada R2 tidak ada dua route dengan next hop sama.

**Answer - Tabel Alokasi Interface (rencana):**

| Router | ether1 (LAN) | ether2 | ether3 |
| --- | --- | --- | --- |
| R1 | 10.4.20.129/28 | 10.4.20.161/30 | - (not used yet) |
| R2 | 10.4.20.65/26 | 10.4.20.162/30 | 10.4.20.165/30 |
| R3 | 10.4.20.1/26 | 10.4.20.166/30 | - (not used yet) |

**Answer - Routing Table (rencana):**

| Router | Destination Network | Next Hop | Outgoing Interface | Metode |
| --- | --- | --- | --- | --- |
| R1 | Ruang Dosen 10.4.20.128/28 | - | ether1 | Terhubung Langsung |
| R1 | Link R1–R2 10.4.20.160/30 | - | ether2 | Terhubung Langsung |
| R1 | IF-105 10.4.20.64/26 | 10.4.20.162 | ether2 | Static (bisa di-summarize) |
| R1 | IF-106 10.4.20.0/26 | 10.4.20.162 | ether2 | Static (bisa di-summarize, summary `10.4.20.0/25`) |
| R2 | IF-105 10.4.20.64/26 | - | ether1 | Terhubung Langsung |
| R2 | Link R1–R2 10.4.20.160/30 | - | ether2 | Terhubung Langsung |
| R2 | Link R2–R3 10.4.20.164/30 | - | ether3 | Terhubung Langsung |
| R2 | Ruang Dosen 10.4.20.128/28 | 10.4.20.161 | ether2 | Static |
| R2 | IF-106 10.4.20.0/26 | 10.4.20.166 | ether3 | Static |
| R3 | IF-106 10.4.20.0/26 | - | ether1 | Terhubung Langsung |
| R3 | Link R2–R3 10.4.20.164/30 | - | ether2 | Terhubung Langsung |
| R3 | Ruang Dosen 10.4.20.128/28 | 10.4.20.165 | ether2 | Static |
| R3 | IF-105 10.4.20.64/26 | 10.4.20.165 | ether2 | Static (tidak bisa di-summarize) |

### Problem A.3 - Beban Konfigurasi Static Routing

**Langkah Kerja:**

Asumsi: tiap router melayani tepat satu LAN, dan yang dihitung adalah route menuju LAN router lain.

1. Satu router harus punya route ke `n - 1` LAN lain, jadi static route per router = `n - 1`. Total seluruh router = `n × (n - 1)`.
2. Jika tiap tujuan butuh satu backup route lewat jalur alternatif, jumlahnya dua kali lipat: `2(n - 1)` per router dan `2n(n - 1)` total.
3. Ruang baru berarti LAN baru yang harus dikenal semua router lama, jadi `n` router lama harus diubah (masing-masing +1 route, atau +2 dengan backup). Router baru juga harus dikonfigurasi dengan `n` route, sehingga total router yang disentuh `n + 1`.

**Answer:**

| Jumlah Router (n) | Static Route per Router | Total Tanpa Cadangan | Total Dengan Cadangan | Router yang Diubah saat Menambah 1 Ruang |
| --- | --- | --- | --- | --- |
| 3 | 2 | 6 | 12 | 3 router lama (+1 router baru = 4) |
| 4 | 3 | 12 | 24 | 4 router lama (+1 router baru = 5) |
| 10 | 9 | 90 | 180 | 10 router lama (+1 router baru = 11) |
| n | n − 1 | n(n − 1) | 2n(n − 1) | n router lama (+1 router baru = n + 1) |

Jumlah route tumbuh kuadratik (O(n²)) dan setiap penambahan ruang menyentuh semua router, sehingga static routing tidak skalabel. Ini yang menjadi alasan penggunaan dynamic routing (OSPF) di Bagian C dan D.

## Bagian B - Static Routing pada Chain Topology

### Tabel Alokasi Acuan

| Segmen | Blok VLSM (/xx) | Alamat IP yang Digunakan |
| --- | --- | --- |
| IF-106 | 10.4.20.0/26 | R3 (ether1): 10.4.20.1 <br> Kazdel (eth0): 10.4.20.10 |
| IF-105 | 10.4.20.64/26 | R2 (ether1): 10.4.20.65 <br> Yan (eth0): 10.4.20.70 |
| Ruang Dosen | 10.4.20.128/28 | R1 (ether1): 10.4.20.129 <br> Lungmen (eth0): 10.4.20.130 <br> Laterano (eth0): 10.4.20.131 |
| Backbone A | 10.4.20.144/29 | R1: 10.4.20.145 / R2: 10.4.20.146 / R3: 10.4.20.147 / R4: 10.4.20.148 (all ether4) |
| Backbone B | 10.4.20.152/29 | R1: 10.4.20.153 / R2: 10.4.20.154 / R3: 10.4.20.155 / R4: 10.4.20.156 (all ether5) |
| Link R1–R2 | 10.4.20.160/30 | R1 (ether2): 10.4.20.161 <br> R2 (ether2): 10.4.20.162 |
| Link R2–R3 | 10.4.20.164/30 | R2 (ether3): 10.4.20.165 <br> R3 (ether2): 10.4.20.166 |
| Link R3–R1 | 10.4.20.168/30 | R1 (ether3): 10.4.20.169 <br> R3 (ether3): 10.4.20.170 |
| Ruang Baca | 10.4.20.192/26 | R4 (ether1): 10.4.20.193 <br> Higashi (eth0): 10.4.20.200 |

### Problem B.1 - Membangun Chain Topology dan Mengamati Connected Route

**Langkah Kerja (berurutan):**

**Langkah 1 - Buat node di GNS3.**
- Tambahkan 3 node MikroTik CHR, beri nama R1, R2, R3. Klik kanan tiap node, **Configure → Network**, atur **Adapters = 5**.
- Tambahkan 3 Ethernet switch (Switch1, Switch2, Switch3) dan host: Lungmen (netics-pc), Laterano (netics-pc-desktop) di Switch1, Yan (netics-pc) di Switch2, Kazdel (netics-pc) di Switch3.

**Langkah 2 - Sambungkan kabel.**
- R1 ether2 ke R2 ether2, dan R2 ether3 ke R3 ether2.
- ether1 R1 ke Switch1 (e2), ether1 R2 ke Switch2 (e1), ether1 R3 ke Switch3 (e1).
- Lungmen dan Laterano ke Switch1 (e0 dan e1), Yan ke Switch2 (e0), Kazdel ke Switch3 (e0).

![Topologi Gambar B.1](img/b1-topologi.png)

**Langkah 3 - Start semua node, login router** (`admin`, password kosong atau Enter, lalu Ctrl+C jika diminta ganti password). Hapus DHCP client bawaan dan beri identitas:

```
# R1
/ip dhcp-client remove [find]
/system identity set name=R1
# R2
/ip dhcp-client remove [find]
/system identity set name=R2
# R3
/ip dhcp-client remove [find]
/system identity set name=R3
```

**Langkah 4 - Alamat interface router:**

```
# R1
/ip address
add address=10.4.20.129/28 interface=ether1 comment="LAN Ruang Dosen"
add address=10.4.20.161/30 interface=ether2 comment="link ke R2"

# R2
/ip address
add address=10.4.20.65/26 interface=ether1 comment="LAN IF-105"
add address=10.4.20.162/30 interface=ether2 comment="link ke R1"
add address=10.4.20.165/30 interface=ether3 comment="link ke R3"

# R3
/ip address
add address=10.4.20.1/26 interface=ether1 comment="LAN IF-106"
add address=10.4.20.166/30 interface=ether2 comment="link ke R2"
```

**Langkah 5 - Alamat host dan default gateway** (di console masing-masing host):

```
# Lungmen
ip addr add 10.4.20.130/28 dev eth0
ip route add default via 10.4.20.129
# Laterano
ip addr add 10.4.20.131/28 dev eth0
ip route add default via 10.4.20.129
# Yan
ip addr add 10.4.20.70/26 dev eth0
ip route add default via 10.4.20.65
# Kazdel
ip addr add 10.4.20.10/26 dev eth0
ip route add default via 10.4.20.1
```

**Langkah 6 - Tampilkan routing table setiap router** (`/ip route print`) lalu bandingkan dengan baris "Terhubung Langsung" di rencana A.2. Yang muncul hanya route berflag `DAc` (connected):
- R1: `10.4.20.128/28` (ether1), `10.4.20.160/30` (ether2)
- R2: `10.4.20.64/26` (ether1), `10.4.20.160/30` (ether2), `10.4.20.164/30` (ether3)
- R3: `10.4.20.0/26` (ether1), `10.4.20.164/30` (ether2)

Hasilnya sama dengan rencana A.2.

**Langkah 7 - Prediksi (ditulis sebelum ping):** Laterano **tidak dapat** menjangkau Yan. R1 tidak punya route ke `10.4.20.64/26`, sehingga paket ke Yan dibuang di R1. Ping ke R2 (`.162`) dari Laterano juga diprediksi gagal. Paket request sampai ke R2 karena `.160/30` terhubung langsung di R1, tetapi R2 tidak punya route kembali ke `10.4.20.128/28`, sehingga reply tidak pernah pulang. Ping dari R1 sendiri ke `.162` akan berhasil karena subnetnya terhubung langsung.

Verifikasi:

```
# Laterano
ping -c 3 10.4.20.162
ping -c 3 10.4.20.70
# R1
/ping 10.4.20.162 count=3
```

**Answer:**

| No | Parameter/Aspek | Field/Perintah | Hasil Observasi (screenshot) |
| --- | --- | --- | --- |
| 1 | Alamat interface R1, R2, R3 | `/ip address print` | ![](img/b1-ip-address-r1.png) ![](img/b1-ip-address-r2.png) ![](img/b1-ip-address-r3.png) |
| 2 | Routing table awal | `/ip route print` | Hanya route connected (`DAc`) seperti Langkah 6. ![](img/b1-route-r1.png) ![](img/b1-route-r2.png) ![](img/b1-route-r3.png) |
| 3 | Hasil ping ke R2 (.162) | `ping` | Dari Laterano: 100% packet loss (R2 tidak punya return route ke 10.4.20.128/28). Dari R1: berhasil. ![](img/b1-ping-r2.png) |

### Problem B.2 - Static Route Multi-Hop, Forward dan Return Path, dan Anatomi Datagram

**Langkah Kerja (berurutan):**

**Langkah 1 - Pasang static route pada R1 dan R2:**

```
# R1
/ip route
add dst-address=10.4.20.64/26 gateway=10.4.20.162 comment="IF-105 via R2"
add dst-address=10.4.20.0/26 gateway=10.4.20.162 comment="IF-106 via R2"

# R2
/ip route
add dst-address=10.4.20.128/28 gateway=10.4.20.161 comment="Ruang Dosen via R1"
add dst-address=10.4.20.0/26 gateway=10.4.20.166 comment="IF-106 via R3"
```

**Langkah 2 - Prediksi (ditulis sebelum ping):** R3 belum punya static route. Ping Laterano ke Kazdel **gagal**. Request berhasil melewati R1 dan R2 lalu sampai ke Kazdel lewat R3 (connected), tetapi reply Kazdel ke `10.4.20.131` berhenti di **R3**, karena R3 tidak punya route ke `10.4.20.128/28`. Traceroute dari R3 ke Laterano diprediksi tidak mendapat jawaban (tidak ada route).

```
# Laterano
ping -c 3 10.4.20.10
# R3
/tool traceroute 10.4.20.131
```

**Langkah 3 - Lengkapi static route R3, lalu ulangi pengujian:**

```
# R3
/ip route
add dst-address=10.4.20.128/28 gateway=10.4.20.165 comment="Ruang Dosen via R2"
add dst-address=10.4.20.64/26 gateway=10.4.20.165 comment="IF-105 via R2"
```

```
# Laterano
ping -c 3 10.4.20.10
```

**Langkah 4 - Verifikasi path.** Jika `mtr` tidak ada, pakai `traceroute`.

```
# Laterano
mtr -r -c 5 10.4.20.70
mtr -r -c 5 10.4.20.10
# Kazdel
mtr -r -c 5 10.4.20.131
```

Perkiraan hop:
- Laterano ke Yan: R1 (`.129`) → R2 → Yan (`.70`) = 3 hop.
- Laterano ke Kazdel: R1 → R2 → R3 → Kazdel (`.10`) = 4 hop.
- Kazdel ke Laterano: R3 (`.1`) → R2 → R1 → Laterano (`.131`) = 4 hop, arah sebaliknya.

(Alamat tiap hop ditulis sesuai output asli.)

**Langkah 5 - Mulai capture.** Klik kanan link R1–R2, **Start capture**. Ulangi pada link R2–R3 dan link Switch1–R1. Pada setiap jendela Wireshark, isi display filter `icmp`.

**Langkah 6 - Kirim 3 ICMP Echo Request dari Laterano:**

```
ping -c 3 10.4.20.10
```

Pada ketiga capture, buka satu paket Echo Request yang sama, lalu lihat bagian Internet Protocol Version 4. Hasil yang diharapkan:

| Capture | Time to Live | Protocol | Header Checksum |
| --- | --- | --- | --- |
| LAN Ruang Dosen (Switch1–R1) | 64 | ICMP (1) | nilai A |
| Link R1–R2 | 63 | ICMP (1) | nilai B (berbeda) |
| Link R2–R3 | 62 | ICMP (1) | nilai C (berbeda) |

TTL turun 1 di setiap router yang dilewati. Header Checksum ikut berubah di setiap hop karena TTL (bagian header) berubah. Protocol tetap ICMP. Nilai checksum diisi sesuai capture aktual.

**Answer:**

| No | Parameter/Aspek | Field/Perintah | Hasil Observasi (screenshot) |
| --- | --- | --- | --- |
| 1 | Prediksi ping ke Kazdel sebelum R3 dikonfigurasi | - | Gagal. Request sampai ke Kazdel, reply berhenti di R3 karena tidak ada route ke 10.4.20.128/28. |
| 2 | Hasil ping dan traceroute dari R3 | `ping`, `/tool traceroute` | ![](img/b2-ping-sebelum.png) ![](img/b2-traceroute-r3.png) |
| 3 | Routing table R1, R2, R3 lengkap | `/ip route print` | ![](img/b2-route-r1.png) ![](img/b2-route-r2.png) ![](img/b2-route-r3.png) |
| 4 | Path Ruang Dosen → IF-105 | `mtr` | 3 hop: R1, R2, Yan. ![](img/b2-mtr-yan.png) |
| 5 | Path Ruang Dosen → IF-106 | `mtr` | 4 hop: R1, R2, R3, Kazdel. ![](img/b2-mtr-kazdel.png) |
| 6 | Path IF-106 → Ruang Dosen | `mtr` | 4 hop: R3, R2, R1, Laterano. ![](img/b2-mtr-balik.png) |
| 7 | TTL pada LAN Ruang Dosen, link R1–R2, link R2–R3 | `ip.ttl` | 64, 63, 62. ![](img/b2-ttl-lan.png) ![](img/b2-ttl-r1r2.png) ![](img/b2-ttl-r2r3.png) |
| 8 | Protocol dan Header Checksum pada ketiga capture | `ip.proto`, `ip.checksum` | Protocol 1 (ICMP) di ketiganya, checksum berbeda di setiap hop. ![](img/b2-checksum.png) |

### Problem B.3 - Manipulasi Route: Summarization, Longest Prefix Match, dan Routing Loop

**Langkah Kerja - (a) Route Summarization:**

```
# R1
/ip route remove [find where comment~"via R2"]
/ip route add dst-address=10.4.20.0/25 gateway=10.4.20.162 comment="IF-105 dan IF-106 via R2 (summary)"
/ip route print
```

Verifikasi dari Laterano: `ping -c 3 10.4.20.70` dan `ping -c 3 10.4.20.10` tetap berhasil. Summary route ini dipertahankan untuk Bagian C.

**Langkah Kerja - (b) Longest Prefix Match:**

Prediksi (ditulis sebelum uji): Kazdel (`.10`) **tidak terjangkau**, karena `/32` blackhole lebih spesifik dari `/25`. R3 ether1 (`.1`) **terjangkau**, dan Yan (`.70`) **terjangkau**, karena keduanya tidak cocok dengan `/32` dan tetap mengikuti summary `/25`.

```
# R1
/ip route add dst-address=10.4.20.10/32 blackhole comment="uji LPM"
```

```
# Laterano
ping -c 3 10.4.20.10
ping -c 3 10.4.20.1
ping -c 3 10.4.20.70
```

Hapus route uji setelah pengamatan:

```
# R1
/ip route remove [find where comment="uji LPM"]
```

**Langkah Kerja - (c) Routing Loop:**

Prediksi (ditulis sebelum uji): datagram dari Laterano ke Kazdel akan berputar antara R1 dan R2. TTL berkurang 1 di setiap lompatan sampai habis, lalu router terakhir mengirim ICMP **Time to live exceeded** ke Laterano. Dengan TTL awal 64, TTL sampai 1 di R2, sehingga R2 (`10.4.20.162`) yang mengirim pesan itu.

Pastikan capture link R1–R2 masih berjalan, lalu:

```
# R2
/ip route set [find where comment="IF-106 via R3"] gateway=10.4.20.161
```

```
# Laterano
ping -c 2 10.4.20.10
```

Pada capture link R1–R2, Echo Request yang sama muncul berulang kali dengan TTL turun (63, 62, 61, ... 1) sampai muncul ICMP Time-to-live exceeded.

Kembalikan konfigurasi dan verifikasi:

```
# R2
/ip route set [find where comment="IF-106 via R3"] gateway=10.4.20.166
```

```
# Laterano
ping -c 3 10.4.20.10
```

**Answer:**

| No | Parameter/Aspek | Field/Perintah | Hasil Observasi (screenshot) |
| --- | --- | --- | --- |
| 1 | Routing table R1 setelah summarization | `/ip route print` | Dua route via R2 diganti satu route `10.4.20.0/25`. ![](img/b3-route-summary.png) |
| 2 | Konektivitas ke IF-105 dan IF-106 setelah summarization | `ping` | Yan dan Kazdel tetap terjangkau. ![](img/b3-ping-summary.png) |
| 3 | Prediksi dan hasil uji LPM (.10, .1, .70) | `ping` | .10 gagal (blackhole /32), .1 dan .70 berhasil. ![](img/b3-lpm.png) |
| 4 | Message yang diterima pengirim saat loop | `ping` | `Time to live exceeded` (dari R2, 10.4.20.162). ![](img/b3-loop-ping.png) |
| 5 | Nilai TTL Echo Request yang sama pada capture R1–R2 | `ip.ttl` | TTL turun berurutan 63, 62, 61, ... sampai 1. ![](img/b3-loop-ttl.png) |
| 6 | Konektivitas setelah konfigurasi dikembalikan | `ping` | Kazdel kembali terjangkau. ![](img/b3-restore.png) |

## Bagian C - Redundansi dan Dynamic Routing OSPF pada Topologi Ring

![Topologi Bagian C (Gambar C.1)](img/c-topologi.png)

### Problem C.1 - Redundant Link dengan Static Routing

#### Langkah Kerja - (a)

**Answer:**

Langkah: sambungkan kabel R1 ether3 ke R3 ether3 di GNS3, lalu beri alamat:

```
# R1
/ip address add address=10.4.20.169/30 interface=ether3 comment="link ke R3"
# R3
/ip address add address=10.4.20.170/30 interface=ether3 comment="link ke R1"
```

Pasang direct route di kedua sisi. Pada R1 route `/26` baru mengalahkan summary `/25` (longest prefix match). Pada R3 route lama ke Ruang Dosen berprefix sama, jadi distance dinaikkan agar menjadi backup:

```
# R1
/ip route add dst-address=10.4.20.0/26 gateway=10.4.20.170 comment="IF-106 langsung via R3"
# R3
/ip route set [find where comment="Ruang Dosen via R2"] distance=10
/ip route add dst-address=10.4.20.128/28 gateway=10.4.20.169 comment="Ruang Dosen langsung via R1"
```

![IP address R1 ether3 dan R3 ether3](img/c1a-ip.png)
![Static route R1 dan R3](img/c1a-route.png)

#### Langkah Kerja - (b)

**Answer:**

```
# Laterano
mtr -r -c 5 10.4.20.10
```

Hasil yang diharapkan: 3 hop (R1 → R3 → Kazdel), lebih pendek dari Problem B.2 yang 4 hop (R1 → R2 → R3 → Kazdel), karena R1 kini lewat link langsung R1–R3.

![mtr Laterano ke Kazdel](img/c1b-mtr.png)

#### Langkah Kerja - (c)

**Answer:**

Prediksi (ditulis sebelum uji): dengan direct route R3 dimatikan, arah pergi Laterano → Kazdel tetap lewat R1 → R3 (3 hop), tetapi arah balik Kazdel → Laterano dari R3 jatuh ke backup via R2 (R3 → R2 → R1, 4 hop). Path menjadi asimetris.

```
# R3
/ip route disable [find where comment="Ruang Dosen langsung via R1"]
```

```
# Laterano
mtr -r -c 5 10.4.20.10
# Kazdel
mtr -r -c 5 10.4.20.131
```

Setelah selesai, aktifkan kembali:

```
# R3
/ip route enable [find where comment="Ruang Dosen langsung via R1"]
```

![mtr dari Laterano ke Kazdel](img/c1c-mtr-pergi.png)
![mtr dari Kazdel ke Laterano](img/c1c-mtr-balik.png)

#### Langkah Kerja - (d)

**Answer:**

Prediksi (ditulis sebelum uji): traffic **tidak** otomatis pindah ke R2. Suspend link di GNS3 hanya membuang paket, interface ether3 R1 tetap berstatus running, sehingga route direct tetap aktif dan ping terus gagal (blackhole). Ini keterbatasan static routing.

```
# Laterano (jalankan dulu, lalu klik kanan link R1-R3 di GNS3 > Suspend)
ping -O 10.4.20.10
```

Selama link suspend, periksa di R1:

```
/interface print where name=ether3
/ip route print where dst-address=10.4.20.0/26
```

Hasil yang diharapkan: ping menampilkan `no answer yet`, ether3 masih berflag `R` (running), dan route `10.4.20.0/26 via 10.4.20.170` masih aktif.

![ping -O saat suspend](img/c1d-ping.png)
![interface print ether3 di R1](img/c1d-interface.png)
![route print R1](img/c1d-route.png)

#### Langkah Kerja - (e)

**Answer:**

Pilih **Resume** pada link R1–R3, lalu aktifkan gateway check di kedua direct route:

```
# R1
/ip route set [find where comment="IF-106 langsung via R3"] check-gateway=ping
# R3
/ip route set [find where comment="Ruang Dosen langsung via R1"] check-gateway=ping
```

Ulangi uji link failure (`ping -O 10.4.20.10` dari Laterano, Suspend link R1–R3). Kali ini route direct ditandai unreachable setelah gateway `.170` tidak membalas ping, sehingga R1 jatuh ke summary `/25` via R2 dan R3 jatuh ke backup via R2. Catat lamanya ping terputus (perkiraan 10-30 detik, tulis sesuai hitungan aktual dari nomor icmp_seq yang hilang).

![check-gateway=ping di R1 dan R3](img/c1e-check-gateway.png)
![Durasi downtime](img/c1e-downtime.png)

### Problem C.2 - OSPF pada Topologi Ring

#### Langkah Kerja - (a)

**Answer:**

Resume link R1–R3 terlebih dahulu. Di R1, R2, dan R3:

```
/ip route remove [find where static]
/ip route print
```

Routing table hanya berisi route connected.

![Routing table hanya connected](img/c2a-route.png)

#### Langkah Kerja - (b)

**Answer:**

```
# R1
/routing ospf instance add name=ospf1 version=2 router-id=1.1.1.1
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
/routing ospf interface-template
add area=backbone interfaces=ether2,ether3 type=ptp comment="link inter-router"
add area=backbone interfaces=ether1 passive comment="LAN"

# R2 (sama, router-id=2.2.2.2)
/routing ospf instance add name=ospf1 version=2 router-id=2.2.2.2
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
/routing ospf interface-template
add area=backbone interfaces=ether2,ether3 type=ptp comment="link inter-router"
add area=backbone interfaces=ether1 passive comment="LAN"

# R3 (sama, router-id=3.3.3.3)
/routing ospf instance add name=ospf1 version=2 router-id=3.3.3.3
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
/routing ospf interface-template
add area=backbone interfaces=ether2,ether3 type=ptp comment="link inter-router"
add area=backbone interfaces=ether1 passive comment="LAN"
```

![OSPF R1](img/c2b-r1.png)
![OSPF R2](img/c2b-r2.png)
![OSPF R3](img/c2b-r3.png)

#### Langkah Kerja - (c)

**Answer:**

```
/routing ospf neighbor print
/routing ospf interface print
/ip route print where ospf
```

Hasil yang diharapkan: tiap router punya 2 neighbor berstatus `Full`, interface ether2/ether3 bertipe `ptp`, ether1 `passive`, dan route berflag `o` (distance 110) menuju semua LAN dan link yang tidak terhubung langsung.

![Neighbor](img/c2c-neighbor.png)
![Interface](img/c2c-interface.png)
![Route OSPF](img/c2c-route.png)

#### Langkah Kerja - (d)

**Answer:**

Prediksi (ditulis sebelum uji): traffic **pulih otomatis** lewat R2 tanpa konfigurasi tambahan, tetapi tidak instan. Suspend di GNS3 tidak membuat interface down, sehingga OSPF baru menyadarinya setelah dead interval (default 40 detik) tanpa Hello dari R3.

```
# Laterano (jalankan dulu, lalu Suspend link R1-R3)
ping -O 10.4.20.10
```

Selama suspend, di R1:

```
/ip route print where dst-address=10.4.20.0/26
/routing ospf neighbor print
```

Hasil yang diharapkan: neighbor R3 hilang dari daftar setelah dead interval dan route `10.4.20.0/26` berpindah ke gateway R2 (`10.4.20.162`, ether2). Catat durasi ping terputus dari nomor icmp_seq yang hilang (perkiraan 30-40 detik).

Lalu **Resume** link R1–R3. Neighbor R3 kembali `Full` setelah Hello berikutnya, dan route `10.4.20.0/26` kembali lewat R3 langsung (cost lebih kecil), terlihat dari `mtr -r -c 5 10.4.20.10` yang kembali 3 hop.

![Ping berkelanjutan dan durasi pemulihan](img/c2d-ping.png)
![Routing table dan neighbor R1 saat suspend](img/c2d-suspend.png)
![Setelah Resume](img/c2d-resume.png)

## Bagian D - Backbone Ganda dan Skalabilitas Dynamic Routing

![Topologi Bagian D (Gambar D.1)](img/d-topologi.png)

### Problem D.1 - Migrasi ke Backbone Ganda

#### Langkah Kerja - (a)

**Answer:**

Tambahkan dua Ethernet switch: SW-BackboneA dan SW-BackboneB. Sambungkan ether4 R1, R2, R3 ke SW-BackboneA, dan ether5 R1, R2, R3 ke SW-BackboneB.

![Topologi setelah Backbone A dan B](img/d1a-topologi.png)

#### Langkah Kerja - (b)

**Answer:**

```
# R1
/interface disable ether2,ether3
/ip address
add address=10.4.20.145/29 interface=ether4 comment="Backbone A"
add address=10.4.20.153/29 interface=ether5 comment="Backbone B"
/routing ospf interface-template
add area=backbone interfaces=ether4,ether5 comment="backbone ganda"

# R2
/interface disable ether2,ether3
/ip address
add address=10.4.20.146/29 interface=ether4 comment="Backbone A"
add address=10.4.20.154/29 interface=ether5 comment="Backbone B"
/routing ospf interface-template
add area=backbone interfaces=ether4,ether5 comment="backbone ganda"

# R3
/interface disable ether2,ether3
/ip address
add address=10.4.20.147/29 interface=ether4 comment="Backbone A"
add address=10.4.20.155/29 interface=ether5 comment="Backbone B"
/routing ospf interface-template
add area=backbone interfaces=ether4,ether5 comment="backbone ganda"
```

![Disable ether2,ether3](img/d1b-disable.png)
![Alamat backbone](img/d1b-address.png)
![Interface-template backbone](img/d1b-template.png)

#### Langkah Kerja - (c)

**Answer:**

```
/routing ospf neighbor print
/routing ospf interface print
```

Prediksi (ditulis sebelum uji): semua router berprioritas sama (128), jadi DR dipilih dari router-id tertinggi: **R3 = DR, R2 = BDR, R1 = DROther** di Backbone A dan B. Catatan: pemilihan DR tidak preemptive, jadi router yang lebih dulu aktif bisa menjadi DR walau router-id-nya lebih kecil. Tulis peran aktual hasil `/routing ospf interface print` pada tabel di bawah.

| Segmen | DR | BDR |
| --- | --- | --- |
| Backbone A | (isi dari hasil aktual) | (isi dari hasil aktual) |
| Backbone B | (isi dari hasil aktual) | (isi dari hasil aktual) |

![Neighbor](img/d1c-neighbor.png)
![Peran DR/BDR](img/d1c-interface.png)

#### Langkah Kerja - (d)

**Answer:**

Prediksi (ditulis sebelum uji): R1 punya **2 gateway** menuju IF-106 (`10.4.20.0/26`), yaitu `10.4.20.147` (R3 di Backbone A) dan `10.4.20.155` (R3 di Backbone B), keduanya cost sama (ECMP).

```
# R1
/ip route print detail where dst-address=10.4.20.0/26
```

![Route R1 dengan dua gateway](img/d1d-route.png)

#### Langkah Kerja - (e)

**Answer:**

```
# Laterano
ping -O 10.4.20.10
```

Saat ping berjalan, di GNS3 klik kanan link antara R3 dan SW-BackboneA, **Suspend**. Amati di R1 (`/ip route print detail where dst-address=10.4.20.0/26`): gateway ECMP turun dari 2 menjadi 1 (tinggal `10.4.20.155` via Backbone B) setelah adjacency di Backbone A timeout (dead interval 40 detik). Downtime mungkin hingga sekitar 40 detik jika flow ping kebetulan memakai Backbone A, dan tidak ada jika flow memakai Backbone B. Catat durasi aktual, lalu **Resume** link.

![ping saat suspend](img/d1e-ping.png)
![Gateway R1 berubah](img/d1e-route.png)

#### Langkah Kerja - (f)

**Answer:**

```
# R1
/routing ospf interface-template set [find where comment="backbone ganda"] priority=0
/interface disable ether4,ether5
/interface enable ether4,ether5
/routing ospf interface print
```

Hasil yang diharapkan: dengan priority 0, R1 tidak boleh menjadi DR maupun BDR, sehingga setelah re-election perannya menjadi DROther. Jika sebelumnya R1 adalah DR atau BDR, perannya berubah. Jika sebelumnya sudah DROther, tidak ada perubahan peran (tulis sesuai hasil aktual).

![Priority 0 di R1](img/d1f-priority.png)
![Disable/enable interface](img/d1f-toggle.png)
![Peran R1 sebelum dan sesudah](img/d1f-role.png)

### Problem D.2 - Penambahan Ruang Baca melalui R4

#### Langkah Kerja - (a)

**Answer:**

Jika memakai static routing dengan backup via backbone kedua:
- R1, R2, R3 masing-masing butuh route ke `10.4.20.192/26`: 1 primary (via R4 di Backbone A, `.148`) + 1 backup (via Backbone B, `.156`) = 2 route per router, total **3 × 2 = 6 route**.
- R4 harus punya route ke 3 LAN lama (Ruang Dosen, IF-105, IF-106), masing-masing primary + backup = 3 × 2 = **6 route**.
- Total **12 static route**, tersebar di **keempat router (R1, R2, R3, R4)**.

Dengan OSPF: 0 route tambahan di R1-R3, cukup konfigurasi R4 saja.

#### Langkah Kerja - (b)

**Answer:**

Tambahkan di GNS3: node R4 (MikroTik CHR, 5 adapter), Switch4, dan Higashi (netics-pc). Sambungkan R4 ether1 ke Switch4 (e1), Higashi eth0 ke Switch4 (e0), R4 ether4 ke SW-BackboneA, R4 ether5 ke SW-BackboneB.

```
# R4
/ip dhcp-client remove [find]
/system identity set name=R4
/ip address
add address=10.4.20.193/26 interface=ether1 comment="LAN Ruang Baca"
add address=10.4.20.148/29 interface=ether4 comment="Backbone A"
add address=10.4.20.156/29 interface=ether5 comment="Backbone B"
/routing ospf instance add name=ospf1 version=2 router-id=4.4.4.4
/routing ospf area add name=backbone area-id=0.0.0.0 instance=ospf1
/routing ospf interface-template
add area=backbone interfaces=ether4,ether5 comment="backbone ganda"
add area=backbone interfaces=ether1 passive comment="LAN"
```

```
# Higashi
ip addr add 10.4.20.200/26 dev eth0
ip route add default via 10.4.20.193
```

![Topologi dengan R4, Switch4, Higashi](img/d2b-topologi.png)
![Alamat R4](img/d2b-address.png)
![OSPF R4](img/d2b-ospf.png)
![Higashi](img/d2b-higashi.png)

#### Langkah Kerja - (c)

**Answer:**

Tanpa menyentuh konfigurasi R1:

```
# R1
/ip route print where dst-address=10.4.20.192/26
# Laterano
mtr -r -c 5 10.4.20.200
```

Hasil yang diharapkan: R1 sudah mempelajari `10.4.20.192/26` via OSPF dengan dua gateway (`10.4.20.148` dan `10.4.20.156`). mtr dari Laterano ke Higashi 3 hop: R1 (`.129`) → R4 (`.148` atau `.156`) → Higashi (`.200`).

![Route R1 ke Ruang Baca](img/d2c-route.png)
![mtr Laterano ke Higashi](img/d2c-mtr.png)

#### Langkah Kerja - (d)

**Answer:**

```
# jalankan di masing-masing router
/export file=case4-R1
/export file=case4-R2
/export file=case4-R3
/export file=case4-R4
```

Download tiap file dari router (WinBox → Files, atau SFTP seperti Modul 4 bagian 0.5), lalu lampirkan ke repository atau drive bersama GNS3 project.

```
Lampiran: case4-R1.rsc, case4-R2.rsc, case4-R3.rsc, case4-R4.rsc
```

## Tugas Mandiri

Asumsi penempatan: RA melayani Lab Jaringan, RB melayani Lab Sistem Cerdas, RC melayani Ruang Server (sesuai urutan di soal). Kebutuhan dihitung sebagai host + 1 alamat router. Ruang Server: 6 host + 1 gateway = 7 alamat, tidak muat di /29 (6 host valid), jadi dipakai /28.

**Gambar Perencanaan VLSM - VLSM Planning Diagram**

```
10.4.50.0/24
├─ 10.4.50.0/26    Lab Jaringan        (.0   - .63)
├─ 10.4.50.64/27   Lab Sistem Cerdas   (.64  - .95)
├─ 10.4.50.96/28   Ruang Server        (.96  - .111)
├─ 10.4.50.112/30  Link RA-RB          (.112 - .115)
├─ 10.4.50.116/30  Link RB-RC          (.116 - .119)
└─ 10.4.50.120 - .255                  sisa, tidak terpakai
```

![Gambar perencanaan VLSM](img/tm-vlsm.jpg)

**Answer - Tabel Perhitungan VLSM:**

| No | Parameter | Lab Jaringan (±60) | Lab Sistem Cerdas (±25) | Ruang Server (±6) | Link RA–RB | Link RB–RC |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Prefix | /26 | /27 | /28 | /30 | /30 |
| 2 | Network Address | 10.4.50.0 | 10.4.50.64 | 10.4.50.96 | 10.4.50.112 | 10.4.50.116 |
| 3 | Broadcast Address | 10.4.50.63 | 10.4.50.95 | 10.4.50.111 | 10.4.50.115 | 10.4.50.119 |
| 4 | Rentang Host Valid | 10.4.50.1 - 10.4.50.62 | 10.4.50.65 - 10.4.50.94 | 10.4.50.97 - 10.4.50.110 | 10.4.50.113 - 10.4.50.114 | 10.4.50.117 - 10.4.50.118 |

**Answer - Soal 2: Rencana alamat interface** (host pertama link diberikan ke router bernomor lebih kecil):

| Router | ether1 (LAN) | ether2 | ether3 |
| --- | --- | --- | --- |
| RA | 10.4.50.1/26 | 10.4.50.113/30 (link ke RB) | - |
| RB | 10.4.50.65/27 | 10.4.50.114/30 (link ke RA) | 10.4.50.117/30 (link ke RC) |
| RC | 10.4.50.97/28 | 10.4.50.118/30 (link ke RB) | - |

**Answer - Soal 3: Routing table:**

| Router | Destination Network | Next Hop | Outgoing Interface | Metode |
| --- | --- | --- | --- | --- |
| RA | Lab Jaringan 10.4.50.0/26 | - | ether1 | Terhubung Langsung |
| RA | Link RA–RB 10.4.50.112/30 | - | ether2 | Terhubung Langsung |
| RA | Lab Sistem Cerdas 10.4.50.64/27 | 10.4.50.114 | ether2 | Static (bisa di-summarize) |
| RA | Ruang Server 10.4.50.96/28 | 10.4.50.114 | ether2 | Static (bisa di-summarize) |
| RA | Link RB–RC 10.4.50.116/30 | 10.4.50.114 | ether2 | Static (bisa di-summarize) |
| RB | Lab Sistem Cerdas 10.4.50.64/27 | - | ether1 | Terhubung Langsung |
| RB | Link RA–RB 10.4.50.112/30 | - | ether2 | Terhubung Langsung |
| RB | Link RB–RC 10.4.50.116/30 | - | ether3 | Terhubung Langsung |
| RB | Lab Jaringan 10.4.50.0/26 | 10.4.50.113 | ether2 | Static |
| RB | Ruang Server 10.4.50.96/28 | 10.4.50.118 | ether3 | Static |
| RC | Ruang Server 10.4.50.96/28 | - | ether1 | Terhubung Langsung |
| RC | Link RB–RC 10.4.50.116/30 | - | ether2 | Terhubung Langsung |
| RC | Lab Jaringan 10.4.50.0/26 | 10.4.50.117 | ether2 | Static (bisa di-summarize) |
| RC | Lab Sistem Cerdas 10.4.50.64/27 | 10.4.50.117 | ether2 | Static (bisa di-summarize) |
| RC | Link RA–RB 10.4.50.112/30 | 10.4.50.117 | ether2 | Static (bisa di-summarize) |

Route yang dapat di-summarize:
- **RA:** tiga route via `10.4.50.114` (`.64/27`, `.96/28`, `.116/30`) menjadi satu `10.4.50.64/26` (`.64 - .127`) via `10.4.50.114`. Link RA–RB `10.4.50.112/30` yang terhubung langsung ke RA tetap menang karena prefix lebih panjang (longest prefix match).
- **RC:** tiga route via `10.4.50.117` (`.0/26`, `.64/27`, `.112/30`) menjadi satu `10.4.50.0/25` (`.0 - .127`) via `10.4.50.117`. Ruang Server `10.4.50.96/28` dan link RB–RC yang terhubung langsung ke RC tetap menang lewat longest prefix match.
- **RB:** tidak ada yang bisa di-summarize karena dua static route-nya menuju arah yang berbeda (RA dan RC).

**Answer - Soal 4 (backup route via RC–RA):**

```
Link RC-RA ditambahkan sebagai jalur backup (subnet baru, misalnya 10.4.50.120/30).
Ada 3 LAN (Lab Jaringan, Lab Sistem Cerdas, Ruang Server). Setiap router punya
2 LAN tujuan jauh, dan masing-masing butuh 1 backup route lewat jalur alternatif:

- RA: 2 backup route (ke LAN RB dan LAN RC, lewat RC)
- RB: 2 backup route (ke LAN RA lewat RC, dan ke LAN RC lewat RA)
- RC: 2 backup route (ke LAN RA dan LAN RB, lewat RA)

Total tambahan = 3 router x 2 = 6 static route (sesuai selisih rumus Problem A.3:
2n(n-1) - n(n-1) = n(n-1) = 6 untuk n = 3). Backup route diberi distance lebih besar
(misalnya 10) agar hanya dipakai saat route utama gagal.

Alasan memilih dynamic routing (OSPF): static route tidak mendeteksi link putus
dengan sendirinya. Seperti hasil Problem C.1, traffic tetap dikirim ke link yang mati
(interface masih running) kecuali ditambah check-gateway dengan konfigurasi manual
di setiap route. Static juga berarti setiap penambahan subnet menyentuh semua router
dan jumlah route tumbuh O(n^2). Dengan OSPF, setiap router hanya meng-advertise
interface miliknya sendiri, jalur utama dan backup dihitung otomatis (berdasarkan cost),
dan failover terjadi tanpa campur tangan admin (Problem C.2). Pada topologi ring ini
OSPF membuat 6 route tambahan tersebut tidak perlu ditulis sama sekali.
```

AJK
