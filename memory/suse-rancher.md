# SUSE Rancher Prime Manager

## Co to jest?
SUSE Rancher Prime to centralna platforma zarządzania Kubernetes, która zapewnia jednolity interfejs do zarządzania wieloma klastrami Kubernetes (on-prem, cloud, edge) z jednego miejsca.

## Rola w projekcie
- **2 instancje** Rancher Prime Manager (MAIN + DR)
- Zarządzanie wszystkimi klastrami RKE2 downstream
- Centralna autentykacja i RBAC
- Integracja z Active Directory klienta
- Backup operator włączony

## Architektura Rancher

### Komponenty
- **API Server** - centralny punkt zarządzania zasobami
- **Authentication Proxy** - centralna autentykacja SSO
- **Cluster Controller** - zarządzanie klastrami downstream
- **Fleet** - GitOps continuous delivery
- **Cluster Agent** - agent w każdym klastrze downstream

### Tryby zarządzania klastrami
1. **Provisioned** - Rancher tworzy klaster (RKE2, K3s)
2. **Hosted** - Rancher zarządza klastrem w chmurze (EKS, AKS, GKE)
3. **Imported** - Rancher importuje istniejący klaster

## Zakres wdrożenia

### Deployment Rancher Manager
- 2 instancje (1 MAIN + 1 DR)
- 3 VM per instancja (HA cluster z etcd)
- 8 vCPU, 64GB RAM, 150GB SSD per VM
- Na infrastrukturze Harvester

### Identity Provider
- Integracja z Microsoft Active Directory
- Konfiguracja RBAC (Role-Based Access Control)

### Backup
- Rancher Backup Operator włączony
- etcd backup & snapshotting
- Testowanie backup/restore

## Autentykacja - Wspierani providerzy (13+)
- Microsoft Active Directory (używany w projekcie)
- LDAP
- GitHub OAuth
- Azure AD
- Google OAuth
- Keycloak (OIDC/SAML)
- Generic OIDC
- Amazon Cognito
- PingIdentity
- Microsoft AD FS
- Okta
- Shibboleth
- FreeIPA

## RBAC - Trzy poziomy
| Poziom | Opis |
|--------|------|
| **Global** | Uprawnienia na poziomie całego Rancher (np. tworzenie klastrów) |
| **Cluster** | Uprawnienia w ramach konkretnego klastra |
| **Project** | Uprawnienia w ramach projektu (namespace'y w klastrze) |

## Token Management
- Sesja domyślnie: 43200 min (30 dni)
- Max TTL: 129600 min (90 dni)
- SHA256 hashing opcjonalnie
- Bearer token + Kubeconfig

## Fleet (GitOps)
Fleet to wbudowany w Rancher system continuous delivery oparty na GitOps:
- Automatyczne wdrażanie z Git repo do klastrów
- Multi-cluster deployment
- Pull-based model (bezpieczniejszy niż push)
- Używany w tym projekcie do synchronizacji konfiguracji MAIN <-> DR

## Wersje dokumentacji
- v2.8, v2.9, v2.10, v2.11, v2.12, **v2.13 (najnowsza)**
- Wersje community + product + SRFA

## Ścieżka do dokumentacji
```
/home/pepe3535/addof/rancher-product-docs/versions/v2.13/modules/en/pages/
```

### Kluczowe pliki dokumentacji (v2.13)
- `about-rancher/what-is-rancher.adoc` - co to Rancher
- `about-rancher/architecture/` - architektura
- `getting-started/` - rozpoczęcie pracy
- `new-user-guides/authentication-permissions-and-global-configuration/` - auth & RBAC
- `how-to-guides/new-user-guides/manage-clusters/` - zarządzanie klastrami
- `integrations-in-rancher/fleet/` - Fleet/GitOps

## Ważne dla Tech Leada
1. Rancher to "single pane of glass" - klient zarządza WSZYSTKIM z jednego UI
2. AD integration jest krytyczna - bez niej klient nie może używać swoich kont
3. Backup Rancher ≠ Backup workloadów (to robi CloudCasa)
4. Fleet (GitOps) jest kluczowy dla spójności między MAIN i DR
5. Rancher zarządza ale NIE hostuje workloady AI - te działają na klastrach RKE2 downstream
