# Dokumentacja Techniczna - Struktura Repo

## Struktura repozytorium
```
/home/pepe3535/addof/
├── project_description.md              ← Główny opis projektu (671 linii)
├── harvester-product-docs/             ← Docs SUSE Virtualization (Git repo)
│   ├── versions/v1.5, v1.6, v1.7/     ← Aktywne wersje
│   ├── archived-versions/v1.3, v1.4/  ← Archiwum
│   ├── Makefile                        ← Build targets
│   ├── package.json                    ← npm dependencies
│   ├── playbook-local.yml              ← Antora build config (local)
│   └── playbook-remote.yml             ← Antora build config (remote)
│
└── rancher-product-docs/               ← Docs SUSE Rancher Manager (Git repo)
    ├── versions/v2.8-v2.13/           ← Wersje produktowe
    ├── community-docs/v2.9-v2.13/     ← Wersje community
    ├── community-supplemental-files/   ← CSS, JS, fonty, ikony
    ├── shared/modules/                 ← Współdzielone moduły
    ├── Makefile                        ← Build targets (6 wariantów)
    ├── package.json                    ← npm dependencies
    └── playbook-*.yml                  ← 6 playbook'ów Antora
```

## Technologia dokumentacji

| Aspekt | Narzędzie |
|--------|-----------|
| **Generator** | Antora 3.1.9 (static site generator) |
| **Format treści** | AsciiDoc (.adoc) |
| **Build system** | npm + Make |
| **Search** | Lunr.js (client-side full-text search) |
| **UI Theme** | dsc-style-bundle (Bulma CSS) |
| **CI/CD** | GitHub Actions |
| **Języki** | EN (oba), ZH (tylko Rancher) |

## Jak budować dokumentację

### Harvester docs
```bash
cd /home/pepe3535/addof/harvester-product-docs
npm ci                           # instalacja dependencies
make local                       # budowanie → build/site/
make preview                     # serwer na port :8000
```

### Rancher docs
```bash
cd /home/pepe3535/addof/rancher-product-docs
npm ci
make product-local               # docs produktowe → build/site-product-local/
make community-local             # docs community → build/site-community-local/
make preview-product-local       # serwer na port :8081
make preview-community-local     # serwer na port :8080
```

## Najważniejsze ścieżki do dokumentacji

### Harvester v1.7 (najnowsza wersja)
```
harvester-product-docs/versions/v1.7/modules/en/pages/
├── introduction/overview.adoc
├── installation/
├── cluster-management/
├── networking/
│   └── deep-dive.adoc          ← szczegóły architektury sieciowej
├── storage/
├── virtual-machines/
├── observability/
└── troubleshooting/
```

### Rancher v2.13 (najnowsza wersja)
```
rancher-product-docs/versions/v2.13/modules/en/pages/
├── about-rancher/
│   ├── what-is-rancher.adoc
│   └── architecture/           ← architektura Rancher
├── getting-started/
├── new-user-guides/
│   └── authentication-permissions-and-global-configuration/  ← auth & RBAC
├── how-to-guides/
│   └── new-user-guides/manage-clusters/
├── integrations-in-rancher/
│   └── fleet/                  ← GitOps
└── troubleshooting/
```

### Harvester API
```
harvester-product-docs/versions/v1.7/modules/en/attachments/dev-swagger.json
```

## Ważne dla Tech Leada
1. Dokumentacja SUSE to **reference** - zawsze sprawdzaj tu w razie wątpliwości
2. Wersje v1.7 (Harvester) i v2.13 (Rancher) to **najnowsze** - preferuj te
3. Docs budują się jako static site - można je hostować lokalnie do przeglądania offline
4. Rancher ma **community** i **product** wersje docs - product ma więcej
5. Dokumentacja w formacie AsciiDoc jest czytelna nawet bez budowania (plain text)
