# Networking

## Filozofia sieci w projekcie
Sieć jest podzielona na **dedykowane fabric'i** wg typu ruchu. Na MAIN site'cie każdy typ ruchu ma osobną sieć fizyczną. Na DR site'cie fabric'i są skonsolidowane dla oszczędności.

## Fabric'i sieciowe

### MAIN Site - 4 oddzielne fabric'i
| Fabric | Przepustowość | Ruch | Switche |
|--------|--------------|------|---------|
| **Front End** | 25 GbE | Management, klienci, UI | Klient (istniejące) |
| **Tuning Fabric** | 400 GbE | Inter-GPU dla training/fine-tuning | 1x Z9664F-ON |
| **Inference Fabric** | 100 GbE | Inter-GPU dla inference | 2x S5232F-ON |
| **Storage Fabric** | 100 GbE | Serwery ↔ PowerScale/PowerStore | 2x S5232F-ON |

### DR Site - 2 fabric'i (skonsolidowane)
| Fabric | Przepustowość | Ruch | Switche |
|--------|--------------|------|---------|
| **Front End** | 25 GbE | Management, klienci | Klient (istniejące) |
| **Consolidated** | 100 GbE | Tuning + Inference + Storage | 2x S5232F-ON |

## Dlaczego 400GbE na Tuning?
Fine-tuning (training) modeli AI wymaga **masywnej wymiany gradientów** między GPU w różnych serwerach. Distributed training (np. FSDP, DeepSpeed) generuje ogromny ruch all-reduce. 400GbE (NDR InfiniBand/Ethernet) minimalizuje bottleneck komunikacyjny.

Inference NIE wymaga takiej przepustowości - każde zapytanie jest obsługiwane lokalnie przez GPU na jednym serwerze.

## Karty sieciowe (NIC) per rola serwera

### XE9680 (Fine-Tuning)
| NIC | Prędkość | Rola |
|-----|---------|------|
| Broadcom 57504 Quad Port | 10/25GbE SFP28 | Front End |
| Mellanox ConnectX-6 DX Dual Port | 100GbE QSFP56 | Storage Fabric |
| 8x NVIDIA ConnectX-7 Single Port | NDR/400GbE OSFP | **Backend (GPU-to-GPU)** |

### XE7745 (Inference / DR)
| NIC | Prędkość | Rola |
|-----|---------|------|
| Broadcom 57504 Quad Port | 10/25GbE SFP28 | Front End |
| Mellanox ConnectX-6 DX Dual Port | 100GbE QSFP56 | Storage Fabric |
| 4x NVIDIA ConnectX-6 DX Dual Port | 100GbE QSFP56 | **Backend (GPU-to-GPU)** |

## SR-IOV (Single Root I/O Virtualization)

### Co to jest?
SR-IOV pozwala jednej fizycznej karcie sieciowej (PF - Physical Function) prezentować się jako wiele wirtualnych kart (VF - Virtual Functions). Każdy VF może być przypisany bezpośrednio do kontenera/VM, omijając hypervisor/kernel dla maksymalnej wydajności.

### W projekcie
- Konfiguracja VF per rola hosta
- IOMMU musi być włączone w BIOS
- VF alokowane do GPU workloadów dla direct memory access
- Deliverable: RDMA & SR-IOV Host Design Document

## RDMA (Remote Direct Memory Access)

### Co to jest?
RDMA pozwala na bezpośredni transfer danych między pamięciami dwóch serwerów BEZ angażowania CPU. To kluczowe dla wydajnego distributed AI training.

### Implementacja w projekcie
- **RoCE** (RDMA over Converged Ethernet) - RDMA po standardowym Ethernet
- Konfiguracja na kartach Mellanox/NVIDIA ConnectX
- MTU (Jumbo Frames) musi być poprawnie ustawiony
- Offload settings na NIC

## Ważne ograniczenia GPU

### Brak Port-Channeling
- GPU NIC **NIE wspiera** port-channelingu ani HA (High Availability)
- Każdy GPU łączy się **jednym portem/jedną ścieżką** do switcha backend
- **Nie można** podłączyć GPU do dwóch switchy jednocześnie

### Konsekwencje
- Na MAIN site (tuning): **1 switch Z9664F** - single point of failure
- Opcja HA: 2 switche, po 4 GPU z każdego serwera do każdego switcha → połowa GPU komunikuje się przy awarii
- Na inference: analogicznie, 2 switche S5232F ale GPU na jednym

## Enterprise SONIC
Wszystkie switche Dell używają **Enterprise SONIC Distribution** - otwarty system operacyjny switchy oparty na Linux. Zalety:
- Unified management
- Automation-friendly (CLI + API)
- Kompatybilny z Dell ecosystem

## Deliverables sieciowe projektu
1. RDMA & SR-IOV Host Design Document
2. Host Configuration Summary (per-role, per-port)
3. RDMA & SR-IOV Integration Plan
4. CSI/CSM Integration Design
5. End-to-End Validation Summary

## Ważne dla Tech Leada
1. **400GbE fabric** = najdroższy element sieci (transceivers NDR są kosztowne)
2. **GPU brak HA** na poziomie sieci - to ryzyko architektoniczne do omówienia z klientem
3. **RDMA/SR-IOV** wymaga precyzyjnej konfiguracji BIOS/firmware - nie da się zrobić remote
4. **Consolidated fabric na DR** = kompromis kosztowy - training będzie wolniejszy niż na MAIN
5. Transceivers 400G są drogie → dlatego 400G fabric TYLKO na tuning, inference ma 100G
6. **NVMeoF over TCP** (storage) vs **RoCE** (GPU) - dwa różne użycia RDMA
