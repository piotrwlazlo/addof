# SUSE Virtualization (Harvester)

## Co to jest?
SUSE Virtualization (dawniej Harvester) to platforma HCI (Hyper-Converged Infrastructure) oparta na Kubernetes, która umożliwia uruchamianie maszyn wirtualnych (VM) obok kontenerów na tej samej infrastrukturze.

## Kluczowe technologie wewnętrzne
- **KubeVirt** - uruchamianie VM na Kubernetes (QEMU/KVM)
- **Longhorn** - rozproszony storage blokowy (dyski VM)
- **Canal** - overlay network (Flannel + Calico)
- **SUSE Linux Micro** - niemutowalny system bazowy (host OS)

## Rola w projekcie
- Zainstalowany na **6 serwerach fizycznych CPU** (3 MAIN + 3 DR)
- Hostuje:
  - VM dla Rancher Management Server (3 VM/site)
  - VM dla master node'ów klastrów downstream RKE2
  - VM dla komponentów Open Innovation Platform
- Zintegrowany z SUSE Rancher Prime (zarządzanie z jednego panelu)

## Zakres wdrożenia

### Instalacja i konfiguracja
- Instalacja na 6 serwerach CPU across 2 site'y
- Konfiguracja storage i networking
- Konfiguracja multi-tenancy
- Upload "golden" image OS (SUSE Liberty / RHEL)
- Integracja z Rancher Prime

### Backup i Restore
- Konfiguracja backupów VM
- Snapshoty VM
- Testowanie restore

## Kluczowe funkcje (z dokumentacji v1.7)

### Zarządzanie VM
- Tworzenie VM (z UI, z image, z template)
- Live migration (przenoszenie VM między hostami bez downtime)
- Snapshoty i backup VM
- Klonowanie VM
- Hotplug (dodawanie dysków/NIC w runtime)
- Cloud-init (automatyczna konfiguracja VM przy starcie)

### Storage
- **Longhorn V1/V2** - wbudowany distributed storage
- Wolumeny RWO (ReadWriteOnce) i RWX (ReadWriteMany)
- Block storage i file storage
- Integracja z third-party CSI (LVM, NFS, Rook RADOS)

### Networking
- Multi-NIC (wiele interfejsów sieciowych na VM)
- VLAN support
- Bridge networks
- Bond aggregation (LACP / manual)
- Multus CNI + Bridge CNI plugins

### Monitoring & Logging
- Wbudowany Prometheus + Grafana
- Fluent Bit / Fluentd do logów
- Alertmanager

## Wersje dokumentacji dostępne
- v1.3, v1.4, v1.5 (archived)
- v1.6, v1.7 (aktywne)

## Ścieżka do dokumentacji
```
/home/pepe3535/addof/harvester-product-docs/versions/v1.7/modules/en/pages/
```

### Kluczowe pliki dokumentacji (v1.7)
- `introduction/overview.adoc` - przegląd produktu
- `installation/` - instalacja
- `cluster-management/` - zarządzanie klastrem
- `networking/` - konfiguracja sieci, deep-dive
- `storage/` - konfiguracja storage
- `virtual-machines/` - zarządzanie VM
- `observability/` - monitoring i logging
- `troubleshooting/` - rozwiązywanie problemów

## Ważne dla Tech Leada
1. Harvester to fundament infrastruktury - wszystkie VM zarządzające działają na nim
2. Multi-tenancy jest wymagane - oddzielenie środowisk/zespołów
3. Backup VM to osobny temat od backup Kubernetes (CloudCasa)
4. Live migration jest kluczowe dla maintenance bez downtime
5. Longhorn storage na Harvester to NIE to samo co storage dla GPU workloadów (tam Dell CSI)
