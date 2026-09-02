# Enlightened Services — Workflows

Centrale herbruikbare GitHub Actions workflows voor de **Enlightened Services** (`enlightenedservices`) GitHub Organisatie.

---

## Beschikbare Workflows

### 1. `reusable-test.yml`
Voert stationaire validatie en tests uit (standaard Vitest unit tests) op GitHub-hosted runners. Optioneel met code-coverage rapportage die als GitHub Actions artifact wordt geüpload. E2E (Playwright) is voorzien voor een latere fase.

#### Inputs
| Input | Type | Default | Beschrijving |
| :--- | :--- | :--- | :--- |
| `node_version` | `string` | `22` | Node.js versie |
| `test_command` | `string` | `pnpm test` | Testcommando; leeg schakelt het uit |
| `check_command` | `string` | *(leeg)* | Statische validatie (bv. `pnpm check`); leeg schakelt het uit |
| `coverage_command` | `string` | *(leeg)* | Coveragecommando (bv. `pnpm vitest run --coverage`); leeg schakelt het uit |
| `coverage_report_dir` | `string` | `coverage` | Map met het coverage rapport |
| `upload_artifacts` | `boolean` | `false` | Of het coverage rapport als artifact geüpload moet worden |
| `timeout_minutes` | `number` | `30` | Job timeout |

#### Secrets
| Secret | Vereist | Beschrijving |
| :--- | :--- | :--- |
| `ORG_READ_TOKEN` | nee | Fine-grained token met leestoegang tot private submodules |

> **Let op:** Repositories die `coverage_command` inschakelen moeten eerst de coverage provider (bv. `@vitest/coverage-v8`) en hun Vitest coverage config toevoegen.

#### Voorbeeld
```yaml
name: Tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    uses: enlightened-services/workflows/.github/workflows/reusable-test.yml@main
    with:
      test_command: pnpm test:unit
      coverage_command: pnpm vitest run --coverage
      upload_artifacts: true
    secrets:
      ORG_READ_TOKEN: ${{ secrets.SHARED_READ_TOKEN }}
```

---

### 2. `reusable-docker-build.yml`
Bouwt multi-arch (`linux/amd64`, `linux/arm64`) Docker images met Docker Buildx & QEMU, en pusht deze naar **GitHub Container Registry (GHCR)**.

#### Inputs
| Input | Type | Default | Beschrijving |
| :--- | :--- | :--- | :--- |
| `dockerfile` | `string` | *(verplicht)* | Pad naar de `Dockerfile` (bv. `./Dockerfile` of `./sib-studio/Dockerfile`) |
| `image_name` | `string` | *(verplicht)* | Naam van de image in GHCR (bv. `es-sib-studio`) |
| `context` | `string` | `.` | Docker build context directory |
| `platforms` | `string` | `linux/amd64,linux/arm64` | Doelplatformen |
| `cache_scope` | `string` | `default` | Unieke cache key scope |
| `push` | `boolean` | `true` | Of de image gepusht moet worden naar GHCR |

---

## Gebruiksvoorbeelden

### A. Vanuit een Monorepo (zoals `enlightened-services-apps`)

```yaml
# .github/workflows/app-sib-studio.yml
name: App — sib-studio

on:
  push:
    branches: [main]
    paths:
      - 'sib-studio/**'
      - 'shared/**'
      - 'package.json'
      - 'pnpm-lock.yaml'
  workflow_dispatch:

jobs:
  build:
    uses: enlightenedservices/workflows/.github/workflows/reusable-docker-build.yml@main
    with:
      context: .
      dockerfile: ./sib-studio/Dockerfile
      image_name: es-sib-studio
      cache_scope: sib-studio
```

### B. Vanuit een Standalone App Repository (bv. `enlightenedservices/sib-studio`)

```yaml
# .github/workflows/build.yml
name: Build & Push

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    uses: enlightenedservices/workflows/.github/workflows/reusable-docker-build.yml@main
    with:
      dockerfile: ./Dockerfile
      image_name: es-sib-studio
```

---

## Organisatie Toegang Instellen

Om deze workflow te kunnen aanroepen vanuit andere repositories binnen de organisatie:
1. Ga naar **Settings** $\rightarrow$ **Actions** $\rightarrow$ **General**.
2. Onder **Access**: vink **"Accessible from repositories in the 'enlightenedservices' organization"** aan.

---

## Self-Hosted Runner (Debian 13)

Zie [docs/runner-setup-debian-13.md](docs/runner-setup-debian-13.md) voor de inrichting van de barebones
Debian 13 (amd64) host met de org-level runner (Git, Docker+Buildx+QEMU, `$HOME`-fix en vereiste org-secrets).
