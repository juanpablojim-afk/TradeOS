# FUNCTIONS.md
## TradeOS — Inventario Real de Funciones

**Versión:** 1.0 · **Extraído de:** tradeOS_base.html (3206 líneas)
**Fecha de auditoría:** Junio 2026
**Total de funciones identificadas en el código:** 73 funciones nombradas

> Nota: La documentación menciona "130 funciones críticas presentes" pero este número no corresponde a funciones JavaScript nombradas. Puede incluir handlers inline de HTML (`onclick=...`), métodos internos o sub-funciones. Este inventario lista solo las funciones JavaScript nombradas con `function` o `var fn =` en el scope global.

---

## Organización por Módulo

---

### SISTEMA — Inicialización y Persistencia

| Función | Descripción | Escribe localStorage |
|---------|-------------|---------------------|
| `loadAll()` | Carga `cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists` desde localStorage. Cada clave en try/catch independiente. Establece `lang` desde `cfg.lang`. | No |
| `saveAll()` | Persiste `cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists` en localStorage. Llamada después de cualquier mutación de estado. | `tt_cfg`, `tt_trades`, `tt_ph`, `tt_cp`, `tt_fd`, `tt_cl` |
| `initApp()` | Inicializa toda la app post-onboarding o post-load. Llama a todas las funciones de build y render en try/catch. Establece idioma, reloj, sidebar. | No |
| `showLoading()` | Muestra pantalla de carga animada con mensajes secuenciales. Llama `initApp()` al finalizar. | No |

---

### ONBOARDING

| Función | Descripción |
|---------|-------------|
| `addAcc()` | Agrega un bloque de cuenta al formulario de onboarding. Máximo 6 cuentas (`accCnt >= 6`). Genera HTML dinámico con los chips de tipo, instrumento y drawdown. |
| `setAccType(n, type)` | Establece el tipo ('own'/'funded') de la cuenta n en `accTypeMap`. Actualiza el placeholder del input de broker. |
| `setAccInst(n, val, el)` | Establece el instrumento ('forex'/'futures') de la cuenta n en `accInstMap`. |
| `setAccDD(n, val, el)` | Establece el drawdown % de la cuenta n en `accDDMap`. Muestra/oculta input personalizado si val es 'other'. |
| `addGoal()` | Agrega un bloque de meta financiera al formulario de onboarding. |
| `addPair()` | Agrega par personalizado al chip de instrumentos en el paso 3. |
| `finishOnboarding()` | Construye el objeto `cfg` completo desde todos los inputs del onboarding. Llama `saveAll()` y `showLoading()`. Punto de entrada de todos los datos del usuario. |

---

### NAVEGACIÓN

| Función | Descripción |
|---------|-------------|
| `sv(id, btn)` | **Función central de navegación.** Quita clase `on` de todas las vistas y botones del sidebar. Agrega `on` a `view-[id]` y al botón activo. Llama la función de render correspondiente a la vista. Único punto autorizado para cambiar de vista. |

---

### SIDEBAR Y UI GLOBAL

| Función | Descripción |
|---------|-------------|
| `updateSidebar()` | Actualiza avatar (primera letra del nombre), nombre y label "TRADER ACTIVO" en el sidebar. |
| `updateClock()` | Actualiza el reloj y fecha en el dashboard y en el checklist. Usa locale `es-CR` o `en-US`. |
| `showN(msg, err)` | Muestra notificación toast. `err=true` para estilo de error. Duración: 3 segundos. |
| `toggleTheme()` | Alterna entre modo oscuro (sin atributo) y claro (`data-theme="light"`). Persiste en `tt_theme`. ⚠️ Declarada dos veces en el código. |
| `loadTheme()` | Lee `tt_theme` de localStorage y aplica el atributo `data-theme` si corresponde. ⚠️ Declarada dos veces en el código. |
| `openSupport()` | Abre Gmail con destinatario `tradeossoporte@gmail.com`. Si falla, usa `mailto:`. |
| `T(key)` | Función de traducción. Retorna `LG[lang][key]` o `key` si no existe. |
| `setLang(l)` | Cambia `lang` global y llama `applyLang()`. |
| `applyLang()` | Itera sobre elementos con atributo `data-t` y aplica la traducción correspondiente. |

---

### DASHBOARD

| Función | Descripción |
|---------|-------------|
| `renderDash()` | Orquesta el render completo del dashboard. Calcula métricas globales (total trades, WR, P&L, racha, R:R promedio, reglas rotas esta semana), Score de Consistencia, alerta de drawdown, y resumen semanal. Llama `renderGoals()`, `renderTrades('dash-recent', trades)`, `setTimeout(renderEquityCurve, 50)`. |
| `renderDashAccPanels()` | Renderiza los paneles mini de cada cuenta en el dashboard. Calcula P&L y drawdown por cuenta en tiempo real filtrando `trades[]`. |
| `renderGoals()` | Renderiza las barras de progreso de metas financieras. Cruza `cfg.goals` con `funds` de tipo `'goal'`. |
| `renderWeeklySummary()` | **Solo visible sábados y domingos.** Muestra comparativa esta semana vs semana anterior. |
| `getWeeklySummary()` | Calcula estadísticas de la semana actual y la anterior. Retorna `{ this: Stats, last: Stats }`. |
| `detectMain()` | Detecta patrones críticos en los últimos 5 trades. Retorna string con el patrón o `null`. Detecta: mood revenge/pressure en 2+ trades, reglas rotas en 2+ trades. |
| `renderEquityCurve()` | Renderiza la curva de equity como SVG inline en `#equity-curve`. Usa curvas Bézier. Requiere mínimo 2 trades. |
| `renderEquity()` | Versión alternativa de la curva de equity en `<canvas>`. Usa Canvas 2D API. Parece coexistir con `renderEquityCurve()`. |

---

### CHECKLIST

| Función | Descripción |
|---------|-------------|
| `buildChecklist()` | Construye el HTML del checklist diario. Lee `cfg.clItems` y `cfg.customCLItems`. Lee estado de hoy desde `checklists[today]`. Agrupa en 4 secciones predefinidas. |
| `toggleCL(id, el)` | Marca/desmarca un ítem del checklist. Persiste en `checklists[fecha]`. Llama `saveAll()`. |
| `resetCL()` | Borra el estado del checklist del día actual. |
| `saveBias(val)` | Guarda el bias del día en `localStorage['tt_bias_' + new Date().toDateString()]`. Clave fuera del sistema `saveAll()`. |
| `loadBias()` | Carga el bias del día desde localStorage y lo pone en `#bias-note`. |
| `startSession()` | Inicia el temporizador de sesión. Muestra `#session-bar`. Guarda `sessionStartTime`. |
| `endSession()` | Detiene el temporizador. Calcula duración en minutos. Pregunta si hay trades para registrar. Si confirma, llama `sv('log', null)`. |

---

### REGISTRO DE TRADE

| Función | Descripción |
|---------|-------------|
| `saveTrade()` | Construye el objeto `trade` desde el formulario. Lo agrega al inicio de `trades[]` (`unshift`). Llama `saveAll()`. Limpia el formulario. Redirige al dashboard después de 600ms. |
| `selR(r, btn)` | Selecciona el resultado del trade ('win'/'loss'/'be'). Actualiza `selResult` y clases CSS de los botones. |
| `prevImg(inp)` | Lee la imagen cargada con FileReader, la convierte a base64 y la guarda en `imgB64`. |
| `clearImg()` | Limpia la imagen del formulario y resetea `imgB64`. |
| `toggleTag(el)` | Agrega/quita un tag de `selectedTags[]`. ⚠️ Existe también versión `toggleTag(tag, btn)` con firma diferente — la del scope global usa el elemento directamente. |
| `toggleRuleBroken(el)` | Maneja la selección múltiple de reglas rotas. Si selecciona 'none', deselecciona todas. Si selecciona una regla, deselecciona 'none'. Mantiene `selectedRules[]`. |
| `buildRuleChips()` | Construye los chips de reglas rotas en el formulario desde `cfg.rules`. Si no hay reglas configuradas, usa las predeterminadas en ES/EN. |
| `toggleNotebook()` | Muestra/oculta el panel de libreta en el formulario. Actualiza `notebookEnabled`. |
| `buildPairSel()` | Puebla el selector de par con `cfg.pairs`. |
| `buildSessSel()` | Puebla el selector de sesión con `cfg.sessions`. |
| `buildAccSel()` | Puebla el selector de cuenta en el formulario de trade. |
| `buildRiskSel()` | Puebla el selector de riesgo % con valores fijos: `[0.1, 0.25, 0.35, 0.5, 0.75, 1, 1.5, 2, 3, 5]`. Pre-selecciona el valor de `cfg.riskPct`. |
| `buildRuleSelect()` | Versión antigua con `<select>` simple. Aparentemente coexiste con `buildRuleChips()` pero usa un elemento diferente (`#t-rule` vs `#t-rule-chips`). Puede ser código no utilizado en la UI actual. |

---

### CALCULADORA DE LOTAJE

| Función | Descripción |
|---------|-------------|
| `calcLot()` | Calcula el lotaje recomendado. Lee par, cuenta, stop y riesgo%. Calcula `realBalance = acc.size + (pnl < 0 ? pnl : 0)`. Para futuros: usa tabla de tick values fijos. Para forex exact (4 pares): $10/pip/lote. Para pares excluidos o no soportados: muestra mensaje explicativo. |
| `buildLotCalcSel()` | Puebla el selector de cuenta en la calculadora integrada en el formulario de trade. |

---

### MIS TRADES

| Función | Descripción |
|---------|-------------|
| `renderTrades(cid, list)` | Renderiza lista de trades en el contenedor `cid`. Si `cid === 'dash-recent'`, muestra solo los primeros 5. Incluye badges MADRE/HIJA, libreta, imagen, reglas rotas. |
| `delTrade(id)` | Elimina un trade por id con confirmación. Llama `saveAll()`, `renderTrades()` y `renderDash()`. **No elimina las réplicas hijas** al eliminar una madre — quedan huérfanas. |
| `buildTradeFilters()` | Puebla los selectores de filtro (cuenta, par) con los valores existentes en `trades[]`. |
| `applyTradeFilters()` | Filtra `trades[]` según los valores de los selectores y llama `renderTrades()`. |
| `clearTradeFilters()` | Resetea los tres filtros y muestra todos los trades. |
| `setTradeView(mode)` | Alterna entre vista lista y calendario. Actualiza `tradeViewMode`. |
| `exportTradesCSV()` | Exporta `trades[]` a CSV. Columnas: Fecha, Par, Sesión, Cuenta (ID, no nombre), Resultado, PnL, RR, Riesgo%, Mood, Regla rota, Nota. No incluye imagen, libreta ni tags. |
| `clearAllTrades()` | Borra todos los trades con confirmación doble. Llama `saveAll()`. |

---

### CALENDARIO

| Función | Descripción |
|---------|-------------|
| `renderCalendar()` | Renderiza el calendario mensual completo en HTML. Agrupa trades por día. Muestra resumen mensual, navegación, cabeceras y grilla de días coloreados. |
| `calDayClick(day)` | Abre el modal de zona con el detalle del día seleccionado. Muestra resumen (trades, WR, P&L) y lista de trades colapsable. |
| `calDayData(key)` | Calcula estadísticas de un día específico desde `trades[]`. Retorna `{ pnl, trades, wins, losses, be, result, wr }`. |
| `calPrev()` | Navega al mes anterior. Maneja rollover de año. |
| `calNext()` | Navega al mes siguiente. Maneja rollover de año. |

---

### FASES

| Función | Descripción |
|---------|-------------|
| `renderPhases()` | Orquesta el render de la sección de fases. Llama `buildPhases()` y renderiza cada tarjeta. |
| `buildPhases()` | Construye y retorna el array de fases a mostrar, combinando las predeterminadas activas + personalizadas. |
| `evalCond(ph)` | Evalúa las condiciones en tiempo real de la fase Disciplina. Cuenta sesiones limpias consecutivas (sin reglas rotas, 1 trade por sesión) y WR. Retorna `{ streak, required, wr, allMet }`. |
| `togglePT(id, i, el)` | Marca/desmarca una tarea dentro de una fase. Persiste en `phases[id][i]`. |
| `completePH(id)` | Marca una fase como completada. Agrega su ID a `completedPh[]`. Llama `saveAll()`. |
| `resetPhase(id)` | Resetea el progreso de una fase (borra de `phases` y `completedPh`). Con confirmación. |
| `saveCustomPhase()` | Crea una nueva fase personalizada. La agrega a `cfg.customPhases[]` y a `cfg.selectedPhases[]`. |
| `openCustomPhaseModal()` | Muestra el modal de creación de fase personalizada. |
| `closeCustomPhaseModal()` | Cierra el modal de creación de fase personalizada. |
| `removeCustomPhase(id)` | Elimina una fase personalizada de `cfg.customPhases` y `cfg.selectedPhases`. Solo si `id.startsWith('ph_custom')`. |
| `getISOWeek(d)` | Calcula el número de semana ISO de una fecha. Usado en `renderWeeklyEvolution()`. |

---

### GESTIÓN DE FONDOS

| Función | Descripción |
|---------|-------------|
| `saveFund()` | Construye el objeto `fund` y lo agrega al inicio de `funds[]`. Llama `saveAll()`. Llama `renderFunds()` y `renderGoals()`. |
| `addFund()` | Alias de `saveFund()`. |
| `delFund(id)` | Elimina un movimiento de fondos por id. Llama `saveAll()` y `renderFunds()`. |
| `renderFunds()` | Renderiza el resumen de fondos (4 tarjetas de totales) y el historial completo. Calcula totales por tipo. |
| `buildFiAccSel()` | Puebla el selector de cuenta en el formulario de fondos. |
| `clearAllFunds()` | Borra todo el historial de fondos con confirmación. |

---

### CUENTAS

| Función | Descripción |
|---------|-------------|
| `renderAccs()` | Renderiza las tarjetas de cada cuenta con P&L, WR, trades, R:R, reglas rotas y barra de drawdown. Todo calculado en runtime desde `trades[]`. |
| `editAccPhase(id)` | Cicla por los estados de fase de una cuenta (`challenge1 → challenge2 → funded → active → paused → evaluation → ...`). Persiste en `cfg.accounts`. |
| `editAccDD(id)` | Edita el drawdown máximo de una cuenta con `prompt()`. Persiste en `cfg.accounts`. |

---

### PATRONES

| Función | Descripción |
|---------|-------------|
| `renderPatterns()` | Detecta y renderiza patrones de comportamiento. Requiere 5+ trades. Detecta: revenge trading vs WR, reglas rotas vs pérdidas, mejor/peor par, mejor sesión, efecto del sueño, pérdida→regla rota, R:R real vs mínimo, peor día de semana, rachas de pérdidas (≥3), break evens frecuentes (≥3). |
| `renderWeeklyEvolution(filteredTrades)` | Renderiza la evolución semanal de WR como tabla. Agrupa por semana ISO. Muestra hasta las últimas 12 semanas. Mínimo 2 semanas para mostrar. |

---

### REPLICADOR

| Función | Descripción |
|---------|-------------|
| `openReplicatorModal()` | Abre el modal del replicador. Llama `buildReplicatorPanel()`. |
| `closeRepModal()` | Cierra el modal del replicador. |
| `toggleReplicator()` | Versión toggle del replicador (panel inline, aparentemente no usado en la UI actual — el modal es el flujo principal). |
| `buildReplicatorPanel()` | Construye el panel con inputs de P&L por cuenta. Excluye la cuenta seleccionada en el formulario. Pre-carga el P&L del campo `#t-pnl`. |
| `applyReplicator()` | Crea los trades hijos (réplicas). Por cada cuenta adicional: crea un trade con `isChild:true`, `isMother:false`, `notebook:''` y el P&L editado. Llama `saveTrade()` al final. ⚠️ El `motherId` puede ser incorrecto (ver SCHEMA.md). |

---

### CONFIGURACIÓN (ZONE SYSTEM)

| Función | Descripción |
|---------|-------------|
| `openZoneCfg(zone)` | Abre el modal de configuración de zona. Genera contenido dinámico según `zone`: `'dashboard'`, `'checklist'`, `'log'`, `'trades'`, `'phases'`, `'funds'`, `'accs'`, `'pat'`. |
| `closeZoneModal()` | Cierra el modal de zona. |
| `saveZoneEdit(field)` | Guarda ediciones en el modal de zona. Actualmente solo maneja `field === 'name'`. |
| `renderZoneCL()` | Renderiza el editor de checklist dentro del modal de zona. Lista todos los ítems activos con botones de renombrar/eliminar. |
| `renderCfgProfile()` | Actualiza los valores de perfil mostrados en la sección Ajustes (nombre, país, plataforma, riesgo%, R:R mínimo, instrumentos). |
| `addCLItem()` | Agrega un ítem personalizado al checklist desde el modal de zona. Lee posición de inserción del select `#ze-cl-pos`. |
| `clRenameBtn(btn)` | Renombra un ítem del checklist (predeterminado o personalizado) con `prompt()`. |
| `clDeleteBtn(btn)` | Elimina un ítem del checklist por índice. No protege los 13 ítems base. ⚠️ Ver nota abajo. |
| `resetCLToDefault()` | Restaura `cfg.clItems` a los 13 ítems predeterminados y limpia `cfg.customCLItems`. |

**⚠️ Nota sobre `clDeleteBtn`:** La regla 4 de PROJECT_RULES.md dice que los 13 ítems predeterminados nunca se eliminan. Sin embargo, `clDeleteBtn()` permite eliminar cualquier ítem por índice sin validar si es un ítem base. Si el usuario llega al modal de checklist, puede eliminar ítems predeterminados.

---

### IMAGEN

| Función | Descripción |
|---------|-------------|
| `openM(src)` | Abre el modal de imagen con la foto del trade. |
| `closeM()` | Cierra el modal de imagen. ⚠️ Declarada dos veces en el código. |

---

### BACKUP Y DATOS

| Función | Descripción |
|---------|-------------|
| `exportD()` | Exporta `{ cfg, trades, phases, completedPh, funds, checklists }` como JSON. Nombre de archivo: `tradeos-backup-YYYY-MM-DD.json`. No incluye bias, tema ni jft_beta. |
| `importD(inp)` | Importa un JSON de backup. Restaura cada clave individualmente en localStorage si existe en el JSON. Recarga la página. No valida versión del esquema. |
| `resetAll()` | Limpia todo el localStorage (`localStorage.clear()`) y recarga. ⚠️ Declarada dos veces en el código. |

---

### CHIPS Y SELECCIÓN (ONBOARDING)

| Función | Descripción |
|---------|-------------|
| `sc(el)` | Selección single de chip en onboarding. Deselecciona todos los chips del mismo grupo y selecciona el clickeado. |
| `sm(el)` | Selección múltiple de chip en onboarding. Toggle del chip. Mantiene array en `mSel[groupId]`. |
| `getM(id)` | Retorna el array de selecciones múltiples de un grupo: `mSel[id] || []`. |
| `getSel(g)` | Retorna la selección single de un grupo: `cSel[g] || null`. |

---

## Funciones Inline Críticas (no nombradas, definidas en HTML)

Estas funciones no están en el JS global pero son parte de la lógica del sistema:

| Expresión | Ubicación | Descripción |
|-----------|-----------|-------------|
| `localStorage.setItem('jft_beta','1')` | Beta splash button | Marca que el usuario aceptó el splash de beta |
| IIFE del beta splash | `<script>` línea 1107 | Verifica `jft_beta` y muestra/oculta el splash |

---

## Hallazgos sobre Funciones

### Funciones duplicadas (declaradas dos veces, la segunda sobrescribe)
- `resetAll()` — líneas ~3136 y ~3184
- `openM()` — líneas ~3137 y ~3185
- `closeM()` — líneas ~3138 y ~3186
- `showN()` — líneas ~3139 y ~3187
- `toggleTheme()` — líneas ~3141 y ~3189
- `loadTheme()` — líneas ~3150 y ~3198

### Funciones posiblemente no utilizadas en la UI actual
- `buildRuleSelect()` — usa `<select id="t-rule">` pero la UI actual usa chips `#t-rule-chips`
- `toggleReplicator()` — el flujo actual usa `openReplicatorModal()` con modal, no panel inline
- Primera versión de `toggleTag(tag, btn)` — reemplazada por la versión con un solo parámetro `el`

### Función con dos implementaciones de curva de equity
- `renderEquity()` — usa `<canvas>` con Canvas 2D API
- `renderEquityCurve()` — usa `<svg>` inline
`initApp()` llama `renderEquity()`. `renderDash()` llama `setTimeout(renderEquityCurve, 50)`. Ambas se ejecutan y renderizan en elementos distintos (`#equity-chart` y `#equity-curve` respectivamente). No es un bug — son dos elementos visuales diferentes en el dashboard.

---

*Documento generado por auditoría directa del código fuente — Junio 2026*
