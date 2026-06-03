# LOCALSTORAGE_AUDIT.md
## TradeOS — Auditoría Real de localStorage

**Versión:** 1.0 · **Extraído de:** tradeOS_base.html (3206 líneas)
**Fecha de auditoría:** Junio 2026
**Método:** Búsqueda de todos los `localStorage.getItem`, `localStorage.setItem`, `localStorage.clear()` en el código fuente

---

## Resumen Ejecutivo

| Métrica | Valor |
|---------|-------|
| Total de claves identificadas | 9 claves distintas |
| Claves gestionadas por `saveAll()` / `loadAll()` | 6 |
| Claves gestionadas directamente (fuera del sistema) | 3 |
| Prefijo principal | `tt_` |
| Prefijo secundario (solo una clave) | `jft_` |
| Prefijo dinámico (bias por fecha) | `tt_bias_[dateString]` |
| Claves incluidas en backup JSON | 6 |
| Claves NO incluidas en backup JSON | 3 (`tt_bias_*`, `tt_theme`, `jft_beta`) |

---

## Inventario Completo de Claves

### Claves del sistema principal (gestionadas por `saveAll()` / `loadAll()`)

| Clave localStorage | Variable JS | Tipo de dato | Escribe | Lee | Incluida en backup |
|--------------------|-------------|--------------|---------|-----|-------------------|
| `tt_cfg` | `cfg` | `Object` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |
| `tt_trades` | `trades` | `Array` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |
| `tt_ph` | `phases` | `Object` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |
| `tt_cp` | `completedPh` | `Array` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |
| `tt_fd` | `funds` | `Array` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |
| `tt_cl` | `checklists` | `Object` | `saveAll()`, `importD()` | `loadAll()` | ✅ Sí |

### Claves gestionadas directamente (fuera de `saveAll()`)

| Clave localStorage | Variable JS | Tipo de dato | Escribe | Lee | Incluida en backup |
|--------------------|-------------|--------------|---------|-----|-------------------|
| `tt_theme` | — (sin variable JS) | `string` ('light' \| 'dark') | `toggleTheme()` | `loadTheme()` | ❌ No |
| `tt_bias_[fecha]` | — (sin variable JS) | `string` (texto libre) | `saveBias(val)` | `loadBias()` | ❌ No |
| `jft_beta` | — (sin variable JS) | `string` ('1') | Inline en beta splash button | IIFE línea 1107 | ❌ No |

---

## Detalle por Clave

### `tt_cfg`

**Tipo:** `JSON.stringify(cfg)` → Object complejo

**Escribe:**
```javascript
// En saveAll():
localStorage.setItem('tt_cfg', JSON.stringify(cfg));

// En importD():
if(d.cfg) localStorage.setItem('tt_cfg', JSON.stringify(d.cfg));
```

**Lee:**
```javascript
// En loadAll():
try { const c = localStorage.getItem('tt_cfg'); if(c) cfg = JSON.parse(c); } catch(e) { cfg = {}; }
```

**Tamaño estimado:** Variable. Depende del número de cuentas, reglas, metas y fases. Sin imágenes: ~2-5 KB típico.

**Riesgo de pérdida:** Bajo. Se sobreescribe en cada `saveAll()` pero con try/catch en lectura.

---

### `tt_trades`

**Tipo:** `JSON.stringify(trades)` → Array de objetos trade

**Escribe:**
```javascript
localStorage.setItem('tt_trades', JSON.stringify(trades));
```

**Lee:**
```javascript
try { trades = JSON.parse(localStorage.getItem('tt_trades') || '[]'); } catch(e) { trades = []; }
```

**Tamaño estimado:** **Esta es la clave más grande.** Cada trade sin imagen: ~200-400 bytes. Cada trade con imagen base64: ~50-300 KB según tamaño de captura. Con 100 trades sin imágenes: ~40 KB. Con 10 trades con imágenes de 200 KB cada una: ~2 MB solo en esta clave.

**Riesgo crítico:** La saturación de localStorage ocurre principalmente aquí. El límite típico del navegador es 5-10 MB para todo el origen. Con 20-30 capturas de alta resolución, puede alcanzarse el límite.

**Comportamiento si localStorage está lleno:** `saveAll()` no tiene manejo de error para `QuotaExceededError`. Si localStorage está lleno, `localStorage.setItem()` lanza una excepción que **no está capturada**, resultando en pérdida silenciosa del trade que se intentaba guardar.

---

### `tt_ph`

**Tipo:** `JSON.stringify(phases)` → Object con estado de tareas

**Escribe/Lee:** Igual que `tt_cfg`. Tamaño muy pequeño (~0.5-2 KB).

---

### `tt_cp`

**Tipo:** `JSON.stringify(completedPh)` → Array de strings

**Tamaño:** Mínimo. Máximo 10-15 IDs de fases.

---

### `tt_fd`

**Tipo:** `JSON.stringify(funds)` → Array de objetos fund

**Tamaño estimado:** Cada movimiento ~150-200 bytes. Con 500 movimientos: ~100 KB.

---

### `tt_cl`

**Tipo:** `JSON.stringify(checklists)` → Object por fecha

**Estructura de clave interna:** `{ "Mon Jun 02 2026": { "sleep_c": true, ... } }`

**Crecimiento:** Crece indefinidamente. Por cada día de uso se agrega una entrada. Después de 1 año de uso diario: ~365 entradas, ~50-80 KB estimado.

**Riesgo a largo plazo:** No hay limpieza automática de entradas antiguas. En 3-4 años de uso intensivo puede contribuir significativamente al uso de localStorage.

---

### `tt_theme`

**Tipo:** `string` literal `'light'` o `'dark'`

**Escribe:**
```javascript
// En toggleTheme():
localStorage.setItem('tt_theme', next); // next = 'light' | 'dark'
```

**Lee:**
```javascript
// En loadTheme():
const saved = localStorage.getItem('tt_theme');
if(saved === 'light') document.documentElement.setAttribute('data-theme', 'light');
```

**Comportamiento:** Si la clave no existe o tiene valor `'dark'`, se usa el modo oscuro (default). No se llama a `saveAll()` — es una clave independiente.

**No incluida en backup:** Si el usuario importa un backup, su preferencia de tema se mantiene porque no se sobrescribe. Comportamiento correcto.

---

### `tt_bias_[dateString]`

**Tipo:** `string` (texto libre del usuario)

**Formato de la clave:** `'tt_bias_' + new Date().toDateString()`
- Ejemplo: `tt_bias_Mon Jun 02 2026`
- Ejemplo: `tt_bias_Tue Jun 03 2026`

**Escribe:**
```javascript
// En saveBias(val):
localStorage.setItem('tt_bias_' + new Date().toDateString(), val);
```

**Lee:**
```javascript
// En loadBias():
var saved = localStorage.getItem('tt_bias_' + new Date().toDateString());
if(saved) el.value = saved;
```

**Crecimiento:** Una clave por día de uso. Después de 1 año: hasta 365 claves separadas (solo días en que el usuario escribió algo).

**No incluida en backup:** Esta es una pérdida real de datos si el usuario importa un backup en un navegador distinto o después de `localStorage.clear()`. Todo el historial de bias del día se pierde.

**`resetAll()` las elimina:** `localStorage.clear()` borra todas las claves del origen, incluyendo las de bias.

---

### `jft_beta`

**Tipo:** `string` literal `'1'`

**Escribe:**
```javascript
// Inline en el botón del beta splash:
localStorage.setItem('jft_beta', '1');
```

**Lee:**
```javascript
// IIFE inmediata en el primer <script>:
if(localStorage.getItem('jft_beta') === '1') { s.style.display = 'none'; }
else { s.style.display = 'flex'; }
```

**Propósito:** Flag permanente que indica que el usuario ya vio y aceptó el splash de beta. Una vez guardado, el splash nunca vuelve a mostrarse.

**Único uso del prefijo `jft_`:** Todas las demás claves usan `tt_`. Esta inconsistencia de prefijo es real y está confirmada en el código.

**`resetAll()` la elimina:** Al llamar `localStorage.clear()`, se borra `jft_beta`, por lo que el splash de beta volvería a mostrarse después de un reset. Comportamiento probablemente intencionado.

---

## Comportamiento de `loadAll()`

```javascript
function loadAll(){
  try{ const c=localStorage.getItem('tt_cfg'); if(c) cfg=JSON.parse(c); }catch(e){ cfg={}; }
  try{ trades=JSON.parse(localStorage.getItem('tt_trades')||'[]'); }catch(e){ trades=[]; }
  try{ phases=JSON.parse(localStorage.getItem('tt_ph')||'{}'); }catch(e){ phases={}; }
  try{ completedPh=JSON.parse(localStorage.getItem('tt_cp')||'[]'); }catch(e){ completedPh=[]; }
  try{ funds=JSON.parse(localStorage.getItem('tt_fd')||'[]'); }catch(e){ funds=[]; }
  try{ checklists=JSON.parse(localStorage.getItem('tt_cl')||'{}'); }catch(e){ checklists={}; }
  if(cfg&&cfg.lang) lang=cfg.lang;
}
```

**Análisis:**
- Cada clave tiene su propio try/catch → un JSON corrupto en una clave no rompe las demás. ✅
- `tt_cfg` falla silenciosamente a `{}` → la app muestra onboarding en lugar de crashear. ✅
- `tt_trades` falla silenciosamente a `[]` → se pierde el historial pero la app no rompe. ✅
- Ninguna clave tiene validación de esquema — si el JSON es válido pero el esquema cambió, los datos se cargan tal como están. ⚠️

## Comportamiento de `saveAll()`

```javascript
function saveAll(){
  localStorage.setItem('tt_cfg', JSON.stringify(cfg));
  localStorage.setItem('tt_trades', JSON.stringify(trades));
  localStorage.setItem('tt_ph', JSON.stringify(phases));
  localStorage.setItem('tt_cp', JSON.stringify(completedPh));
  localStorage.setItem('tt_fd', JSON.stringify(funds));
  localStorage.setItem('tt_cl', JSON.stringify(checklists));
}
```

**Análisis:**
- **Sin try/catch.** Si `localStorage.setItem()` lanza `QuotaExceededError` (localStorage lleno), la excepción se propaga sin manejo. ⚠️ **Crítico**
- Se sobreescriben las 6 claves en cada llamada, incluso si solo cambió una.
- `saveAll()` se llama frecuentemente: en cada toggle de checklist, en cada trade guardado, en cada movimiento de fondos, en cada tarea de fase marcada.

---

## Comportamiento de `importD()`

```javascript
function importD(inp){
  const f=inp.files[0]; if(!f) return;
  const r=new FileReader();
  r.onload=e=>{
    try{
      const d=JSON.parse(e.target.result);
      if(d.cfg)        localStorage.setItem('tt_cfg',JSON.stringify(d.cfg));
      if(d.trades)     localStorage.setItem('tt_trades',JSON.stringify(d.trades));
      if(d.phases)     localStorage.setItem('tt_ph',JSON.stringify(d.phases));
      if(d.completedPh)localStorage.setItem('tt_cp',JSON.stringify(d.completedPh));
      if(d.funds)      localStorage.setItem('tt_fd',JSON.stringify(d.funds));
      showN(lang==='es'?'Importado':'Imported');
      setTimeout(()=>location.reload(),800);
    }catch(e){ showN(lang==='es'?'Error al importar':'Import error',true); }
  };
  r.readAsText(f);
}
```

**Análisis:**
- Solo importa las 6 claves del sistema principal. `tt_theme`, `tt_bias_*` y `jft_beta` no se tocan. ✅
- No valida versión de esquema — importa lo que venga en el JSON. ⚠️
- No importa `checklists` (`tt_cl`). **Aunque `d.checklists` esté en el JSON, no hay una línea para `tt_cl`**. ⚠️ **Bug confirmado** — el historial del checklist no se restaura al importar un backup.
- El `catch` solo muestra una notificación de error sin información de la causa. ⚠️

---

## Estimación de Uso de localStorage

### Sin imágenes (uso típico conservador)

| Clave | Estimado por entrada | Entradas típicas | Total estimado |
|-------|---------------------|-----------------|----------------|
| `tt_cfg` | — | 1 objeto | ~3-5 KB |
| `tt_trades` | ~300 bytes/trade | 200 trades | ~60 KB |
| `tt_ph` | — | 1 objeto | ~1 KB |
| `tt_cp` | — | 1 array | < 1 KB |
| `tt_fd` | ~200 bytes/movimiento | 100 movimientos | ~20 KB |
| `tt_cl` | ~200 bytes/día | 365 días | ~73 KB |
| `tt_theme` | — | 1 string | < 0.1 KB |
| `tt_bias_*` | ~100 bytes/día | 200 días | ~20 KB |
| `jft_beta` | — | 1 string | < 0.1 KB |
| **TOTAL** | | | **~177 KB** |

### Con imágenes (riesgo real)

| Escenario | Tamaño adicional | Total estimado |
|-----------|-----------------|----------------|
| 10 imágenes de 100 KB c/u | +1 MB | ~1.2 MB |
| 30 imágenes de 200 KB c/u | +6 MB | ~6.2 MB |
| **Límite típico del navegador** | — | **~5-10 MB** |

Con 25-30 capturas de pantalla de resolución normal, el usuario puede alcanzar el límite de localStorage. El comportamiento actual ante este escenario es: **pérdida silenciosa del trade** que se intentaba guardar.

---

## Tabla de Inclusión en Backup

| Clave | En backup JSON | Si no está: qué se pierde |
|-------|---------------|--------------------------|
| `tt_cfg` | ✅ Sí | Todo el perfil, cuentas, configuración |
| `tt_trades` | ✅ Sí | Historial de trades |
| `tt_ph` | ✅ Sí | Progreso de tareas en fases |
| `tt_cp` | ✅ Sí | Fases marcadas como completadas |
| `tt_fd` | ✅ Sí | Historial de fondos |
| `tt_cl` | ✅ Exportado · ❌ No importado | Historial de checkboxes del checklist (**bug en importD**) |
| `tt_theme` | ❌ No | Preferencia de tema (se mantiene en el navegador destino) |
| `tt_bias_*` | ❌ No | Todo el historial de bias del día |
| `jft_beta` | ❌ No | Flag de beta (se volvería a mostrar el splash) |

---

## Bugs Confirmados en el Sistema de Persistencia

### Bug 1 — `importD()` no restaura el checklist
**Severidad:** Media
La función `importD()` no tiene la línea para `tt_cl`:
```javascript
// Esta línea NO EXISTE en importD():
if(d.checklists) localStorage.setItem('tt_cl', JSON.stringify(d.checklists));
```
El objeto `checklists` sí se incluye en el JSON de `exportD()`, pero al importar no se restaura. El historial del checklist se pierde en migraciones.

### Bug 2 — `saveAll()` sin manejo de `QuotaExceededError`
**Severidad:** Alta
Si localStorage está lleno, `saveAll()` falla silenciosamente desde la perspectiva del usuario. El toast de "Trade guardado ✓" se muestra, pero el dato no se persistió. El usuario cree que su trade está guardado cuando no lo está.

### Bug 3 — Bias del día fuera del sistema de backup
**Severidad:** Baja-Media
El historial de bias del día (notas de análisis pre-sesión) no se incluye en el backup. Es un dato que el trader puede considerar valioso para revisiones semanales. No tiene forma de migrar este dato a otro navegador o dispositivo.

---

## Comportamiento en Casos Edge de localStorage

| Escenario | Resultado actual |
|-----------|-----------------|
| Modo incógnito | localStorage funciona pero se borra al cerrar la ventana. El usuario pierde todos los datos. No hay aviso. |
| Safari iOS con archivo local | localStorage bloqueado por política de Safari. La app no funciona. |
| localStorage lleno | `saveAll()` lanza excepción no capturada. Trade/dato no se guarda. Usuario no recibe aviso de error. |
| JSON corrupto en una clave | `loadAll()` lo captura y resetea esa variable a su valor default. Las otras claves se cargan normales. |
| `localStorage.clear()` externo | Equivale a `resetAll()`. La app muestra onboarding en el próximo load. |
| Dos pestañas abiertas | Pueden sobreescribirse mutuamente. No hay sincronización entre pestañas. |

---

*Documento generado por auditoría directa del código fuente — Junio 2026*
