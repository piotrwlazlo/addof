# Słownik Pojęć i Skrótów

## Infrastruktura

| Termin | Pełna nazwa | Opis |
|--------|------------|------|
| **HCI** | Hyper-Converged Infrastructure | Serwer łączący compute, storage i networking w jednym |
| **RDMA** | Remote Direct Memory Access | Bezpośredni dostęp do pamięci zdalnego serwera bez CPU |
| **RoCE** | RDMA over Converged Ethernet | Implementacja RDMA na standardowym Ethernet |
| **SR-IOV** | Single Root I/O Virtualization | Wirtualizacja I/O - dzielenie jednej karty NIC na wiele wirtualnych |
| **VF** | Virtual Function | Wirtualna instancja karty NIC w SR-IOV |
| **PF** | Physical Function | Fizyczna karta NIC w SR-IOV |
| **IOMMU** | Input/Output Memory Management Unit | Jednostka zarządzania pamięcią I/O (wymagana dla SR-IOV) |
| **NVMeoF** | NVMe over Fabrics | Protokół NVMe rozszerzony na sieć (tu: TCP) |
| **NDR** | Next Data Rate | Standard InfiniBand/Ethernet 400Gbps |
| **SXM** | (form factor) | Socket GPU bezpośrednio na płycie głównej (najwydajniejszy) |
| **NVL** | NVLink | Połączenie GPU-GPU (szybsze niż PCIe) |
| **BMC** | Baseboard Management Controller | Zarządzanie serwerem out-of-band (IPMI/iDRAC) |
| **BOSS** | Boot Optimized Storage Solution | Moduł Dell z dyskami boot |
| **MTU** | Maximum Transmission Unit | Maksymalny rozmiar pakietu (jumbo frames = 9000) |
| **LACP** | Link Aggregation Control Protocol | Agregacja portów sieciowych |

## Kubernetes / SUSE

| Termin | Pełna nazwa | Opis |
|--------|------------|------|
| **RKE2** | Rancher Kubernetes Engine 2 | Dystrybucja K8s od SUSE (FIPS-ready) |
| **K3s** | (lekki K8s) | Lekka dystrybucja K8s od SUSE |
| **etcd** | (baza danych) | Rozproszona baza key-value, serce Kubernetes |
| **CRD** | Custom Resource Definition | Rozszerzenie API Kubernetes |
| **CSI** | Container Storage Interface | Standard integracji storage z K8s |
| **CSM** | Container Storage Modules | Moduły Dell do zaawansowanego storage w K8s |
| **PV** | Persistent Volume | Trwały wolumen w Kubernetes |
| **PVC** | Persistent Volume Claim | Żądanie trwałego wolumenu przez pod |
| **RWO** | ReadWriteOnce | Wolumen montowany przez 1 node |
| **RWX** | ReadWriteMany | Wolumen montowany przez wiele node'ów |
| **CNI** | Container Network Interface | Standard pluginów sieciowych K8s |
| **RBAC** | Role-Based Access Control | Kontrola dostępu oparta na rolach |
| **Fleet** | (GitOps engine) | Silnik GitOps wbudowany w Rancher |
| **Longhorn** | (storage) | Rozproszony storage blokowy dla K8s (SUSE) |
| **KubeVirt** | (virtualization) | VM na Kubernetes |
| **Harvester** | (HCI platform) | Platforma HCI od SUSE (KubeVirt + Longhorn) |
| **Multus** | (CNI plugin) | Plugin pozwalający na wiele NIC w pod'ach |

## Storage Dell

| Termin | Opis |
|--------|------|
| **PowerScale** | Scale-out NAS (dawniej Isilon) - pliki/dane niestrukturalne |
| **PowerStore** | Enterprise block/unified storage - bazy danych, block |
| **PowerSwitch** | Switche sieciowe Dell |
| **SyncIQ** | Replikacja file-level na PowerScale (MAIN → DR) |
| **SmartConnect** | Load balancing klientów NAS |
| **SmartQuotas** | Kwoty dyskowe per tenant |
| **SmartPools** | Tiering danych (hot/cold) |
| **SnapshotIQ** | Snapshoty danych na PowerScale |
| **InsightIQ** | Performance monitoring PowerScale |
| **CloudIQ** | AIOps monitoring Dell storage |

## AI / ML

| Termin | Opis |
|--------|------|
| **OICM** | Open Innovation Cluster Manager - platforma AI |
| **Fine-Tuning** | Dostrajanie modelu na nowych danych |
| **Training** | Trenowanie modelu od zera |
| **Inference** | Serwowanie modelu (odpowiedzi na zapytania) |
| **RAG** | Retrieval-Augmented Generation - LLM + baza wiedzy |
| **Vector DB** | Baza danych wektorowych (embeddingi) |
| **MLOps** | DevOps dla modeli ML (CI/CD + monitoring) |
| **LLM** | Large Language Model |
| **NVAIE** | NVIDIA AI Enterprise - licencje na software AI |
| **CUDA** | Compute Unified Device Architecture - framework GPU NVIDIA |
| **DCGM** | Data Center GPU Manager - monitoring GPU |
| **NVSwitch** | Wewnętrzny switch GPU-GPU w SXM platforms |
| **HBM3e** | High Bandwidth Memory 3e - pamięć GPU H200 (141GB) |

## Backup / DR

| Termin | Opis |
|--------|------|
| **CloudCasa** | Kubernetes backup & DR by Catalogic |
| **Velero** | Open-source backup Kubernetes |
| **RPO** | Recovery Point Objective - ile danych można stracić |
| **RTO** | Recovery Time Objective - jak szybko odtworzyć |
| **DR** | Disaster Recovery |
| **Failover** | Przełączenie na zapasowy system |
| **Runbook** | Procedura krok-po-kroku dla operacji DR |

## Dokumentacja

| Termin | Opis |
|--------|------|
| **Antora** | Generator static site dla dokumentacji |
| **AsciiDoc** | Format markup dokumentacji (alternatywa Markdown) |
| **Playbook** | Plik konfiguracyjny build Antora |
| **SRFA** | (wariant docs Rancher - specyficzny) |

## Nazwy klastrów (w projekcie)

| Klaster | Site | Rola |
|---------|------|------|
| **C1** | MAIN | Fine-Tuning #1 |
| **C2** | MAIN | Fine-Tuning #2 |
| **C3** | MAIN | Inference/RAG |
| **C4** | DR | DR Fine-Tuning |
| **C5** | DR | DR Inference |
