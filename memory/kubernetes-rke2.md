# Kubernetes i RKE2 Downstream Clusters

## Co to jest RKE2?
RKE2 (Rancher Kubernetes Engine 2) to dystrybucja Kubernetes od SUSE, zoptymalizowana pod kątem bezpieczeństwa i zgodności (FIPS, CIS benchmark). Jest zarządzana przez Rancher Prime.

## Rola w projekcie
Klastry RKE2 downstream to WŁAŚCIWE środowisko uruchomieniowe dla workloadów AI. To na nich działają kontenery z modelami AI, training joby, inference servery itd.

## Topologia klastrów

### MAIN Site
| Klaster | Rola | Serwer | GPU |
|---------|------|--------|-----|
| **C1** | Fine-Tuning | XE9680 #1 | 8x H200 SXM |
| **C2** | Fine-Tuning | XE9680 #2 | 8x H200 SXM |
| **C3** | Inference/RAG | 2x XE7745 | 16x H200 NVL |

### DR Site
| Klaster | Rola | Serwer | GPU |
|---------|------|--------|-----|
| **C4** | DR Fine-Tuning | XE7745 #1 | 8x H200 NVL |
| **C5** | DR Inference | XE7745 #2 | 8x H200 NVL |

**Łącznie: 5 klastrów RKE2 na bare-metal GPU nodes**

### Architektura każdego klastra
- Master nodes (etcd + controlplane) → działają jako VM na Harvester
- Worker nodes → bare-metal serwery GPU
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
1. **5 klastrów** to serce platformy AI - tu działają wszystkie workloady
2. **GPU Operator** jest KRYTYCZNY - bez niego pody nie widzą GPU
3. **GPU Operator NIE wspiera SUSE OS** - worker nodes muszą być Ubuntu lub RHEL (decyzja zależy od licencji klienta)
4. **SR-IOV + RDMA** dają 10-100x lepszą wydajność sieci dla GPU - ważne dla distributed training
5. **Master nodes na VM** (Harvester), worker nodes na **bare-metal** - typowa architektura dla GPU
6. **Dell CSI** = persistent storage dla AI data (modele, datasety, checkpointy)
7. **Każdy klaster** jest zarządzany przez Rancher, ale operacyjnie niezależny
8. **RKE2** jest FIPS-compliant out-of-the-box - ważne dla enterprise klientów
9. **CNI** - OICM i SUSE wspólnie wybiorą konkretny CNI (w trakcie ustaleń)
