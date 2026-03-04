# AI Platform - Open Innovation (OICM+) v1.11.0

## Co to jest?
OICM+ (Open Innovation Cluster Manager) to bezpieczna, elastyczna platforma do orkiestracji infrastruktury GPU w środowiskach multi-tenant, multi-cluster i multi-cloud. Wersja w projekcie: **v1.11.0**.

Oficjalna dokumentacja: https://oicm.docs.openinnovation.ai/v1.11.0/

## Trzy główne warstwy

### 1. AI Cluster Manager (warstwa infra)
Zarządzanie zasobami GPU i orkiestracja klastrów.

| Obszar | Funkcje |
|--------|---------|
| **Resource Management** | Alokacja zasobów, zarządzanie GPU node'ami, multi-cluster management, monitoring i raporty zużycia |
| **Security** | Bezpieczne zarządzanie infrastrukturą AI, RBAC, izolacja danych |
| **Scalability** | Deployment i skalowanie AI workloadów, do tysięcy GPU node'ów |
| **Efficiency** | Optymalizacja dla inference, fine-tuning, training, job management |

**Autonomiczne operacje:**
- GPU provisioning i inteligentny job scheduling
- Job lifecycle: submit → track → monitor → reschedule → terminate
- Multi-level isolation: tenant → workspace → user → namespace
- Quota management na poziomie organizacji
- Auto-scaling, priorytetyzacja, failover handling
- Retry policies, checkpointing, recovery support
- Cost/usage tracking: GPU hours, storage, API calls, job history

### 2. AI Ops (warstwa ML/AI)
Kompleksowa platforma AI: od infrastruktury po deployment i bezpieczeństwo.

| Obszar | Funkcje |
|--------|---------|
| **Training & Fine-Tuning** | UI-based fine-tuning multi-GPU/nodes, distributed training, optymalizacja treningu, **LoRA adapters** |
| **Deployment & Inference** | Skalowalne deployowanie modeli, elastyczne Inference APIs (REST + SDK), optymalizacja alokacji, model agnostic |
| **Notebooks** | Alokacja i zarządzanie notebookami, idle notebook optimization, wersjonowanie i sharing |
| **Resource Management** | Alokacja per projekt/zespół, scheduling optymalizacji workloadów |
| **Collaboration & Versioning** | Narzędzia kolaboracji, **wersjonowanie modeli**, **wersjonowanie datasetów** |

**Dodatkowe funkcje AI Ops (z dokumentacji):**
- Model Hub - odkrywanie modeli
- Model registration, versioning, bundling
- Custom Docker images
- Performance benchmarking
- Knowledge i ASR benchmarking
- Experiment i run tracking
- Blueprint creation (workflow automation)
- Dataset i dataframe management z API
- API key management per workspace

### 3. Gen AI Studio — ~~OPCJONALNY~~ **WYŁĄCZONY Z SCOPE (2026-02-24)**
> **Decyzja:** GenAI Studio NIE będzie wdrażane w ramach tego projektu.

Interfejs do pracy z generatywnym AI (referencyjnie):

| Obszar | Funkcje |
|--------|---------|
| **Model Integration** | Dostęp do modeli z prywatnych źródeł, modele proprietary, modele open-source |
| **Knowledge Base Management** | Przetwarzanie danych niestrukturalnych, RBAC, zarządzanie embeddingami |
| **RAG Builder** | Budowanie aplikacji RAG, prosty UX |
| **Agentic Workflows** | Projektowanie i automatyzacja workflow z AI agentami, redukcja ręcznej pracy |

## Integracje (z dokumentacji oficjalnej)

| Kategoria | Integracje |
|-----------|-----------|
| **Identity** | Keycloak, LDAP, Active Directory, OAuth2, SAML |
| **Monitoring** | Prometheus, Grafana, ELK/EFK stacks |
| **Model Sources** | Hugging Face, wewnętrzne repozytoria |
| **Billing** | GPU hours, token usage, API calls, storage metrics export |
| **APIs** | RESTful interfaces + SDK do programmatic control |
| **Hardware** | NVIDIA GPU, AMD (ROCm), extensible na przyszły hardware |

## Security & Data Isolation (szczegóły z docs)

### RBAC
- Enforced na poziomie: **cluster → tenant → workspace**
- Fine-grained permissions per level
- Integracja z Keycloak SSO (OAuth2, SAML)
- Integracja z LDAP / Active Directory
- Scoped API keys per workspace
- Full audit logging (user actions, admin activities, system operations)

### Data Isolation - trzy warstwy
| Warstwa | Mechanizm |
|---------|-----------|
| **Database** | Unique tenant ID per record, queries auto-scoped do tenanta |
| **Storage** | Dedicated folders per tenant (NFS/object storage), tenant-specific credentials |
| **Network** | Kubernetes network segmentation via **Calico**, firewall rules per pod/namespace z tenant labels |

### Modele deploymentu
| Model | Opis | Kiedy |
|-------|------|-------|
| **Shared Cluster** | Wielu tenantów, izolacja via K8s namespaces + Calico | Standard |
| **Dedicated Cluster** | Jeden tenant per klaster | High-security, gov, air-gapped |

## SUSE AI Suite
OICM jest częścią ekosystemu SUSE AI:
- SUSE AI Suite (cały pakiet), SUSE Application Collection, SUSE Observability
- SUSE Security, SUSE Storage (Longhorn), Rancher, RKE/K3s, SUSE Linux, Harvester

### Licencjonowanie
| Metryka | Zastosowanie |
|---------|-------------|
| **vCPU** (Cloud/Virtual) | 2 Cores lub 4 vCPUs (hyperthreading) |
| **Socket** (Bare-Metal) | 2 Sockets, do 64 Cores |

## NVAIE (NVIDIA AI Enterprise)
- Licencje WYMAGANE na serwerach GPU (jeśli OI nie jest w scope)
- 8 licencji per serwer XE7745

## Ważne dla Tech Leada
1. **OICM ma własny RBAC + Keycloak** - trzeba wyjaśnić integrację z Rancher RBAC i AD klienta
2. **LoRA fine-tuning** jest wprost wspierany - to standard dla enterprise LLM customization
3. **CNI** - OICM i SUSE dogadają się i wspólnie wybiorą konkretny CNI (decyzja 2026-02-24)
4. **Shared cluster z logiczną izolacją** - 2 fizyczne klastry RKE2, 5 logicznych podziałów (namespace'y + OICM quotas)
5. **Model versioning + dataset versioning** - jest wbudowane, nie trzeba zewnętrznego MLflow
6. **REST API + SDK** - klient może programatycznie zarządzać platformą
7. **~~Gen AI Studio jest OPCJONALNY~~** → **Gen AI Studio WYŁĄCZONE z scope** (decyzja 2026-02-24)
8. **Billing/metering** - OICM eksportuje metryki GPU hours/storage/API calls - ważne dla cost allocation
9. **Checkpointing + retry** - autonomiczne zarządzanie long-running training jobs
10. **Blueprint creation** - workflow automation, ważne dla powtarzalnych pipeline'ów
