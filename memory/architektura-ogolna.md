# Architektura Ogólna Platformy AI

## Cel Projektu
Dostarczenie klientowi w pełni funkcjonalnej platformy AI on-premises, składającej się z:
- Serwerów Dell (compute + GPU)
- Sieci (Dell PowerSwitch z Enterprise SONIC)
- Storage (Dell PowerScale + PowerStore)
- SUSE Kubernetes (Rancher + RKE2 + Harvester)
- Backup & DR (CloudCasa by Catalogic)
- Platformy AI (Open Innovation Platform / OICM)

## Architektura Dwóch Centrów Danych

### MAIN (Produkcja)
```
┌─────────────────────────────────────────────────────────────────┐
│                    MAIN AI SETUP (Produkcja)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NETWORKING:                                                     │
│  ┌──────────┐ ┌──────────────┐ ┌─────────────┐ ┌────────────┐  │
│  │ Front End│ │Tuning Fabric │ │  Inference   │ │  Storage   │  │
│  │  25 GbE  │ │   400 GbE    │ │   100 GbE   │ │  100 GbE   │  │
│  └──────────┘ └──────────────┘ └─────────────┘ └────────────┘  │
│                                                                  │
│  COMPUTE (1 fizyczny klaster RKE2, 3 logiczne podziały):        │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐   │
│  │  C1 Fine-Tuning  │ │ C2 Inference    │ │  C3 TEST / DEV  │   │
│  │  2x XE9680       │ │ 12 GPU XE7745   │ │  4 GPU XE7745   │   │
│  │  16x H200 SXM    │ │ (1.5 serwera)   │ │  (0.5 serwera)  │   │
│  │  400GbE backend  │ │ 100GbE backend  │ │  100GbE backend │   │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘   │
│  Izolacja logiczna: namespace'y + OICM scheduling/quotas        │
│                                                                  │
│  CONTROL PLANE:  3 VM (RKE2 master etcd/controlplane + worker)  │
│                  8 core CPU, 64GB RAM, 150GB SSD per VM         │
│                                                                  │
│  STORAGE:                                                        │
│  ┌────────────────────┐  ┌───────────────────┐                  │
│  │ PowerScale (NAS)   │  │ PowerStore (Block) │                 │
│  │ 3x F710, 76TB      │  │ NVMeoF-TCP         │                 │
│  │ Dane niestruktur.  │  │ Dane strukt./VecDB │                 │
│  └────────────────────┘  └───────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

### DR (Disaster Recovery) - wersja okrojona
```
┌─────────────────────────────────────────────────────────────────┐
│                    DR AI SETUP (Disaster Recovery)                │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  NETWORKING:                                                     │
│  ┌──────────┐ ┌────────────────────────────────────────────┐    │
│  │ Front End│ │     Consolidated Fabric 100 GbE            │    │
│  │  25 GbE  │ │  (tuning + inference + storage w jednym)   │    │
│  └──────────┘ └────────────────────────────────────────────┘    │
│                                                                  │
│  COMPUTE (1 fizyczny klaster RKE2, 2 logiczne podziały):        │
│  ┌─────────────────┐ ┌─────────────────┐                        │
│  │ C4 DR Fine-Tune  │ │ C5 DR Inference │                        │
│  │  1x XE7745       │ │  1x XE7745      │                        │
│  │  8x H200 NVL     │ │  8x H200 NVL    │                        │
│  └─────────────────┘ └─────────────────┘                        │
│  Izolacja logiczna: namespace'y + OICM scheduling/quotas        │
│                                                                  │
│  CONTROL PLANE:  3 VM (jak MAIN)                                │
│                                                                  │
│  STORAGE:                                                        │
│  ┌────────────────────┐  ┌───────────────────┐                  │
│  │ PowerScale (NAS)   │  │ PowerStore (Block) │                 │
│  │ 3x F210, 62TB      │  │ NVMeoF-TCP         │                 │
│  └────────────────────┘  └───────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

## Warstwy Stosu Technologicznego (od góry)

| Warstwa | Komponenty |
|---------|------------|
| **Gen AI Studio** | Interfejs użytkownika do pracy z AI |
| **OICM: AI & Data Science** | Inferencing, Fine-Tuning, Training, Notebooks, A/B Testing, LLM Tools, MLOps |
| **OICM: GPU & Workload Mgmt** | Multi-Tenancy, Multi-Cluster, Scheduling, Monitoring, Logging, Security/IAM, Workspaces |
| **Kubernetes (RKE2)** | Pod Lifecycle, Load Balancing, Service Discovery, Auto-Scaling, Health Checks, Secrets |
| **System Software** | Libraries, Compilers, Runtime, GPU Drivers (CUDA), NVIDIA GPU Operator |
| **OS** | SUSE Linux Enterprise / SUSE Linux Micro |
| **Wirtualizacja** | SUSE Virtualization (Harvester) - KubeVirt + Longhorn |
| **Hardware** | Dell PowerEdge (XE9680, XE7745), PowerScale, PowerStore, PowerSwitch |

## Kluczowe Decyzje Architektoniczne

1. **Separacja fabric'ów sieciowych na MAIN** - osobne sieci dla tuning (400G), inference (100G), storage (100G), frontend (25G) = maksymalna wydajność
2. **Konsolidacja fabric'ów na DR** - jeden fabric 100GbE = oszczędność kosztów przy zachowaniu funkcjonalności
3. **Różne serwery wg roli** - XE9680 (8 GPU SXM per serwer) do fine-tuningu, XE7745 (8 GPU NVL PCIe) do inference
4. **DR na XE7745 dla obu ról** - oszczędność, DR nie musi mieć pełnej mocy MAIN
5. **2 fizyczne klastry RKE2, 5 logicznych** - izolacja via namespace'y i OICM (nie oddzielne klastry)
6. **Logiczny podział GPU na MAIN** - C2 (inference) dostaje 12 GPU, C3 (test/dev) 4 GPU z 2 serwerów XE7745
7. **Identyczny storage na obu site'ach** - PowerScale + PowerStore = łatwa replikacja i failover
8. **GitOps/CI-CD** - jeden Git repo synchronizuje konfigurację na oba site'y
9. **Środowisko connected** (nie air-gapped) - decyzja 2026-03-04

## Partnerzy i Dostawcy

| Partner | Rola |
|---------|------|
| **Dell Technologies** | Hardware (serwery, storage, networking), CSI/CSM drivers |
| **SUSE** | Virtualization, Rancher, RKE2, Storage, Linux, AI Suite |
| **NVIDIA** | GPU (H200), CUDA, GPU Operator, NVAIE licencje |
| **Catalogic (CloudCasa)** | Kubernetes backup & disaster recovery |
| **Open Innovation** | Platforma AI (OICM) - orkiestracja workloadów AI |
| **Klient** | Infrastruktura DC, sieć, Active Directory, wymagania biznesowe |

## Fazy Projektu

1. **Discovery, Design & Architecture** - warsztaty, dokumenty projektowe
2. **Implementation** - instalacja i konfiguracja komponentów
3. **Testing** - testy GPU, DR, backup/restore
4. **Engagement Review & Hand-Over** - dokumentacja, knowledge transfer
