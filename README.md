# PT Cahaya Timur — 3-Site VLAN & Static Routing

Studi kasus jaringan enterprise 3 site (hub & spoke) dengan VLSM, VLAN, trunking, multilayer switching, ROAS, dan static routing.

## Topologi

```
                 HQ Makassar
                /            \
               /              \
      Branch Manado      Branch Palu
```

- **HQ Makassar ↔ Branch Manado** — link WAN langsung
- **HQ Makassar ↔ Branch Palu** — link WAN langsung
- **Manado ↔ Palu** — TIDAK ada link langsung, traffic antar kedua branch ini wajib lewat HQ (hub & spoke)

## Perangkat per Site

| Site | Perangkat | Metode Inter-VLAN |
|---|---|---|
| HQ Makassar | Router + Multilayer Switch + 2x Access Switch | SVI (multilayer switching) |
| Branch Manado | Router + Access Switch | ROAS (router-on-a-stick) |
| Branch Palu | Router + Multilayer Switch + Access Switch | SVI (multilayer switching) |

**Catatan trunk:** Native VLAN pada seluruh trunk port (switch-to-switch maupun switch-to-router) diubah ke **VLAN 99** — bukan default VLAN 1.

## Pengalamatan IP (VLSM)

Blok dasar: `172.25.0.0/22`

### HQ Makassar

| VLAN | Divisi | Host | Network | Broadcast | Mask | Prefix |
|---|---|---|---|---|---|---|
| 30 | IT | 50 | 172.25.0.0 | 172.25.0.63 | 255.255.255.192 | /26 |
| 40 | Guest | 30 | 172.25.0.64 | 172.25.0.95 | 255.255.255.224 | /27 |
| 20 | Finance | 25 | 172.25.0.96 | 172.25.0.127 | 255.255.255.224 | /27 |
| 10 | Management | 10 | 172.25.0.128 | 172.25.0.143 | 255.255.255.240 | /28 |

### Branch Palu

| VLAN | Divisi | Host | Network | Broadcast | Mask | Prefix |
|---|---|---|---|---|---|---|
| 20 | Finance | 15 | 172.25.0.160 | 172.25.0.191 | 255.255.255.224 | /27 |
| 40 | Guest | 10 | 172.25.0.192 | 172.25.0.207 | 255.255.255.240 | /28 |
| 10 | Management | 5 | 172.25.0.208 | 172.25.0.215 | 255.255.255.248 | /29 |

### Branch Manado

| VLAN | Divisi | Host | Network | Broadcast | Mask | Prefix |
|---|---|---|---|---|---|---|
| 30 | IT | 20 | 172.25.1.0 | 172.25.1.31 | 255.255.255.224 | /27 |
| 40 | Guest | 15 | 172.25.1.32 | 172.25.1.63 | 255.255.255.224 | /27 |
| 10 | Management | 5 | 172.25.1.64 | 172.25.1.71 | 255.255.255.248 | /29 |

### Link WAN (point-to-point)

| Link | Network | Prefix | Usable |
|---|---|---|---|
| HQ – Manado | 172.25.2.0 | /30 | 172.25.2.1 – 172.25.2.2 |
| HQ – Palu | 172.25.2.4 | /30 | 172.25.2.5 – 172.25.2.6 |

## Task Implementasi

1. Konfigurasi VLAN, trunk (native VLAN 99), dan access port di ketiga site.
2. HQ & Palu: konfigurasi SVI untuk inter-VLAN routing di multilayer switch.
3. Manado: konfigurasi sub-interface ROAS di router untuk inter-VLAN routing.
4. Konfigurasi static route di ketiga router agar seluruh VLAN di seluruh site dapat saling ping — termasuk Manado ↔ Palu yang wajib lewat HQ sebagai next-hop.
5. Verifikasi akhir:
   - Ping antar-VLAN dalam satu site
   - Ping antar-site langsung (HQ ↔ Manado, HQ ↔ Palu)
   - Ping Manado ↔ Palu (melalui HQ)
   - `show ip route` di ketiga router menunjukkan rute static yang benar
