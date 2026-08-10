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
      - preprod

  workflow_dispatch:

jobs:
  sync:
    # Recomendado: referenciar el workflow por tag de release o SHA de commit
    # en lugar de @main, para evitar cambios no auditados (supply chain).
    uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
    with:
      target_repo: repo-pruebaSec-destino
      allowed_source_branch: preprod
    secrets:
      SYNC_TOKEN: ${{ secrets.SYNC_TOKEN }}
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
      - preprod

  workflow_dispatch:

jobs:
  sync:
    uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
    with:
      target_repo: repo-pruebaSec-destino
      allowed_source_branch: preprod
    secrets:
      SYNC_TOKEN: ${{ secrets.SYNC_TOKEN }}
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
  - preprod
```

En este caso, cada cambio enviado a la rama `preprod` (stage) ejecutará automáticamente el proceso de sincronización. La rama debe coincidir con el parámetro `allowed_source_branch`.

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

> **Seguridad:** el workflow reutilizable valida internamente una allowlist de mapeos `repo fuente → repo espejo`. Al dar de alta un nuevo proyecto, se debe agregar el mapeo correspondiente en `repository-sync.yml` (paso *Validate source repo -> target repo mapping*). Un repositorio no autorizado no podrá escribir en el espejo de otro proyecto.

---

### allowed_source_branch

Rama del repositorio fuente desde la cual está permitido sincronizar. El workflow falla si el sync se dispara desde cualquier otra rama, evitando que una feature branch sin revisar pise el espejo.

Ejemplo:

```yaml
allowed_source_branch: preprod
```

---

### secrets

Los secretos deben pasarse de forma **explícita** al workflow reutilizable. No usar `secrets: inherit`, ya que heredaría *todos* los secretos de la organización al workflow, ampliando innecesariamente la superficie de exposición.

Ejemplo:

```yaml
secrets:
  SYNC_TOKEN: ${{ secrets.SYNC_TOKEN }}
```

El workflow requiere la existencia del secreto organizacional:

```text
SYNC_TOKEN
```

utilizado para autenticarse contra el repositorio destino y realizar la sincronización.

> **Seguridad:** el `SYNC_TOKEN` debe ser un **fine-grained token o GitHub App** con permiso `contents: write` únicamente sobre los repositorios espejo. No utilizar un PAT clásico con acceso de escritura a toda la organización.

---

# Recomendaciones de Seguridad Adicionales

Estas medidas complementan el workflow y deben configurarse fuera de este repositorio:

- **Estado del commit publicado solo por Checkmarx:** el estado (válido/inválido) del commit debe ser publicado exclusivamente por la GitHub App o servicio de Checkmarx mediante la Checks/Status API, nunca por workflows o usuarios con permisos genéricos.
- **Branch protection en `main`:** exigir el check de Checkmarx como *required status check* en el repositorio de desarrollo, verificando el SHA exacto del head del PR `preprod → main`.
- **Repositorio espejo privado:** el espejo debe ser privado; el clone ya se realiza autenticado con el `SYNC_TOKEN`.
- **Referencias inmutables:** consumir este workflow reutilizable por tag de release o SHA de commit en lugar de `@main`.
