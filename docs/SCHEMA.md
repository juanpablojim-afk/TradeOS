# SCHEMA.md
## TradeOS — Esquema Real de Datos

**Versión:** 1.0 · **Extraído de:** tradeOS_base.html (3206 líneas)
**Fecha de auditoría:** Junio 2026
**Método:** Lectura directa del código fuente — funciones `finishOnboarding()`, `saveTrade()`, `saveFund()`, `saveAll()`, `loadAll()`

> Este documento describe la estructura **real** de los datos tal como existen en el código.
> No es documentación de intención — es lo que realmente se guarda.

---

## Variables de Estado Global (runtime)

```javascript
let cfg = {}           // Objeto de configuración completo del usuario
let trades = []        // Array de objetos trade
let phases = {}        // Estado de tareas por fase: { [phaseId]: { [taskIndex]: boolean } }
let completedPh = []   // Array de IDs de fases completadas: string[]
let funds = []         // Array de movimientos de fondos
let checklists = {}    // Estado del checklist por fecha: { [dateString]: { [itemId]: boolean } }

// Variables auxiliares de runtime (NO se persisten en localStorage)
let selResult = ''          // Resultado seleccionado en formulario ('win'|'loss'|'be')
let imgB64 = ''             // Imagen en base64 del trade activo en formulario
let lang = 'es'             // Idioma activo ('es'|'en')
let cSel = {}               // Selecciones single de chips en onboarding
let mSel = {}               // Selecciones múltiples de chips en onboarding
let accInstMap = {}         // Instrumento por cuenta en onboarding: { [n]: 'forex'|'futures' }
let accCnt = 0              // Contador de cuentas en onboarding
let goalCnt = 0             // Contador de metas en onboarding
let accTypeMap = {}         // Tipo por cuenta en onboarding: { [n]: 'own'|'funded' }
let accDDMap = {}           // Drawdown % por cuenta en onboarding: { [n]: number }
let customRules = []        // Reglas personalizadas escritas en onboarding
let customCLItems = []      // Items de checklist personalizados en onboarding
let customPairs = []        // Pares personalizados en onboarding
let customPlats = []        // Plataformas personalizadas en onboarding
let selectedTags = []       // Tags seleccionados en formulario de trade activo
let selectedRules = []      // Reglas rotas seleccionadas en formulario activo
let notebookEnabled = false // Estado del panel de libreta en formulario
let sessionStartTime = null // Timestamp de inicio de sesión de checklist
let sessionInterval = null  // Interval ID del temporizador de sesión
let calYear = new Date().getFullYear()  // Año visible en el calendario
let calMonth = new Date().getMonth()    // Mes visible en el calendario
let tradeViewMode = 'list'  // Modo de vista de trades ('list'|'calendar')
```

---

## 1. Objeto `cfg` — Configuración del Usuario

Generado en `finishOnboarding()`. Persistido en `localStorage` bajo la clave `tt_cfg`.

```javascript
cfg = {
  // Perfil (Paso 1)
  name: string,               // Nombre o alias del trader. Default: 'Trader'
  age: string,                // Edad. Puede ser string vacío si no se completó
  country: string,            // País. Puede ser string vacío
  experience: string,         // Nivel: 'beginner'|'inter'|'advanced'|'pro'. Default: 'inter'

  // Estilo (Paso 3)
  sessions: string[],         // Sesiones: ['London','New York','Asia','Overlap']
  strategy: string,           // Estrategia: 'pa'|'ict'|'smc'|'sc'|'fund'|'other'. Default: 'pa'
  pairs: string[],            // Instrumentos: ['XAU/USD','EUR/USD',...]. Default si vacío: ['XAU/USD','EUR/USD']
  riskProfile: string,        // 'conservative'|'moderate'|'aggressive'. Default: 'moderate'
  riskPct: number,            // % de riesgo por trade. Default: 1
  rrMin: number,              // R:R mínimo aceptado. Default: 1.8
  maxTrades: string,          // Máx trades por sesión: '1'|'2'|'3'|'4'|'unlimited'. Default: '2'

  // Reglas (Paso 4)
  rules: string[],            // Reglas personalizadas del trader (sin prefijo)

  // Checklist (Paso 6)
  clItems: string[],          // IDs de items de checklist activos. Si vacío, usa defaultCL completo
  customCLItems: string[],    // Labels de items personalizados de checklist

  // Horario (Paso 5)
  hasJob: boolean,            // ¿Tiene trabajo además del trading?
  jobHours: string,           // Horario de trabajo (texto libre)
  daysOff: string,            // Días libres (texto libre)
  sleepTime: string,          // Hora de ir a dormir
  wakeTime: string,           // Hora de despertar para operar
  platform: string,           // Plataforma: 'MT5'|'MT4'|'ctrader'|'tradingview'|custom. Default: 'MT5'

  // Metas (Paso 7)
  monthlyExpenses: number,    // Gastos fijos mensuales en $. Default: 0
  goals: Goal[],              // Array de metas financieras

  // Psicología (Paso 9)
  mainError: string,          // Error más frecuente: 'overtrading'|'revenge'|'fomo'|'impatience'|'overconfidence'|'no_plan'. Default: 'overtrading'
  badConditions: string[],    // Condiciones en que toma peores decisiones
  motivation: string,         // Por qué quiere ser trader (texto libre)

  // Fases (Paso 8)
  selectedPhases: string[],   // IDs de fases seleccionadas en onboarding
  customPhases: CustomPhase[], // Fases personalizadas creadas por el usuario (se agrega post-onboarding)

  // Cuentas (Paso 2)
  accounts: Account[],        // Array de cuentas

  // Sistema
  lang: string,               // 'es'|'en'
  customRules: string[],      // Copia de customRules del onboarding
  customPairs: string[],      // Copia de customPairs del onboarding
}
```

### 1.1 Sub-objeto `Account`

Generado en `finishOnboarding()` desde los bloques `.acc-blk`.

```javascript
Account = {
  id: number,           // Índice posicional (1, 2, 3...) asignado en onboarding. NO es Date.now()
  type: string,         // 'own' | 'funded'
  firm: string,         // Nombre del broker o prop firm. Default: '—'
  size: number,         // Tamaño original de la cuenta en $. Default: 10000
  maxDD: number,        // Drawdown máximo en %. Default: 10
  balance: number,      // Igual a size en el momento de creación (no se actualiza dinámicamente)
  drawdown: number,     // 0 en el momento de creación (no se actualiza dinámicamente)
  pnl: number,          // 0 en el momento de creación (no se actualiza dinámicamente)
  phase: string,        // Estado: 'active'|'challenge1'|'challenge2'|'funded'|'paused'|'evaluation'. Default: 'active'
  instrument: string,   // 'forex' | 'futures'. Default: 'forex'
}
```

**Nota importante:** Los campos `balance`, `drawdown` y `pnl` del objeto `Account` se inicializan en el onboarding pero **no se actualizan** cuando se registran trades. El P&L real se calcula dinámicamente en runtime filtrando `trades[]` por `account.id`. Estos tres campos son vestigios de una arquitectura anterior y contienen valores desactualizados.

### 1.2 Sub-objeto `Goal`

```javascript
Goal = {
  name: string,     // Nombre de la meta (texto libre)
  target: number,   // Monto objetivo en $
  saved: number,    // Monto ahorrado. Inicializado en 0. Se puede actualizar vía fondos tipo 'goal'
}
```

### 1.3 Sub-objeto `CustomPhase`

Generado en `saveCustomPhase()`. Se guarda en `cfg.customPhases[]`.

```javascript
CustomPhase = {
  id: string,       // 'custom_' + Date.now()
  icon: string,     // Emoji. Default: '📌'
  name: string,     // Nombre en mayúsculas
  time: string,     // Subtítulo / timeframe
  desc: string,     // Descripción larga
  tasks: string[],  // Array de strings con las tareas
}
```

---

## 2. Objeto `trade` — Trade Individual

Generado en `saveTrade()`. Persistido en `trades[]` → `localStorage['tt_trades']`.

```javascript
trade = {
  id: number,           // Date.now() en el momento de guardar. Único por trade madre.
                        // En réplicas: Date.now() + Math.random() (puede colisionar si se crean muy rápido)
  date: string,         // ISO 8601: new Date().toISOString(). Ej: "2026-06-02T14:30:00.000Z"
  pair: string,         // Instrumento: 'XAU/USD', 'EUR/USD', 'NQ', etc.
  session: string,      // 'London' | 'New York' | 'Asia' | 'Overlap'
  account: string,      // ID de la cuenta (referencia a Account.id). Tipo string en el trade.
  result: string,       // 'win' | 'loss' | 'be'
  pnl: number,          // P&L en $. Positivo o negativo. Default parseFloat: 0
  rr: number,           // R:R real obtenido. Default parseFloat: 0
  risk: number,         // % de riesgo usado. De selector (0.1 a 5)
  mood: string,         // Estado emocional: 'calm'|'confident'|'anxious'|'pressure'|'tired'|'revenge'
  ruleBroken: string,   // Reglas rotas separadas por ', '. Si ninguna: 'none'
  note: string,         // Descripción del setup (textarea libre)
  img: string,          // Base64 completo de la imagen. String vacío si no hay imagen.
  tags: string[],       // Tags de setup: ['Barrida de liquidez','FVG','Order Block',...]
  isMother: boolean,    // true si es trade original. Siempre true en saveTrade()
  isChild: boolean,     // false si es trade original. Siempre false en saveTrade()
  notebook: string,     // Contenido de la libreta. String vacío si notebook no estaba activado.
                        // En hijos: siempre '' (nunca se copia)

  // Solo en réplicas (isChild: true) — generado en applyReplicator()
  motherId: number,     // id del trade madre. Referencia a trades[0].id en el momento de replicar.
                        //   ⚠️ BUG POTENCIAL: usa trades[0].id antes de que saveTrade() haya guardado el madre.
}
```

**Tags disponibles en el formulario (hardcodeados en el HTML):**
`'Barrida de liquidez'`, `'Ruptura'`, `'Pullback'`, `'S&R'`, `'FVG'`, `'Order Block'`, `'News play'`

**Moods disponibles (hardcodeados):**
`'calm'`, `'confident'`, `'anxious'`, `'pressure'`, `'tired'`, `'revenge'`

---

## 3. Objeto `fund` — Movimiento de Fondos

Generado en `saveFund()`. Persistido en `funds[]` → `localStorage['tt_fd']`.

```javascript
fund = {
  id: number,         // Date.now() en el momento de guardar
  date: string,       // ISO 8601: new Date().toISOString()
  type: string,       // Tipo de movimiento (ver valores abajo)
  amount: number,     // Monto en $. Siempre positivo en el input. El render determina si es gasto o ingreso.
  accountId: string,  // ID de la cuenta asociada. String vacío si no aplica.
  note: string,       // Nota opcional (texto libre)
  source: string,     // Nombre del firm de la cuenta asociada, o '—' si no hay cuenta
}
```

**Tipos de movimiento (`fund.type`):**
| Valor | Etiqueta ES | Etiqueta EN | ¿Es gasto? |
|-------|-------------|-------------|-----------|
| `own_payout` | Payout cuenta propia | Own account payout | No |
| `funded_payout` | Payout fondeada | Funded account payout | No |
| `salary` | Salario / Trabajo | Job salary | No |
| `goal` | Meta alcanzada | Goal reached | No |
| `challenge` | Challenge comprado | Challenge purchase | **Sí** |
| `expense` | Gasto | Operational expense | **Sí** |
| `other` | Otro | Other | No |

**Lógica de totales en `renderFunds()`:**
- Ingresos totales = `own_payout + funded_payout + salary + goal + other`
- Gastos totales = `expense + challenge`
- Neto = Ingresos - Gastos

---

## 4. Objeto `phases` — Estado de Tareas por Fase

Persistido en `localStorage['tt_ph']`. Actualizado en `togglePT()`.

```javascript
phases = {
  [phaseId: string]: {
    [taskIndex: number]: boolean  // true si la tarea está marcada como completada
  }
}
// Ejemplo real:
phases = {
  "discipline": { 0: true, 1: true, 2: false, 3: false, 4: false, 5: false },
  "challenge": { 0: true },
  "ph_custom_custom_1234567890": { 0: false }
}
```

**IDs de fases predeterminadas:**
`'discipline'`, `'challenge'`, `'funded'`, `'scaling'`, `'own_capital'`, `'goals'`

**ID de fases personalizadas:**
Formato: `'ph_custom_' + customPhase.id` donde `customPhase.id = 'custom_' + Date.now()`
Resultado: `'ph_custom_custom_1234567890'`

---

## 5. Array `completedPh` — Fases Completadas

Persistido en `localStorage['tt_cp']`. Actualizado en `completePH()`.

```javascript
completedPh = string[]
// Array de phaseIds que el usuario marcó como completadas
// Ejemplo: ['discipline', 'challenge', 'ph_custom_custom_1234567890']
```

---

## 6. Objeto `checklists` — Estado del Checklist por Fecha

Persistido en `localStorage['tt_cl']`. Actualizado en `toggleCL()`.

```javascript
checklists = {
  [dateString: string]: {          // Clave: new Date().toDateString() — ej: "Mon Jun 02 2026"
    [itemId: string]: boolean      // true si el ítem está checkeado
  }
}
// Ejemplo:
checklists = {
  "Mon Jun 02 2026": {
    "sleep_c": true,
    "no_rev_c": true,
    "bias_c": false,
    "cx_Mi regla personalizada": true
  }
}
```

**IDs de ítems predeterminados (los 13 base, inmutables):**
`'sleep_c'`, `'no_rev_c'`, `'bias_c'`, `'risk_c'`, `'mental_c'`, `'pending_c'`, `'dd_c'`, `'news_c'`, `'setup_c'`, `'rr_c'`, `'lot_c'`, `'close_c'`, `'journal_c'`

**IDs de ítems personalizados:**
Formato: `'cx_' + label` donde label es el texto que escribió el usuario.

---

## 7. Objeto exportado en backup JSON (`exportD()`)

```javascript
backup = {
  cfg: cfg,
  trades: trades,
  phases: phases,
  completedPh: completedPh,
  funds: funds,
  checklists: checklists,
}
```

**Lo que NO incluye el backup:**
- El bias del día (`tt_bias_*` — claves individuales por fecha)
- El tema de color (`tt_theme`)
- El flag de beta splash (`jft_beta`)

---

## Inconsistencias y Hallazgos Reales

### 1. ID de cuenta: número vs string
`Account.id` se asigna como número (`id: n` donde `n` es el índice entero 1-6).
Al guardar un trade, `account` se guarda como string (`document.getElementById('t-accs').value`).
En los filtros, se compara con `String(t.account) === String(a.id)`.
La conversión es consistente en el código, pero el tipo es ambiguo — en el objeto trade es string, en Account es number.

### 2. Campo `account` en trade: ID vs nombre
`trade.account` guarda el `id` de la cuenta (ej: `"1"`), no el nombre.
El CSV exportado muestra el `id` en la columna "Cuenta", no el nombre del firm.
Esto hace el CSV menos legible sin cruzar con `cfg.accounts`.

### 3. Los campos `balance`, `drawdown`, `pnl` de `Account` son datos muertos
Se inicializan en el onboarding con valores fijos y nunca se actualizan.
El P&L real de cada cuenta se calcula filtrando `trades[]` en runtime.
Estos campos existen en el objeto guardado pero no tienen efecto funcional.

### 4. `motherId` en réplicas puede ser incorrecto
En `applyReplicator()`, el `motherId` se asigna como `trades[0]?trades[0].id:null` **antes** de que `saveTrade()` guarde el trade madre. En ese momento, `trades[0]` es el último trade guardado antes de esta operación, no el trade madre actual. El trade madre se guarda dentro de `saveTrade()` al final de `applyReplicator()`. Esto puede hacer que `motherId` apunte al trade anterior, no al madre real.

### 5. IDs de fases personalizadas: doble prefijo `custom_`
En `saveCustomPhase()`:
- `customPhase.id = 'custom_' + Date.now()`
- Se agrega a `cfg.selectedPhases` como `'custom_' + id` → resultado: `'custom_custom_1234567890'`
- Se guarda en `cfg.customPhases` con `id = 'custom_1234567890'`
- En `renderPhases()`, el ID para phases es `'ph_custom_' + 'custom_1234567890'` → `'ph_custom_custom_1234567890'`
El doble `custom_` es real en el código actual.

### 6. `selectedTags` tiene doble declaración
Existe `window._selectedTags` (función `toggleTag` primera versión, línea ~1609) y `selectedTags` global (segunda versión, línea ~2813). Ambas coexisten. El `saveTrade()` usa `selectedTags` (la versión global), no `window._selectedTags`. La primera versión es código muerto.

### 7. `resetAll()`, `openM()`, `closeM()`, `showN()`, `toggleTheme()`, `loadTheme()` están duplicadas
Estas funciones aparecen dos veces en el código (líneas ~3134-3153 y ~3184-3201). La segunda declaración sobrescribe la primera en runtime — JavaScript no produce error, simplemente usa la última declaración. Funcionalmente no causa bugs pero es código redundante.

### 8. `renderWeeklySummary()` solo muestra en sábado y domingo
La función tiene un guard `if(now.getDay()!==0&&now.getDay()!==6){el.style.display='none';return;}`. De lunes a viernes el panel "Resumen de la semana" no aparece, incluso si hay trades de esa semana.

### 9. El bias del día no se incluye en el backup
Las claves `tt_bias_[fecha]` se guardan directamente en localStorage pero no se incluyen en el JSON de backup de `exportD()`. Al importar un backup, el historial de bias del día se pierde.

### 10. Prefijo de localStorage: inconsistencia real confirmada
- `cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists`, `theme` → prefijo `tt_`
- `beta_splash` → prefijo `jft_`
- `bias del día` → prefijo `tt_bias_`
La documentación decía prefijo `jft_` para todo, pero la realidad es `tt_` para casi todo.

---

*Documento generado por auditoría directa del código fuente — Junio 2026*
