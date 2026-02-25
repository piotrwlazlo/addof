# ADDOF - Baza Wiedzy Projektu (PL)

## Kontekst
Projekt: On-prem AI Platform dla klienta enterprise
Rola użytkownika: Tech Lead (kontakt z klientami i partnerami)
Ja: Zaufany architekt systemów / ekspert techniczny

## Struktura Bazy Wiedzy
- [Architektura ogólna](architektura-ogolna.md) - Pełny przegląd platformy, komponenty, diagramy
- [Infrastruktura sprzętowa](infrastruktura-sprzet.md) - Serwery Dell, GPU, storage, networking
- [SUSE Virtualization (Harvester)](suse-virtualization.md) - Wirtualizacja, VM, HCI
- [SUSE Rancher Prime](suse-rancher.md) - Zarządzanie Kubernetes, klastry, RBAC
- [Kubernetes i RKE2](kubernetes-rke2.md) - Klastry downstream, GPU operator, CSI
- [Storage](storage.md) - Longhorn, PowerScale, PowerStore, wolumeny
- [Networking](networking.md) - Fabric 400GbE/100GbE/25GbE, SR-IOV, RDMA
- [Disaster Recovery](disaster-recovery.md) - DR, backup, CloudCasa, replikacja
- [AI Platform (OICM)](ai-platform-oicm.md) - Open Innovation, training, inference, MLOps
- [Dokumentacja techniczna](dokumentacja-techniczna.md) - Struktura repo, Antora, budowanie docs
- [Slownik pojec](slownik.md) - Kluczowe terminy i skróty

## Kluczowe Fakty
- 2 data center: Production (MAIN) + Disaster Recovery (DR)
- 6 serwerów CPU (wirtualizacja) + 5 serwerów GPU (AI workloads)
- SUSE stack: Virtualization (Harvester) + Rancher Prime + RKE2 + Storage (Longhorn)
- Dell stack: PowerScale (NAS) + PowerStore (block) + PowerSwitch (SONIC)
- GPU: NVIDIA H200 (SXM na XE9680, NVL PCIe na XE7745)
- Backup: CloudCasa by Catalogic + etcd snapshots + SyncIQ
- AI: Open Innovation Platform (OICM) - multi-tenant, multi-cluster
- Dokumentacja: Antora 3.1.9 + AsciiDoc, wersje Harvester v1.3-v1.7, Rancher v2.8-v2.13
