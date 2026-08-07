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

Cada proyecto debe crear el archivo:

```text
.github/workflows/sync.yml
```

con el siguiente contenido:

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

## Parámetros del Workflow

### name

Nombre que aparecerá en GitHub Actions para identificar el workflow.

Ejemplo:

```yaml
name: Sync Repository to Checkmarx
```

---

### branches

Define qué rama disparará automáticamente la sincronización hacia el repositorio espejo.

Ejemplo:

```yaml
branches:
  - main
```

En este caso, cada cambio enviado a la rama `main` ejecutará automáticamente el proceso de sincronización.

---

### workflow_dispatch

Permite ejecutar el workflow manualmente desde la interfaz de GitHub Actions.

Ejemplo:

```yaml
workflow_dispatch:
```

Es útil para pruebas, validaciones o reejecuciones sin necesidad de realizar nuevos commits.

---

### uses

Referencia al workflow reutilizable corporativo responsable de realizar la sincronización.

Ejemplo:

```yaml
uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
```

Donde:

- `acacoop/Security-Workflows` es el repositorio que contiene el workflow reutilizable.
- `.github/workflows/repository-sync.yml` es el workflow que ejecutará la sincronización.
- `@main` indica la rama desde la cual se utilizará dicho workflow.

---

### target_repo

Nombre del repositorio espejo destino donde se copiará el código para su análisis en Checkmarx.

Ejemplo:

```yaml
target_repo: repo-pruebaSec-destino
```

Cada proyecto debe utilizar su propio repositorio espejo.

---

### secrets: inherit

Permite heredar automáticamente los secretos definidos a nivel de organización o repositorio para que puedan ser utilizados por el workflow reutilizable.

Ejemplo:

```yaml
secrets: inherit
```

Esto evita tener que redefinir los mismos secretos en cada repositorio que implemente el template.

El workflow requiere la existencia del secreto organizacional:

```text
SYNC_TOKEN
```

utilizado para autenticarse contra el repositorio destino y realizar la sincronización.
