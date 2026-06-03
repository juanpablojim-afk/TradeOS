# SECURITY_AUDIT.md
## TradeOS — Reporte de Riesgos de Seguridad y Datos

**Versión auditada:** 1.0 Beta  
**Fecha:** Junio 2026  
**Rol:** Security Auditor  
**Fuentes analizadas:** `SCHEMA.md` · `FUNCTIONS.md` · `LOCALSTORAGE_AUDIT.md` · `tradeOS_base.html`  
**Método:** Auditoría directa de código fuente + análisis de documentación de arquitectura

> Este documento identifica riesgos reales encontrados en el código.
> No propone soluciones — eso corresponde al CTO.
> Los riesgos están ordenados por prioridad real, no por categoría.

---

## Resumen Ejecutivo

| Total de riesgos identificados | 23 |
|-------------------------------|-----|
| 🔴 Críticos | 2 |
| 🟠 Altos | 7 |
| 🟡 Medios | 8 |
| 🟢 Bajos | 6 |

**Estado de privacidad:** ✅ Conforme — No se detectaron llamadas a APIs externas, tracking, telemetría ni exfiltración de datos. La app respeta los 4 principios de privacidad no negociables del producto.

**Riesgo principal inmediato:** Pérdida silenciosa de trades cuando localStorage se satura. El usuario recibe confirmación de guardado pero el dato no se persistió.

---

## 🔴 CRÍTICOS — Pérdida de datos confirmada o probable

---

### C1 — `saveAll()` sin captura de `QuotaExceededError`

**Categoría:** Pérdida de datos  
**Función afectada:** `saveAll()`  
**Archivo:** `tradeOS_base.html` línea ~1485

Cuando localStorage alcanza su límite (~5-10 MB según navegador), `localStorage.setItem()` lanza una excepción `QuotaExceededError`. `saveAll()` no tiene ningún bloque `try/catch`. La excepción se propaga sin manejo.

El toast de confirmación ("Trade guardado ✓") se dispara antes del `setItem`, por lo que se muestra correctamente al usuario. El dato, sin embargo, no se persistió. El trader cree que su trade está guardado. No lo está.

Este es el único escenario en el sistema donde la app miente activamente al usuario sobre el estado de sus datos. No hay aviso, no hay detección posterior, no hay recuperación.

```javascript
// Estado actual — sin protección:
function saveAll(){
  localStorage.setItem('tt_cfg', JSON.stringify(cfg));      // puede fallar silenciosamente
  localStorage.setItem('tt_trades', JSON.stringify(trades)); // puede fallar silenciosamente
  // ...
}
```

---

### C2 — Saturación de localStorage por imágenes base64 sin control

**Categoría:** Pérdida de datos / Escalabilidad  
**Función afectada:** `prevImg()`, `saveTrade()`, `saveAll()`  
**Variable afectada:** `imgB64`, campo `trade.img`

Cada trade con imagen ocupa entre 50 KB y 300 KB dependiendo de la resolución de la captura. No existe límite de tamaño implementado, ni compresión antes de guardar, ni aviso al usuario sobre el espacio disponible.

Escenarios de saturación según estimaciones de `LOCALSTORAGE_AUDIT.md`:

| Escenario | Tamaño adicional | Total estimado |
|-----------|-----------------|----------------|
| 10 imágenes de 100 KB c/u | +1 MB | ~1.2 MB |
| 30 imágenes de 200 KB c/u | +6 MB | ~6.2 MB |
| Límite típico del navegador | — | ~5-10 MB |

Con 25-30 capturas de resolución normal, el límite se alcanza. En ese punto, el riesgo C1 se vuelve inevitable para cualquier `saveAll()`. El uso de screenshots es el flujo esperado del producto — esto no es un escenario edge sino el uso normal de un trader activo.

---

## 🟠 ALTOS — Riesgo real para datos del usuario en condiciones normales

---

### A1 — `importD()` no restaura `checklists` — Bug confirmado

**Categoría:** Pérdida de datos  
**Función afectada:** `importD()`  
**Severidad dentro del rango:** Alta (bug confirmado con pérdida silenciosa)

El JSON exportado por `exportD()` sí incluye la clave `checklists`. La función `importD()` no tiene la línea correspondiente para restaurarla en `tt_cl`. Al importar un backup, el historial completo del checklist diario queda en el JSON pero nunca se escribe en localStorage.

```javascript
// Lo que existe en importD():
if(d.cfg)         localStorage.setItem('tt_cfg', JSON.stringify(d.cfg));
if(d.trades)      localStorage.setItem('tt_trades', JSON.stringify(d.trades));
if(d.phases)      localStorage.setItem('tt_ph', JSON.stringify(d.phases));
if(d.completedPh) localStorage.setItem('tt_cp', JSON.stringify(d.completedPh));
if(d.funds)       localStorage.setItem('tt_fd', JSON.stringify(d.funds));
// Esta línea NO EXISTE:
// if(d.checklists) localStorage.setItem('tt_cl', JSON.stringify(d.checklists));
```

El usuario asume que su historial fue restaurado porque no recibe ningún error. El historial del checklist se pierde en toda migración entre navegadores o dispositivos.

---

### A2 — Importación sin validación de esquema ni tipo

**Categoría:** Corrupción de datos  
**Función afectada:** `importD()`

`importD()` valida que `d.cfg` sea truthy antes de escribirlo en localStorage, pero no valida que sea un objeto, que contenga las propiedades esperadas del esquema, ni que `d.trades` sea un array. Un JSON malformado o de otro sistema puede pasar la verificación:

```javascript
// Este JSON pasa todas las validaciones actuales:
{ "cfg": "texto cualquiera", "trades": {}, "funds": null }
```

Si el usuario importa un archivo incorrecto (un JSON de otra aplicación, un backup truncado, o un archivo renombrado por error), los datos actuales se sobreescriben sin que la app detecte la incompatibilidad. El resultado es corrupción de estado que puede manifestarse como pantalla en blanco, datos incorrectos en métricas, o errores de renderizado al cargar.

---

### A3 — Importación sin rollback — sobreescritura irreversible parcial

**Categoría:** Pérdida de datos  
**Función afectada:** `importD()`

`importD()` sobreescribe las claves de localStorage de forma secuencial e irreversible. No existe snapshot del estado previo antes de iniciar la operación. Si el proceso falla a mitad (por ejemplo, error al procesar `d.trades` después de haber escrito `d.cfg` correctamente), el estado queda parcialmente sobreescrito: parte de los datos son del backup nuevo, parte son de la sesión anterior.

No existe mecanismo de rollback. El `catch` global solo muestra un toast de error sin información de causa y sin intentar restaurar el estado anterior.

---

### A4 — `motherId` en réplicas apunta al trade incorrecto

**Categoría:** Corrupción de datos  
**Función afectada:** `applyReplicator()`  
**Documentado en:** `SCHEMA.md` — Hallazgo 4

En `applyReplicator()`, el `motherId` se asigna como `trades[0]?.id` antes de que el trade madre se guarde. En ese momento, `trades[0]` es el último trade registrado anteriormente, no el madre actual que se está creando. El trade madre se guarda dentro de `saveTrade()` al final del flujo.

Resultado: todas las réplicas hijas tienen un `motherId` que referencia el trade anterior, no al que las generó. El vínculo estructural madre-hija en los datos está roto para cada operación del replicador.

El impacto actual es limitado porque no hay funcionalidad que navegue explícitamente de hija a madre. El riesgo es de regresión garantizada cuando esa funcionalidad se implemente.

---

### A5 — `delTrade()` deja réplicas hijas huérfanas

**Categoría:** Corrupción de datos  
**Función afectada:** `delTrade()`  
**Documentado en:** `FUNCTIONS.md`

Al eliminar una trade madre, las réplicas hijas (`isChild: true`) no se eliminan. Quedan en `trades[]` con un `motherId` que ya no existe en el array. Estas hijas huérfanas:

- Aparecen en el historial con badge HIJA pero sin madre referenciable
- Se cuentan en el total de trades
- Afectan el WR, P&L y R:R promedio del dashboard
- Se incluyen en el cálculo de drawdown por cuenta
- Contaminan los patrones detectados por `renderPatterns()`

El impacto es inmediato y acumulativo: cada vez que el usuario elimina una trade madre que tenía réplicas, el estado de los datos se degrada.

---

### A6 — Modo incógnito: pérdida total de datos sin aviso

**Categoría:** Pérdida de datos  
**Documentado en:** `LOCALSTORAGE_AUDIT.md`

En modo incógnito, localStorage funciona durante la sesión pero se elimina completamente al cerrar la ventana o la pestaña. Si el trader abre TradeOS en modo incógnito (accidental o no), trabaja una sesión completa registrando trades, y cierra el navegador, pierde todos los datos sin posibilidad de recuperación.

No existe detección de modo incógnito ni aviso al usuario. La app se comporta con normalidad durante toda la sesión, confirmando guardados que no sobrevivirán al cierre.

---

### A7 — Dos pestañas abiertas: sobreescritura mutua sin sincronización

**Categoría:** Pérdida de datos  
**Función afectada:** `saveAll()`  
**Documentado en:** `LOCALSTORAGE_AUDIT.md`

Si el trader tiene dos instancias de TradeOS abiertas simultáneamente (dos pestañas del mismo archivo), cada `saveAll()` sobreescribe el localStorage completo con el estado en memoria de esa pestaña. Los cambios realizados en la otra pestaña se borran.

No existe sincronización entre pestañas (no se usa el evento `storage` de la Web API). En un escenario concreto: el trader registra un trade en la pestaña A, luego marca un ítem del checklist en la pestaña B — `saveAll()` de B sobreescribe `tt_trades` con el array que no incluye el trade de A.

---

## 🟡 MEDIOS — Riesgos en condiciones específicas o con impacto parcial

---

### M1 — `clDeleteBtn()` puede eliminar los 13 ítems predeterminados del checklist

**Categoría:** Corrupción de datos  
**Función afectada:** `clDeleteBtn()`  
**Regla violada:** PROJECT_RULES.md — Regla 4

PROJECT_RULES.md establece explícitamente: "Los ítems predeterminados del checklist nunca se eliminan al agregar nuevos — siempre incluir los 13 base." Sin embargo, `clDeleteBtn()` elimina ítems por índice posicional sin validar si el ítem es predeterminado o personalizado. El usuario puede romper esta invariante desde la UI sin restricción técnica.

La única recuperación disponible es `resetCLToDefault()`, que restaura los 13 ítems base pero elimina todos los personalizados.

---

### M2 — Colisión de IDs en trades réplica

**Categoría:** Corrupción de datos  
**Función afectada:** `applyReplicator()`  
**Documentado en:** `SCHEMA.md` — campo `id` en objeto `trade`

El ID de trades réplica se genera como `Date.now() + Math.random()`. `Math.random()` retorna un float que se suma al timestamp pero no garantiza unicidad matemática. Si el replicador crea múltiples hijos en la misma invocación con el mismo timestamp (posible en hardware moderno rápido), existe probabilidad de colisión de IDs.

Un ID duplicado rompería `delTrade(id)`, que filtra por ID para eliminar, potencialmente eliminando más de un trade o el trade incorrecto.

---

### M3 — IDs de fases personalizadas con doble prefijo `custom_`

**Categoría:** Corrupción de datos / Escalabilidad  
**Documentado en:** `SCHEMA.md` — Hallazgo 5

El ID real en `cfg.selectedPhases` es `'custom_custom_[timestamp]'` y en `phases{}` la clave es `'ph_custom_custom_[timestamp]'`. Esta inconsistencia no causa bugs hoy porque el código la genera y la consume con la misma lógica defectuosa.

Si cualquier función futura construye este ID manualmente asumiendo un solo prefijo `custom_`, el vínculo entre `selectedPhases` y `phases{}` se rompe silenciosamente. El progreso de tareas de esa fase queda desconectado de la fase visible en UI.

---

### M4 — `checklists` crece indefinidamente sin limpieza

**Categoría:** Escalabilidad  
**Clave afectada:** `tt_cl`  
**Documentado en:** `LOCALSTORAGE_AUDIT.md`

Por cada día de uso se agrega una entrada permanente al objeto `checklists`. No existe TTL, no hay limpieza automática, no hay límite implementado.

| Período de uso | Entradas acumuladas | Tamaño estimado |
|---------------|--------------------|-----------------| 
| 1 año diario | ~365 entradas | ~73 KB |
| 3 años diario | ~1,095 entradas | ~219 KB |
| 5 años diario | ~1,825 entradas | ~365 KB |

Aislado no es crítico, pero contribuye al agotamiento del límite de localStorage junto con C2. El historial de checkboxes de hace 3 años no tiene valor operativo para el trader.

---

### M5 — Bias del día: claves sueltas sin backup, sin migración posible

**Categoría:** Pérdida de datos / Privacidad  
**Claves afectadas:** `tt_bias_[dateString]`  
**Documentado en:** `LOCALSTORAGE_AUDIT.md`

El historial de bias se guarda como claves independientes en localStorage, una por día (`tt_bias_Mon Jun 02 2026`, etc.). Esto crea varios problemas simultáneos:

- No se incluyen en el backup JSON — se pierden en toda migración
- No hay inventario de cuántas claves existen ni cuánto espacio ocupan
- `localStorage.clear()` las elimina junto con todo lo demás
- En v2.0 con Supabase, no hay forma de migrar estas claves a cloud sin un proceso de inventario ad-hoc
- Contribuyen al crecimiento no controlado del localStorage

El bias del día es un dato de análisis que el trader puede considerar tan valioso como las notas de sus trades.

---

### M6 — Campos muertos `balance`, `drawdown`, `pnl` en objeto `Account`

**Categoría:** Corrupción de datos (riesgo de regresión futura)  
**Documentado en:** `SCHEMA.md` — Hallazgo 3

Los campos `balance`, `drawdown` y `pnl` del objeto `Account` se inicializan en el onboarding y nunca se actualizan. El P&L real de cada cuenta se calcula en runtime filtrando `trades[]`.

Estos tres campos existen en `cfg` serializado en localStorage con valores incorrectos (siempre reflejan el estado del onboarding, no el estado actual). Cualquier función futura que lea `account.pnl` directamente en lugar de calcularlo desde `trades[]` obtendrá siempre 0 sin producir ningún error visible.

---

### M7 — Tipo inconsistente: `Account.id` (number) vs `trade.account` (string)

**Categoría:** Corrupción de datos (riesgo de regresión futura)  
**Documentado en:** `SCHEMA.md` — Hallazgo 1

`Account.id` es un número entero (1, 2, 3...). `trade.account` guarda ese mismo ID como string (el valor del `<select>` del formulario). La comparación funciona porque el código usa `String(t.account) === String(a.id)` consistentemente.

Sin embargo, cualquier nueva función que compare `t.account === a.id` directamente producirá falsos negativos silenciosos en todos los filtros de cuenta — ningún trade se asociará a ninguna cuenta. El riesgo crece con cada función nueva que necesite cruzar trades con cuentas.

---

### M8 — `saveZoneEdit()` descarta silenciosamente campos no manejados

**Categoría:** Pérdida de datos  
**Función afectada:** `saveZoneEdit(field)`

La función solo tiene lógica para `field === 'name'`. Cualquier otro valor de `field` no produce error pero tampoco guarda nada. Si en el futuro se agregan zonas editables y se reutiliza esta función sin notar la limitación, los cambios del usuario se pierden silenciosamente con confirmación de éxito.

---

## 🟢 BAJOS — Buenas prácticas y riesgos de mantenimiento

---

### B1 — 6 funciones declaradas dos veces

**Categoría:** Mantenimiento  
**Funciones:** `resetAll()`, `openM()`, `closeM()`, `showN()`, `toggleTheme()`, `loadTheme()`  
**Documentado en:** `FUNCTIONS.md` — Hallazgos sobre funciones duplicadas

En JavaScript, la segunda declaración sobrescribe la primera silenciosamente. No causa bugs en el estado actual, pero si una edición modifica la primera copia creyendo que es la única activa, el cambio no tiene efecto. Riesgo de mantenimiento garantizado en cada edición del archivo.

---

### B2 — Código muerto en scope global

**Categoría:** Mantenimiento  
**Funciones:** `buildRuleSelect()`, primera versión de `toggleTag(tag, btn)`, `toggleReplicator()`  
**Documentado en:** `FUNCTIONS.md`

Funciones que ya no se usan en la UI actual pero permanecen en el scope global. Contribuyen a la confusión al leer el código, aumentan el tamaño del archivo y pueden ser accidentalmente invocadas si algún elemento HTML remanente las referencia.

---

### B3 — CSV exportado usa ID de cuenta en lugar del nombre del firm

**Categoría:** Experiencia de datos  
**Función afectada:** `exportTradesCSV()`  
**Documentado en:** `SCHEMA.md` — Hallazgo 2

El CSV muestra `"1"` en la columna "Cuenta" en lugar de `"FTMO"` o el nombre del broker. El archivo exportado es ilegible como documento standalone sin cruzarlo manualmente con `cfg.accounts`. Para análisis externos en Excel o herramientas similares, esto elimina gran parte del valor del export.

---

### B4 — `tt_cl` incluida en export pero no en import — inconsistencia de documentación

**Categoría:** Mantenimiento  
**Relacionado con:** A1

Además del bug funcional (A1), el JSON exportado contiene la clave `checklists` visible, lo que lleva a cualquier desarrollador que inspeccione el backup a asumir que se restaura al importar. La inconsistencia entre lo que está en el JSON y lo que `importD()` procesa no está documentada en ningún aviso de la UI ni en un comentario de código.

---

### B5 — Prefijo de claves inconsistente (`tt_` vs `jft_`)

**Categoría:** Mantenimiento  
**Documentado en:** `LOCALSTORAGE_AUDIT.md` y `SCHEMA.md` — Hallazgo 10

Una sola clave (`jft_beta`) usa el prefijo `jft_`. Todas las demás usan `tt_`. La documentación original del proyecto indicaba `jft_` como prefijo universal. Si alguna función de limpieza selectiva o migración filtra por prefijo, necesita manejar ambos prefijos o perderá `jft_beta`.

---

### B6 — Email de soporte hardcodeado en código cliente

**Categoría:** Privacidad  
**Función afectada:** `openSupport()`

`tradeossoporte@gmail.com` está hardcodeado en el JS del archivo. En v1.0 local es irrelevante. En v1.2 con hosting web, cualquier persona que inspeccione el código fuente del navegador tiene acceso a ese email. No es un riesgo crítico para los datos del trader, pero es una superficie de spam que crece con la audiencia del producto.

---

## Tabla de Riesgos — Resumen Priorizado

| ID | Severidad | Categoría | Descripción breve | Función(es) afectada(s) |
|----|-----------|-----------|-------------------|------------------------|
| C1 | 🔴 Crítico | Pérdida de datos | `saveAll()` silencia `QuotaExceededError` — trade "guardado" que no existe | `saveAll()` |
| C2 | 🔴 Crítico | Pérdida / Escalabilidad | Base64 sin control agota localStorage inevitablemente | `prevImg()`, `saveTrade()`, `saveAll()` |
| A1 | 🟠 Alto | Pérdida de datos | Bug confirmado: `importD()` no restaura `checklists` | `importD()` |
| A2 | 🟠 Alto | Corrupción | Importación sin validación de tipo ni esquema | `importD()` |
| A3 | 🟠 Alto | Pérdida de datos | Importación sin rollback — sobreescritura irreversible parcial | `importD()` |
| A4 | 🟠 Alto | Corrupción | `motherId` en réplicas apunta al trade incorrecto | `applyReplicator()` |
| A5 | 🟠 Alto | Corrupción | Eliminar madre deja hijas huérfanas contaminando métricas | `delTrade()` |
| A6 | 🟠 Alto | Pérdida de datos | Modo incógnito: pérdida total sin aviso al usuario | — |
| A7 | 🟠 Alto | Pérdida de datos | Dos pestañas: sobreescritura mutua sin sincronización | `saveAll()` |
| M1 | 🟡 Medio | Corrupción | `clDeleteBtn()` puede eliminar los 13 ítems base del checklist | `clDeleteBtn()` |
| M2 | 🟡 Medio | Corrupción | Colisión de IDs en trades réplica — probabilidad baja, impacto alto | `applyReplicator()` |
| M3 | 🟡 Medio | Corrupción / Escalabilidad | Doble prefijo `custom_` — frágil ante construcción manual de IDs | `saveCustomPhase()` |
| M4 | 🟡 Medio | Escalabilidad | `checklists` crece indefinidamente sin limpieza automática | `toggleCL()`, `saveAll()` |
| M5 | 🟡 Medio | Pérdida / Privacidad | Bias: claves sueltas, fuera de backup, sin migración posible | `saveBias()` |
| M6 | 🟡 Medio | Corrupción (futura) | Campos muertos en `Account` — trampa para regresiones futuras | — |
| M7 | 🟡 Medio | Corrupción (futura) | Tipo ambiguo `account` number vs string — frágil en funciones nuevas | Múltiples |
| M8 | 🟡 Medio | Pérdida de datos | `saveZoneEdit()` descarta silenciosamente campos no manejados | `saveZoneEdit()` |
| B1 | 🟢 Bajo | Mantenimiento | 6 funciones declaradas dos veces | Múltiples |
| B2 | 🟢 Bajo | Mantenimiento | Código muerto activo en scope global | Múltiples |
| B3 | 🟢 Bajo | Experiencia de datos | CSV exportado usa ID de cuenta, no nombre del firm | `exportTradesCSV()` |
| B4 | 🟢 Bajo | Mantenimiento | `tt_cl` incluida en export pero no en import — sin documentar | `importD()`, `exportD()` |
| B5 | 🟢 Bajo | Mantenimiento | Prefijo de claves inconsistente (`tt_` vs `jft_`) | Múltiples |
| B6 | 🟢 Bajo | Privacidad | Email de soporte hardcodeado en código cliente | `openSupport()` |

---

## Criterios mínimos de seguridad para v2.0 (Supabase)

Estos criterios deben estar resueltos antes del primer deploy a cloud. No son negociables.

1. **Row Level Security (RLS)** activado en todas las tablas de Supabase — ningún usuario puede leer datos de otro bajo ninguna circunstancia
2. **Magic link tokens** deben tener expiración ≤ 15 minutos
3. **Imágenes base64** no deben ir a Supabase sin compresión previa — definir límite máximo de tamaño por imagen antes de migración
4. **Validación de esquema en servidor** antes de insertar en PostgreSQL — no confiar exclusivamente en validaciones del cliente
5. **Política de retención de datos** definida antes del lanzamiento: ¿qué sucede con los datos de un usuario que cancela su suscripción?
6. **Migración de `tt_bias_*`** requiere inventario ad-hoc — estas claves no tienen estructura centralizada y deben procesarse clave por clave
7. **Los IDs de trades deben ser UUIDs** en v2.0 — `Date.now()` no es suficiente para un entorno multiusuario y multidispositivo
8. **Los campos muertos `balance`, `drawdown`, `pnl` de `Account`** deben eliminarse del esquema antes de migrar a PostgreSQL — no persistir campos que nunca se actualizan

---

*Documento generado por auditoría directa del código fuente — Junio 2026*  
*Autor: Security Auditor — TradeOS*  
*Estado: Solo identificación de riesgos — sin propuestas de solución (responsabilidad del CTO)*
