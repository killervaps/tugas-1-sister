# Simulasi Time Synchronization (Lamport & Vector Clock)

Proyek ini merupakan implementasi simulasi sinkronisasi waktu dalam sistem terdistribusi menggunakan algoritma **Lamport Clock** dan **Vector Clock**. Simulasi dijalankan di lingkungan emulasi **GNS3** menggunakan *image* Docker `royyana/netics-pc:debi-latest`.

## Spesifikasi VM GNS3

Simulasi ini dijalankan pada GNS3 VM dengan spesifikasi sebagai berikut:

| Komponen | Spesifikasi |
| --- | --- |
| **Memory** | 4 GB |
| **Processors** | 1 |
| **Hard Disk 1 (SCSI)** | 20 GB |
| **Hard Disk 2 (SCSI)** | 1 TB |
| **Network Adapter 1** | Host-only |
| **Network Adapter 2** | NAT |

## Arsitektur Jaringan & Alokasi IP
<img width="983" height="656" alt="Screenshot 2025-12-18 105959" src="https://github.com/user-attachments/assets/249805d7-2401-4c2c-a9d7-6c20c5d8be11" />
Topologi terdiri dari 4 node komputer yang terhubung melalui sebuah Ethernet Switch ke NAT1.

### Konfigurasi IP Statis

Setiap node dikonfigurasi secara manual pada file `/etc/network/interfaces` dengan format berikut:

```bash
# Static config for eth0
auto eth0
iface eth0 inet static
	address 192.168.122.[x]
	netmask 255.255.255.0
	gateway 192.168.122.1
	up echo nameserver 8.8.8.8 > /etc/resolv.conf

```

### Tabel Pembagian IP & Peran Node

| Device | IP Address | `time_sync` Nodes | Role |
| --- | --- | --- | --- |
| **ds-computer-1** | 192.168.122.2 | Node A | Peer Node |
| **ds-computer-2** | 192.168.122.3 | Node B | Peer Node |
| **ds-computer-3** | 192.168.122.4 | Node C | Logger |
| **ds-computer-4** | 192.168.122.5 | Node D | Peer Node |

## Konfigurasi `run.bash`

Setiap node memiliki skrip eksekusi yang berbeda untuk mengatur *clock offset* dan target komunikasi.

### Node A

```bash
#!/bin/bash
python3 peer_node.py \
          --name A --listen 0.0.0.0 5000 \
          --peers A@192.168.122.2:5000 B@192.168.122.3:5001 D@192.168.122.5:5002 \
          --logger 192.168.122.4 9999 \
          --offset-ms 600 \
          --initiate-broadcast --msg "Hello from A"

```

### Node B

```bash
#!/bin/bash
python3 peer_node.py \
          --name B --listen 0.0.0.0 5001 \
          --peers A@192.168.122.2:5000 B@192.168.122.3:5001 D@192.168.122.5:5002 \
          --logger 192.168.122.4 9999 \
          --offset-ms -600

```

### Node D

```bash
#!/bin/bash
python3 peer_node.py \
          --name D --listen 0.0.0.0 5002 \
          --peers A@192.168.122.2:5000 B@192.168.122.3:5001 D@192.168.122.5:5002 \
          --logger 192.168.122.4 9999 \
          --offset-ms -600

```

## Langkah-Langkah Menjalankan Simulasi

1. **Akses Terminal Node:**
Buka 4 terminal (misalnya menggunakan WSL) dan hubungkan ke konsol node GNS3 via Telnet:
* **Node A:** `telnet 192.168.179.128 5001`
* **Node B:** `telnet 192.168.179.128 5004`
* **Node C:** `telnet 192.168.179.128 5005`
* **Node D:** `telnet 192.168.179.128 5007`


2. **Pindah ke Direktori Kerja:**
Pada masing-masing terminal, masuk ke folder proyek yang sesuai:
* `cd ds25/synchronization/time_sync/node_a` (dst untuk b, c, dan d).


3. **Jalankan Logger:**
Pada **Node C**, jalankan skrip logger terlebih dahulu:
```bash
bash run.bash

```


4. **Jalankan Peer Nodes:**
Jalankan skrip pada **Node A, B, dan D** secara bergantian:
```bash
bash run.bash

```
