# MASTER_CONTEXT.md
## TradeOS — Documento Maestro del Proyecto
**Versión:** 1.0 · **Fecha:** Junio 2026  
**Autor:** Juan Pa · **Contacto:** tradeossoporte@gmail.com  
**Instagram empresa:** @justfortraders_fx · **Instagram personal:** @juanpa05cr

> Este documento es la **fuente principal de verdad** para todos los agentes de IA y desarrolladores que trabajen en el proyecto. Cualquier decisión técnica, de diseño o de negocio debe consultarse aquí primero.

---

## 1. Nombre del Proyecto

**TradeOS**  
Eslogan EN: *"The operating system for serious traders."*  
Eslogan ES: *"El sistema operativo para traders serios."*

---

## 2. Objetivo Principal

Ser el sistema operativo personal de un trader de prop firms — un archivo HTML local que funciona sin internet, sin cuentas, sin servidores, y que centraliza todo lo que un trader necesita para operar con consistencia y disciplina: journal de trades, checklist diario, calculadora de lotaje, seguimiento de cuentas, gestión de fondos, análisis de patrones y fases de progreso.

---

## 3. Problema que Resuelve

Los traders serios operan múltiples cuentas en distintas prop firms (Funding Pips, FTMO, E8, Apex), usan replicadores MT5, y mezclan forex con futuros CME. No existe una herramienta que:

- Se adapte a esa realidad multiuenta y multi-firma
- Funcione offline sin depender de servidores externos
- Combine journal + psicología + calculadora + gestión de fondos en un solo lugar
- Sea personalizable desde el primer uso sin registro ni suscripción
- Trate al trader como un profesional, no como un principiante

TradeOS resuelve exactamente eso.

---

## 4. Público Objetivo

**Primario:** Traders de prop firms latinoamericanos (18-35 años) que operan forex y/o futuros CME, con al menos 1 cuenta fondeada o en challenge activo.

**Secundario:** Traders independientes con capital propio que quieren un journal profesional sin pagar suscripciones mensuales.

**Perfil específico:**
- Opera London Open y/o New York
- Usa MetaTrader 5 (MT5)
- Tiene entre 1 y 5 cuentas activas simultáneas
- Usa replicador de trades entre cuentas
- Instrumentos: XAU/USD, GBP/USD, EUR/USD, NQ, ES, MNQ
- Entiende conceptos como drawdown, R:R, win rate, consistencia

---

## 5. Funciones Existentes (completo)

### 5.1 Onboarding — 9 pasos personalizados

| Paso | ID | Contenido |
|------|----|-----------|
| 1 | s1 | Perfil: nombre/alias, edad, país, años de experiencia |
| 2 | s2 | Cuentas: broker/firma, tamaño, tipo (propia/fondeada), instrumento (Forex/Futuros CME), drawdown máximo |
| 3 | s3 | Estilo: pares operados, sesiones, estrategia, perfil de riesgo, % riesgo/trade, R:R mínimo |
| 4 | s4 | Reglas: reglas que no se pueden romper |
| 5 | s5 | Horario: trabajo externo, horario de sueño, cuando opera |
| 6 | s6 | Checklist: 13 ítems predeterminados siempre incluidos + personalizables |
| 7 | s7 | Metas financieras: objetivos en $ con nombre |
| 8 | s8 | Fases: selección de fase actual en la carrera |
| 9 | s9 | Psicología: error más frecuente, peor momento del día |

### 5.2 Dashboard

- **Score de Consistencia** (reemplaza XP): promedio entre win rate y cumplimiento de reglas. Muestra porcentaje + label (Excelente/Sólido/Regular/Mejorar) con color (verde/amarillo/rojo)
- **6 métricas**: Trades, Win Rate, P&L neto, Racha limpia, R:R promedio, Reglas rotas esta semana
- **Curva de equity**: canvas SVG con P&L acumulado, aparece desde 2 trades
- **Paneles mini por cuenta**: P&L, barra de drawdown con color, fase activa con código de color
- **Alerta de drawdown**: aparece cuando el drawdown supera el 70% del máximo configurado
- **Metas financieras**: barras de progreso por objetivo
- **Resumen semanal**: auto-generado con estadísticas de la semana
- **Últimos 5 trades**: listado rápido con resultado y monto

### 5.3 Checklist Diario

- 13 ítems predeterminados organizados en 4 secciones: *Antes de operar / Al abrir la plataforma / Al ver un setup / Después del trade*
- Ítems predeterminados permanentes — nunca se eliminan al agregar nuevos
- Selector de posición al agregar nuevos ítems (insertar en cualquier lugar de la lista)
- Campo de **bias del día** — texto libre guardado por fecha en localStorage
- Botón **▶ Iniciar sesión** — activa temporizador visible; al cerrar pregunta si hay trades para registrar y redirige al formulario
- Botón **✕ Resetear para hoy** — limpia los checks sin borrar historial
- Edición en tiempo real: renombrar, eliminar, agregar ítems sin cerrar el modal

### 5.4 Registrar Trade

**Campos del formulario:**
- Par / Instrumento (selector poblado con pares del onboarding)
- Sesión (London / New York / Asia / Overlap)
- Cuenta (selector de cuentas configuradas)
- Resultado (Ganador / Perdedor / Break Even)
- P&L en $ (positivo o negativo)
- R:R real obtenido
- Riesgo % usado
- Estado emocional (Tranquilo / Confiado / Ansioso / Presionado / Cansado / Revenge)
- **Reglas rotas — multi-selección**: chips activables, múltiples reglas seleccionables simultáneamente
- Tags de setup (Barrida de liquidez / Ruptura / Pullback / S&R / FVG / Order Block / News play)
- Descripción del setup (textarea libre)
- **📓 LIBRETA**: activable/desactivable por trade. Solo se guarda en el **trade madre** (`isMother:true`), nunca en réplicas hijas (`isChild:true`). Texto libre para análisis de mercado y reflexión post-trade
- Captura del trade: upload de imagen con botón **✕ eliminar foto**

**Calculadora de lotaje integrada:**
- Funciona sobre el **balance real** (tamaño original - pérdidas acumuladas), no el tamaño original
- Futuros CME exactos: NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL
- Forex exactos: EUR/USD, GBP/USD, AUD/USD, NZD/USD
- Pares excluidos con mensaje claro: USD/JPY, GBP/JPY, XAU/USD, XAG/USD, todos los índices CFD
- Muestra lotaje en estándar, mini y micro (forex) o contratos (futuros)

**Replicar trade:**
- Botón **⊕ Replicar** abre modal con explicación clara: "TRADE MADRE + HIJOS"
- El modal muestra todas las cuentas con campo de P&L editable por cuenta
- El trade madre lleva `isMother:true`, las réplicas llevan `isChild:true, motherId`
- La libreta NO se copia a los hijos

### 5.5 Mis Trades

**Vista lista:**
- Filtros por cuenta, par y resultado
- Badge MADRE/HIJA por trade
- Libreta visible si tiene contenido (panel dorado colapsado)
- Reglas rotas en rojo
- Imagen del gráfico como miniatura clicable
- Botón eliminar por trade

**Vista calendario:**
- Resumen mensual (trades, WR, P&L)
- Días coloreados: verde musgo (ganancia), rojo ladrillo (pérdida), amarillo (BE)
- Dots de trades por día
- Click en día: modal con trades **colapsables** (auto-colapsa si hay más de 2)
- Cada trade en el modal muestra: par, sesión, P&L, estado emocional, R:R, regla rota, nota, libreta
- Navegación mes anterior/siguiente

### 5.6 Fases del Plan

6 fases predeterminadas auto-populadas al abrir la sección por primera vez:

| ID | Ícono | Nombre |
|----|-------|--------|
| discipline | 🌱 | Disciplina |
| challenge | 🎯 | Challenge |
| funded | 💰 | Fondeado |
| scaling | 🚀 | Escalando |
| own_capital | 🏦 | Capital propio |
| goals | ❤ | Metas cumplidas |

- Condiciones en tiempo real para la fase Disciplina (racha de sesiones limpias)
- Tareas por fase con checkbox de progreso
- Crear fase personalizada: ícono, nombre, subtítulo, descripción, tareas
- Botón Editar para resetear progreso o eliminar fases custom

### 5.7 Gestión de Fondos

Tipos de movimiento: Payout cuenta propia / Payout fondeada / Salario / Meta alcanzada / Challenge comprado / Gasto / Otro

- Cada payout **vinculado a una cuenta** (selector de cuentas en el formulario)
- La firma se guarda automáticamente desde el nombre de la cuenta
- Historial con ícono, tipo, fecha, nota y monto
- Botón eliminar por movimiento
- 4 métricas resumen: Total ingresos, Payouts fondeadas, Cuenta propia, Gastos/Challenges

### 5.8 Mis Cuentas

Paneles detallados por cuenta con:
- Nombre de firma y tipo (propia/fondeada) + instrumento (Forex/Futuros)
- **Barra de drawdown siempre visible**: % usado vs máximo, margen disponible en $ y %
- Color de barra: verde (<50%), amarillo (50-80%), rojo (>80%)
- P&L, Win Rate, Trades totales, R:R promedio, Reglas rotas
- Botón **FASE**: cicla entre estados (Challenge P1, P2, Fondeada, Activa, Pausada, Evaluación)
- Botón **DD**: edita el drawdown máximo permitido
- También visible en dashboard como mini-cards por cuenta

### 5.9 Análisis de Patrones

- Gráfico SVG de **evolución del Win Rate** semana a semana (últimas 12 semanas)
- Línea de referencia en 50%, color verde/rojo según esté sobre o bajo
- 10 patrones de comportamiento detectados automáticamente desde el trade 5
- Filtro por cuenta
- Patrones detectados incluyen: revenge trading, peor día de la semana, mejor sesión, R:R promedio, etc.

### 5.10 Ajustes del Sistema

- **Perfil actual**: grilla 2×3 con Nombre, País, Platform, Riesgo/trade, R:R mín, Instrumentos
- **Backup/Restore**: exportar JSON con todos los datos, importar desde archivo
- **Soporte**: abre Gmail con email pre-completado a tradeossoporte@gmail.com
- **Tema**: toggle modo oscuro (Obsidian Gold) / modo claro (Crema dorado)
- **Zona de peligro**: reset completo con confirmación

### 5.11 Sistema de idiomas

Bilingüe completo ES/EN. Toggle en la esquina superior derecha de la pantalla de bienvenida. Todas las cadenas de texto se almacenan en el objeto `LG` con claves `es` y `en`. La función `T(key)` devuelve la traducción según `lang` actual.

### 5.12 Beta Splash

Modal de bienvenida para colaboradores beta. Se muestra una vez y se descarta con un botón. Estado persistido en `localStorage('jft_beta')`.

---

## 6. Flujo Completo del Usuario

```
PRIMERA VEZ
────────────
1. Abre trader-os.html en el navegador
2. Beta splash aparece → click "Entendido →"
3. Pantalla de bienvenida → click "Empezar configuración →"
4. Onboarding pasos 1-9 (< 5 minutos)
5. Click "Crear mi dashboard →" → app se inicializa con sus datos

USO DIARIO
────────────
06:00  → Abrir TradeOS
       → Ir a Checklist → completar los 13 ítems
       → Escribir bias del día
       → Click "▶ Iniciar sesión" → temporizador activo

~02:00 (London Open CR) → Identificar setup
       → Ir a Registrar trade
       → Calcular lotaje exacto en la calculadora integrada
       → Ejecutar trade en MT5

Después del trade:
       → Registrar trade en TradeOS
       → Activar libreta si el trade requiere análisis
       → Si operó en múltiples cuentas → usar ⊕ Replicar
       → Click "GUARDAR TRADE →"
       → La sesión finaliza → click "Cerrar sesión"

SEMANAL
────────────
- Revisar Dashboard → Score de Consistencia, curva de equity
- Revisar Patrones → evolución win rate semanal
- Revisar Cuentas → estado de drawdown por firma
- Exportar backup desde Ajustes

FONDOS
────────────
- Al recibir payout → registrar en Gestión de Fondos
- Vincular al nombre de la cuenta de la firma correspondiente
```

---

## 7. Tecnologías Utilizadas

| Categoría | Tecnología | Detalle |
|-----------|-----------|---------|
| Frontend | HTML5 | Single file, sin framework |
| Estilos | CSS3 con variables | Paleta completa en `:root`, modo claro/oscuro |
| Lógica | Vanilla JavaScript ES6+ | Sin dependencias externas de runtime |
| Persistencia | localStorage | 8 claves propias (ver §13) |
| Gráficos | Canvas API + SVG inline | Curva de equity, gráfico de evolución semanal |
| Fuentes | Google Fonts CDN | Bebas Neue (únicamente) |
| DM Sans / DM Mono | Importadas desde fonts.gstatic.com | Tipografía principal y monospace |
| Distribución | Archivo `.html` descargable | Compatible con Chrome, Firefox, Edge en Windows y macOS |

**No hay:**
- Backend
- Base de datos externa
- Autenticación
- APIs de terceros (excepto fuentes)
- Node.js / npm
- Frameworks (React, Vue, etc.)

---

## 8. Estructura General de la Aplicación

```
trader-os.html
├── <head>
│   ├── Meta tags (charset, viewport, cache-control)
│   ├── Google Fonts (Bebas Neue)
│   └── <style> — 33KB de CSS con variables, componentes, responsive
│
├── <body>
│   ├── #ld — Loader splash (animación de carga)
│   ├── #ob — Onboarding (pasos s0-s9, oculto tras setup)
│   ├── #app — App principal
│   │   ├── .sb — Sidebar con 9 nav items
│   │   └── .main — Área de contenido
│   │       ├── #view-dash (class="view on") — Dashboard
│   │       ├── #view-cl (class="view") — Checklist
│   │       ├── #view-log (class="view") — Registrar trade
│   │       ├── #view-trades (class="view") — Mis trades
│   │       ├── #view-phases (class="view") — Fases
│   │       ├── #view-funds (class="view") — Fondos
│   │       ├── #view-accs (class="view") — Cuentas
│   │       ├── #view-pat (class="view") — Patrones
│   │       └── #view-cfg (class="view") — Ajustes
│   ├── #rep-modal — Modal replicador de trades
│   ├── #zone-modal — Modal de configuración de zonas
│   ├── #custom-phase-modal — Modal nueva fase
│   ├── #img-modal — Modal visor de imagen
│   └── #beta-splash — Splash de bienvenida beta
│
├── <script> (beta-splash inline — separado)
│   └── (function(){...})() — Chequea localStorage('jft_beta')
│
└── <script> (script principal — 135KB, 2094 líneas, 130 funciones)
    ├── STATE — Variables globales
    ├── LANG — Sistema bilingüe LG{es,en}
    ├── ONBOARDING — goS(), finish(), helpers
    ├── CORE — sv(), saveAll(), loadAll(), initApp()
    ├── DASHBOARD — renderDash(), renderEquity(), renderGoals(), renderWeeklySummary(), renderDashAccPanels()
    ├── CHECKLIST — buildChecklist(), toggleCL(), renderZoneCL()
    ├── TRADES — saveTrade(), renderTrades(), renderCalendar(), calDayClick()
    ├── PHASES — ensureDefaultPhases(), renderPhases(), buildPhases(), evalCond()
    ├── FUNDS — saveFund(), renderFunds(), delFund(), buildFiAccSel()
    ├── ACCOUNTS — renderAccs(), editAccPhase(), editAccDD()
    ├── PATTERNS — renderPatterns(), renderWeeklyEvolution(), getISOWeek()
    ├── CALCULATOR — calcLot(), buildLotCalcSel()
    ├── REPLICATOR — openReplicatorModal(), buildReplicatorPanel(), applyReplicator()
    ├── NOTEBOOK — toggleNotebook()
    ├── SESSION — startSession(), endSession()
    ├── ZONE CONFIG — openZoneCfg(), closeZoneModal(), saveZoneEdit()
    ├── CONFIG — renderCfgProfile(), exportD(), importD(), resetAll()
    └── UTILS — showN(), toggleTheme(), openM(), T(), openSupport()
```

### Objeto de configuración (`cfg`)

```javascript
cfg = {
  name, age, country, experience,        // Perfil
  platform, riskPct, rrMin,             // Estilo
  pairs[], sessions[], strategies[],    // Instrumentos
  rules[], clItems[],                   // Reglas y checklist
  goals[{name, amount}],               // Metas
  accounts[{
    id, firm, type, size, maxDD,
    balance, pnl, drawdown,
    phase, instrument            // 'forex' | 'futures'
  }],
  psychProfile, mainError,             // Psicología
}

// Datos separados en arrays globales:
trades[]    // Historial completo
phases{}    // Estado de tareas por fase
completedPh[] // Fases completadas
funds[]     // Historial de movimientos
checklists{} // Estado de checklists por fecha
```

### Objeto de trade

```javascript
{
  id, date,
  pair, session, account, result,
  pnl, rr, risk, mood,
  ruleBroken,        // string, múltiplos separados por ', '
  note,              // descripción del setup
  notebook,          // libreta (solo en trade madre)
  img,               // base64 de la captura
  tags[],            // tags de setup
  isMother,          // true en trade original
  isChild,           // true en réplicas
  motherId           // id del trade madre (en hijos)
}
```

---

## 9. Diseño Visual Actual

### Paleta — Modo Oscuro (Obsidian Gold)

| Variable | Valor | Uso |
|----------|-------|-----|
| `--bg` | `#07070f` | Fondo base |
| `--bg2` | `#0d0d1a` | Cards, panels |
| `--bg3` | `#131320` | Inputs, backgrounds secundarios |
| `--bg4` | `#1a1a2e` | Borders, highlights muy sutiles |
| `--accent` | `#c8a96e` | Dorado principal — elementos clave |
| `--accent2` | `#e8c98e` | Dorado claro |
| `--accent3` | `#a07840` | Dorado oscuro |
| `--win` | `#5a7a4a` | Verde musgo — ganancias |
| `--loss` | `#8a4a35` | Rojo ladrillo — pérdidas |
| `--warn` | `#c49a2a` | Amarillo — advertencias |
| `--text` | `#e8e8f5` | Texto principal |
| `--text2` | `#72729a` | Texto secundario |
| `--text3` | `#3a3a5a` | Texto terciario / placeholders |

### Paleta — Modo Claro (Crema Dorado)

`--bg: #f5f0e8` / `--bg2: #ede8dc` / `--accent: #8b6520` / `--text: #1a1510`

### Tipografía

- **Display / Títulos grandes:** Bebas Neue (Google Fonts CDN)
- **UI principal:** DM Sans (Google Fonts CDN)
- **Monospace / Métricas:** DM Mono (Google Fonts CDN)
- **Fallbacks:** `Georgia, serif` / `sans-serif` / `monospace`

### Border radius

- `--r: 14px` — Cards principales
- `--r2: 9px` — Inputs, badges, elementos secundarios

### Principios de diseño

1. **Sin decoración innecesaria** — cada elemento tiene función
2. **Datos primero** — métricas grandes, labels pequeños en monospace
3. **Dorado = atención** — el dorado señala elementos activos, importantes o de acción
4. **Verde musgo = bien, Rojo ladrillo = mal** — consistente en toda la app
5. **Modo oscuro por defecto** — traders operan de madrugada
6. **Densidad de información alta** — grids compactos, no espacios vacíos

---

## 10. Funciones Pendientes o Sugeridas

### Alta prioridad (más valor para el trader)

- **Diario de mercado** — sección separada para análisis pre-sesión por día (HTF bias, niveles clave, contexto), independiente del checklist y de los trades
- **Notas post-sesión** — campo libre después de cerrar sesión para aprendizajes del día
- **Precio de entrada + SL/TP en el trade** — campos para documentar niveles exactos (sin afectar calculadora)
- **Editar trade registrado** — actualmente no se puede modificar un trade ya guardado
- **Exportar a CSV** — exportar historial completo para análisis en Excel/Sheets

### Media prioridad

- **Estadísticas avanzadas por sesión** — cuál sesión tiene mejor WR, mejor R:R
- **Filtro de calendario por cuenta** — ver solo los días de una cuenta específica
- **Notificaciones de drawdown** — alerta visual más prominente cuando se acerca al límite
- **Heatmap de días de la semana** — visualización de rendimiento Lun-Vie
- **Modo compact para sidebar** — sidebar colapsable a solo íconos

### Baja prioridad / Futuro SaaS

- **Sincronización cloud** — pasar de localStorage a backend (Supabase, Firebase)
- **Multi-device** — acceso desde móvil y desktop con los mismos datos
- **Cuentas de usuario** — sistema de registro/login
- **Plan gratuito / premium** — feature gating por plan
- **Calculadora con precio en tiempo real** — API de precios para pares no exactos (JPY, XAU)
- **Integración MT5** — lectura automática de trades desde MetaTrader
- **App móvil** — versión PWA o React Native

---

## 11. Modelos de Monetización

### Actual (versión beta)

- **Venta directa en Gumroad:** $19 USD por copia
- **Split:** Gumroad 10% → vendedor recibe 90%
- **Afiliados:** 30% para creadores de contenido que refieran ventas

### Proyectado (versión SaaS)

| Modelo | Descripción | Precio sugerido |
|--------|-------------|-----------------|
| **One-time** | Compra única del archivo HTML | $19-$29 USD |
| **Freemium** | Gratis (1 cuenta, 50 trades) / Pro ilimitado | $9/mes o $79/año |
| **Pro** | Todas las funciones, sync cloud, multi-device | $15/mes o $119/año |
| **Teams** | Para grupos de traders o academias | $49/mes (hasta 10 usuarios) |
| **Afiliados** | 30% de comisión para referidos | Recurrente |
| **Licencias institucionales** | Para prop firms que quieren darlo a sus traders | Negociado |

---

## 12. Reglas que NUNCA Deben Romperse

> Estas reglas son invariantes del proyecto. Ningún agente de IA ni desarrollador debe violarlas.

### 12.1 Reglas de datos

1. **Los trades madre nunca se convierten en hijos** — `isMother` y `isChild` son mutuamente excluyentes
2. **La libreta solo vive en el trade madre** — nunca copiar `notebook` a réplicas hijas
3. **El backup es la única fuente de verdad** — nunca transformar datos al importar, siempre restaurar exacto
4. **Los ítems predeterminados del checklist nunca se eliminan** al agregar nuevos — siempre incluir los 13 base
5. **El balance real para la calculadora** = `account.size + pnl_negativo_acumulado` — nunca usar `size` a secas si hay pérdidas

### 12.2 Reglas de UI/UX

6. **El dorado `#c8a96e` es el color de identidad de marca** — no cambiar, no reemplazar
7. **El verde musgo `#5a7a4a` = ganancia / bien** — no usar verde limón ni verde neón
8. **El rojo ladrillo `#8a4a35` = pérdida / mal** — no usar rojo brillante
9. **Todas las métricas numéricas van en DM Mono** — nunca en DM Sans
10. **El sidebar tiene exactamente 9 secciones** — no agregar ni quitar sin rediseño completo

### 12.3 Reglas de código

11. **Siempre validar con Node.js después de cualquier cambio** — `node --check script.js` debe pasar
12. **Cero closing tags sin escapar en JS strings** — `</tag>` dentro de strings debe ser `<\/tag>`
13. **Todos los `getElementById` deben tener null check** — nunca `.textContent =` sin `if(el)`
14. **sv() es la única función que navega entre vistas** — no manipular `.view.on` directamente desde otros lugares
15. **El `window.addEventListener('load')` debe ser único** — nunca duplicar
16. **Todos los elementos dentro de `.main` deben tener `class="view"`** — sin excepción
17. **Los datos nunca se envían a servidores externos** — la app es 100% local

### 12.4 Reglas de negocio

18. **El eslogan no cambia** — "The operating system for serious traders."
19. **La calculadora solo calcula si el resultado es exacto** — si el par no tiene valor por pip fijo, mostrar mensaje explicativo, nunca dar número aproximado sin avisarlo
20. **La calculadora usa el balance real, no el tamaño original** — ya documentado en §5.4

---

## 13. Dependencias Externas y APIs

### Runtime (necesarias para que la app funcione)

| Dependencia | URL | Uso | Fallback si falla |
|------------|-----|-----|-------------------|
| Google Fonts — Bebas Neue | `fonts.googleapis.com` | Tipografía de display | `sans-serif` — funciona pero cambia visual |
| Google Fonts — DM Sans | `fonts.gstatic.com` | Tipografía UI | `sans-serif` |
| Google Fonts — DM Mono | `fonts.gstatic.com` | Tipografía monospace | `monospace` |

> **La app funciona offline.** Las fuentes se cargan la primera vez y el browser las cachea. Si no hay internet en el primer uso, se usan los fallbacks del sistema.

### localStorage — claves usadas

| Clave | Contenido |
|-------|-----------|
| `tt_cfg` | Objeto `cfg` completo (perfil, cuentas, reglas, etc.) |
| `tt_trades` | Array de trades |
| `tt_ph` | Estado de fases y tareas |
| `tt_fd` | Array de movimientos de fondos |
| `tt_cl` | Estado del checklist |
| `tt_cp` | Fases completadas |
| `tt_bias_[fecha]` | Bias del día por fecha (ej: `tt_bias_2026-06-02`) |
| `tt_theme` | `'light'` o `'dark'` |
| `jft_beta` | `'1'` si ya vio el splash de beta |

### Ninguna API de terceros

No hay llamadas a:
- APIs de precios (Forex, futuros)
- APIs de brokers o prop firms
- Servicios de analytics
- Tracking o telemetría
- Servicios de autenticación

---

## 14. Riesgos Técnicos Identificados

### Críticos

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|-----------|
| **Pérdida de datos por cache del navegador** | Alta | Crítico | Recordar al usuario exportar backup periódicamente. El botón "Exportar backup" está visible en Ajustes |
| **localStorage lleno** | Media (con muchas imágenes base64) | Alto | Comprimir imágenes antes de guardar (pendiente) o limitar a 1 imagen por trade |
| **Corrupción del archivo HTML en edición manual** | Alta (historial) | Crítico | Nunca editar el archivo con editores que auto-formateen HTML. Solo editar con herramientas que respeten la estructura |

### Moderados

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|-------------|---------|-----------|
| **Safari iOS bloquea localStorage en archivos locales** | Alta | Alto | La app no funciona en iPhone/iPad. Solución: hospedar en Netlify/GitHub Pages |
| **Cambio de política de Google Fonts** | Baja | Medio | Las fuentes tienen fallbacks en CSS |
| **El archivo crece demasiado** | Media | Medio | Actualmente 225KB — el límite práctico es ~500KB antes de que cargue lento |
| **Imágenes base64 saturan localStorage** | Media | Alto | localStorage tiene límite de ~5-10MB según navegador. Si el trader sube muchas capturas, puede llegar al límite |

### Técnicos de código

| Riesgo | Estado | Nota |
|--------|--------|------|
| Script tag partido por `</script>` en strings JS | **Resuelto** — estructura separada en 2 `<script>` | El beta-splash tiene su propio script tag |
| Closing tags sin escapar en strings JS | **Monitoreo continuo** | Siempre verificar con `node --check` |
| Duplicación de `window.addEventListener('load')` | **Resuelto** — 1 solo | Verificar en cada edición mayor |
| Vistas fuera del `.main` | **Resuelto** — auditado | El div depth de `.main` debe terminar en 0 antes del script |
| Clases CSS duplicadas con conflicto | **Parcialmente resuelto** — `.fi-ic` tenía conflicto, renombrado a `.cfg-cell` | Revisar al agregar nuevas secciones |

---

## 15. Resumen Ejecutivo

TradeOS es una aplicación web de archivo único (225KB, sin backend, sin dependencias de runtime) que funciona como el sistema operativo personal de un trader de prop firms. Está construida en HTML/CSS/JavaScript vanilla, persiste todo en localStorage, y no requiere internet para funcionar después de la carga inicial.

**En su estado actual (v1.0 beta):**
- Onboarding personalizado en 9 pasos que adapta toda la experiencia al trader específico
- Soporte completo para forex majors, índices CFD y futuros CME (NQ, ES, YM, etc.)
- Gestión de múltiples cuentas de distintas prop firms con drawdown en tiempo real
- Journal de trades con libreta por trade, replicador para múltiples cuentas, calculadora de lotaje exacta
- Análisis de patrones con evolución semanal de win rate
- Checklist diario con temporizador de sesión
- Sistema bilingüe ES/EN completo
- Modo oscuro (Obsidian Gold) y modo claro (Crema dorado)
- Exportación/importación de backup en JSON

**El producto resuelve un problema real:** los traders de prop firms latinoamericanos que operan múltiples cuentas con replicador MT5 no tienen una herramienta que combine journal + calculadora + gestión + análisis en un lugar que funcione offline y se adapte a su realidad.

**Modelo de distribución actual:** venta directa en Gumroad a $19 USD.

**Camino al SaaS:** el siguiente paso lógico es migrar la persistencia de localStorage a un backend cloud (Supabase recomendado por su plan gratuito y SDKs modernos), agregar autenticación simple (magic link por email), y hospedar la app en un dominio propio. El código frontend puede mantenerse en vanilla JS o migrarse a React. La lógica de negocio ya está completamente desarrollada y probada.

**Estado de código al momento de este documento:**
- Node.js syntax check: PASS
- Closing HTML tags en JS strings: 0
- Div depth en el HTML body: 0
- window.addEventListener: 1 (no duplicado)
- Todas las vistas dentro de `.main`: confirmado
- Funciones críticas presentes: 130/130
- Tamaño del archivo: 225KB

---

*Última actualización: Junio 2026*  
*Creado por: Juan Pa — TradeOS*  
*Contacto: tradeossoporte@gmail.com*
