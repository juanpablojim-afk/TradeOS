# FILE_NAMING_AUDIT.md
## TradeOS — Auditoría de Nomenclatura del Archivo Principal

**Fecha:** Junio 2026  
**Método:** Análisis exhaustivo de referencias en toda la base de código y documentación  
**Archivos analizados:** 24 documentos `.md` + `tradeos_base.html.html` + `.gitignore` + `.gitattributes` + `.claude/settings.local.json`

---

## 1. Situación Actual

El archivo principal de la aplicación existe en disco con el nombre:

```
tradeos_base.html.html
```

Este nombre tiene **doble extensión** (`.html.html`) y fue commiteado así en el `Initial commit` del repositorio (`67d7849`). No existe ningún otro archivo `.html` en el proyecto.

---

## 2. Referencias en Documentación

### 2.1 Variante `tradeOS_base.html` — 22 referencias

Es la variante más usada en la documentación técnica. Aparece en **8 documentos**:

| Documento | Líneas donde aparece | Contexto |
|-----------|---------------------|---------|
| `FUNCTIONS.md` | 4 | `**Extraído de:** tradeOS_base.html (3206 líneas)` |
| `LOCALSTORAGE_AUDIT.md` | 4 | `**Extraído de:** tradeOS_base.html (3206 líneas)` |
| `SCHEMA.md` | 4 | `**Extraído de:** tradeOS_base.html (3206 líneas)` |
| `SECURITY_AUDIT.md` | 7, 39 | Fuente de la auditoría + referencia a línea ~1485 |
| `EXECUTIVE_REPORT.md` | 7 | `**Fuentes:** tradeOS_base.html · MASTER_CONTEXT…` |
| `DEVELOPMENT_BACKLOG.md` | 303 | `Configurar Netlify o Vercel con el archivo tradeOS_base.html` |
| `TradeOS_Department_Prompts.md` | 46, 54, 93, 459, 575 | 5 referencias al nombre del archivo |
| `TradeOS_Department_Prompts_1.md` | 46, 54, 93, 459, 575 | 5 referencias (copia exacta) |
| `TradeOS_Department_Prompts_2.md` | 46, 54, 93, 459, 575 | 5 referencias (copia exacta) |

### 2.2 Variante `trader-os.html` — 4 referencias

Usada exclusivamente en documentos orientados al **usuario final y reglas del proyecto**:

| Documento | Línea | Contexto |
|-----------|-------|---------|
| `MASTER_CONTEXT.md` | 210 | `1. Abre trader-os.html en el navegador` |
| `MASTER_CONTEXT.md` | 276 | Diagrama de estructura de carpetas del archivo |
| `PROJECT_RULES.md` | 30 | `node --check trader-os.html  # debe retornar sin errores` |
| `PROJECT_STATUS.md` | 14 | Generado en esta sesión — heredó el nombre de MASTER_CONTEXT |

### 2.3 Variante `tradeos_base.html.html` — 0 referencias en documentación

El nombre real del archivo en disco **no aparece en ningún documento**. Ninguna referencia en los 24 archivos `.md`.

### 2.4 Variante `tradeos_base.html` (minúsculas, sin doble extensión) — 0 referencias

No existe como nombre oficial en ningún documento.

---

## 3. Referencias en el Propio Archivo HTML

El archivo **no se auto-referencia por nombre** en ningún lugar de su código interno.

| Búsqueda | Resultado |
|----------|-----------|
| `download=` con nombre `.html` | ❌ No existe |
| `href=` apuntando al archivo | ❌ No existe |
| `filename` o `saveAs` con nombre `.html` | ❌ No existe |
| Comentario con nombre del archivo | Solo `<!-- TradeOS v1.0 -->` en línea 2 — sin nombre de archivo |
| `<title>` | `TradeOS — The Operating System for Serious Traders` — sin nombre de archivo |

El archivo sí genera nombres para sus exports internos:
- Backup JSON: `tradeos-backup-YYYY-MM-DD.json`
- Export CSV: `trades-YYYY-MM-DD.csv`

Estos nombres son independientes del nombre del archivo HTML y son consistentes con la marca.

---

## 4. Referencias en Scripts y GitHub Actions

| Tipo de archivo | Estado | Detalle |
|----------------|--------|---------|
| GitHub Actions (`.yml`) | ❌ No existe | No hay workflows configurados en el repositorio |
| Scripts de shell (`.sh`) | ❌ No existe | No hay scripts |
| `package.json` | ❌ No existe | No hay proyecto Node.js en v1.x |
| `vercel.json` | ❌ No existe | Solo documentado en `BUILD_ORDER_V2.md` para v2.0 |
| `Makefile` | ❌ No existe | — |

No existe ninguna automatización que haga referencia al nombre del archivo.

---

## 5. Referencias en Configuraciones

| Archivo | Referencia | Detalle |
|---------|-----------|---------|
| `.gitignore` | ❌ Ninguna | El `.gitignore` es una plantilla genérica de Node.js — no menciona el HTML |
| `.gitattributes` | ❌ Ninguna | Solo `* text=auto` para normalización de saltos de línea |
| `.claude/settings.local.json` | ⚠️ Sí | `"Bash(node --check \"…\\tradeos_base.html.html\")"` — referencia al nombre actual con doble extensión, generada automáticamente al ejecutar el comando en esta sesión |

---

## 6. Historial Git

El `Initial commit` (`67d7849`) muestra que el archivo fue subido al repositorio directamente con el nombre `tradeos_base.html.html`. El historial de git no evidencia un rename previo — el nombre con doble extensión es el nombre original del commit, no el resultado de una operación posterior.

```
tradeos_base.html.html               | 3206 ++++++++++++++++++++++++++++++++++
```

---

## 7. Tabla Resumen de Variantes

| Variante | Documentos | Referencias | Dónde se usa |
|----------|-----------|-------------|-------------|
| `tradeOS_base.html` | 8 | **22** | Auditorías técnicas, prompts de agentes, backlog |
| `trader-os.html` | 2 | **4** | Reglas del proyecto, contexto para usuario final |
| `tradeos_base.html.html` | 0 | **0** | Solo en disco y en `.claude/settings.local.json` |
| `tradeos_base.html` | 0 | **0** | No existe en ningún lugar |
| `index.html` | 1 | **3** | Solo en `BUILD_ORDER_V2.md` — corresponde a v2.0 React, no a v1.x |

---

## 8. Análisis de la Doble Extensión

El archivo `tradeos_base.html.html` tiene doble extensión. Las causas más probables:

1. **Guardado desde un editor con extensión automática:** el archivo tenía el nombre `tradeos_base.html` y se guardó "como" archivo en un diálogo que añadió `.html` automáticamente al detectar que el nombre ya terminaba en `.html`.

2. **Descarga desde un sistema web:** algunos sistemas de descarga añaden la extensión del Content-Type aunque el nombre ya la tenga.

3. **Renombrado manual accidental:** se editó el nombre del archivo y se escribió la extensión manualmente sin notar que ya tenía una.

**Consecuencia técnica:** el comando de validación del proyecto falla con doble extensión:
```bash
node --check tradeos_base.html.html
# TypeError [ERR_UNKNOWN_FILE_EXTENSION]: Unknown file extension ".html"
```

---

## 9. Recomendación: Nombre Oficial Definitivo

### El nombre oficial debe ser:

```
tradeOS_base.html
```

### Justificación

| Criterio | `tradeOS_base.html` | `trader-os.html` |
|----------|--------------------|--------------------|
| Frecuencia en documentación técnica | **22 referencias** | 4 referencias |
| Documentos que lo usan | Auditorías, prompts de agentes, backlog, reporte ejecutivo | Solo MASTER_CONTEXT y PROJECT_RULES |
| Consistencia con el nombre en disco | Solo difiere en la doble extensión | Nombre completamente distinto |
| Convención de naming | `camelCase` consistente con el proyecto | Kebab-case, distinto al resto |
| Aprobado para uso por agentes de IA | Sí — todos los prompts de agentes lo usan | No — solo documentos de usuario |
| Contiene el nombre del producto | `TradeOS` → `tradeOS` | `trader-os` (versión reducida) |

**`tradeOS_base.html`** es el nombre que:
- Aparece en 22 de 26 referencias totales
- Es usado por todos los documentos técnicos (SCHEMA, FUNCTIONS, SECURITY_AUDIT, EXECUTIVE_REPORT)
- Es usado en los 3 archivos de prompts para agentes de IA (`TradeOS_Department_Prompts*.md`)
- Refleja correctamente el nombre del producto (`tradeOS`)
- Es el nombre que los auditores técnicos del proyecto usaron al analizar el archivo real

**`trader-os.html`** debe entenderse como el **nombre de distribución al usuario final** — es el nombre que ve el comprador en Gumroad cuando descarga el archivo. Es diferente al nombre de trabajo interno. Esta distinción es válida y puede mantenerse: el archivo de trabajo se llama `tradeOS_base.html`, el archivo que se entrega al comprador puede llamarse `trader-os.html`.

---

## 10. Impacto del Renombre

### Archivos que requieren actualización si se aplica el renombre

| Archivo | Cambio requerido |
|---------|----------------|
| `docs/MASTER_CONTEXT.md` líneas 210, 276 | `trader-os.html` → `tradeOS_base.html` |
| `docs/PROJECT_RULES.md` línea 30 | `node --check trader-os.html` → `node --check tradeOS_base.html` |
| `docs/PROJECT_STATUS.md` línea 14 | `trader-os.html` → `tradeOS_base.html` |
| `.claude/settings.local.json` | Actualizar el permiso con el nombre correcto |
| El archivo en disco | Renombrar `tradeos_base.html.html` → `tradeOS_base.html` |

### Archivos que NO requieren cambio

Todo lo demás ya usa `tradeOS_base.html`. Son 22 referencias ya correctas.

---

## 11. Acción Recomendada (sin prioridad de implementación)

```
1. Renombrar en disco:
   tradeos_base.html.html  →  tradeOS_base.html

2. Actualizar MASTER_CONTEXT.md (2 líneas)

3. Actualizar PROJECT_RULES.md (1 línea)

4. Actualizar PROJECT_STATUS.md (1 línea)

5. Actualizar .claude/settings.local.json (1 línea del permiso)

Total: renombrar 1 archivo + editar 4 líneas en 4 documentos.
```

Este cambio resuelve:
- La doble extensión que rompe `node --check`
- La inconsistencia entre el nombre en disco y las 22 referencias en documentación
- La ambigüedad entre `trader-os.html` y `tradeOS_base.html` como nombres de trabajo

---

*Documento generado por auditoría de referencias — TradeOS*  
*Fecha: Junio 2026*  
*Este documento no modifica ningún archivo — solo registra el análisis y la recomendación*
