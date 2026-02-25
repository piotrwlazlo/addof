# Pytania na spotkanie z architektem

## Słowniczek (przeczytaj przed spotkaniem)

**Failover** - automatyczne lub manualne przełączenie z systemu głównego (MAIN) na zapasowy (DR) gdy główny przestaje działać. Wyobraź sobie że masz dwa identyczne biura - jedno główne i jedno zapasowe. Gdy w głównym wyłączą prąd, przenosisz ludzi do zapasowego. Failover to właśnie ten proces "przenoszenia się". Kluczowe pytanie: czy to automatyczne (system sam przełącza) czy manualne (ktoś musi kliknąć)?

**Control Plane** - "mózg" klastra Kubernetes. Nie uruchamia Twoich kontenerów z AI - on nimi ZARZĄDZA. Decyduje gdzie uruchomić pod, restartuje go jak padnie, skaluje gdy trzeba więcej. Bez control plane klaster działa (pody żyją) ale jest "głuchy" - nie możesz nic zmienić, zdeployować, usunąć.

**CNI (Container Network Interface)** - plugin który decyduje jak kontenery komunikują się ze sobą w Kubernetes. Pomyśl o tym jak o "operatorze telefonicznym" w klastrze. **Canal** i **Calico** to dwie różne "firmy telekomunikacyjne" - nie są w pełni kompatybilne. Canal jest domyślny w SUSE, ale OICM wymaga Calico do izolacji tenantów.

**RBAC** - system uprawnień "kto może co robić". Jak w firmie: dyrektor widzi wszystko, manager widzi swój dział, pracownik widzi swoje projekty. W tym projekcie RBAC jest w TRZECH miejscach: Rancher, OICM (przez Keycloak) i Active Directory klienta.

**Air-gapped** - środowisko odcięte od internetu. Jak komputer w banku który nie ma dostępu do sieci. Ma ogromny wpływ na wdrożenie bo WSZYSTKO (obrazy kontenerów, modele AI, aktualizacje) musi być dostarczone ręcznie na nośnikach lub przez wewnętrzną sieć.

**Overlay network** - "wirtualna sieć" nakładana na fizyczną sieć w Kubernetes. Domyślnie CAŁY ruch między kontenerami idzie przez overlay (wolniejszy, ~25GbE). Problem: GPU powinny rozmawiać przez fizyczną sieć 400GbE, nie przez overlay. Jeśli nikt tego nie skonfiguruje - GPU mają Ferrari (400GbE) ale jeżdżą po polnej drodze (overlay 25GbE).

---

## Pytanie 1: Jak OICM jest zainstalowany na tych 5 klastrach GPU?

### Kontekst do zrozumienia
W naszym projekcie mamy 5 klastrów Kubernetes z GPU (C1-C5). OICM to platforma AI która nimi zarządza. Ale OICM sam musi gdzieś "mieszkać" - i tu jest pytanie: gdzie?

Są dwie opcje:
- **Jedna centralna instancja OICM** (np. na oddzielnej VM na Harvester) która zarządza wszystkimi 5 klastrami zdalnie. Jak jeden pilot sterujący 5 dronami.
- **Osobna instancja OICM na każdym klastrze** - 5 oddzielnych platform. Jak 5 pilotów, każdy ze swoim dronem.

Pierwsza opcja jest prostsza operacyjnie ale wymaga dobrej sieci między OICM a klastrami. Druga opcja jest bardziej odporna na awarie ale 5x więcej pracy z utrzymaniem.

### Co zapytać
"Jak OICM jest rozłożony na klastrach - czy jest jedna centralna instancja zarządzająca wszystkimi klastrami, czy każdy klaster ma swoją? Gdzie fizycznie działa OICM control plane - na Harvester VM czy na jednym z GPU serwerów?"

---

## Pytanie 2: Jak łączą się systemy logowania - AD, Keycloak, Rancher?

### Kontekst do zrozumienia
Klient ma Active Directory (AD) - to jego "książka telefoniczna" z kontami pracowników. Gdy ktoś odchodzi z firmy, usuwają go z AD i traci dostęp do wszystkiego.

W naszym projekcie mamy **trzy systemy** które potrzebują wiedzieć "kim jest ten użytkownik":
1. **Rancher** - zarządzanie klastrami (admini infra)
2. **OICM** (przez Keycloak) - platforma AI (data scientists)
3. **Active Directory** klienta - źródło prawdy o użytkownikach

Problem: jeśli te systemy nie są połączone w jedną spójną ścieżkę, to:
- Ktoś jest usunięty z AD ale nadal ma dostęp do OICM
- Data scientist musi pamiętać 2 różne hasła
- Admin musi ręcznie synchronizować uprawnienia w 2 miejscach

Ideał: AD → Keycloak (pośrednik) → zarówno Rancher jak i OICM. Jedno konto, jedno hasło, jedno miejsce zarządzania.

### Co zapytać
"Jak wygląda flow logowania użytkownika end-to-end? Czy data scientist loguje się jednym kontem AD do OICM? Czy Keycloak jest centralnym brokerem tożsamości dla całej platformy (i Rancher i OICM), czy Rancher ma osobną integrację z AD?"

---

## Pytanie 3: Czy OICM wymaga Calico - i czy to działa z RKE2?

### Kontekst do zrozumienia
OICM izoluje dane między zespołami (tenantami) za pomocą reguł sieciowych - "zespół A nie widzi danych zespołu B". Do tego używa **Calico** - specyficznego pluginu sieciowego.

Problem: RKE2 (nasz Kubernetes) domyślnie używa **Canal**, a nie Calico. To jak próba podłączenia ładowarki iPhone do Samsunga - podobne ale nie kompatybilne.

Jeśli OICM WYMAGA Calico a my mamy Canal na klastrach - trzeba to zmienić. Zmiana CNI na działającym klastrze to przeinstalowanie sieci = ryzyko przestoju. Lepiej wiedzieć TERAZ niż odkryć to w trakcie wdrożenia.

### Co zapytać
"OICM używa Calico do izolacji sieciowej między tenantami. Klastry RKE2 domyślnie mają Canal. Czy testowaliście OICM z Canal? Czy musimy skonfigurować RKE2 z Calico od początku?"

---

## Pytanie 4: Czy distributed training faktycznie używa 400GbE fabric?

### Kontekst do zrozumienia
Distributed training to trenowanie jednego modelu AI na wielu GPU w wielu serwerach jednocześnie. GPU muszą ciągle wymieniać dane między sobą (gradienty). Im szybsza wymiana → tym szybszy training.

W projekcie mamy dedykowaną sieć 400GbE (szybka) do tego celu. ALE - domyślnie Kubernetes kieruje ruch kontenerów przez overlay network (~25GbE, wolna). To jak mieć autostradę obok ale GPS ciągle kieruje przez miasto.

Żeby GPU faktycznie używały 400GbE trzeba specjalnej konfiguracji (SR-IOV, host networking, lub Multus). OICM mówi "wspieramy distributed training" ale nie precyzuje JAK ruch GPU trafia na fizyczną sieć 400GbE.

### Co zapytać
"OICM wspiera distributed training multi-node. Jak ruch NCCL (komunikacja między GPU) trafia na 400GbE backend fabric? Czy to przez SR-IOV, host networking, czy Multus? Czy macie to przetestowane na RKE2 z ConnectX-7?"

---

## Pytanie 5: Gen AI Studio - jest w scope czy nie?

### Kontekst do zrozumienia
OICM ma 3 warstwy:
- **AI Cluster Manager** - zarządzanie GPU i klastrami (infra)
- **AI Ops** - training, inference, notebooks (platforma ML)
- **Gen AI Studio** - RAG, knowledge base, agenci AI (aplikacje generatywne) ← **OPTIONAL**

Gen AI Studio jest oznaczony jako OPCJONALNY, ale to właśnie ta warstwa daje największą wartość biznesową klientowi:
- **RAG Builder** - klient buduje chatbota na swoich dokumentach bez kodowania
- **Knowledge Base** - zarządzanie bazą wiedzy dla LLM
- **Agentic Workflows** - automatyzacja procesów z AI agentami

Bez Gen AI Studio klient dostaje "platformę do odpalania modeli" ale nie "narzędzie do budowania aplikacji AI". To duża różnica w propozycji wartości.

### Co zapytać
"Gen AI Studio jest oznaczony jako opcjonalny. Czy jest w scope tego projektu? Jeśli tak - kto odpowiada za wdrożenie i integrację z Vector DB na PowerStore? Jeśli nie - czy klient o tym wie i czy planuje go w przyszłości?"

---

## Pytanie 6: Jak działa DR (disaster recovery) dla platformy OICM?

### Kontekst do zrozumienia
DR to plan na wypadek katastrofy - co robimy gdy główne data center przestaje działać.

W naszym projekcie mamy 5 oddzielnych narzędzi do DR:
| Co chroni | Narzędzie | Vendor |
|-----------|-----------|--------|
| Pliki AI (datasety, modele) | SyncIQ | Dell |
| Bazy danych (Vector DB) | CSM Replication | Dell |
| Kubernetes (pody, konfiguracja) | CloudCasa | Catalogic |
| Maszyny wirtualne | VM Replication | SUSE |
| Konfiguracja deploymentów | GitOps/Fleet | SUSE |

**Problem 1:** Nikt nie koordynuje tych 5 narzędzi. To jak ewakuacja budynku gdzie straż pożarna, policja, pogotowie i ochrona budynku działają osobno bez wspólnego dowodzenia.

**Problem 2:** OICM ma własny stan (konta użytkowników w Keycloak, konfiguracja tenantów, quotas, model registry, audit logi). CloudCasa chroni Kubernetes ale NIE koniecznie wewnętrzne bazy danych OICM. Po failoverze możemy mieć działający klaster ale pustą platformę AI.

### Co zapytać
"Jak wygląda DR dla warstwy OICM? Czy Keycloak, model registry i konfiguracja tenantów są objęte backupem CloudCasa? Czy jest zdefiniowana kolejność failover - co uruchamiamy najpierw, co potem? Czy istnieje runbook opisujący cały proces krok po kroku?"

---

## Pytanie 7: Czy środowisko będzie air-gapped (odcięte od internetu)?

### Kontekst do zrozumienia
Duzi klienci enterprise często nie pozwalają na dostęp do internetu z serwerów produkcyjnych (bezpieczeństwo). To się nazywa "air-gapped".

Wpływ na nasz projekt jest ogromny:
- **Modele AI** - skąd pobrać LLM jeśli nie ma Hugging Face? Trzeba mirror albo ręczny transfer
- **Container images** - każdy kontener (OICM, Rancher, GPU Operator) musi być pobrany z internetu. Bez niego potrzebny lokalny registry (np. Harbor)
- **Aktualizacje** - OS, CUDA, drivers, security patche - wszystko trzeba dostarczyć offline
- **CloudCasa** - wymaga "outbound connectivity" do control plane - czy to działa w air-gap?

Jeśli klient oczekuje air-gapped a architektura tego nie przewiduje, to dodatkowe tygodnie pracy na budowę mirror infrastructure.

### Co zapytać
"Czy środowisko klienta jest air-gapped czy ma dostęp do internetu? Czy jest planowany lokalny container registry? Jak będą dostarczane modele AI i aktualizacje software?"

