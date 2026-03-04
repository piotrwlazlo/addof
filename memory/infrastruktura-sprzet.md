# Infrastruktura Sprzętowa

## Serwery Compute (GPU)

### Fine-Tuning: 2x Dell XE9680 (MAIN)
| Parametr | Wartość |
|----------|---------|
| **CPU** | 2x Intel Xeon Gold 6548Y+ 2.5GHz, 32C |
| **RAM** | 2TB DRAM (64GB x 32) |
| **Dyski lokalne** | 4x 3.84TB NVMe + BOSS 2x 960GB |
| **GPU** | NVIDIA HGX H200 8-GPU SXM 141GB, 700W |
| **Frontend NIC** | Broadcom 57504 Quad Port 10/25GbE SFP28 |
| **Storage Fabric NIC** | Mellanox ConnectX-6 DX Dual Port 100GbE QSFP56 |
| **Backend NIC (tuning)** | 8x NVIDIA ConnectX-7 Single Port NDR 400GbE OSFP PCIe |
| **Zasilanie** | 3+3 FTR, 2800W Titanium (200-240Vac) |
| **Transceivers (backend)** | 8x NVIDIA 400Gbps NDR OSFP MPO12 APC 850nm MMF (do 50m) |
| **Wsparcie** | ProSupport Plus 4Hr Mission Critical, 36 mies. |

**Uwagi:**
- GPU Power Brake Enabled
- H200 SXM = najwydajniejsza wersja (SXM socket, NVSwitch fabric wewnątrz serwera)
- 8 GPU x 141GB HBM3e = ~1.1TB pamięci GPU na serwer

### Inferencing / RAG: 2x Dell XE7745 (MAIN) + 2x Dell XE7745 (DR)
| Parametr | Wartość |
|----------|---------|
| **CPU** | 2x AMD EPYC 9455 3.15GHz, 48C/96T |
| **RAM** | 1.5TB DRAM (24x 64GB DDR5 6400MT/s) |
| **Dyski lokalne** | 4x 3.84TB NVMe + BOSS 2x 480GB |
| **GPU** | 8x NVIDIA H200 NVL PCIe 141GB, 450-600W |
| **NVLINK** | 2x 4-way NVLINK Bridges (8 GPU) |
| **Frontend NIC** | Broadcom 57504 Quad Port 10/25GbE SFP28 |
| **Storage Fabric NIC** | Mellanox ConnectX-6 DX Dual Port 100GbE QSFP56 |
| **Backend NIC** | 4x NVIDIA ConnectX-6 DX Dual Port 100GbE QSFP56 |
| **Zasilanie** | 8x PSU, 3200W Titanium (230-240Vac) |
| **Licencje** | 8x NVAIE per serwer |
| **Wsparcie** | ProSupport Plus 4Hr Mission Critical, 36 mies. |

**Uwagi:**
- H200 NVL = wersja PCIe z NVLink bridge (nie SXM)
- Na DR te same serwery służą do fine-tuningu i inference
- NVAIE licencje WYMAGANE jeśli Open Innovation nie jest proponowane

## Serwery CPU (Wirtualizacja / Control Plane) - Dell PowerEdge R570

### Specyfikacja potwierdzona (2026-02-24)
| Parametr | Wartość |
|----------|---------|
| **Model** | Dell PowerEdge R570, Enterprise |
| **CPU** | Intel Xeon 6 Performance 6731P 2.5GHz, 32C/64T, 144MB Cache, 245W, DDR5-6400 |
| **RAM** | 512GB (8x 64GB RDIMM, 6400MT/s, Dual Rank) |
| **Dyski** | 3x 3.84TB NVMe Read Intensive E3.S Gen5 (EDSFF E3.S chassis, max 8 drives) |
| **BOSS** | M.2 Interposer card with 2x M.2 480GB (22x80) |
| **RAID** | C30, No RAID for NVMe chassis |
| **Riser** | Riser Config 3, Front Full Height 2x16 FH (Gen5) |
| **Zasilanie** | Dual Redundant (1+1) Hot-Plug MHS PSU, 1500W MM Titanium (100-240Vac) |
| **Zarządzanie** | iDRAC10 Enterprise 17G, OpenManage Enterprise Advanced |
| **Bezpieczeństwo** | Secure Enterprise Key Manager License 3.0, Secured Component Verification |
| **BIOS** | Power Saving BIOS Settings, UEFI Boot Mode with GPT |
| **NIC** | Legacy Password, No LOM, No OCP (do potwierdzenia - brak OCP NIC!) |
| **Szyny** | PowerEdge Blind Mate Rail (B35) |

- 6 serwerów fizycznych (3 MAIN + 3 DR)
- Hostują: Harvester (wirtualizacja), Rancher Management, Master nodes RKE2, Open Innovation components
- Wymagania na VM dla Rancher Manager: 3 VM/site, 8 core, 64GB RAM, 150GB SSD

> **⚠️ UWAGA (2026-02-24):** OICM zgłosił, że 32 rdzenie w R570 mogą być niewystarczające - chcą 64 rdzeni. Trwa weryfikacja możliwości zwiększenia CPU (np. Xeon 6 z 64C). Decyzja oczekiwana.
>
> **⚠️ UWAGA:** Specyfikacja nie zawiera karty sieciowej OCP - do potwierdzenia jak R570 łączy się z siecią (tylko iDRAC?)

## Networking

### MAIN Site - Switche
| Switch | Rola | Porty |
|--------|------|-------|
| **1x Z9664F-ON** | Backend Fabric (Tuning) | 64x 400GbE QSFP56-DD |
| **2x S5232F-ON** | Storage Fabric | 32x 100GbE QSFP28 |
| **2x S5232F-ON** | Inferencing Fabric | 32x 100GbE QSFP28 |

Łącznie: 5 switchy na MAIN + switche PowerScale backend

### DR Site - Switche
| Switch | Rola | Porty |
|--------|------|-------|
| **2x S5232F-ON** | Consolidated Fabric | 32x 100GbE QSFP28 |

Łącznie: 2 switche na DR + switche PowerScale backend

**Ważne uwagi dotyczące GPU i switchy:**
- GPU NIE wspierają port-channelingu ani HA - każdy GPU łączy się jednym portem do switcha
- Nie ma możliwości podłączenia GPU do dwóch switchy jednocześnie
- Opcja HA: 2 switche, po 4 GPU do każdego (częściowa komunikacja przy awarii switcha)
- Wszystkie switche: Enterprise SONIC Distribution

## Storage

### PowerScale (Dane Niestrukturalne - NAS) - specyfikacja potwierdzona 2026-02-24

**MAIN:** 3x PowerScale F710
- ~39TB raw per node (10x 3.84TB ISE NVMe SSD)
- 2x 2.6 GHz Processor per node
- 512GB RAM (5600 MT/s DIMM) per node
- FE: 2x 100GbE w/o Optics per node
- BE: 2x 100GbE w/o Optics per node
- 2x S5232F backend switches (intra-cluster)
- Dual PSU 1400W RDNT, iDRAC 16G Enterprise
- Software: Enterprise Bundle, SmartConnect, SnapshotIQ, SmartQuotas, SyncIQ (replikacja), SmartPools, SmartDedupe, AIOps/CloudIQ, InsightIQ, HDFS

**DR:** 3x PowerScale F210
- ~31TB raw per node (4x 7.68TB ISE NVMe SSD)
- 2.0 GHz Processor per node
- 128GB RAM (5600 MT/s DIMM) per node
- FE: 2x 100GbE w/o Optics per node
- BE: 2x 100GbE w/o Optics per node
- 2x S5232 backend switches
- Taka sama konfiguracja software

### PowerStore (Dane Strukturalne / Vector DB - Block)
- Istniejący PowerStore na obu site'ach
- Upgrade: 2x 100GB NIC per site
- Protokół: NVMeoF over TCP
- Wystarczająca pojemność, możliwość rozbudowy

## Podsumowanie Hardware

| Komponent | MAIN | DR |
|-----------|------|----|
| Serwery Fine-Tuning (C1/C4) | 2x XE9680 (16 GPU SXM) | 1x XE7745 (8 GPU NVL) |
| Serwery Inference+Test (C2+C3/C5) | 2x XE7745 (12+4 GPU NVL) | 1x XE7745 (8 GPU NVL) |
| Serwery CPU (Harvester) | 3x R570 | 3x R570 |
| Klastry RKE2 (fizyczne) | 1 | 1 |
| Klastry logiczne | C1, C2, C3 | C4, C5 |
| PowerScale | 3x F710 (~117TB raw) | 3x F210 (~93TB raw) |
| PowerStore | istniejący (wystarczająca pojemność) + 2x100G NIC | istniejący + 2x100G NIC |
| Switche | 5 (1x400G + 4x100G) | 2 (100G consolidated) |
| GPU łącznie | 32x H200 | 16x H200 |
