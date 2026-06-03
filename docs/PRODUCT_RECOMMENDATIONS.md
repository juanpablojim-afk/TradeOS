# PRODUCT_RECOMMENDATIONS.md
## TradeOS — Recomendaciones de Producto

**Autor:** Product Manager  
**Fecha:** Junio 2026  
**Basado en:** SECURITY_AUDIT.md · MASTER_CONTEXT.md · TASKS.md · ROADMAP.md · BUSINESS_VISION.md · PROJECT_RULES.md  
**Versión evaluada:** 1.0 Beta

---

## Resumen Ejecutivo

La auditoría de seguridad reveló 23 riesgos. Desde el punto de vista de producto, el diagnóstico es claro: **TradeOS tiene un excelente conjunto de funciones pero una capa de persistencia de datos frágil**. El mayor riesgo no es técnico — es de reputación. Un trader que pierde sus trades porque `saveAll()` falló en silencio, o que importa un backup y pierde todo su historial de checklist, no vuelve a comprar ni recomienda el producto. En un mercado que se mueve por boca a boca entre traders, un solo caso así puede costar decenas de ventas.

La priorización de este sprint responde a una pregunta simple: **¿qué puede destruir la confianza del usuario antes de que el producto llegue a su segunda venta?**

---

## 1. Riesgos que deben corregirse ANTES del lanzamiento público

> Criterio: cualquier riesgo que pueda causar pérdida de datos sin que el usuario lo sepa, o que pueda romper el flujo de trabajo diario del trader.

---

### 🔴 BLOQUEANTES DE LANZAMIENTO

#### BL-1 — `saveAll()` miente al usuario cuando localStorage está lleno (C1 + C2 combinados)

**Severidad:** Crítica  
**Fuente:** SECURITY_AUDIT.md — C1 y C2  

**Problema:** Cuando localStorage se satura (inevitable con uso normal de imágenes), el trade "se guarda" con toast de confirmación pero no persiste. El trader opera convencido de tener registro de sus trades. No lo tiene.

**Por qué es bloqueante:** Viola la confianza fundamental del producto. Un trader que descubre que perdió 2 semanas de trades sin advertencia no recomienda TradeOS — lo destruye públicamente. El modelo de distribución depende de recomendaciones orgánicas.

**Lo que debe quedar resuelto:**
- `saveAll()` debe capturar `QuotaExceededError` y mostrar aviso crítico visible (no un toast — un modal que bloquee la UI hasta que el usuario actúe)
- Las imágenes deben comprimirse antes de guardarse (`canvas.toDataURL` con calidad reducida)
- Una barra de uso de localStorage debe ser visible en Ajustes con alerta al 80%
- Al llegar al 90%, la app debe avisar proactivamente y sugerir exportar backup + eliminar fotos antiguas

**No entregar v1.0 hasta que esto esté resuelto.**

---

#### BL-2 — Importar backup no restaura el historial de checklist (A1)

**Severidad:** Alta  
**Fuente:** SECURITY_AUDIT.md — A1  

**Problema:** Bug confirmado. `importD()` omite la línea que restaura `tt_cl`. El historial de checklist está en el JSON exportado pero nunca se escribe al importar. El usuario migra entre dispositivos y pierde todo su historial de disciplina diaria.

**Por qué es bloqueante:** El checklist es una de las funciones más usadas diariamente. Si un usuario reinstala Chrome, cambia de computadora, o restaura un backup, pierde datos que confía haber guardado. Es un bug con impacto directo y confirmado.

**Lo que debe quedar resuelto:**
- Una línea de código: agregar `if(d.checklists) localStorage.setItem('tt_cl', JSON.stringify(d.checklists));` en `importD()`
- Verificar que la línea existente de `exportD()` ya incluye `checklists` (confirmado en auditoría — sí lo incluye)

---

#### BL-3 — Importar un JSON incorrecto destruye todos los datos sin rollback (A2 + A3)

**Severidad:** Alta  
**Fuente:** SECURITY_AUDIT.md — A2 y A3  

**Problema:** `importD()` no valida el esquema antes de escribir, y no hace snapshot del estado previo. Un archivo JSON de otra aplicación, un backup truncado, o un archivo renombrado por error puede pasar todas las validaciones actuales y sobreescribir datos reales con basura. Si falla a mitad del proceso, el estado queda parcialmente corrompido.

**Por qué es bloqueante:** El backup/import es la única herramienta de recuperación de datos del producto. Si esa herramienta puede destruir datos, el safety net no existe.

**Lo que debe quedar resuelto:**
- Antes de escribir: snapshot del estado actual en memoria (no requiere localStorage extra)
- Validación mínima de esquema: `d.cfg` debe ser objeto, `d.trades` debe ser array, `d.cfg.name` debe existir como string
- Si alguna validación falla: rollback al snapshot + mensaje de error específico indicando qué campo está mal
- Si el proceso falla a mitad: rollback automático

---

#### BL-4 — Eliminar un trade madre contamina métricas con réplicas huérfanas (A5)

**Severidad:** Alta  
**Fuente:** SECURITY_AUDIT.md — A5  

**Problema:** Al borrar una trade madre, las réplicas hijas quedan en `trades[]` con `motherId` que no existe. Estas hijas afectan win rate, P&L, R:R promedio, drawdown por cuenta y análisis de patrones. Cada eliminación de una madre degrada silenciosamente todos los números del dashboard.

**Por qué es bloqueante:** El dashboard muestra métricas incorrectas de forma acumulativa. Un trader que opera con replicador (caso de uso central del producto) y elimina una operación mala verá sus estadísticas contaminadas permanentemente.

**Lo que debe quedar resuelto:**
- `delTrade()` debe verificar si el trade es madre (`isMother: true`) y eliminar también todas las hijas (`isChild: true, motherId === id`)
- O mostrar modal de confirmación: "¿Eliminar también las X réplicas hijas de este trade?"

---

### 🟠 IMPORTANTES — Corregir antes del primer sprint de distribución (primeras 2 semanas post-lanzamiento)

Estos riesgos no bloquean el lanzamiento pero deben resolverse antes de activar afiliados o campañas de distribución.

---

#### P1 — El CSV exportado muestra IDs en lugar de nombres de firma (B3)

**Fuente:** SECURITY_AUDIT.md — B3  
**Impacto usuario:** Un trader que exporta a Excel para análisis externo ve `"1"`, `"2"`, `"3"` en la columna Cuenta en lugar de `"FTMO"`, `"Funding Pips"`. El archivo es inutilizable como documento standalone.  
**Valor de corregir:** Alto — el CSV es una promesa implícita de portabilidad de datos. Si ese dato no funciona, el usuario siente que el producto lo traicionó.  
**Esfuerzo:** Muy bajo. Una línea de código en `exportTradesCSV()`.

---

#### P2 — Modo incógnito destruye todos los datos sin aviso (A6)

**Fuente:** SECURITY_AUDIT.md — A6  
**Impacto usuario:** Un usuario que abre TradeOS en modo incógnito (accidental o deliberado) puede registrar una sesión completa de trading y perderla al cerrar el navegador. Sin aviso.  
**Valor de corregir:** Alto — el escenario accidental es real y el impacto es máximo.  
**Esfuerzo:** Bajo. Detectar modo incógnito al cargar y mostrar banner de advertencia prominente.

---

#### P3 — Dos pestañas destruyen datos mutuamente (A7)

**Fuente:** SECURITY_AUDIT.md — A7  
**Impacto usuario:** Raro, pero posible. Un trader que tiene TradeOS abierto en dos pestañas puede perder un trade registrado.  
**Valor de corregir:** Medio.  
**Esfuerzo:** Bajo. Usar el evento `storage` de la Web API para sincronizar o al menos detectar conflicto y avisar.

---

## 2. Riesgos que pueden esperar

> Criterio: riesgos reales pero que no destruyen datos del usuario en uso normal, o cuya probabilidad en v1.0 es baja.

---

| ID | Riesgo | Por qué puede esperar | Versión objetivo |
|----|--------|-----------------------|-----------------|
| A4 | `motherId` apunta al trade incorrecto | No hay función actual que navegue de hija a madre. El bug existirá cuando se implemente esa feature. Corregir antes de construir navegación madre↔hija | v1.1 |
| M1 | `clDeleteBtn()` puede borrar ítems base del checklist | El usuario debe hacer un esfuerzo deliberado. Recuperación disponible con `resetCLToDefault()` | v1.1 |
| M2 | Colisión de IDs en trades réplica | Probabilidad matemáticamente baja en hardware normal. Colisión requeriría microsegundos exactos | v1.1 |
| M3 | Doble prefijo `custom_` en fases | No causa bugs hoy. Solo bloquea futura extensión | v1.1 |
| M4 | `checklists` crece sin límite | A 73KB/año, no es crítico en v1.0. Implementar TTL de 90-180 días en v1.1 | v1.1 |
| M5 | Bias no incluido en backup | El bias es valioso pero no operacional. Agregar al backup export en v1.1 | v1.1 |
| M6 | Campos muertos en `Account` | Trampa para regresiones futuras. Limpiar antes de v2.0 / migración a Supabase | Pre-v2.0 |
| M7 | `account` number vs string | Funciona hoy por uso consistente de `String()`. Normalizar en v1.1 | v1.1 |
| M8 | `saveZoneEdit()` descarta campos silenciosamente | Solo afecta si se agrega lógica futura al switch. Documentar como deuda técnica | v1.1 |
| B1 | 6 funciones duplicadas | No causa bugs, solo confusión en mantenimiento | v1.1 refactor |
| B2 | Código muerto en scope global | Mismo caso que B1 | v1.1 refactor |
| B5 | Prefijo `jft_` vs `tt_` inconsistente | Solo importa si se hace limpieza selectiva por prefijo | Pre-v2.0 |
| B6 | Email de soporte hardcodeado | Irrelevante en v1.0 local. Revisar en v1.2 con hosting público | v1.2 |

---

## 3. Mejoras que aportan más valor al usuario

> Criterio: funciones que el trader nota, usa todos los días, y que generan el "esto es exactamente lo que necesitaba".

---

### MU-1 — Filtro por rango de fechas en Mis Trades

**Prioridad:** Alta  
**Frecuencia:** Diaria / semanal  
**Justificación:** Un trader con 6 meses de historia necesita ver "solo esta semana" o "este mes" para analizar rachas. Hoy tiene que recorrer manualmente. Es la queja más obvia de cualquier journal que crece.  
**Esfuerzo:** Medio.

---

### MU-2 — Estadísticas por par y por sesión

**Prioridad:** Alta  
**Frecuencia:** Semanal  
**Justificación:** "¿En qué par tengo mejor win rate? ¿London o NY me va mejor?" Son preguntas que todo trader serio se hace. Hoy no hay respuesta directa en la app. El análisis de patrones menciona esto pero no lo resuelve con un número claro.  
**Esfuerzo:** Medio. Los datos ya están en `trades[]`. Solo falta la vista.

---

### MU-3 — Exportar trades a PDF

**Prioridad:** Media  
**Frecuencia:** Mensual  
**Justificación:** Los traders de prop firms necesitan documentar su performance para renovaciones, escalado o como evidencia ante la firma. Un PDF con los trades del mes es un entregable real. El CSV no sirve para eso.  
**Esfuerzo:** Medio-alto. Requiere generar PDF desde el navegador (librería o impresión CSS).

---

### MU-4 — Tooltip en métricas del dashboard explicando el cálculo

**Prioridad:** Media  
**Frecuencia:** Diaria (primera semana de uso intenso)  
**Justificación:** "¿Cómo se calcula el Score de Consistencia?" es una pregunta legítima de un profesional. El trader no necesita que le expliquen qué es un pip, pero sí necesita saber exactamente cómo se computa su score para confiar en él.  
**Esfuerzo:** Bajo. Tooltips con `title` o mini-modales.

---

### MU-5 — Bias del día incluido en el backup exportado

**Prioridad:** Media  
**Frecuencia:** Diaria  
**Justificación:** El trader escribe su bias todos los días. No incluirlo en el backup es inconsistente con la promesa de "tus datos son tuyos". Actualmente se pierde en toda migración.  
**Esfuerzo:** Medio. Requiere centralizar las claves `tt_bias_*` dispersas.

---

## 4. Mejoras que aportan más valor al negocio

> Criterio: funciones que aumentan conversión, retención, distribución o preparan el camino al SaaS.

---

### MN-1 — Hospedar en dominio propio (v1.2)

**Impacto negocio:** Crítico  
**Justificación:** Safari iOS bloquea localStorage en archivos locales. El mercado objetivo es Latinoamérica móvil. Un trader en iPhone no puede usar TradeOS hoy. Resolver esto abre un segmento completo del ICP y es requisito para cualquier campaña de distribución seria.  
**Decisión:** Netlify o GitHub Pages como primer paso. Costo: $0-10/mes.

---

### MN-2 — Mecanismo de actualización claro para compradores existentes

**Impacto negocio:** Alto  
**Justificación:** En el modelo de archivo único, ¿cómo sabe un comprador que hay una nueva versión? ¿Cómo la descarga sin pagar de nuevo? Hoy no hay respuesta clara. Cada venta sin mecanismo de update es un comprador que se siente abandonado en v1.1.  
**Recomendación:** Splash en la app que mencione "v1.0 Beta — actualizaciones gratuitas para compradores" + notificación en Gumroad a compradores previos en cada nueva versión.

---

### MN-3 — Página de testimonios / prueba social dentro de la app (o landing)

**Impacto negocio:** Alto  
**Justificación:** El canal de distribución principal son creadores de contenido y boca a boca. El producto necesita una landing page con capturas reales del producto, testimonios de traders reales y video demo. Hoy la conversión depende solo del afiliado.  
**Nota PM:** Esto está fuera del scope del archivo HTML — es una decisión de marketing. Documentar como acción para Juan Pa.

---

### MN-4 — Guardar métricas del momento de fondeo como hito histórico

**Impacto negocio:** Medio-Alto  
**Justificación:** Cuando un trader pasa de Challenge a Fondeado, ese es el momento emocional más importante de su carrera reciente. Si la app captura y celebra ese hito (con las métricas exactas de ese día), el trader tiene un recuerdo que quiere compartir en redes. Contenido orgánico gratuito para el negocio.  
**Esfuerzo:** Medio.

---

### MN-5 — Preparar esquema de datos para v2.0 Supabase

**Impacto negocio:** Estratégico (no inmediato)  
**Justificación:** El SECURITY_AUDIT.md listó 8 criterios no negociables antes de cualquier migración cloud. Los más urgentes son: normalizar tipos de ID (`Date.now()` → UUIDs), eliminar campos muertos de `Account`, y centralizar claves `tt_bias_*`. Hacer este trabajo en v1.1 significa que v2.0 no requiere migración de datos sino solo un cambio de capa de persistencia.  
**Recomendación:** Incluir como refactor silencioso en v1.1 antes de construir features nuevas.

---

## 5. Sprint Recomendado — v1.1

> Objetivo del sprint: cerrar todos los riesgos bloqueantes de lanzamiento y las mejoras de alta prioridad que aumentan confianza del usuario.

**Duración estimada:** 2-3 semanas de desarrollo  
**Criterio de éxito:** Ningún riesgo 🔴 abierto. Ningún bug que cause pérdida silenciosa de datos.

---

### Bloque A — Críticos (no negociables para el sprint)

| # | Tarea | ID Origen | Prioridad en sprint |
|---|-------|-----------|---------------------|
| 1 | Capturar `QuotaExceededError` en `saveAll()` con modal de bloqueo | C1 | 🔴 Primero |
| 2 | Comprimir imágenes antes de guardar en `prevImg()` | C2 | 🔴 Primero |
| 3 | Barra de uso de localStorage en Ajustes con alerta al 80% | C2 | 🔴 Primero |
| 4 | Fix `importD()`: restaurar `tt_cl` (checklists) | A1 | 🔴 Primero |
| 5 | Validación de esquema en `importD()` antes de escribir | A2 | 🔴 Primero |
| 6 | Rollback de estado previo si `importD()` falla | A3 | 🔴 Primero |
| 7 | `delTrade()`: eliminar réplicas hijas al borrar madre | A5 | 🔴 Primero |

---

### Bloque B — Importantes (mismo sprint si hay capacidad, o sprint inmediato siguiente)

| # | Tarea | ID Origen | Prioridad |
|---|-------|-----------|-----------|
| 8 | Fix CSV: mostrar nombre de firma, no ID de cuenta | B3 | 🟠 Alta |
| 9 | Detectar modo incógnito al cargar y mostrar advertencia | A6 | 🟠 Alta |
| 10 | Sincronizar pestañas o detectar conflicto con evento `storage` | A7 | 🟠 Alta |
| 11 | Incluir `tt_bias_*` en exportación de backup | M5 | 🟠 Alta |

---

### Bloque C — Deuda técnica (mismo sprint como refactor silencioso)

| # | Tarea | ID Origen | Nota |
|---|-------|-----------|------|
| 12 | Eliminar 6 funciones duplicadas | B1 | Sin impacto en UI |
| 13 | Limpiar código muerto del scope global | B2 | Sin impacto en UI |
| 14 | Normalizar `account` como string consistente en todo el código | M7 | Previene regresiones |
| 15 | Documentar limitación de `saveZoneEdit()` en comentario de código | M8 | 10 minutos de trabajo |
| 16 | Limpiar doble prefijo `custom_` en fases | M3 | Previene regresiones |

---

### Bloque D — Features nuevas (entran en v1.1 si el Bloque A y B están cerrados)

| # | Feature | ID Origen | Justificación |
|---|---------|-----------|---------------|
| 17 | Filtro por rango de fechas en Mis Trades | TASKS.md | Alta frecuencia de uso, datos ya disponibles |
| 18 | Estadísticas por par y por sesión | TASKS.md | Pregunta que todo trader se hace semanalmente |
| 19 | Tooltip en métricas del dashboard | TASKS.md | Genera confianza en el Score de Consistencia |

---

## Decisiones que quedan fuera de este análisis

Las siguientes decisiones requieren validación de Juan Pa o del rol correspondiente antes de implementar:

| Decisión | Requiere | Estado |
|----------|----------|--------|
| ¿Hospedar en Netlify/GitHub Pages ahora o esperar a dominio propio? | Juan Pa | Pendiente |
| ¿Mecanismo de notificación de updates a compradores Gumroad? | Juan Pa | Pendiente |
| ¿Límite máximo de imagen en KB antes de comprimir? | CTO + Trading Expert | Pendiente (sugerencia: 100KB post-compresión) |
| ¿TTL de historial de checklist? ¿90 días, 180 días, 1 año? | Trading Expert | Pendiente |
| ¿Modal de confirmación al borrar madre con hijas, o eliminación automática? | UX review | Pendiente |

---

## Tabla de priorización completa

| ID | Descripción | Tipo | Impacto usuario | Impacto negocio | Esfuerzo | Sprint |
|----|------------|------|-----------------|-----------------|----------|--------|
| BL-1 | saveAll() silencia error crítico + compresión imágenes | Bug crítico | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Medio | v1.1 Bloque A |
| BL-2 | importD() no restaura checklists | Bug confirmado | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Muy bajo | v1.1 Bloque A |
| BL-3 | Importación sin validación ni rollback | Bug crítico | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Medio | v1.1 Bloque A |
| BL-4 | delTrade() deja huérfanas que contaminan métricas | Bug confirmado | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Bajo | v1.1 Bloque A |
| P1 | CSV con nombres de firma, no IDs | Bug menor | ⭐⭐⭐⭐ | ⭐⭐⭐ | Muy bajo | v1.1 Bloque B |
| P2 | Detección de modo incógnito | Prevención | ⭐⭐⭐⭐ | ⭐⭐⭐ | Bajo | v1.1 Bloque B |
| P3 | Sincronización de pestañas | Prevención | ⭐⭐⭐ | ⭐⭐ | Bajo | v1.1 Bloque B |
| MU-5 | Bias en backup exportado | Feature | ⭐⭐⭐⭐ | ⭐⭐⭐ | Medio | v1.1 Bloque B |
| MU-1 | Filtro por rango de fechas | Feature | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | Medio | v1.1 Bloque D |
| MU-2 | Stats por par y sesión | Feature | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | Medio | v1.1 Bloque D |
| MU-4 | Tooltips en métricas dashboard | Feature | ⭐⭐⭐ | ⭐⭐⭐ | Bajo | v1.1 Bloque D |
| MN-1 | Hosting en dominio propio | Infraestructura | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Bajo | v1.2 |
| MN-2 | Mecanismo de updates para compradores | Negocio | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Bajo | v1.1 / marketing |
| MU-3 | Exportar trades a PDF | Feature | ⭐⭐⭐⭐ | ⭐⭐⭐ | Alto | v1.2 |
| MN-4 | Hito histórico al pasar a Fondeado | Feature | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Medio | v1.2 |
| MN-5 | Preparar esquema para Supabase | Deuda técnica | ⭐ | ⭐⭐⭐⭐⭐ | Alto | Pre-v2.0 |

---

## Criterios de salida del sprint v1.1

El sprint v1.1 se considera cerrado cuando:

- [ ] `saveAll()` captura y muestra error visible ante `QuotaExceededError`
- [ ] Las imágenes se comprimen antes de guardarse
- [ ] La barra de uso de localStorage está visible en Ajustes
- [ ] `importD()` restaura `tt_cl` correctamente
- [ ] `importD()` valida esquema y hace rollback ante fallo
- [ ] `delTrade()` limpia réplicas huérfanas
- [ ] CSV exportado muestra nombres de firma
- [ ] Modo incógnito muestra advertencia al cargar
- [ ] Ningún riesgo 🔴 abierto en SECURITY_AUDIT.md
- [ ] `node --check` pasa en el archivo final
- [ ] CHANGELOG.md actualizado con todos los cambios

---

*Documento generado por Product Manager — TradeOS*  
*Próxima revisión: al cierre del sprint v1.1*  
*Para implementación técnica: pasar Bloque A al CTO con este documento como brief*
