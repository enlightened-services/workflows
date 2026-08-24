# Enlightened Services — Workflows

Centrale herbruikbare GitHub Actions workflows voor de **Enlightened Services** (`enlightenedservices`) GitHub Organisatie.

---

## Beschikbare Workflows

### 1. `reusable-docker-build.yml`
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
