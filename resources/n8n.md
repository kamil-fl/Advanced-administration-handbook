# Rozszerzenia i ulepszenia dla instancji n8n

Ten dokument zbiera praktyczne sposoby na **rozszerzenie** i **ulepszenie** instancji n8n. Zawiera zarówno dodatki funkcjonalne (community nodes), jak i konfiguracje poprawiające bezpieczeństwo, wydajność oraz niezawodność.

## 1) Rozszerzenia funkcjonalne (community nodes)

### Włączanie i instalacja community nodes

1. **Włącz instalację zewnętrznych pakietów** (tylko, jeśli rozumiesz ryzyko i masz politykę bezpieczeństwa):
   - W self-hosted n8n ustaw zmienne środowiskowe:
     - `N8N_COMMUNITY_PACKAGES_ENABLED=true`
     - (opcjonalnie) `N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true` (pozwala na narzędzia w community packages; używaj ostrożnie).
2. **Zainstaluj pakiety** przez interfejs n8n (Community Nodes) lub CLI, zgodnie z procedurą n8n.
3. **Zdefiniuj listę dozwolonych pakietów** w polityce organizacji (np. tylko podpisane i utrzymywane).

### Co instalować?

Poniżej kategorie, które zwykle przynoszą najwięcej korzyści:

- **Integracje SaaS/CRM/ERP** (np. dodatkowe API usług branżowych).
- **Rozszerzenia AI/LLM** (dodatkowe konektory do modeli lub wektorowych baz danych).
- **Narzędzia DevOps** (monitoring, CI/CD, ticketing, komunikatory).
- **Źródła danych** (bazy danych i hurtownie, API branżowe).
- **Obsługa dokumentów** (PDF, OCR, konwersje formatów).

> Uwaga: zamiast próbować „zainstalować wszystko”, wybieraj **konkretne pakiety** pod swoje procesy. Ułatwia to utrzymanie i bezpieczeństwo.

### AI, modele i multi-agentowe automatyzacje

Jeśli Twoim celem jest praca z AI i budowa agentów, skup się na integracjach, które umożliwiają:

- **Dostęp do modeli LLM** (np. dostawcy chmurowi i lokalni).
- **Orkiestrację agentów** (narzędzia do planowania zadań, pamięci i narzędzi zewnętrznych).
- **Wektorowe bazy danych** do wyszukiwania semantycznego (np. rozwiązania hostowane lub self-hosted).
- **Łączenie narzędzi** (HTTP, bazy danych, pliki, komunikatory).

W praktyce oznacza to połączenie wbudowanych node’ów n8n (HTTP Request, Code, AI Agent, bazy danych) oraz community nodes dla konkretnych dostawców. Nie wszystkie pakiety będą Ci potrzebne — dobieraj je pod swoje procesy.

## 2) Ulepszenia bezpieczeństwa

- **Oddzielne środowiska** (dev/test/prod) i osobne bazy danych dla każdego środowiska.
- **Wymuszanie HTTPS** na reverse proxy (np. Nginx/Traefik) i poprawne ustawienie `N8N_HOST`, `N8N_PROTOCOL`, `WEBHOOK_URL`.
- **Silne szyfrowanie**:
  - Ustaw `N8N_ENCRYPTION_KEY` (stały, bezpieczny klucz) dla danych uwierzytelniających.
- **Ograniczenia dostępu**:
  - Ogranicz dostęp do edytora (IP allowlist, VPN, SSO).
  - Włącz MFA/SSO, jeśli jest dostępne w Twojej edycji.
- **Zasada najmniejszych uprawnień**:
  - Dedykowane konta serwisowe dla integracji, minimalne scope’y tokenów.

## 3) Ulepszenia wydajności i skalowania

- **Tryb kolejkowy (queue mode)** dla skalowania i niezawodności:
  - Oddzielny worker/workerzy i serwer webowy.
  - Zewnętrzna kolejka (np. Redis) zgodnie z zaleceniami n8n.
- **Podział obciążeń**:
  - Oddziel procesy do webhooków i do pracy w tle.
- **Optymalizacja pamięci**:
  - Włącz przechowywanie binariów poza bazą (`N8N_BINARY_DATA_MODE=filesystem`), jeśli obsługujesz duże pliki.

## 4) Szybkie wdrożenie z HTTPS (docker-compose)

Poniżej przykład **minimalnego** wdrożenia z reverse proxy. Działa na typowych VPS-ach (np. Mikrus), a HTTPS zapewnia Caddy. Dostosuj domenę, hasła i ścieżki.

```yaml
version: "3.8"
services:
  n8n:
    image: n8nio/n8n:latest
    restart: unless-stopped
    environment:
      - N8N_HOST=automation.example.com
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://automation.example.com/
      - N8N_ENCRYPTION_KEY=__WSTAW_LOSOWY_32_ZNAKOWY_KLUCZ__
      - N8N_COMMUNITY_PACKAGES_ENABLED=true
      - N8N_BINARY_DATA_MODE=filesystem
    volumes:
      - n8n_data:/home/node/.n8n
  caddy:
    image: caddy:latest
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - caddy_data:/data
      - caddy_config:/config
volumes:
  n8n_data:
  caddy_data:
  caddy_config:
```

`Caddyfile`:

```
automation.example.com {
  reverse_proxy n8n:5678
}
```

Po uruchomieniu `docker-compose up -d` Caddy automatycznie pobierze certyfikat TLS.

## 5) Czyszczenie i ponowne wdrożenie (docker-compose)

Jeśli musisz „wyczyścić wszystko” i postawić instancję od nowa:

```bash
docker-compose down -v --remove-orphans
docker image prune -a -f
docker-compose up -d
```

## 6) Niezawodność i monitoring

- **Monitoring**:
  - Eksport metryk (Prometheus/Grafana) lub zewnętrzne APM.
- **Logowanie**:
  - Ustaw poziomy logów i centralizację (np. ELK/Opensearch).
- **Kopie zapasowe**:
  - Backup bazy danych i danych binarnych, testy odtwarzania.

## 7) Ulepszenia UX i zarządzania

- **Standaryzacja workflow**:
  - Konwencje nazewnictwa workflowów, tagi, foldery, readme w opisie.
- **Biblioteka szablonów**:
  - Wspólne workflowy jako szablony do reużycia.
- **Walidacja danych wejściowych**:
  - Ujednolicone node’y do sanitizacji, walidacji, mapowania.

## 8) Najlepsze praktyki wdrożeniowe

- **Separacja sekretów**:
  - Nie przechowuj sekretów w workflowach; używaj managerów sekretów (np. Vault).
- **Zarządzanie zmianami**:
  - Wersjonowanie workflowów i kontrola zmian w repozytorium.
- **Testy**:
  - Automatyczne testy krytycznych workflowów, scenariusze „happy path” i błędów.

## 9) Szybka lista kontrolna

- [ ] Włączone community nodes z polityką bezpieczeństwa.
- [ ] Zdefiniowane środowiska i separacja bazy danych.
- [ ] Ustawiony `N8N_ENCRYPTION_KEY`.
- [ ] HTTPS i poprawne `WEBHOOK_URL`.
- [ ] Tryb kolejkowy + workerzy (dla większego obciążenia).
- [ ] Monitoring i logowanie.
- [ ] Backup i testy odtwarzania.

## Gdzie dodać konkretne rozszerzenia?

Jeśli chcesz, dodaj listę **konkretnych integracji** (np. nazwy usług, bazy danych, CRM), a przygotuję listę community nodes i konfiguracji dla Twojego przypadku.
