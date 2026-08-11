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
- Repositorio espejo creado con la convención `<repo>-security-check` (ver [Convención de nombres del repositorio espejo](#convención-de-nombres-del-repositorio-espejo)).
- Proyecto creado en Checkmarx.

---

# Instalación

Cada proyecto debe crear el archivo `.github/workflows/sync.yml` con el contenido indicado en la sección [Configuración](#configuración).

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
    # Recomendado: referenciar el workflow por tag de release o SHA de commit
    # en lugar de @main, para evitar cambios no auditados (supply chain).
    uses: acacoop/Security-Workflows/.github/workflows/repository-sync.yml@main
    with:
      allowed_source_branch: preprod
    secrets:
      SYNC_TOKEN: ${{ secrets.SYNC_TOKEN }}
```

> El repositorio espejo **no se configura**: se deriva automáticamente del nombre del repositorio con la convención `<repo>-security-check` (ver [Convención de nombres del repositorio espejo](#convención-de-nombres-del-repositorio-espejo)).

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

### Convención de nombres del repositorio espejo

El repositorio espejo destino **no se configura por parámetro**: el workflow lo deriva automáticamente del nombre del repositorio fuente aplicando la convención:

```text
<nombre-del-repo>-security-check
```

Ejemplos:

| Repositorio fuente     | Repositorio espejo               |
| ---------------------- | -------------------------------- |
| `acacoop/mi-app`       | `acacoop/mi-app-security-check`  |
| `acacoop/api-pagos`    | `acacoop/api-pagos-security-check` |

> **Seguridad:** el nombre del repo fuente proviene de `github.repository`, un valor asignado por GitHub que no puede falsificarse. Por lo tanto, cada repositorio solo puede escribir en **su propio** espejo: es imposible que un proyecto pise el espejo de otro. Esta convención reemplaza la necesidad de mantener una allowlist central.

**Al dar de alta un proyecto nuevo** solo hay que:

1. Crear el repositorio espejo (privado) siguiendo la convención: `<repo>-security-check`.
2. Otorgar acceso de escritura al espejo al `SYNC_TOKEN` (si es fine-grained o GitHub App).
3. Crear el `sync.yml` en el repo fuente (ver [Configuración](#configuración)).
4. Crear el proyecto en Checkmarx apuntando al espejo.

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
