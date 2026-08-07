# Checkmarx Repository Sync

Este repositorio utiliza el workflow corporativo de sincronización hacia Checkmarx para implementar el Security Gate requerido para promociones a producción.

## Objetivo

Permitir que el código sea analizado por Checkmarx utilizando un repositorio espejo centralizado, reduciendo el consumo de licencias y garantizando que únicamente se promocione código previamente validado.

---

# Arquitectura

```text
feature/*
    ↓
preprod
    ↓
Workflow Sync
    ↓
Repositorio Checkmarx
    ↓
Checkmarx Scan
    ↓
PASS
    ↓
PR preprod → main
    ↓
checkmarx-validation
    ↓
Producción
```

---

# Requisitos Previos

Antes de implementar este workflow se debe contar con:

- Acceso al repositorio de workflows corporativos.
- Permisos para utilizar workflows reutilizables.
- Secret organizacional `SYNC_TOKEN`.
- Repositorio espejo configurado para Checkmarx.
- Proyecto creado en Checkmarx.

---

# Instalación

Crear el siguiente archivo:

```text
.github/workflows/sync.yml
```

Contenido:

```yaml
name: Sync Repository to Checkmarx

on:
  push:
    branches:
      - main

  workflow_dispatch:

jobs:
  sync:
    uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
    with:
      target_repo: repo-pruebaSec-destino
    secrets: inherit
```

---

# Configuración

Cada proyecto debe definir únicamente el repositorio espejo donde se sincronizará el código para su análisis en Checkmarx.

Ejemplo:

```yaml
jobs:
  sync:
    uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
    with:
      target_repo: repo-pruebaSec-destino
    secrets: inherit
