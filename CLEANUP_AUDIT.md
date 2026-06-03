# CLEANUP_AUDIT.md
## TradeOS — Auditoría Completa de Limpieza del Proyecto

**Fecha:** Junio 2026  
**Método:** Análisis exhaustivo de árbol de carpetas, hashes MD5, configuraciones git, contenido de documentos y referencias cruzadas  
**Estado:** Solo análisis — ningún archivo modificado, eliminado ni renombrado

---

## Mapa de la Situación Actual

```
C:\Users\juanp\OneDrive\Documentos\TradeOS\          ← RAÍZ (repo git principal)
│
├── tradeOS_base.html.html    ← APP PRINCIPAL (doble extensión ⚠️)
├── .gitignore                ← Plantilla genérica de Node.js (no personalizada)
├── .gitattributes            ← Solo `* text=auto`
├── .claude/
│   └── settings.local.json
│
├── docs/                     ← 25 documentos .md
│   ├── 3 archivos IDÉNTICOS (MD5 igual) ⚠️
│   ├── 2 documentos en conflicto directo
│   └── varios con solapamiento de contenido
│
├── exports/                  ← VACÍA
├── screenshots/              ← VACÍA
│
├── TradeOS/                  ← 🔴 BASURA — solo .git anidado, sin archivos
│   └── TradeOS/              ← 🔴 BASURA — solo .git anidado, sin archivos
│
└── backups/                  ← 🔴 BASURA — solo .git anidados, sin archivos
    └── TradeOS/
        └── TradeOS/          ← solo .git
```

---

# 1. ARCHIVOS ESENCIALES

Archivos que son obligatorios para que el proyecto funcione. No tocar bajo ninguna circunstancia.

| Archivo | Ruta | Por qué es esencial |
|---------|------|---------------------|
| `tradeOS_base.html.html` | `/` | La aplicación completa — HTML + CSS + JS en un solo archivo |
| `.git/` | `/` | Control de versiones, historial, conexión a GitHub |
| `.gitattributes` | `/` | Normalización de saltos de línea en Git |
| `.gitignore` | `/` | Exclusiones del repositorio (aunque necesita personalización) |
| `.claude/settings.local.json` | `/.claude/` | Configuración de permisos para Claude Code |

**Total archivos esenciales: 5** (más el directorio `.git/`)

---

# 2. DOCUMENTACIÓN ESENCIAL

Documentos que deben conservarse tal como están. Son la base de conocimiento del proyecto.

| Documento | Propósito | Por qué conservar |
|-----------|-----------|------------------|
| `MASTER_CONTEXT.md` | Contexto completo del proyecto | Fuente primaria de verdad declarada en PROJECT_RULES |
| `SAAS_ARCHITECTURE_PLAN.md` | Arquitectura oficial v2.0 — 11 decisiones cerradas | Todo el código de v2.0 depende de este documento |
| `PROJECT_RULES.md` | Reglas obligatorias del proyecto | Gobernanza — todo agente/dev debe leerlo primero |
| `SCHEMA.md` | Esquema real extraído del código | Fuente única de verdad para los datos de v1.x |
| `FUNCTIONS.md` | Inventario de 73 funciones reales | Fuente única de verdad para las funciones |
| `SECURITY_AUDIT.md` | 23 riesgos identificados con severidad | Base para correcciones de v1.1 |
| `SECURITY_REQUIREMENTS_V2.md` | 55 requisitos críticos para producción | Checklist de lanzamiento de v2.0 |
| `PRODUCT_REQUIREMENTS_V2.md` | Definición completa del producto v2.0 | Contrato del producto para el equipo |
| `BUILD_ORDER_V2.md` | 20 etapas de construcción con criterios | Guía exacta para construir v2.0 |
| `EXECUTIVE_REPORT.md` | Análisis CTO con top 20 acciones priorizadas | Estado real del producto y hoja de ruta |
| `UI_GUIDELINES.md` | Sistema de diseño: colores, tipografía, tokens | Referencia para todo trabajo visual |
| `BUSINESS_VISION.md` | Visión de negocio y modelo de monetización | Contexto estratégico |
| `LOCALSTORAGE_AUDIT.md` | Auditoría detallada de localStorage | Datos que no están en SCHEMA.md |
| `CHANGELOG.md` | Historial de versiones | Trazabilidad de cambios |
| `ROADMAP.md` | Hoja de ruta del producto | Dirección a largo plazo |
| `TASKS.md` | Tareas activas en curso | Estado operacional del trabajo |
| `TRADING_EXPERT_REVIEW.md` | Revisión por experto — validación funcional | Validación del dominio de negocio |
| `PROJECT_STATUS.md` | Estado actual completo (generado en esta sesión) | Punto de referencia inicial del proyecto |
| `FILE_NAMING_AUDIT.md` | Auditoría del nombre del archivo principal | Guía para resolver el problema de nomenclatura |
| `CLEANUP_AUDIT.md` | Este documento | Guía para limpiar el proyecto |

**Total documentos esenciales: 20**

---

# 3. DOCUMENTOS DUPLICADOS

## Duplicado confirmado: `TradeOS_Department_Prompts*.md` (3 copias idénticas)

**Verificación MD5:**
```
9bc8c92c5d161a729cc9f451b8b6d163  TradeOS_Department_Prompts.md
9bc8c92c5d161a729cc9f451b8b6d163  TradeOS_Department_Prompts_1.md
9bc8c92c5d161a729cc9f451b8b6d163  TradeOS_Department_Prompts_2.md
```

Los tres archivos son **byte-a-byte idénticos**. Mismo contenido, mismo tamaño (886 líneas), mismo hash.

| Atributo | Detalle |
|----------|---------|
| Documento original | `TradeOS_Department_Prompts.md` |
| Duplicados | `TradeOS_Department_Prompts_1.md`, `TradeOS_Department_Prompts_2.md` |
| Diferencias encontradas | **Ninguna.** Hash MD5 idéntico en los tres archivos. |
| Cuál conservar | `TradeOS_Department_Prompts.md` (el original sin sufijo numérico) |
| Por qué existen los duplicados | Probablemente generados en sesiones de IA distintas sin verificar si ya existía el archivo |

**Contenido del archivo:** Prompts listos para configurar 5 roles en Claude (CTO, Product Manager, Trading Expert, Marketing & Growth, Security Auditor). Es un documento de alto valor operacional — debe conservarse **uno solo**.

---

# 4. DOCUMENTOS OBSOLETOS

## 4.1 `IDEAS.md` — Parcialmente obsoleto

**Problema:** Contiene ideas sin prioridad que ya están capturadas con más detalle en otros documentos:
- Las ideas de integración (MT5, TradingView, CSV de prop firms) están en `PRODUCT_REQUIREMENTS_V2.md §4` como "Fuera de v2.0"
- Las ideas de IA están en `PRODUCT_REQUIREMENTS_V2.md §4` como planificadas para v2.2+
- Las ideas de comunidad están en `SAAS_ARCHITECTURE_PLAN.md` como v2.1

**Decisión recomendada:** Conservar pero marcar en el documento que fue **supersedido por PRODUCT_REQUIREMENTS_V2.md**. No eliminarlo — puede contener ideas aún no capturadas formalmente.

## 4.2 `PRODUCT_RECOMMENDATIONS.md` — Supersedido por `EXECUTIVE_REPORT.md`

**Problema:** `PRODUCT_RECOMMENDATIONS.md` (360 líneas) contiene recomendaciones de producto. `EXECUTIVE_REPORT.md` (492 líneas) cubre el mismo terreno con más detalle, más contexto técnico y un Top 20 priorizado con estimaciones de esfuerzo. El Executive Report fue escrito después y es más completo.

**Solapamiento verificado:** Ambos documentos recomiendan las mismas acciones (edición de trades, stats por par, edición de perfil, fix de saveAll, etc.) pero el Executive Report lo hace con mayor rigor.

**Decisión recomendada:** Conservar `PRODUCT_RECOMMENDATIONS.md` solo si contiene recomendaciones que NO están en `EXECUTIVE_REPORT.md`. Candidato a archivar o fusionar en el futuro.

## 4.3 `DEVELOPMENT_BACKLOG.md` — Solapamiento con `TASKS.md` y `EXECUTIVE_REPORT.md`

**Problema:** `DEVELOPMENT_BACKLOG.md` (750 líneas) es la "traducción operacional del Executive Report". Contiene los mismos bugs y acciones que:
- `EXECUTIVE_REPORT.md §TOP 20 ACCIONES`  
- `TASKS.md §Crítico / Alta Prioridad`

Tres documentos rastrean el mismo conjunto de trabajo pendiente con distintos niveles de detalle.

**Decisión recomendada:** `DEVELOPMENT_BACKLOG.md` es el más detallado (tiene pasos concretos de implementación por cada bug). Conservar como referencia técnica. Pero `TASKS.md` es el que se debe actualizar con el estado real del trabajo. Documentar explícitamente en cada uno cuál es la fuente activa.

---

# 5. DOCUMENTOS EN CONFLICTO

Documentos con información técnicamente contradictoria. Requieren resolución explícita de Juan Pa.

## Conflicto 1 — Número de secciones del sidebar

| Documento | Afirmación |
|-----------|-----------|
| `PROJECT_RULES.md` Regla 18 | *"El sidebar tiene exactamente **9** secciones. No agregar una sección décima sin rediseño aprobado."* |
| `PRODUCT_REQUIREMENTS_V2.md §6` | Define **10 módulos** en el sidebar de v2.0 (agrega Calculadora como sección propia) |

**Causa:** La Regla 18 fue escrita para v1.x. v2.0 agrega la Calculadora como sección independiente (en v1.x era un panel integrado en el formulario de trade, no una vista propia).

**Resolución recomendada:** Actualizar `PROJECT_RULES.md` Regla 18 para indicar que aplica solo al archivo HTML v1.x y que v2.0 tiene 10 secciones según `PRODUCT_REQUIREMENTS_V2.md`.

---

## Conflicto 2 — Fuente principal de verdad del proyecto

| Documento | Afirmación |
|-----------|-----------|
| `PROJECT_RULES.md` Regla 21 | *"MASTER_CONTEXT.md es la **fuente principal de verdad**. Cualquier decisión técnica, de diseño o de negocio debe estar reflejada ahí."* |
| `SAAS_ARCHITECTURE_PLAN.md` intro | *"Todo lo que se construya en v2.0 debe consultarse aquí primero. Las decisiones marcadas como CERRADA no se reabren sin aprobación."* |

**Causa:** `MASTER_CONTEXT.md` documenta el producto actual (v1.x). `SAAS_ARCHITECTURE_PLAN.md` governa v2.0. Son fuentes de verdad para versiones distintas, no documentos en conflicto real — pero la ambigüedad confunde a agentes de IA y nuevos colaboradores.

**Resolución recomendada:** Agregar una nota explícita en ambos documentos: `MASTER_CONTEXT.md` = verdad de producto para v1.x. `SAAS_ARCHITECTURE_PLAN.md` = verdad técnica para v2.0.

---

## Conflicto 3 — Scope de v2.0 vs v2.1 en el Roadmap

| Documento | Afirmación |
|-----------|-----------|
| `ROADMAP.md` | v2.0 = "Cloud y Multi-dispositivo". v2.1 = "Plan Freemium" (Stripe, planes, feature gating) |
| `SAAS_ARCHITECTURE_PLAN.md` | v2.0 incluye **ambas cosas**: Supabase cloud + Stripe + planes Free/Pro |

**Causa:** `ROADMAP.md` fue escrito antes de `SAAS_ARCHITECTURE_PLAN.md`. El roadmap está desactualizado.

**Resolución recomendada:** Actualizar `ROADMAP.md` para que refleje el scope de v2.0 según `SAAS_ARCHITECTURE_PLAN.md`. El roadmap es el documento que los usuarios y colaboradores leen primero.

---

## Conflicto 4 — Cantidad de funciones en el código

| Documento | Afirmación |
|-----------|-----------|
| `MASTER_CONTEXT.md §8` | *"Funciones críticas presentes: **130/130**"* |
| `FUNCTIONS.md` cabecera | *"Total de funciones identificadas: **73** funciones nombradas"* |

**Causa:** `MASTER_CONTEXT.md` cuenta handlers inline de HTML (`onclick=...`), métodos internos y sub-funciones. `FUNCTIONS.md` solo cuenta funciones JavaScript con declaración `function` o `const fn =` en scope global. Ambos son correctos con distintas definiciones.

**Resolución recomendada:** Agregar una nota aclaratoria en `MASTER_CONTEXT.md §8` explicando que "130 funciones" incluye handlers inline, y que el inventario de funciones nombradas en JS está en `FUNCTIONS.md` (73).

---

## Conflicto 5 — Nombre del archivo principal (3 variantes)

| Documento(s) | Nombre usado |
|-------------|--------------|
| 8 documentos técnicos (SCHEMA, FUNCTIONS, SECURITY_AUDIT, etc.) | `tradeOS_base.html` |
| MASTER_CONTEXT.md, PROJECT_RULES.md | `trader-os.html` |
| Archivo real en disco | `tradeOS_base.html.html` (doble extensión) |

**Resolución recomendada:** Ver `FILE_NAMING_AUDIT.md`. El nombre oficial debe ser `tradeOS_base.html`.

---

# 6. REPOSITORIOS GIT ANIDADOS

Se detectaron **4 repositorios Git anidados** dentro del repositorio principal. Todos están vacíos (sin archivos de proyecto). Todos son **artefactos de operaciones git fallidas o mal ejecutadas**.

---

## REPO ANIDADO 1 — `TradeOS/.git`

| Atributo | Valor |
|----------|-------|
| **Ruta completa** | `C:\...\TradeOS\TradeOS\.git` |
| **Remote configurado** | ❌ Ninguno |
| **Archivos de proyecto** | ❌ Ninguno (solo `.git`) |
| **Último commit msg** | `Initial commit` |
| **Branch** | `main` (sin commits) |
| **Propósito probable** | Resultado de `git init` ejecutado dentro de la carpeta del proyecto después de que el repo raíz ya existía. Probablemente ocurrió al intentar inicializar el repositorio dos veces. |
| **¿Debe eliminarse?** | ✅ **SÍ — eliminar toda la carpeta `TradeOS/`** |
| **Nivel de riesgo** | 🟢 Bajo. No tiene archivos reales ni historial valioso. Pero su presencia como untracked en el repo raíz contamina `git status`. |

---

## REPO ANIDADO 2 — `TradeOS/TradeOS/.git`

| Atributo | Valor |
|----------|-------|
| **Ruta completa** | `C:\...\TradeOS\TradeOS\TradeOS\.git` |
| **Remote configurado** | ✅ `https://github.com/juanpablojim-afk/TradeOS.git` |
| **Archivos de proyecto** | Solo `.gitattributes` (sin HTML, sin docs) |
| **FETCH_HEAD** | `8bc6a6ba79a4232a5a8b54846242790aa12610c4` (branch main del repo remoto) |
| **packed-refs** | Apunta a `refs/remotes/origin/main` |
| **Propósito probable** | Resultado de `git clone` ejecutado dentro de la carpeta `TradeOS/` ya existente. El clone descargó solo la estructura de git (quizás con `--no-checkout`) y no los archivos reales. |
| **¿Debe eliminarse?** | ✅ **SÍ — eliminar junto con `TradeOS/`** |
| **Nivel de riesgo** | 🟡 Medio. Tiene el mismo remote que el repo raíz — podría confundir comandos git si se ejecutan desde esta ruta por error. |

---

## REPO ANIDADO 3 — `backups/TradeOS/.git`

| Atributo | Valor |
|----------|-------|
| **Ruta completa** | `C:\...\TradeOS\backups\TradeOS\.git` |
| **Remote configurado** | ✅ `https://github.com/juanpablojim-afk/TradeOS.git` |
| **Archivos de proyecto** | ❌ Ninguno |
| **FETCH_HEAD** | Vacío (fetch iniciado pero sin resultado) |
| **Propósito probable** | `git clone` iniciado en la carpeta `backups/` pero interrumpido o fallido antes de completar la descarga de objetos. |
| **¿Debe eliminarse?** | ✅ **SÍ — eliminar toda la carpeta `backups/`** |
| **Nivel de riesgo** | 🟢 Bajo. Sin archivos reales. Pero la presencia de esta carpeta como untracked en el repo raíz contamina `git status`. |

---

## REPO ANIDADO 4 — `backups/TradeOS/TradeOS/.git`

| Atributo | Valor |
|----------|-------|
| **Ruta completa** | `C:\...\TradeOS\backups\TradeOS\TradeOS\.git` |
| **Remote configurado** | ✅ `https://github.com/juanpablojim-afk/TradeOS.git` |
| **Archivos de proyecto** | ❌ Ninguno |
| **FETCH_HEAD** | `8bc6a6ba79a4232a5a8b54846242790aa12610c4` (mismo que repo anidado 2) |
| **Propósito probable** | Segundo intento de `git clone` dentro de `backups/TradeOS/`, también fallido sin archivos descargados. |
| **¿Debe eliminarse?** | ✅ **SÍ — eliminar junto con `backups/`** |
| **Nivel de riesgo** | 🟡 Medio. Mismo remote que el repo raíz — misma confusión potencial. |

---

## Resumen de repos anidados

| Ruta | Remote | Archivos | Eliminar |
|------|--------|----------|----------|
| `TradeOS/.git` | ❌ Sin remote | ❌ Ninguno | ✅ Sí |
| `TradeOS/TradeOS/.git` | ✅ GitHub | solo `.gitattributes` | ✅ Sí |
| `backups/TradeOS/.git` | ✅ GitHub | ❌ Ninguno | ✅ Sí |
| `backups/TradeOS/TradeOS/.git` | ✅ GitHub | ❌ Ninguno | ✅ Sí |

**Los 4 repos anidados son artefactos de clones/inits fallidos. Ninguno contiene código ni historial valioso. Todos deben eliminarse.**

---

# 7. CARPETAS BASURA O TEMPORALES

## Carpeta `TradeOS/` — Eliminar

**Ruta:** `C:\Users\juanp\OneDrive\Documentos\TradeOS\TradeOS\`

**Contenido real:** Solo dos repositorios git vacíos anidados. Cero archivos de proyecto.

**Origen probable:** Se ejecutó `git init` o `git clone` dentro de la carpeta del proyecto. El nombre `TradeOS/TradeOS/` sugiere que se clonó el repo en la ubicación equivocada — dentro de sí mismo.

**Impacto actual:**
- Aparece como `?? TradeOS/` en `git status` del repo raíz
- Confunde el árbol de carpetas del proyecto
- Puede causar comportamientos inesperados en herramientas que recorren el directorio recursivamente (Claude Code, grep, find)

**Nivel de riesgo al eliminar:** 🟢 **Ninguno.** No tiene archivos de proyecto.

---

## Carpeta `backups/` — Eliminar

**Ruta:** `C:\Users\juanp\OneDrive\Documentos\TradeOS\backups\`

**Contenido real:** Dos repositorios git vacíos anidados. Cero archivos de proyecto.

**Origen probable:** Intento de crear backups del repositorio mediante `git clone`. El proceso falló o se interrumpió antes de descargar los archivos, dejando solo las estructuras `.git`.

**Impacto actual:**
- Aparece como `?? backups/TradeOS/` en `git status` del repo raíz
- El nombre `backups/` es engañoso — no contiene ningún backup real

**Nivel de riesgo al eliminar:** 🟢 **Ninguno.** No tiene archivos de proyecto.

---

## Carpetas `exports/` y `screenshots/` — Conservar (vacías por diseño)

**Ruta:** `/exports/` y `/screenshots/`

**Estado:** Ambas vacías.

**Propósito legítimo:** Son destinos para los exports CSV y capturas de pantalla generados por el usuario durante el uso de la app. Son carpetas de trabajo, no basura.

**Problema:** Están vacías y Git no trackea carpetas vacías. Si se eliminan, se pierden en el repositorio. Si se conservan sin contenido, desaparecen al hacer checkout en otro equipo.

**Solución recomendada:** Agregar un archivo `.gitkeep` en cada una para que Git las mantenga en el repositorio.

---

# 8. PROPUESTA DE ESTRUCTURA FINAL

Así debería quedar el proyecto después de la limpieza:

```
TradeOS/                              ← Repositorio Git principal
│
├── tradeOS_base.html                 ← App principal (renombrado: sin doble extensión)
│
├── docs/                             ← Documentación del proyecto (20 archivos)
│   ├── MASTER_CONTEXT.md
│   ├── SAAS_ARCHITECTURE_PLAN.md
│   ├── PROJECT_RULES.md
│   ├── SCHEMA.md
│   ├── FUNCTIONS.md
│   ├── LOCALSTORAGE_AUDIT.md
│   ├── SECURITY_AUDIT.md
│   ├── SECURITY_REQUIREMENTS_V2.md
│   ├── PRODUCT_REQUIREMENTS_V2.md
│   ├── BUILD_ORDER_V2.md
│   ├── EXECUTIVE_REPORT.md
│   ├── UI_GUIDELINES.md
│   ├── BUSINESS_VISION.md
│   ├── TRADING_EXPERT_REVIEW.md
│   ├── CHANGELOG.md
│   ├── ROADMAP.md
│   ├── TASKS.md
│   ├── DEVELOPMENT_BACKLOG.md
│   ├── PRODUCT_RECOMMENDATIONS.md
│   ├── IDEAS.md
│   ├── TradeOS_Department_Prompts.md  ← Solo 1 copia (eliminar _1 y _2)
│   ├── PROJECT_STATUS.md
│   ├── FILE_NAMING_AUDIT.md
│   └── CLEANUP_AUDIT.md               ← Este documento
│
├── exports/
│   └── .gitkeep                       ← Para que Git trackee la carpeta vacía
│
├── screenshots/
│   └── .gitkeep                       ← Para que Git trackee la carpeta vacía
│
├── .gitignore                         ← Actualizar: agregar exports/ y screenshots/ como ignorados
├── .gitattributes                     ← Sin cambios
└── .claude/
    └── settings.local.json            ← Actualizar: nombre correcto del archivo HTML
```

**Carpetas eliminadas:** `TradeOS/`, `backups/`  
**Archivos eliminados:** `TradeOS_Department_Prompts_1.md`, `TradeOS_Department_Prompts_2.md`  
**Archivos renombrados:** `tradeOS_base.html.html` → `tradeOS_base.html`  
**Archivos agregados:** `exports/.gitkeep`, `screenshots/.gitkeep`

---

# 9. PLAN DE LIMPIEZA

Acciones ordenadas por prioridad y dependencia. Ejecutar en el orden indicado.

---

**ACCIÓN 1: Eliminar carpeta `TradeOS/` completa**
```
Eliminar: C:\Users\juanp\OneDrive\Documentos\TradeOS\TradeOS\
Motivo: Solo contiene repos git vacíos sin archivos de proyecto.
Riesgo: Ninguno.
```

---

**ACCIÓN 2: Eliminar carpeta `backups/` completa**
```
Eliminar: C:\Users\juanp\OneDrive\Documentos\TradeOS\backups\
Motivo: Solo contiene repos git vacíos, no hay backups reales.
Riesgo: Ninguno.
```

---

**ACCIÓN 3: Renombrar el archivo principal**
```
Origen:  C:\Users\juanp\OneDrive\Documentos\TradeOS\tradeOS_base.html.html
Destino: C:\Users\juanp\OneDrive\Documentos\TradeOS\tradeOS_base.html
Motivo: Eliminar la doble extensión. Nombre correcto según 22 referencias en documentación.
Nota: En Windows (filesystem case-insensitive) usar `git mv` para que Git registre el rename.
```

---

**ACCIÓN 4: Eliminar `TradeOS_Department_Prompts_1.md`**
```
Eliminar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\TradeOS_Department_Prompts_1.md
Motivo: Copia byte-a-byte idéntica de TradeOS_Department_Prompts.md (MD5: 9bc8c92c)
Riesgo: Ninguno. El contenido está íntegro en el original.
```

---

**ACCIÓN 5: Eliminar `TradeOS_Department_Prompts_2.md`**
```
Eliminar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\TradeOS_Department_Prompts_2.md
Motivo: Copia byte-a-byte idéntica de TradeOS_Department_Prompts.md (MD5: 9bc8c92c)
Riesgo: Ninguno. El contenido está íntegro en el original.
```

---

**ACCIÓN 6: Crear `exports/.gitkeep`**
```
Crear archivo vacío: C:\Users\juanp\OneDrive\Documentos\TradeOS\exports\.gitkeep
Motivo: Git no trackea carpetas vacías. Sin este archivo, la carpeta desaparece al hacer checkout.
```

---

**ACCIÓN 7: Crear `screenshots/.gitkeep`**
```
Crear archivo vacío: C:\Users\juanp\OneDrive\Documentos\TradeOS\screenshots\.gitkeep
Motivo: Mismo motivo que ACCIÓN 6.
```

---

**ACCIÓN 8: Actualizar `.gitignore`**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\.gitignore
Cambio: Reemplazar la plantilla genérica de Node.js por reglas específicas del proyecto.
Agregar al inicio del archivo:
  # TradeOS v1.x — Archivos específicos del proyecto
  exports/*          # Exports generados por el usuario — no versionar
  screenshots/*      # Screenshots del usuario — no versionar
  !exports/.gitkeep  # Conservar la carpeta vacía en git
  !screenshots/.gitkeep
  .env               # Variables de entorno (ya cubierto por la plantilla)
Motivo: El .gitignore actual es una plantilla de Node.js que no tiene sentido para un 
        proyecto HTML vanilla. Ninguna de las reglas actuales aplica al proyecto v1.x.
        Las reglas de Node.js serán necesarias en v2.0, pero no ahora.
```

---

**ACCIÓN 9: Actualizar `PROJECT_RULES.md` línea 30**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\PROJECT_RULES.md
Línea 30 actual:  node --check trader-os.html  # debe retornar sin errores
Línea 30 nueva:   node --check tradeOS_base.html  # debe retornar sin errores
Motivo: Consistencia con el nombre oficial del archivo (ver FILE_NAMING_AUDIT.md).
```

---

**ACCIÓN 10: Actualizar `MASTER_CONTEXT.md` líneas 210 y 276**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\MASTER_CONTEXT.md
Línea 210 actual:  1. Abre trader-os.html en el navegador
Línea 210 nueva:   1. Abre tradeOS_base.html en el navegador

Línea 276 actual:  trader-os.html
Línea 276 nueva:   tradeOS_base.html

Motivo: El nombre oficial es tradeOS_base.html (ver FILE_NAMING_AUDIT.md).
```

---

**ACCIÓN 11: Actualizar `ROADMAP.md` — Scope de v2.0**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\ROADMAP.md
Cambio: Actualizar las secciones v2.0 y v2.1 para reflejar que SAAS_ARCHITECTURE_PLAN.md
        ya define v2.0 como la versión que incluye cloud + freemium + Stripe.
        El roadmap actual está desactualizado (Conflicto 3).
Motivo: ROADMAP.md es el documento que nuevos colaboradores y Claude Code leen como 
        primera orientación del plan. Tener el scope incorrecto confunde la planificación.
```

---

**ACCIÓN 12: Actualizar `PROJECT_RULES.md` Regla 18**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\PROJECT_RULES.md
Regla 18 actual: "El sidebar tiene exactamente 9 secciones."
Agregar aclaración: "En v1.x: 9 secciones. En v2.0: 10 secciones según PRODUCT_REQUIREMENTS_V2.md."
Motivo: Resolver Conflicto 1. La regla aplica solo a v1.x.
```

---

**ACCIÓN 13: Actualizar `.claude/settings.local.json`**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\.claude\settings.local.json
Cambio: Actualizar el permiso que referencia el nombre del archivo con doble extensión.
Actual:  "Bash(node --check \"...\\tradeos_base.html.html\")"
Nuevo:   "Bash(node --check \"...\\tradeOS_base.html\")"
Motivo: Después de ACCIÓN 3, el archivo con doble extensión ya no existe.
```

---

**ACCIÓN 14: Actualizar `PROJECT_STATUS.md` línea 14**
```
Modificar: C:\Users\juanp\OneDrive\Documentos\TradeOS\docs\PROJECT_STATUS.md
Línea 14 actual:  `trader-os.html`
Línea 14 nueva:   `tradeOS_base.html`
Motivo: Consistencia con el nombre oficial (menor prioridad).
```

---

# 10. COMANDOS GIT RECOMENDADOS

Ejecutar después de completar todas las acciones del Plan de Limpieza. En el orden indicado.

```bash
# Desde: C:\Users\juanp\OneDrive\Documentos\TradeOS

# ── PASO 1: Verificar estado limpio antes de empezar ──────────────────────────
git status

# ── PASO 2: Renombrar el archivo principal (git-aware rename) ─────────────────
# IMPORTANTE: usar `git mv` para que Git registre el rename como tal,
# no como delete + create. En Windows con filesystem case-insensitive,
# el rename de tradeOS_base.html.html → tradeOS_base.html requiere
# un paso intermedio para que Git lo detecte correctamente.
git mv "tradeOS_base.html.html" "tradeOS_base.html.html.tmp"
git mv "tradeOS_base.html.html.tmp" "tradeOS_base.html"

# ── PASO 3: Eliminar los duplicados de documentos ────────────────────────────
git rm "docs/TradeOS_Department_Prompts_1.md"
git rm "docs/TradeOS_Department_Prompts_2.md"

# ── PASO 4: Agregar los nuevos archivos .gitkeep ──────────────────────────────
git add "exports/.gitkeep"
git add "screenshots/.gitkeep"

# ── PASO 5: Agregar los documentos actualizados ──────────────────────────────
git add "docs/PROJECT_RULES.md"
git add "docs/MASTER_CONTEXT.md"
git add "docs/ROADMAP.md"
git add "docs/PROJECT_STATUS.md"
git add ".gitignore"
git add ".claude/settings.local.json"

# ── PASO 6: Agregar los nuevos documentos de auditoría ───────────────────────
git add "docs/FILE_NAMING_AUDIT.md"
git add "docs/CLEANUP_AUDIT.md"
git add "docs/PROJECT_STATUS.md"

# ── PASO 7: Verificar que el staging es correcto antes de commitear ───────────
git status
git diff --staged --stat

# ── PASO 8: Commit de limpieza ────────────────────────────────────────────────
git commit -m "chore: limpieza estructural del proyecto

- Renombrar tradeOS_base.html.html → tradeOS_base.html (eliminar doble extensión)
- Eliminar 2 copias duplicadas idénticas de TradeOS_Department_Prompts.md
- Agregar .gitkeep en exports/ y screenshots/ para preservar carpetas vacías
- Actualizar referencias al nombre del archivo en MASTER_CONTEXT y PROJECT_RULES
- Actualizar ROADMAP.md con scope correcto de v2.0
- Agregar FILE_NAMING_AUDIT.md y CLEANUP_AUDIT.md como documentación de auditoría"

# ── PASO 9: Eliminar las carpetas basura (FUERA de git — ya no son trackeadas) ─
# Las carpetas TradeOS/ y backups/ son untracked, no están en git.
# Eliminarlas con el sistema de archivos, no con git rm.
# En PowerShell:
#   Remove-Item -Recurse -Force "TradeOS"
#   Remove-Item -Recurse -Force "backups"
# En bash/cmd:
#   rm -rf TradeOS
#   rm -rf backups

# ── PASO 10: Verificar estado final ──────────────────────────────────────────
git status
# Esperado: "nothing to commit, working tree clean"

# ── PASO 11: Push al repositorio remoto ──────────────────────────────────────
git push origin main

# ── PASO 12: Verificar en GitHub ─────────────────────────────────────────────
# Abrir: https://github.com/juanpablojim-afk/TradeOS
# Confirmar que:
# - tradeOS_base.html existe (sin doble extensión)
# - TradeOS_Department_Prompts_1.md y _2.md NO existen
# - exports/ y screenshots/ existen con .gitkeep
```

---

## Notas importantes sobre la eliminación de carpetas con repos git anidados

Las carpetas `TradeOS/` y `backups/` contienen repos `.git` internos. En Windows, `git rm -rf` del repo raíz **no puede eliminar** estas carpetas porque Git las detecta como "nested repositories" y las trata como submodules incompletos.

**La eliminación debe hacerse a nivel del sistema de archivos:**

```powershell
# PowerShell (ejecutar como Administrador si hay permisos)
Remove-Item -Recurse -Force "C:\Users\juanp\OneDrive\Documentos\TradeOS\TradeOS"
Remove-Item -Recurse -Force "C:\Users\juanp\OneDrive\Documentos\TradeOS\backups"
```

Después de eliminarlas, `git status` no las mostrará más como `??`.

---

## Resumen de impacto de la limpieza

| Métrica | Antes | Después |
|---------|-------|---------|
| Repositorios git en el proyecto | 5 (1 real + 4 basura) | 1 (solo el real) |
| Archivos untracked en git status | 3 (TradeOS/, backups/, FILE_NAMING_AUDIT.md) | 0 |
| Documentos duplicados | 3 (Department_Prompts ×3) | 1 |
| Variantes del nombre del archivo principal | 3 | 1 |
| Carpetas con repos git vacíos | 4 anidadas | 0 |
| Carpetas vacías sin .gitkeep | 2 | 0 (tienen .gitkeep) |
| `node --check` funcional | ❌ Falla por doble extensión | ✅ Funciona |
| `git status` limpio en repo raíz | ❌ 3 archivos untracked | ✅ Working tree clean |

---

*Documento generado por auditoría completa del proyecto — TradeOS*  
*Fecha: Junio 2026*  
*Ningún archivo fue modificado, eliminado ni renombrado durante la generación de este documento*
