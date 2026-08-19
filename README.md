# GitHub Project - Automatización del flujo Dev → QA → Main

## Descripción

Este repositorio contiene una prueba del flujo de gestión de historias de usuario mediante **GitHub Projects + GitHub Actions**.

El tablero utiliza dos campos principales:

### Status

- `Backlog`
- `Ready`
- `In Progress`
- `In Review`
- `Done`

### Environment

- `Dev`
- `QA`
- `Production`

El flujo de código utiliza tres ramas principales:

- `dev`
- `qa`
- `main`

No se trabaja directamente sobre las ramas principales. Para cada HU se utiliza una rama hija.

---

## Flujo general

El flujo de una HU es:

```text
Backlog
   ↓
Ready
   ↓
In Progress / Dev
   ↓
PR → dev
   ↓
In Review / Dev
   ↓
Merge → dev
   ↓
In Progress / QA
   ↓
PR → qa
   ↓
In Review / QA
   ↓
Merge → qa
   ↓
In Progress / Production
   ↓
PR → main
   ↓
In Review / Production
   ↓
Merge → main
   ↓
Done / Production
```

---

## 1. Crear la HU

La HU se crea como una **Issue de GitHub** y se agrega automáticamente al Project mediante el workflow nativo `Auto-add to project`.

Al entrar al Project, la HU queda:

```text
Status: Backlog
Environment: vacío
```

### Acción manual

Mover la HU de:

```text
Backlog → Ready
```

Cuando la HU está lista para desarrollo.

---

## 2. Desarrollo en Dev

Desde la rama `dev` se crea una rama hija para la HU.

Ejemplo:

```text
hu-011-dev
```

La rama debe seguir la convención:

```text
hu-XXX-dev
```

### Acción manual

Al comenzar el desarrollo:

```text
Status: In Progress
Environment: Dev
```

Se realizan los cambios, commits y push normalmente.

---

## 3. Pull Request hacia Dev

Se crea:

```text
hu-011-dev → dev
```

### Referencia obligatoria de la HU

En el título o cuerpo del PR debe aparecer la referencia:

```text
github-project-test#11
```

El formato general es:

```text
<repositorio>#<numero-de-issue>
```

Ejemplo:

```text
Implement HU-011

github-project-test#11
```

La referencia permite al workflow identificar qué Issue/HU debe actualizar.

### Automatización

Al abrir o reabrir el PR:

```text
Status → In Review
Environment → Dev
```

No es necesario cambiar estos valores manualmente.

---

## 4. Merge hacia Dev

Cuando el PR es aprobado, se hace merge:

```text
hu-011-dev → dev
```

### Automatización

El workflow detecta:

```text
base = dev
merged = true
```

y actualiza:

```text
Status → In Progress
Environment → QA
```

Esto indica que el trabajo de desarrollo ya terminó y que la siguiente etapa es preparar la HU para QA.

---

## 5. Preparación para QA

Después del merge a `dev`, se crea una nueva rama hija desde `qa`:

```text
hu-011-qa
```

Convención:

```text
hu-XXX-qa
```

Los cambios de la HU se copian manualmente a esta rama siguiendo el flujo definido para el proyecto.

### Acción manual

No se crea una nueva Issue.

Se continúa trabajando con la misma HU.

La tarjeta permanece:

```text
Status: In Progress
Environment: QA
```

---

## 6. Pull Request hacia QA

Se crea:

```text
hu-011-qa → qa
```

Y se mantiene la referencia de la misma HU:

```text
github-project-test#11
```

### Automatización

Al abrir o reabrir el PR:

```text
Status → In Review
Environment → QA
```

---

## 7. Merge hacia QA

Cuando el PR es aprobado:

```text
hu-011-qa → qa
```

### Automatización

El workflow detecta:

```text
base = qa
merged = true
```

y actualiza:

```text
Status → In Progress
Environment → Production
```

Esto indica que la etapa de QA terminó y la siguiente etapa es preparar la integración final.

---

## 8. Preparación para Main

Desde `main` se crea una nueva rama hija:

```text
hu-011-main
```

Convención:

```text
hu-XXX-main
```

Los cambios de la HU se copian manualmente a esta rama.

La tarjeta permanece:

```text
Status: In Progress
Environment: Production
```

---

## 9. Pull Request hacia Main

Se crea:

```text
hu-011-main → main
```

Y se mantiene la misma referencia:

```text
github-project-test#11
```

### Automatización

Al abrir o reabrir el PR:

```text
Status → In Review
Environment → Production
```

---

## 10. Merge hacia Main

Cuando el PR es aprobado y se hace merge:

```text
hu-011-main → main
```

### Automatización

El workflow detecta:

```text
base = main
merged = true
```

y actualiza:

```text
Status → Done
Environment → Production
```

En este punto la HU se considera terminada.

---

# Responsabilidades manuales y automáticas

## Acciones manuales

El equipo realiza manualmente:

```text
1. Crear la Issue/HU.
2. Mover Backlog → Ready.
3. Crear la rama hija de dev.
4. Mover Ready → In Progress al comenzar el desarrollo.
5. Desarrollar y hacer commits.
6. Crear el PR hacia dev.
7. Revisar y aprobar el PR.
8. Copiar los cambios a la rama hija de qa.
9. Crear el PR hacia qa.
10. Revisar y aprobar el PR.
11. Copiar los cambios a la rama hija de main.
12. Crear el PR hacia main.
13. Revisar y aprobar el PR.
14. Realizar los merges correspondientes.
```

## Acciones automáticas

GitHub Project:

```text
Issue creada
    ↓
Auto-add al Project
    ↓
Status = Backlog
```

GitHub Action:

```text
PR abierto/reabierto → dev
    ↓
In Review / Dev

PR mergeado → dev
    ↓
In Progress / QA

PR abierto/reabierto → qa
    ↓
In Review / QA

PR mergeado → qa
    ↓
In Progress / Production

PR abierto/reabierto → main
    ↓
In Review / Production

PR mergeado → main
    ↓
Done / Production
```

---

# Convenciones

## Issues

Las historias de usuario se administran como Issues de GitHub.

Ejemplo:

```text
HU-011 - Crear módulo de autenticación
```

## Branches

```text
hu-011-dev
hu-011-qa
hu-011-main
```

No se trabaja directamente sobre:

```text
dev
qa
main
```

## Pull Requests

Los PR deben apuntar a la rama correspondiente:

```text
hu-011-dev  → dev
hu-011-qa   → qa
hu-011-main → main
```

Y deben incluir la referencia de la HU:

```text
github-project-test#11
```

---

# Workflow de GitHub Actions

El repositorio utiliza:

```text
.github/
└── workflows/
    └── project-env-tracking.yml
```

El workflow escucha:

```yaml
pull_request:
  types: [opened, reopened, closed]
  branches: [dev, qa, main]
```

La lógica principal depende de:

- la rama destino (`dev`, `qa` o `main`);
- si el PR está abierto/reabierto;
- si el PR fue cerrado mediante merge;
- la Issue/HU referenciada en el PR.

El workflow actualiza los campos `Status` y `Environment` del Project mediante la API GraphQL de GitHub.

---

# Resultado esperado

Una HU debe avanzar de esta forma:

```text
Backlog
  ↓
Ready
  ↓
In Progress / Dev
  ↓
In Review / Dev
  ↓
In Progress / QA
  ↓
In Review / QA
  ↓
In Progress / Production
  ↓
In Review / Production
  ↓
Done / Production
```

El objetivo es que el tablero muestre siempre **qué estado tiene la HU y cuál es la siguiente etapa de integración**, evitando que una HU que ya terminó en `dev` siga apareciendo como si todavía estuviera desarrollándose en `dev`.

Hu-005