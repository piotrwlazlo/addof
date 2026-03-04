# Kubernetes i RKE2 Downstream Clusters

## Co to jest RKE2?
RKE2 (Rancher Kubernetes Engine 2) to dystrybucja Kubernetes od SUSE, zoptymalizowana pod kątem bezpieczeństwa i zgodności (FIPS, CIS benchmark). Jest zarządzana przez Rancher Prime.

## Rola w projekcie
Klastry RKE2 downstream to WŁAŚCIWE środowisko uruchomieniowe dla workloadów AI. To na nich działają kontenery z modelami AI, training joby, inference servery itd.

## Topologia klastrów (zaktualizowana 2026-03-04)

### Model: 2 klastry fizyczne, 5 klastrów logicznych
Zamiast 5 oddzielnych klastrów RKE2, projekt stosuje **2 fizyczne klastry RKE2** (1 MAIN + 1 DR) z **logiczną izolacją** (namespace'y, OICM multi-tenancy) wewnątrz każdego klastra.

### MAIN Site — 1 fizyczny klaster RKE2
| Klaster logiczny | Rola | Serwery | GPU |
|------------------|------|---------|-----|
| **C1** | Fine-Tuning | 2x XE9680 | 16x H200 SXM |
| **C2** | Inference/RAG | 1.5x XE7745 (12 GPU) | 12x H200 NVL |
| **C3** | Test/Dev | 0.5x XE7745 (4 GPU) | 4x H200 NVL |

- Fizycznie: 2x XE9680 + 2x XE7745 = **32 GPU SXM + 16 GPU NVL**
- C2 i C3 współdzielą 2 serwery XE7745 (12+4 GPU, podział logiczny via OICM scheduling)

### DR Site — 1 fizyczny klaster RKE2
| Klaster logiczny | Rola | Serwery | GPU |
|------------------|------|---------|-----|
| **C4** | DR Fine-Tuning | 1x XE7745 | 8x H200 NVL |
| **C5** | DR Inference | 1x XE7745 | 8x H200 NVL |

- Fizycznie: 2x XE7745 = **16 GPU NVL**

**Łącznie: 2 fizyczne klastry RKE2, 5 logicznych podziałów (namespace'y)**

### Architektura klastra fizycznego
- Master nodes (etcd + controlplane) → działają jako VM na Harvester
- Worker nodes → bare-metal serwery GPU
- Izolacja logiczna → namespace'y + OICM resource management/quotas
- Najnowsza wersja Kubernetes

### OS na GPU Worker Nodes (decyzja 2026-02-24)
> **GPU Operator NIE wspiera SUSE OS.** Na worker nodes GPU będzie zainstalowany **Ubuntu** lub **RHEL** — decyzja zależy od tego, jakie licencje OS klient już posiada.

## NVIDIA GPU Operator
Zainstalowany na każdym klastrze RKE2, odpowiada za:
- Automatyczne wykrywanie i konfigurację GPU
- Instalację GPU drivers
- Instalację CUDA toolkit
- Container runtime (nvidia-container-runtime)
- Device plugin (eksponowanie GPU do podów)
- GPU Feature Discovery
- Monitoring GPU (DCGM)
- **Wspierane OS:** Ubuntu, RHEL (NIE SUSE/SLE)

## Dell PowerStore CSI
Na każdym klastrze RKE2 zainstalowany Dell CSI driver:
- Tworzenie StorageClass dla persistent volumes
- Dynamiczne provisionowanie wolumenów
- Snapshot/restore wolumenów
- Protokół: NVMeoF over TCP (100GbE)

## Dell CSM (Container Storage Modules)
Dodatkowe moduły Dell do zaawansowanego zarządzania storage:
- **CSM Observability** - monitoring storage
- **CSM Resiliency** - odporność na awarie
- **CSM Replication** - replikacja danych między MAIN i DR

## SR-IOV i RDMA
Zaawansowane technologie sieciowe dla GPU workloadów:
- **SR-IOV** (Single Root I/O Virtualization) - bezpośredni dostęp GPU do NIC (bypass kernel)
- **RDMA** (Remote Direct Memory Access) - bezpośredni transfer pamięci między serwerami
- **RoCE** (RDMA over Converged Ethernet) - RDMA po Ethernet
- Konfiguracja per-host: BIOS/firmware, IOMMU, MTU, offload settings
- Mapowanie VF (Virtual Functions) per rola hosta

## Kluczowe usługi Kubernetes w projekcie
| Usługa | Opis |
|--------|------|
| Pod Lifecycle | Zarządzanie cyklem życia kontenerów |
| Load Balancing | Rozkładanie ruchu |
| Service Discovery | Automatyczne odkrywanie usług |
| Auto-Scaling | Automatyczne skalowanie (ważne dla inference) |
| Health Checks | Monitorowanie zdrowia podów |
| Secrets Management | Bezpieczne przechowywanie credentials |
| Network Policies | Izolacja sieciowa między tenantami |

## Ważne dla Tech Leada
1. **2 fizyczne klastry RKE2** (MAIN+DR) z 5 logicznymi podziałami - to serce platformy AI
2. **GPU Operator** jest KRYTYCZNY - bez niego pody nie widzą GPU
3. **GPU Operator NIE wspiera SUSE OS** - worker nodes muszą być Ubuntu lub RHEL (decyzja zależy od licencji klienta)
4. **SR-IOV + RDMA** dają 10-100x lepszą wydajność sieci dla GPU - ważne dla distributed training
5. **Master nodes na VM** (Harvester), worker nodes na **bare-metal** - typowa architektura dla GPU
6. **Dell CSI** = persistent storage dla AI data (modele, datasety, checkpointy)
7. **Oba klastry fizyczne** zarządzane przez Rancher, logiczna izolacja via namespace'y i OICM
8. **RKE2** jest FIPS-compliant out-of-the-box - ważne dla enterprise klientów
9. **CNI** - OICM i SUSE wspólnie wybiorą konkretny CNI (w trakcie ustaleń)
