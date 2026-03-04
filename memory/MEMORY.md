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
- 2 data center: Production (MAIN) + Disaster Recovery (DR), **connected** (nie air-gapped)
- 6x R570 (Harvester) + 2x XE9680 + 4x XE7745 (GPU) = 48 GPU H200
- **2 fizyczne klastry RKE2**, 5 logicznych (C1-C5 via namespace'y + OICM)
- SUSE stack: Harvester + Rancher Prime + RKE2 + Longhorn
- Dell stack: PowerScale (NAS) + PowerStore (block) + PowerSwitch (SONIC)
- GPU: NVIDIA H200 (SXM na XE9680, NVL PCIe na XE7745)
- Backup: CloudCasa + etcd snapshots + SyncIQ
- AI: OICM v1.11.0 - multi-tenant, BEZ GenAI Studio
- **⚠️ Harvester + PowerScale CSI = nie wspierane** — wymaga workaround
