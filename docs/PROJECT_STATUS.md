# PROJECT_STATUS.md
## TradeOS — Estado Completo del Proyecto

**Generado:** Junio 2026  
**Auditor:** Claude Code — basado en lectura directa de los 22 documentos del proyecto  
**Fuentes:** MASTER_CONTEXT · SCHEMA · FUNCTIONS · LOCALSTORAGE_AUDIT · SECURITY_AUDIT · EXECUTIVE_REPORT · SAAS_ARCHITECTURE_PLAN · PROJECT_RULES · PRODUCT_REQUIREMENTS_V2 · SECURITY_REQUIREMENTS_V2 · BUILD_ORDER_V2 · ROADMAP · CHANGELOG · TASKS · IDEAS · UI_GUIDELINES · BUSINESS_VISION · TRADING_EXPERT_REVIEW · PRODUCT_RECOMMENDATIONS · DEVELOPMENT_BACKLOG

---

## Resumen Ejecutivo

### Estado actual del producto

TradeOS es una aplicación web de archivo único (`trader-os.html`, 225 KB, 3.206 líneas) distribuida en Gumroad a $19 USD como pago único. Funciona 100% offline sin backend, sin autenticación y sin servidor. Persiste todos los datos en `localStorage` del navegador del usuario.

**En una frase:** el mejor journal de trading en español para traders de prop firms latinoamericanos, con diferenciación técnica real, pero con 2 bugs críticos activos que pueden hacer que el usuario pierda datos sin saberlo.

El producto está en **v1.0 Beta**. No se ha lanzado v1.1 ni se ha iniciado la construcción de v2.0. La documentación del proyecto está en un nivel de madurez extraordinariamente alto para un producto en esta etapa, pero el código aún no ha incorporado las correcciones documentadas.

### Nivel de madurez del proyecto

| Dimensión | Estado | Calificación |
|-----------|--------|-------------|
| Funcionalidad del producto | 14/14 módulos operativos | ✅ Sólido |
| Documentación técnica | 22 documentos completos | ✅ Excelente |
| Estabilidad de datos | 2 bugs críticos activos | 🔴 Crítico |
| Arquitectura v2.0 | Diseñada, no implementada | ⚠️ Pendiente |
| Modelo de negocio | Definido, sin implementar SaaS | ⚠️ Pendiente |
| Infraestructura cloud | Ninguna cuenta creada aún | ⏸️ Sin iniciar |
| Tests automatizados | Ninguno | ⏸️ Sin iniciar |

### Riesgos más importantes

1. **Bug C1 activo en producción:** `saveAll()` no captura `QuotaExceededError`. Un trader puede perder trades registrados sin ningún aviso — la app muestra toast de "guardado" aunque el dato no se persistió. Es el escenario con mayor potencial de daño reputacional.

2. **Bug C2 activo en producción:** las imágenes en base64 no tienen compresión ni límite. Con 25-30 capturas de resolución normal, el localStorage se satura y el bug C1 se vuelve inevitable para cada guardado subsiguiente.

3. **La v2.0 no ha comenzado:** toda la arquitectura SaaS está diseñada con un nivel de detalle extraordinario (SAAS_ARCHITECTURE_PLAN, SECURITY_REQUIREMENTS_V2, BUILD_ORDER_V2, PRODUCT_REQUIREMENTS_V2), pero no existe una línea de código de React, no hay proyecto Supabase creado, no hay repositorio GitHub. El tiempo hasta el lanzamiento del SaaS es de 12-16 semanas desde el inicio real.

4. **Sin mecanismo de actualización para compradores actuales:** los usuarios de v1.0 con los bugs críticos activos no tienen forma de saber que existe una versión más segura. No hay canal de notificación establecido.

---

## Estado Técnico

### Arquitectura actual

El producto actual (v1.0) es un único archivo HTML con tres capas:

- **Presentación:** CSS3 con variables (~33 KB inline). Sistema de design tokens completo, bilingüe ES/EN, modo oscuro/claro.
- **Lógica:** JavaScript ES6+ vanilla (~135 KB, ~2.094 líneas, 73 funciones nombradas). Estado global en 6 variables (`cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists`).
- **Persistencia:** `localStorage` del navegador con 9 claves distintas. Límite efectivo: ~5-10 MB según navegador.

No hay backend, no hay servidor, no hay APIs externas (excepto Google Fonts CDN), no hay build process, no hay npm.

**La arquitectura v2.0 está completamente diseñada:** React 18 + Vite + Zustand + IndexedDB (Dexie.js) + Supabase (PostgreSQL + Auth + Storage) + Stripe + Resend + Sentry + Vercel. 20 etapas de construcción documentadas en `BUILD_ORDER_V2.md`. El schema SQL completo con RLS definido en `SAAS_ARCHITECTURE_PLAN.md`. Sin embargo, esta arquitectura no existe en código todavía.

### Calidad de documentación

La documentación es uno de los puntos más fuertes del proyecto. 22 documentos en la carpeta `docs/`:

| Documento | Propósito | Calidad |
|-----------|-----------|---------|
| `MASTER_CONTEXT.md` | Contexto completo del proyecto | ✅ Excelente |
| `SAAS_ARCHITECTURE_PLAN.md` | Arquitectura oficial v2.0 | ✅ Excelente — 11 decisiones cerradas |
| `PROJECT_RULES.md` | Reglas obligatorias del proyecto | ✅ Excelente |
| `SCHEMA.md` | Esquema real de datos de v1.x | ✅ Excelente — extraído del código |
| `FUNCTIONS.md` | Inventario de 73 funciones | ✅ Completo |
| `SECURITY_AUDIT.md` | 23 riesgos identificados | ✅ Excelente |
| `SECURITY_REQUIREMENTS_V2.md` | 55 requisitos críticos para producción | ✅ Completo |
| `PRODUCT_REQUIREMENTS_V2.md` | Definición del producto v2.0 | ✅ Excelente |
| `BUILD_ORDER_V2.md` | 20 etapas de construcción con criterios de finalización | ✅ Excelente |
| `EXECUTIVE_REPORT.md` | Análisis CTO con top 20 acciones | ✅ Excelente |
| `LOCALSTORAGE_AUDIT.md` | Auditoría de localStorage | ✅ Completo |
| `ROADMAP.md` | Hoja de ruta del producto | ✅ Presente |
| `CHANGELOG.md` | Historial de cambios | ✅ Actualizado (v1.0 documentado) |
| `UI_GUIDELINES.md` | Guías de diseño | ✅ Presente |
| `BUSINESS_VISION.md` | Visión de negocio | ✅ Presente |
| `TRADING_EXPERT_REVIEW.md` | Revisión por experto en trading | ✅ Presente |
| `PRODUCT_RECOMMENDATIONS.md` | Recomendaciones de producto | ✅ Presente |
| `DEVELOPMENT_BACKLOG.md` | Backlog de desarrollo | ✅ Presente |
| `TASKS.md` | Estado de tareas activas | ✅ Presente |
| `IDEAS.md` | Ideas no priorizadas | ✅ Presente |
| `TradeOS_Department_Prompts*.md` | Prompts de departamentos (3 archivos) | ⚠️ Propósito específico |

### Calidad del código

| Métrica | Estado |
|---------|--------|
| Node.js syntax check | ✅ PASS |
| `window.addEventListener('load')` | ✅ Único (no duplicado) |
| Tags HTML sin escapar en JS | ✅ 0 |
| Funciones duplicadas en el código | ⚠️ 6 (resetAll, openM, closeM, showN, toggleTheme, loadTheme) |
| Funciones de código muerto | ⚠️ 3+ (buildRuleSelect, toggleTag versión 1, toggleReplicator) |
| Bugs críticos de pérdida de datos | 🔴 2 confirmados (C1, C2) |
| Bugs altos de corrupción | 🟠 5 confirmados (A1-A5) |
| Inconsistencias de tipo en datos | 🟡 3 (Account.id number vs trade.account string, etc.) |
| Tests automatizados | ❌ 0 tests |
| Control de versiones (Git) | ❌ No hay repositorio Git documentado |

### Deuda técnica detectada

**Crítica (bloquea v2.0 si no se resuelve):**
- `saveAll()` sin manejo de `QuotaExceededError` — el dato más importante del sistema se guarda sin protección
- Imágenes base64 sin compresión — causa directa de la saturación de localStorage

**Alta:**
- 6 funciones declaradas dos veces — confusión garantizada al editar
- Tipos inconsistentes: `Account.id` (number) vs `trade.account` (string) — trampa para funciones nuevas
- Campos muertos `balance`, `drawdown`, `pnl` en objeto `Account` — valores siempre incorrectos
- `motherId` en réplicas puede apuntar al trade anterior, no al padre real

**Media:**
- `checklists` crece indefinidamente sin TTL ni limpieza automática
- Bias del día (`tt_bias_*`) fuera del sistema de backup — se pierde en toda migración
- ID de fases personalizadas con doble prefijo `custom_` — frágil en cualquier función nueva
- `renderWeeklySummary()` solo visible sábados y domingos (guard deliberado pero erróneo desde UX)

**Baja:**
- Código muerto en scope global (3+ funciones)
- CSV exportado muestra ID de cuenta en lugar del nombre del firm
- Email de soporte hardcodeado en código cliente
- Prefijo de claves `localStorage` inconsistente (`tt_` vs `jft_`)

---

## Seguridad

### Riesgos críticos (🔴)

| ID | Descripción | Función afectada |
|----|-------------|-----------------|
| C1 | `saveAll()` no captura `QuotaExceededError` — trade confirmado como "guardado" que no se persistió | `saveAll()` |
| C2 | Imágenes base64 sin control de tamaño agotan localStorage inevitablemente con uso normal | `prevImg()`, `saveTrade()` |

### Riesgos altos (🟠)

| ID | Descripción |
|----|-------------|
| A1 | Bug confirmado: `importD()` no restaura `checklists` — historial de disciplina se pierde en toda migración |
| A2 | Importación sin validación de esquema — un JSON malformado puede destruir todos los datos del usuario |
| A3 | Importación sin rollback — sobreescritura irreversible parcial si el proceso falla a mitad |
| A4 | `motherId` en réplicas puede apuntar al trade incorrecto (trade anterior, no el padre real) |
| A5 | Eliminar trade madre deja réplicas hijas huérfanas que contaminan métricas, WR y drawdown |
| A6 | Modo incógnito: pérdida total de datos sin ningún aviso al usuario |
| A7 | Dos pestañas abiertas: sobreescritura mutua sin sincronización — cada `saveAll()` borra cambios de la otra pestaña |

### Riesgos medios (🟡)

| ID | Descripción |
|----|-------------|
| M1 | `clDeleteBtn()` puede eliminar los 13 ítems predeterminados del checklist (viola PROJECT_RULES Regla 4) |
| M2 | Colisión de IDs en trades réplica — `Date.now() + Math.random()` no garantiza unicidad |
| M3 | Doble prefijo `custom_` en fases personalizadas — frágil para cualquier función futura |
| M4 | `checklists` crece indefinidamente sin limpieza automática |
| M5 | Bias del día: claves sueltas fuera del backup, sin migración posible a v2.0 |
| M6 | Campos muertos en `Account` — trampa para regresiones futuras |
| M7 | Tipo ambiguo `account`: number en Account, string en trade — frágil en funciones nuevas |
| M8 | `saveZoneEdit()` descarta silenciosamente campos no manejados |

### Riesgos bajos (🟢)

| ID | Descripción |
|----|-------------|
| B1 | 6 funciones declaradas dos veces |
| B2 | Código muerto activo en scope global (3+ funciones) |
| B3 | CSV exportado usa ID de cuenta, no nombre del firm |
| B4 | `tt_cl` incluida en export pero no en import — inconsistencia no documentada en UI |
| B5 | Prefijo de claves inconsistente (`tt_` vs `jft_`) |
| B6 | Email de soporte hardcodeado en código cliente |

**Nota positiva sobre privacidad:** el sistema es completamente conforme en términos de privacidad. Se verificaron 0 llamadas a APIs externas con datos del usuario, 0 analytics, 0 telemetría. Los datos del trader no salen del navegador bajo ninguna circunstancia.

---

## Preparación para SaaS

### Estado de Supabase

**No iniciado.** No existe ninguna cuenta de Supabase creada. No hay proyecto de staging. No hay migrations SQL ejecutadas. No hay conexión configurada.

El schema SQL completo está documentado con precisión en `SAAS_ARCHITECTURE_PLAN.md §DECISIÓN 2` (8 tablas: `profiles`, `accounts`, `trades`, `funds`, `goals`, `phases`, `checklists`, `subscriptions`). Las políticas RLS están definidas. Los índices están especificados. El schema puede ejecutarse hoy.

**Lo que falta:** crear la cuenta, crear el proyecto, ejecutar `001_initial_schema.sql`, verificar RLS.

### Estado de Stripe

**No iniciado.** No existe ninguna cuenta de Stripe configurada. No hay Price IDs. No hay webhooks.

El flujo completo está documentado en `SAAS_ARCHITECTURE_PLAN.md §DECISIÓN 5`: planes y precios definidos (`price_pro_monthly $9`, `price_pro_yearly $79`, `price_teams_monthly $49`), flujo de checkout diseñado, Edge Functions de webhook especificadas. Las 4 Edge Functions requeridas están detalladas en `BUILD_ORDER_V2.md §ETAPA 20`.

**Lo que falta:** crear cuenta, configurar productos en Stripe Dashboard, obtener Price IDs.

### Estado de autenticación

**No iniciado.** No existe sistema de autenticación en v1.x (por diseño — es un archivo local). La arquitectura para v2.0 está completamente diseñada: Magic Link + Google OAuth via Supabase Auth, sesiones de 7 días, JWT en memoria (no localStorage), protección de rutas documentada.

**Decisión de seguridad relevante:** el magic link debe configurarse con expiración ≤ 15 minutos (requisito AUTH-C1 de `SECURITY_REQUIREMENTS_V2.md`). La nota en `BUILD_ORDER_V2.md §ETAPA 07` establece que el mensaje de confirmación debe ser idéntico independientemente de si el email existe (AUTH-A3 — evitar enumeración de usuarios).

### Estado de almacenamiento de imágenes

**No iniciado en v2.0; problema activo en v1.0.**

En v1.x: las imágenes se guardan como base64 completo en localStorage — este es el bug crítico C2 activo.

En v2.0 (diseñado): bucket privado `tradeos-screenshots` en Supabase Storage con prefijo `/{user_id}/{trade_id}.jpg`. Compresión en cliente antes del upload (target ≤500 KB, 1200px de ancho, JPEG 0.7). Límite Free: 50 screenshots / 5 MB. Límite Pro: 2 GB. URLs firmadas con expiración de 1 hora (requisito IMG-M1).

### Estado de migración v1 → v2

**Diseñado, no implementado.** La función `migrateFromV1Backup()` está completamente especificada en `SAAS_ARCHITECTURE_PLAN.md §DECISIÓN 10` y `BUILD_ORDER_V2.md §ETAPA 12`. El mapeo de campos está documentado:
- `cfg` → `profiles` + `accounts` + `goals` + `phases`
- `trades[]` → tabla `trades` (IDs timestamp → UUID, `motherId` resuelto, `ruleBroken` string → `TEXT[]`)
- `checklists{}` → tabla `checklists` (fecha `"Mon Jun 02 2026"` → DATE `'2026-06-02'`)
- `tt_bias_*` → campo `bias_note` en tabla `checklists`

**Problema específico de migración a resolver:** las claves `tt_bias_[fecha]` son claves sueltas en localStorage que no pasan por `saveAll()` y no se incluyen en el backup actual. Para la migración v1→v2, el trader necesitará exportar un backup especial que incluya estas claves. Esto está documentado como requisito BCK-M2 en `SECURITY_REQUIREMENTS_V2.md`.

---

## Documentación

### Documentos existentes (22 archivos en `/docs`)

Todos listados en la sección "Calidad de documentación" del Estado Técnico.

### Documentos faltantes

| Documento | Por qué hace falta |
|-----------|-------------------|
| `DEPLOYMENT_RUNBOOK.md` | Proceso paso a paso para restaurar la BD desde un backup de Supabase. Requerido explícitamente en BCK-A4 de `SECURITY_REQUIREMENTS_V2.md`. |
| `AFFILIATE_PROGRAM.md` | El programa de afiliados (30%) está referenciado en PROJECT_RULES y BUSINESS_VISION pero no tiene documento operativo. Sin instrucciones concretas, el canal no puede activarse. |
| `MIGRATION_GUIDE_V1_TO_V2.md` | Guía para el usuario final sobre cómo migrar sus datos del archivo HTML a la web app. La función técnica existe en el diseño; la guía para el usuario, no. |
| `PRIVACY_POLICY.md` | Requerido en DATA-M1 de `SECURITY_REQUIREMENTS_V2.md` antes del lanzamiento. |
| `ONBOARDING_FLOW.md` | Aunque el flujo de 9 pasos está documentado en varios documentos, no hay un documento unificado de UX del onboarding con estados de error y casos edge. |

### Documentos desactualizados

| Documento | Inconsistencia detectada |
|-----------|--------------------------|
| `ROADMAP.md` | Muestra v2.0 como "Cloud" y v2.1 como "Plan Freemium" en fases separadas. `SAAS_ARCHITECTURE_PLAN.md` unifica ambas en una sola v2.0. Los scopes no coinciden. |
| `PROJECT_RULES.md` Regla 18 | Dice "El sidebar tiene exactamente 9 secciones". `PRODUCT_REQUIREMENTS_V2.md §6` define 10 módulos en el sidebar de v2.0. La regla aplica a v1.x pero puede generar confusión si se lee como regla absoluta. |
| `PROJECT_RULES.md` Regla 21 | Dice "MASTER_CONTEXT.md es la fuente principal de verdad". `SAAS_ARCHITECTURE_PLAN.md` declara ser la arquitectura oficial. No hay documento que clarifique cuál prevalece en caso de conflicto entre ambos. |
| `MASTER_CONTEXT.md §8` | Dice "130 funciones críticas presentes". `FUNCTIONS.md` documenta 73 funciones nombradas. La discrepancia está explicada en `FUNCTIONS.md` (la diferencia incluye handlers inline de HTML), pero el número en `MASTER_CONTEXT.md` es confuso. |
| `TASKS.md` | Requiere verificación manual de si refleja el estado real actual del trabajo. |

---

## Top 10 Prioridades Técnicas

Ordenadas de mayor a menor impacto real para el proyecto.

### 1. Cerrar los 2 bugs críticos de pérdida silenciosa de datos

**Impacto:** máximo. Un usuario que pierde datos sin saberlo y lo documenta públicamente destruye el canal de distribución por recomendaciones orgánicas.

- Envolver `saveAll()` en try/catch. Si falla, mostrar modal bloqueante (no toast) con instrucciones concretas.
- Comprimir imágenes con `canvas.toDataURL('image/jpeg', 0.6)` y limitar ancho a 1200px antes de asignar a `imgB64`.

### 2. Corregir los 3 bugs altos de `importD()` (A1, A2, A3)

**Impacto:** alto. La función de recuperación de datos no puede ser ella misma un punto de destrucción de datos.

- Agregar la línea faltante que restaura `checklists` (A1 — una línea de código).
- Agregar validación de esquema antes de escribir (A2).
- Agregar snapshot + rollback si el proceso falla a mitad (A3).

### 3. Corregir `delTrade()` para eliminar réplicas huérfanas (A5)

**Impacto:** alto. El replicador es el diferenciador más importante del producto. Métricas corruptas en el dashboard afectan la decisión de riesgo del trader.

### 4. Hospedar en dominio propio (`app.tradeos.io`)

**Impacto:** alto en distribución. Desbloquea Safari iOS (40-60% del tráfico móvil latinoamericano), profesionaliza el producto para afiliados, y es prerequisito de v2.0.

Costo: ~$10-15/año de dominio + Vercel Hobby (gratis). Configuración: horas, no días.

### 5. Corregir `motherId` en el replicador (A4 / DT9)

**Impacto:** alto. La relación madre-hija es el núcleo del feature más diferenciador. La migración a v2.0 con UUIDs hace esto más urgente — el mapeo de IDs en `migrateFromV1Backup()` depende de que los `motherId` sean correctos.

### 6. Implementar mecanismo de actualizaciones para compradores de Gumroad

**Impacto:** alto en negocio. Sin notificación, los compradores de v1.0 con bugs activos nunca sabrán que existe una versión más segura. El número de versión debe ser visible en la UI.

### 7. Iniciar la Fase 0 de v2.0 (preparación de infraestructura)

**Impacto:** estratégico. La arquitectura está diseñada en detalle. El próximo paso no es escribir código React — es crear las cuentas (Supabase, Stripe, Vercel, Resend, Sentry, GitHub) y ejecutar las migrations SQL.

Según `SAAS_ARCHITECTURE_PLAN.md §HOJA DE RUTA`, la Fase 0 puede correr en paralelo con las correcciones de v1.1. No hay razón para esperar.

### 8. Normalizar tipos de datos antes de la migración (DT3, DT4)

**Impacto:** estratégico. El tipo de `Account.id` (number) vs `trade.account` (string) debe resolverse antes de que existan usuarios reales en v2.0 con datos reales en PostgreSQL. Los campos muertos `balance`, `drawdown`, `pnl` deben eliminarse del schema antes de la migración.

### 9. Incluir `tt_bias_*` en el backup exportado (DT6)

**Impacto:** medio directo, alto en migración. El historial de análisis pre-sesión del trader es tan valioso como sus trades. Su pérdida en la migración v1→v2 es inaceptable para un usuario activo de 6 meses.

### 10. Implementar edición de trades ya guardados

**Impacto:** alto en retención. La ausencia de edición es la limitación funcional más citada en journals de trading. Su ausencia desincentiva el registro inmediato (el trader espera tener todos los datos perfectos antes de registrar).

---

## Recomendación del CTO

### Situación real al día de hoy

TradeOS tiene un producto con diferenciación genuina, un ICP definido con precisión, una arquitectura SaaS diseñada con nivel de detalle inusual para este estado del proyecto, y documentación que permite incorporar colaboradores — o agentes de IA — sin pérdida de contexto.

El problema no es el product-market fit ni la visión. El problema es la brecha entre la calidad de la documentación y la calidad del código ejecutable. El sistema está documentado para v2.0 con 55 requisitos de seguridad definidos, 20 etapas de construcción con criterios de finalización, y un schema SQL completo — pero ninguna de esas cosas existe en código todavía, y mientras tanto hay 2 bugs críticos activos en el código que sí tiene usuarios reales.

### El siguiente paso

**No es iniciar v2.0. Es cerrar v1.1.**

Las acciones #1 al #5 del `EXECUTIVE_REPORT.md` son no negociables antes de cualquier campaña de distribución o inicio de v2.0. Son 5 correcciones de bugs en el mismo archivo HTML. El esfuerzo estimado es 2-3 semanas. El costo de no hacerlas es un incidente público de pérdida de datos en un mercado pequeño y vocal.

**El orden concreto recomendado:**

**Semana 1-2 (v1.1 — Bugs críticos):**
1. `saveAll()` con try/catch + modal bloqueante de error + barra de uso en Ajustes
2. Compresión de imágenes en `prevImg()` con Canvas API antes de guardar
3. Restauración de `checklists` en `importD()` (una línea)
4. Validación de esquema + rollback en `importD()`
5. Eliminación de hijas huérfanas en `delTrade()`

**Semana 2-3 (v1.1 — Fixes secundarios):**
6. Corrección de `motherId` en `applyReplicator()`
7. Resumen semanal visible todos los días (eliminar guard de fin de semana)
8. Inclusión de `tt_bias_*` en `exportD()` e `importD()`
9. CSV con nombre de firm en lugar de ID

**Semana 3-4 (v1.2 — Distribución):**
10. Hospedar en dominio propio (Vercel + dominio)
11. Notificación a compradores de Gumroad sobre v1.1
12. Número de versión visible en la UI

**Semana 5 en adelante (Fase 0 de v2.0 — en paralelo):**
- Crear cuentas: GitHub repo, Supabase, Stripe (modo test), Vercel, Resend, Sentry
- Ejecutar `001_initial_schema.sql` en Supabase staging
- Verificar RLS con prueba manual: JWT de usuario A → 0 filas de usuario B

**A partir de la Fase 0 completada:** ejecutar `BUILD_ORDER_V2.md` en orden numérico estricto (E01 → E20), sin saltarse ninguna etapa, verificando el criterio de finalización de cada una.

### Qué no hacer ahora mismo

- No iniciar features nuevas (edición de trades, estadísticas por par, diario de mercado) mientras los bugs críticos estén activos en producción.
- No escalar distribución ni activar afiliados en escala mientras el producto tenga bugs que puedan destruir datos del usuario.
- No iniciar v2.0 sin completar primero v1.1 — el `SAAS_ARCHITECTURE_PLAN.md` lo dice explícitamente: "nada de código de v2.0 antes de cerrar todos los bugs de v1.1".

### Estimación de tiempo hasta el lanzamiento de v2.0

| Fase | Duración estimada |
|------|-----------------|
| v1.1 (bugs críticos) | 2-3 semanas |
| v1.2 (hosting) | 1 semana |
| v2.0 Fase 0 (preparación) | 2-3 semanas (en paralelo) |
| v2.0 Fase 1 (esqueleto React + Auth + Supabase) | 3-4 semanas |
| v2.0 Fase 2 (features completos) | 4-6 semanas |
| v2.0 Fase 3 (monetización Stripe) | 2-3 semanas |
| v2.0 Fase 4 (migración + lanzamiento) | 1-2 semanas |
| **TOTAL** | **~16-22 semanas** |

**El potencial está ahí. La ejecución es lo que determina el resultado.**

---

## Inconsistencias Entre Documentos Detectadas

Para registro y resolución explícita:

| ID | Documentos en conflicto | Descripción |
|----|------------------------|-------------|
| INC-1 | `PROJECT_RULES.md` Regla 18 vs `PRODUCT_REQUIREMENTS_V2.md §6` | Regla 18 dice sidebar con "exactamente 9 secciones". PRD v2.0 define 10 módulos en el sidebar. **Resolución:** la Regla 18 aplica a v1.x. Debe actualizarse para indicar que aplica solo al archivo HTML actual. |
| INC-2 | `PROJECT_RULES.md` Regla 21 vs `SAAS_ARCHITECTURE_PLAN.md` intro | Regla 21 dice "`MASTER_CONTEXT.md` es la fuente principal de verdad". `SAAS_ARCHITECTURE_PLAN.md` se declara a sí mismo la arquitectura oficial. **Resolución sugerida:** `MASTER_CONTEXT.md` = verdad de negocio y funcionalidad; `SAAS_ARCHITECTURE_PLAN.md` = verdad técnica para v2.0. |
| INC-3 | `ROADMAP.md` vs `SAAS_ARCHITECTURE_PLAN.md` | `ROADMAP.md` define v2.0 solo como "Cloud/Multi-dispositivo" y v2.1 como "Plan Freemium". `SAAS_ARCHITECTURE_PLAN.md` incluye ambas cosas en v2.0. **Resolución:** el `ROADMAP.md` está desactualizado. `SAAS_ARCHITECTURE_PLAN.md` es la versión definitiva. |
| INC-4 | `MASTER_CONTEXT.md §8` vs `FUNCTIONS.md` | `MASTER_CONTEXT.md` dice "130 funciones críticas presentes". `FUNCTIONS.md` documenta 73. **Resolución:** 130 incluye handlers inline de HTML (`onclick=...`) y sub-funciones. El número exacto de funciones nombradas en JS es 73. Ambos pueden ser ciertos con definiciones distintas de "función". |
| INC-5 | `SECURITY_AUDIT.md` vs `SECURITY_REQUIREMENTS_V2.md` sobre magic link | `SECURITY_AUDIT.md` §Criterios mínimos dice "expiración ≤ 15 minutos". `BUILD_ORDER_V2.md §ETAPA 07` reproduce el mensaje AUTH-A3 de forma levemente diferente al texto de `SECURITY_REQUIREMENTS_V2.md`. La intención es la misma; el texto literal difiere. Impacto: bajo. |

---

*Documento generado por auditoría completa del proyecto — TradeOS*  
*Fecha: Junio 2026*  
*Basado en lectura directa de 22 documentos en `/docs` y análisis cruzado de todas las fuentes*  
*Este documento no modifica ningún código ni propone implementación — solo registra el estado real del proyecto*
