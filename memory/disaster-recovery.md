# Disaster Recovery (DR)

## Strategia DR - Active-Passive
Projekt stosuje model **active-passive** (MAIN aktywny, DR pasywny/standby). DR site jest okrojoną wersją MAIN, zaprojektowaną do przejęcia operacji w razie awarii.

## Komponenty DR

### 1. Replikacja danych - trzy mechanizmy

| Typ danych | Technologia | Kierunek | Poziom |
|-----------|------------|----------|--------|
| Pliki/datasety AI | Dell PowerScale **SyncIQ** | MAIN → DR | File-level |
| Vector DB / dane strukturalne | **Application-Based** replication | MAIN → DR | App-level |
| Block storage (PV) | Dell **CSM Replication** | MAIN → DR | Storage-level |

### 2. Kubernetes Protection - CloudCasa

**CloudCasa by Catalogic** - centralny komponent ochrony Kubernetes.

#### Co chroni CloudCasa?
- Wszystkie zasoby klastra K8s (CRDs, RBAC, services, ingress, storage classes)
- Persistent Volumes (dane aplikacji)
- Metadane klastra (konfiguracja)
- VM na SUSE Virtualization (Harvester)
- Amazon RDS (jeśli dotyczy)

#### Kluczowe funkcje
| Funkcja | Opis |
|---------|------|
| **Cross-cluster restore** | Odtwarzanie na innym klastrze (MAIN → DR) |
| **Cross-account restore** | Odtwarzanie na innym koncie |
| **Cross-cloud restore** | Odtwarzanie w innym cloud |
| **Application-aware backups** | Hooki dla baz danych, message brokerów |
| **Policy-driven** | Automatyczne polityki backup wg priorytetów |
| **Self-hosted** | Działa na infrastrukturze klienta (air-gapped) |
| **Velero management** | Zarządzanie istniejącymi instalacjami Velero |

#### CloudCasa + SUSE Storage (Longhorn)
Specjalna integracja: **CloudCasa DR for Storage** - replikacja na poziomie storage z niskim RTO (szybki failover między klastrami).

### 3. VM Replication
- Replikacja VM między MAIN i DR
- Synchronizacja środowisk compute

### 4. GitOps / CI-CD
- Centralny **Git Repository** + CI/CD Pipeline
- Łączy 2 fizyczne klastry RKE2 (MAIN i DR) z logicznymi podziałami (C1-C5)
- Identyczne deploymenty i konfiguracje na obu site'ach
- Fleet (Rancher) jako mechanizm synchronizacji

### 5. Rancher Backup
- **Rancher Backup Operator** - backup konfiguracji Rancher
- **etcd backup & snapshotting** - backup bazy danych K8s
- Testowanie: restore backup Rancher na nowy klaster = symulacja DR

## Plan wdrożenia CloudCasa (4 dni)

### Dzień 1 - Assessment & Deployment
- Przegląd architektury SUSE Virtualization, Harvester, bare-metal
- Identyfikacja klastrów wymagających DR
- Weryfikacja connectivity między site'ami
- Instalacja agenta CloudCasa na klastrach K8s
- Rejestracja w CloudCasa Control Plane
- Konfiguracja RBAC, service accounts, storage destinations

### Dzień 2 - Backup Policy Design
- Mapowanie workloadów na tiery backupów:
  - **Krytyczne** - stateful apps z PV backup + app hooks
  - **Standardowe** - regularne workloady
  - **Niekrytyczne** - metadata-only backups
- Definiowanie RPO/RTO per grupa workloadów
- Tworzenie polityk: namespaces, metadata, PV data
- Konfiguracja schedulingu, retencji, expiracji
- Application-aware backups (hooki dla DB)
- Testowe backup joby

### Dzień 3 - DR Workflow & Testing
- Projektowanie DR workflow: Site A → Site B
- Recovery do klastrów na SUSE Virtualization/Harvester
- Symulowany failover:
  - Restore metadanych klastra
  - Restore stateful workloadów
  - Restore persistent volumes na DR
- Walidacja: networking, services, ingress/egress po restore

### Dzień 4 - Optimization & Handover
- Tuning polityk (retencja, concurrency, storage efficiency)
- Walidacja cross-site DR workflow
- Dokumentacja:
  - Backup architecture diagram
  - DR workflow diagram
  - Cluster-specific CloudCasa config
  - Known limitations & recommendations

## RPO i RTO

| Metryka | Opis | Do ustalenia z klientem |
|---------|------|------------------------|
| **RPO** (Recovery Point Objective) | Ile danych można stracić? | np. 1h, 4h, 24h |
| **RTO** (Recovery Time Objective) | Jak szybko odtworzyć? | np. 15min, 1h, 4h |

RPO i RTO definiowane per grupa workloadów - nie ma jednego RPO/RTO dla całej platformy.
> **Status (2026-03-04):** RPO/RTO ustali inżynier CloudCasa na spotkaniu z klientem.

## Ważne dla Tech Leada
1. **DR ≠ Backup** - DR to cały workflow failover, nie tylko kopie danych
2. **3 warstwy replikacji** muszą być zsynchronizowane (file, block, k8s)
3. **CloudCasa jest self-hosted** - działa on-prem, OK dla air-gapped
4. **RPO/RTO** to decyzja biznesowa klienta - my dostarczamy technologię
5. **Test DR** jest częścią scope'u - nie tylko konfiguracja, ale rzeczywisty test failover
6. **GitOps** zapewnia spójność konfiguracji, ale dane muszą być replikowane osobno
7. **CloudCasa + Velero** - CloudCasa może zarządzać istniejącym Velero (nie zastępuje, rozszerza)
8. **Runbook** dla DR obejmuje tylko produkty SUSE - Dell/CloudCasa/OI mają osobne runbooki
