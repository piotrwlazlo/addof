# Storage

## Przegląd warstw storage

W projekcie są **trzy oddzielne warstwy storage** - nie mylić!

| Warstwa | Technologia | Rola | Gdzie |
|---------|------------|------|-------|
| **Harvester Storage** | SUSE Storage (Longhorn) | Dyski VM, local storage | 6 CPU nodes |
| **AI Unstructured** | Dell PowerScale (NAS) | Datasety, obrazy, pliki treningowe | Osobne nody |
| **AI Structured** | Dell PowerStore (Block) | Vector DB, bazy danych | Istniejące |

## 1. SUSE Storage (Longhorn) - Storage dla Harvester

### Co to jest?
Longhorn to rozproszony system storage blokowy natywny dla Kubernetes. Zapewnia replikowane wolumeny trwałe.

### Rola w projekcie
- Dyski dla VM działających na Harvester
- Storage lokalny dla control plane (etcd, Rancher data)
- Deploy na **6 CPU nodach** (zarówno MAIN jak DR)

### Zakres wdrożenia
- Walidacja best practices i wymagań sprzętowych
- Konfiguracja wolumenów:
  - **RWO** (ReadWriteOnce) - block storage (jedna replika pisze)
  - **RWX** (ReadWriteMany) - file storage (wiele replik czyta/pisze)
- Uwzględnienie rozmiaru wolumenów vs. dostępna pojemność

### Longhorn V1 vs V2
- **V1** - stabilny, domyślny engine
- **V2** - nowszy data engine, lepsza wydajność (dostępny od nowszych wersji Harvester)

## 2. Dell PowerScale - Dane Niestrukturalne (NAS)

### Co to jest?
Dell PowerScale (dawniej Isilon) to scale-out NAS klasy enterprise, idealny dla dużych zbiorów danych AI (obrazy, tekst, dane treningowe).

### Konfiguracja MAIN (specyfikacja potwierdzona 2026-02-24)
- 3x PowerScale F710
- **~39TB raw per node** (10x 3.84TB ISE NVMe SSD)
- FE: 2x 100GbE w/o Optics per node
- BE: 2x 100GbE w/o Optics per node
- 2x 2.6 GHz Processor per node
- 512GB RAM (5600 MT/s DIMM) per node
- 2x S5232F backend switches (intra-cluster)
- Dual PSU 1400W RDNT, iDRAC 16G Enterprise
- Secure Component Validation, CSV TRIG License, Secure Enterprise Key Management

### Konfiguracja DR (specyfikacja potwierdzona 2026-02-24)
- 3x PowerScale F210
- **~31TB raw per node** (4x 7.68TB ISE NVMe SSD)
- FE: 2x 100GbE w/o Optics per node
- BE: 2x 100GbE w/o Optics per node
- 2.0 GHz Processor per node
- 128GB RAM (5600 MT/s DIMM) per node
- 2x S5232 backend switches

### Oprogramowanie PowerScale
| Moduł | Funkcja |
|-------|---------|
| **SmartConnect Advanced** | Load balancing połączeń klientów |
| **SnapshotIQ** | Snapshoty danych |
| **SmartQuotas** | Kwoty dyskowe per tenant/projekt |
| **SyncIQ** | **Replikacja file-level MAIN → DR** |
| **SmartPools** | Tiering danych (hot/cold) |
| **SmartDedupe** | Deduplikacja |
| **AIOps/CloudIQ** | Monitoring i analytics |
| **InsightIQ** | Performance monitoring |
| **HDFS** | Hadoop Distributed File System connector |

### Replikacja MAIN → DR
- Technologia: **SyncIQ** (file-level replication)
- Automatyczna synchronizacja danych między site'ami
- Konfiguracja RPO (Recovery Point Objective) per policy

## 3. Dell PowerStore - Dane Strukturalne (Block)

### Co to jest?
Dell PowerStore to enterprise block/unified storage, idealny dla baz danych (w tym Vector DB) i danych transakcyjnych.

### Konfiguracja
- **Istniejący** PowerStore na obu site'ach
- Upgrade: 2x 100GB NIC per site
- Protokół: **NVMeoF over TCP** (najnowszy, najszybszy)

### Replikacja MAIN → DR
- Technologia: **Vector-DB Application Based** replication
- Synchronizacja na poziomie aplikacji (nie storage)

## 4. Dell CSI (Container Storage Interface)

### Rola
CSI driver od Dell zainstalowany na klastrach RKE2 umożliwia Kubernetes automatyczne tworzenie i zarządzanie wolumenami na PowerStore i PowerScale.

### Zakres wdrożenia
- Instalacja CSI drivers na klastrach downstream
- Konfiguracja endpoints, auth, storage pool mappings
- Tworzenie StorageClasses (różne tiery, SLA)
- Snapshot/restore workflows

## 5. Dell CSM (Container Storage Modules)

### Moduły
| Moduł | Funkcja |
|-------|---------|
| **CSM Observability** | Monitoring storage z Kubernetes |
| **CSM Resiliency** | Automatyczne odzyskiwanie po awariach |
| **CSM Replication** | Replikacja danych storage między site'ami dla DR |

### CSM Replication
- Kluczowy komponent DR
- Replikacja storage-level między MAIN i DR
- Integracja z CloudCasa dla pełnego DR workflow
- Docs: https://dell.github.io/csm-docs/

## Ważne dla Tech Leada
1. **Nie mylić 3 warstw storage!** Longhorn ≠ PowerScale ≠ PowerStore
2. **PowerScale** = duże pliki AI (datasety, modele) → NAS/file access
3. **PowerStore** = Vector DB + dane strukturalne → block/NVMeoF
4. **Longhorn** = dyski VM i local k8s storage → internal Harvester
5. **SyncIQ** replicates files, **CSM** replicates block storage, **CloudCasa** replicates Kubernetes state
6. **NVMeoF over TCP** = nowoczesny protokół, wymaga 100GbE → najlepsza wydajność block storage
7. **HDFS connector** na PowerScale = gotowość na Spark/Hadoop workloady w przyszłości
