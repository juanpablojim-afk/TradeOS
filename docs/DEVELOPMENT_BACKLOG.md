# DEVELOPMENT_BACKLOG.md
## TradeOS — Backlog de Desarrollo

**Fuente:** EXECUTIVE_REPORT.md (auditoría directa de código, Junio 2026)  
**Versión de referencia:** v1.0 Beta  
**Mantenedor:** Juan Pa · tradeossoporte@gmail.com  
**Última actualización:** Junio 2026

> Este documento es la traducción operacional del Executive Report. Cada ítem tiene origen trazable, categoría clara y estimación de esfuerzo. Las categorías son mutuamente excluyentes — ningún ítem aparece en dos secciones.

---

## ÍNDICE

1. [Correcciones Críticas](#1-correcciones-críticas)
2. [Arquitectura SaaS](#2-arquitectura-saas)
3. [Monetización](#3-monetización)
4. [Nuevas Funciones](#4-nuevas-funciones)

---

## 1. CORRECCIONES CRÍTICAS

> Bugs activos en producción y deuda técnica que pone en riesgo los datos del usuario o la integridad de las métricas. Nada de esta sección puede quedar pendiente al lanzar una campaña de distribución.

---

### C-01 — `saveAll()` sin manejo de `QuotaExceededError`

**Severidad:** 🔴 Crítico  
**Sprint objetivo:** v1.1 — Bloque A  
**Esfuerzo:** Medio  
**Origen:** D1, DT1, RC1, Acción #1

**Problema:**  
`saveAll()` no tiene try/catch. Cuando localStorage se llena, la función falla silenciosamente. El trader recibe el toast "guardado ✓" pero el trade no persiste. La pérdida ocurre sin ningún aviso.

**Fix requerido:**
- Envolver `saveAll()` en try/catch
- Si falla con `QuotaExceededError`: mostrar **modal bloqueante** (no toast) con instrucciones claras: exportar backup y eliminar fotos de trades
- Agregar barra de uso de localStorage en Ajustes con alerta visual proactiva al 80%

**Riesgo de no corregir:** Un caso documentado públicamente de pérdida de datos daña el canal de recomendaciones orgánicas. El mercado latinoamericano de trading es pequeño y vocal.

---

### C-02 — Imágenes guardadas sin compresión en `prevImg()`

**Severidad:** 🔴 Crítico  
**Sprint objetivo:** v1.1 — Bloque A  
**Esfuerzo:** Bajo  
**Origen:** D1, RC1, Acción #2

**Problema:**  
Las capturas de gráficos se guardan como base64 sin compresión. Promedio estimado: ~200 KB por imagen. Con 30+ capturas (uso esperado del producto), localStorage se satura antes de los 5 MB de límite navegador.

**Fix requerido:**
- En `prevImg()`: usar `canvas.toDataURL('image/jpeg', 0.6)` antes de asignar a `imgB64`
- Limitar ancho máximo a 1200px antes de comprimir
- Mostrar tamaño estimado de la imagen en el formulario de trade
- Target: de ~200 KB por imagen a ~30-50 KB

**Relación con C-01:** C-02 ataca la causa raíz; C-01 maneja las consecuencias. Ambos deben implementarse.

---

### C-03 — `importD()` no restaura historial de checklist (`tt_cl`)

**Severidad:** 🔴 Crítico  
**Sprint objetivo:** v1.1 — Bloque A  
**Esfuerzo:** Trivial  
**Origen:** D1, DT6, Acción #3

**Problema:**  
La línea que restaura `tt_cl` al importar backup simplemente no existe en el código. Todo usuario que migra de dispositivo o restaura un backup pierde su historial completo de disciplina diaria.

**Fix requerido:**
```javascript
if(d.checklists) localStorage.setItem('tt_cl', JSON.stringify(d.checklists));
```
Una línea. Impacto máximo en confianza del usuario.

---

### C-04 — `importD()` sin validación de esquema ni rollback

**Severidad:** 🔴 Crítico  
**Sprint objetivo:** v1.1 — Bloque A  
**Esfuerzo:** Medio  
**Origen:** RC1, Acción #4

**Problema:**  
Si el usuario importa un JSON malformado o incompleto, `importD()` puede sobrescribir datos válidos con datos corruptos. La herramienta de recuperación de datos no puede ser ella misma un vector de destrucción de datos.

**Fix requerido:**
- Snapshot del estado actual en memoria antes de escribir
- Validar: `d.cfg` es objeto con `d.cfg.name` como string, `d.trades` es array, `d.funds` es array
- Si validación falla: no escribir nada + mostrar error específico + mantener estado previo
- Si el proceso falla a mitad: rollback automático al snapshot

---

### C-05 — `delTrade()` deja réplicas huérfanas al eliminar una madre

**Severidad:** 🔴 Crítico  
**Sprint objetivo:** v1.1 — Bloque A  
**Esfuerzo:** Bajo  
**Origen:** RC3, D8, Acción #5

**Problema:**  
Al eliminar un trade con `isMother: true`, las réplicas hijas quedan en `trades[]` con `motherId` apuntando a un trade inexistente. Win rate, P&L y análisis de patrones muestran datos incorrectos de forma acumulativa — el caso de uso más diferenciador del producto genera métricas corruptas.

**Fix requerido:**
- Al eliminar una madre: buscar todos los trades con `motherId === id`
- Mostrar modal de confirmación: "Este trade tiene X réplicas en otras cuentas. ¿Eliminar todas?"
- Opción A: eliminar madre + todas las hijas
- Opción B: eliminar solo la madre, promover hijas a trades independientes (`isChild: false`, eliminar `motherId`)

---

### C-06 — CSV exportado muestra ID de cuenta, no nombre de firma

**Severidad:** 🟠 Alto  
**Sprint objetivo:** v1.1 — Bloque B  
**Esfuerzo:** Trivial  
**Origen:** D6, Acción #10

**Problema:**  
La columna "Cuenta" del CSV contiene `"1"`, `"2"`, `"3"` en lugar de `"Funding Pips"`, `"FTMO"`, `"E8"`. El archivo exportado requiere una tabla de referencia para interpretarse — no es un archivo de datos, es basura para análisis externo.

**Fix requerido:**
- En `exportTradesCSV()`: cruzar `t.account` con `cfg.accounts` para obtener `a.firm`
- Incluir el nombre de la firma en la columna correspondiente del CSV

---

### C-07 — Resumen semanal oculto de lunes a viernes

**Severidad:** 🟠 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Trivial  
**Origen:** D3, Acción #11

**Problema:**  
Un guard deliberado en `renderWeeklySummary()` oculta el panel de lunes a viernes. El resumen semanal es exactamente la información que el trader necesita el lunes por la mañana antes de operar — no solo el fin de semana.

**Fix requerido:**
- Eliminar el guard `if(now.getDay()!==0&&now.getDay()!==6)` en `renderWeeklySummary()`
- El panel debe ser visible todos los días de la semana

---

### C-08 — `motherId` puede apuntar al trade anterior (no al madre real)

**Severidad:** 🟠 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Bajo  
**Origen:** DT9, Acción implícita

**Problema:**  
En `applyReplicator()`, el `motherId` se asigna como `trades[0]?.id` antes de que `saveTrade()` guarde el trade madre. En ese momento, `trades[0]` es el penúltimo trade guardado, no el madre actual. El bug no tiene consecuencias visibles hoy, pero cualquier feature de navegación hija→madre lo manifestará.

**Fix requerido:**
- Asignar `motherId` después de que `saveTrade()` retorne el ID real del trade madre guardado
- Verificar que el ID asignado corresponda al trade recién guardado

---

### C-09 — Bias del día excluido del sistema de backup

**Severidad:** 🟡 Medio  
**Sprint objetivo:** v1.1 — Bloque B  
**Esfuerzo:** Bajo  
**Origen:** DT6, Acción #13

**Problema:**  
Las claves `tt_bias_[fecha]` son claves independientes de localStorage que no pasan por `saveAll()` ni se incluyen en el backup exportado. El historial de análisis pre-sesión del trader desaparece en cualquier migración de dispositivo.

**Fix requerido:**
- En `exportD()`: iterar sobre todas las claves de localStorage que empiecen con `tt_bias_` e incluirlas en el JSON de backup
- En `importD()`: restaurar esas claves al importar

---

### C-10 — Funciones duplicadas y código muerto en scope global

**Severidad:** 🟡 Medio  
**Sprint objetivo:** v1.1 — Bloque C  
**Esfuerzo:** Trivial  
**Origen:** D8, DT2, DT8, Acción #16

**Problema:**  
6 funciones declaradas dos veces: `resetAll()`, `openM()`, `closeM()`, `showN()`, `toggleTheme()`, `loadTheme()`. La segunda declaración sobrescribe a la primera sin error — cualquier modificación futura debe hacerse en ambos lugares o se crea un bug silencioso. Adicionalmente: `buildRuleSelect()`, primera versión de `toggleTag()` y `toggleReplicator()` son código muerto que nunca se invoca.

**Fix requerido:**
- Eliminar la segunda declaración de las 6 funciones duplicadas
- Eliminar o deprecar las 3 funciones de código muerto
- Reducción estimada del archivo: 2-3 KB

---

### C-11 — Doble prefijo `custom_custom_` en ID de fases personalizadas

**Severidad:** 🟡 Medio  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Trivial  
**Origen:** DT7

**Problema:**  
Las fases personalizadas generan IDs con formato `'ph_custom_custom_[timestamp]'`. No causa errores visibles hoy, pero cualquier feature que navegue por ID de fase (analytics de progreso, exportación) generará inconsistencias.

**Fix requerido:**
- Corregir la generación del ID para producir `'ph_custom_[timestamp]'`
- Migrar IDs existentes con doble prefijo al formato correcto al cargar el estado

---

### C-12 — Tipos inconsistentes: `Account.id` (number) vs `trade.account` (string)

**Severidad:** 🟡 Medio  
**Sprint objetivo:** Pre-v2.0  
**Esfuerzo:** Bajo  
**Origen:** DT3

**Problema:**  
`Account.id` es number. `trade.account` es string. Se comparan mediante `String(t.account) === String(a.id)`. Funciona hoy por la coerción, pero en una migración a Supabase (tipos estrictos) requiere una transformación de datos sobre datos reales de usuarios — el escenario más riesgoso posible.

**Fix requerido:**
- Normalizar `Account.id` a string UUID en toda la codebase
- Actualizar la comparación en todos los puntos donde se cruzan
- Documentar el tipo final en SCHEMA.md

---

### C-13 — Campos muertos en el objeto Account

**Severidad:** 🟡 Medio  
**Sprint objetivo:** Pre-v2.0  
**Esfuerzo:** Trivial  
**Origen:** DT4

**Problema:**  
`Account.balance`, `Account.drawdown` y `Account.pnl` se inicializan en el onboarding y nunca se actualizan. El P&L real se calcula en runtime. Estos campos ocupan espacio, confunden a quien lee el esquema, y son una trampa para features futuras que podrían leer el valor almacenado (incorrecto) en lugar de calcularlo.

**Fix requerido:**
- Eliminar `balance`, `drawdown` y `pnl` del objeto Account en el onboarding
- Verificar que ningún código lee estos campos (si existe: refactorizar al cálculo en runtime)
- Actualizar SCHEMA.md

---

### C-14 — `checklists` crece sin límite en localStorage

**Severidad:** 🟢 Bajo  
**Sprint objetivo:** v1.1 — Bloque C  
**Esfuerzo:** Bajo  
**Origen:** D7, DT5

**Problema:**  
Cada día agrega una entrada nueva a `tt_cl`. Sin limpieza automática, a 73 KB/año, en 3-4 años de uso diario el acumulado contribuye significativamente al límite de localStorage.

**Fix requerido:**
- Implementar limpieza automática de entradas de checklist de más de 18 meses de antigüedad
- Ejecutar la limpieza al cargar la app o en `saveAll()` con throttle (1 vez por semana)

---

### C-15 — Sin detección de modo incógnito

**Severidad:** 🟢 Bajo  
**Sprint objetivo:** v1.1 — Bloque B  
**Esfuerzo:** Bajo  
**Origen:** Acción #14

**Problema:**  
Un trader en modo incógnito registra una sesión entera y cierra el navegador — pierde todo sin advertencia previa. No es frecuente pero es devastador cuando ocurre.

**Fix requerido:**
- Al cargar la app: intentar escribir un byte en localStorage
- Si falla o si la cuota disponible es cero: mostrar banner prominente de advertencia
- No bloquear el uso — informar y dejar la decisión al trader

---

## 2. ARQUITECTURA SAAS

> Decisiones técnicas que habilitan la evolución del producto de archivo HTML local a plataforma web con backend. Sin estas bases, v2.0 requerirá una reescritura en lugar de una migración.

---

### A-01 — Hospedar en dominio propio (`app.tradeos.io`)

**Prioridad:** 🔴 Alta  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Muy bajo  
**Origen:** RC2, ON1, Acción #6

**Contexto:**  
Safari iOS bloquea localStorage para archivos locales. El 40-60% del tráfico móvil en Latinoamérica usa iOS. En el modelo de distribución actual (archivo HTML descargable), la app no funciona en iPhone/iPad.

**Acciones requeridas:**
- Registrar dominio `tradeos.io` (~$10-15 USD/año)
- Configurar Netlify o Vercel con el archivo `tradeOS_base.html` (gratis en ambas plataformas)
- Resolver Safari iOS como efecto inmediato
- Habilitar URL compartible (reemplaza "descargar un archivo HTML" como proceso de onboarding)
- Prerequisito para: PWA (A-02), actualizaciones automáticas (A-03)

---

### A-02 — PWA — Instalable sin App Store

**Prioridad:** 🟠 Media  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** ROADMAP v1.2

**Contexto:**  
Una vez que la app está en un dominio propio (A-01), convertirla en PWA requiere: un `manifest.json` con nombre, íconos y colores del producto, y un Service Worker mínimo para cache offline. El trader la instala en el homescreen de iOS/Android sin App Store.

**Acciones requeridas:**
- Crear `manifest.json` con los metadatos del producto
- Implementar Service Worker básico con cache del archivo HTML y fuentes
- Agregar meta tags de PWA en el HTML
- Probar instalación en iOS Safari y Android Chrome

**Dependencia:** A-01 debe estar completo.

---

### A-03 — Versión visible en UI + política de actualizaciones documentada

**Prioridad:** 🟠 Media  
**Sprint objetivo:** Inmediato  
**Esfuerzo:** Muy bajo  
**Origen:** RC4, ON2, Acción #8

**Contexto:**  
Cuando se lance v1.1 con los bugs críticos corregidos, no hay forma automatizada de notificar a los compradores de v1.0 que existe una versión más segura. Los usuarios con bugs activos siguen en riesgo indefinidamente.

**Acciones requeridas:**
- Mostrar número de versión (ej: `v1.0.3`) en el sidebar o en la sección Ajustes
- Documentar proceso de notificación en Gumroad (email a compradores existentes con link a versión actualizada)
- Crear política de actualizaciones gratuitas para compradores de v1.x
- Implementar en el email de Gumroad un changelog breve por release

---

### A-04 — Normalizar esquema de datos para migración a Supabase

**Prioridad:** 🟠 Media  
**Sprint objetivo:** Pre-v2.0  
**Esfuerzo:** Alto  
**Origen:** DT3, DT4, Acción #20

**Contexto:**  
La migración a Supabase es el paso más importante del roadmap — habilita el modelo SaaS de suscripción mensual. Hacer el trabajo de normalización en v1.x significa que v2.0 es un cambio de capa de persistencia, no una reescritura. Hacerlo directamente en v2.0 implica transformar datos reales de usuarios en producción: el escenario de mayor riesgo.

**Acciones requeridas:**
- Normalizar `Account.id` de number a string UUID (C-12)
- Eliminar campos muertos `balance`, `drawdown`, `pnl` de Account (C-13)
- Centralizar claves `tt_bias_[fecha]` como parte de la estructura de checklists
- Documentar el esquema SQL equivalente para Supabase en SCHEMA.md
- Agregar campo `schemaVersion` al objeto exportado para controlar migraciones futuras

**Dependencia:** C-12 y C-13 deben estar resueltos antes de ejecutar este ítem.

---

### A-05 — Backend Supabase + autenticación por magic link

**Prioridad:** 🟢 Baja (v2.0)  
**Sprint objetivo:** v2.0  
**Esfuerzo:** Alto  
**Origen:** ROADMAP v2.0

**Contexto:**  
Migrar de localStorage a PostgreSQL vía Supabase. La autenticación por magic link (sin contraseña) elimina la fricción de registro. La sincronización entre dispositivos es el argumento más fuerte para convertir compradores de $19 a suscriptores de $9/mes.

**Acciones requeridas:**
- Crear proyecto en Supabase (plan gratuito)
- Implementar autenticación por magic link (email)
- Migrar las 6 variables de estado (`cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists`) a tablas normalizadas
- Mantener `saveAll()` como interfaz pero reemplazar `localStorage.setItem` por `supabase.from(...).upsert(...)`
- Implementar sincronización automática entre dispositivos
- Backup automático en la nube (reemplaza exportación manual en JSON)

**Dependencia:** A-04 debe estar completo. El esquema SQL debe estar documentado antes de tocar código.

---

## 3. MONETIZACIÓN

> Acciones que generan o protegen ingresos directamente. Separadas de producto y arquitectura porque tienen dueño diferente (Juan Pa) y no requieren desarrollo de código en su mayoría.

---

### M-01 — Activar programa de afiliados en Gumroad

**Prioridad:** 🔴 Alta  
**Sprint objetivo:** Inmediato  
**Esfuerzo:** Muy bajo  
**Origen:** ON3

**Contexto:**  
El programa de afiliados (30% de comisión) está documentado en PROJECT_RULES y BUSINESS_VISION. Gumroad tiene programa de afiliados integrado. Lo que falta es activarlo formalmente y definir el proceso operacional.

**Acciones requeridas:**
- Activar el programa de afiliados en Gumroad (configuración en dashboard)
- Crear formulario de aplicación para afiliados (Google Forms es suficiente en esta etapa)
- Documentar el proceso de pago mensual (qué umbral mínimo, qué método)
- Preparar kit de materiales para afiliados: capturas del producto, copy sugerido, propuesta de valor
- Meta inicial: 5 afiliados activos con audiencia de traders latinoamericanos

**Nota:** Este ítem no requiere desarrollo de código. Es decisión y ejecución de Juan Pa.

---

### M-02 — Landing page con video demo

**Prioridad:** 🔴 Alta  
**Sprint objetivo:** Pre-escala  
**Esfuerzo:** Medio  
**Origen:** ON4, Acción #19

**Contexto:**  
El modelo de distribución depende de creadores de contenido. Hoy no existe una landing page. El afiliado está vendiendo un producto que su audiencia no puede ver antes de comprar. Sin destino que convierta de forma independiente, el ROI del canal de afiliados es menor al potencial.

**Acciones requeridas:**
- Página web standalone (no la app) con propuesta de valor clara
- Capturas reales del producto (dashboard, checklist, calculadora)
- Video demo de 3-5 minutos: flujo real de un trader (onboarding → checklist → registrar trade → dashboard)
- Sección de testimonios (primeros compradores)
- Botón de compra directo a Gumroad
- Hosting: puede usar el mismo dominio `tradeos.io` en la raíz

**Dependencia:** A-01 (dominio propio) facilita el hosting de la landing en el mismo dominio.

---

### M-03 — Card compartible de hito de fondeo

**Prioridad:** 🟠 Media  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** ON5, OP7

**Contexto:**  
Cuando un trader pasa de Challenge a Fondeado es el momento emocional más compartible de su carrera. Si la app genera una card con sus métricas exactas, cada trader fondeado que la comparte es publicidad orgánica de costo cero con el nombre del producto visible.

**Acciones requeridas:**
- Al completar la fase Challenge y marcar como Fondeado: mostrar modal de celebración
- Generar card con métricas del período: trades, WR, P&L, reglas cumplidas, drawdown máximo usado
- Botón "Descargar como imagen" usando `canvas` + `toDataURL`
- Incluir el nombre del producto y URL en la card (no intrusivo, como marca de agua discreta)

---

### M-04 — Licencias para academias (pre-v2.0)

**Prioridad:** 🟡 Media  
**Sprint objetivo:** Evaluación  
**Esfuerzo:** Bajo  
**Origen:** ON6

**Contexto:**  
Hay academias de trading con 50-300 alumnos que recomiendan herramientas. Una licencia de academia ($99 para 20 copias del archivo HTML) es un canal B2B que no requiere backend — solo un modelo de distribución de archivos y facturación directa. Puede probarse antes de v2.0.

**Acciones requeridas:**
- Definir modelo de precio: ej. $99 para 20 licencias, $199 para 50
- Crear proceso de venta directa (fuera de Gumroad — negociación con la academia)
- Preparar una versión del archivo con branding opcional de la academia (nombre de la academia visible en el sidebar)
- Identificar 3-5 academias target para pitch inicial

**Nota:** Requiere evaluación de Juan Pa antes de ejecutar. No hay desarrollo de código obligatorio en la fase de prueba.

---

### M-05 — Plan freemium en v2.1

**Prioridad:** 🟢 Baja (v2.1)  
**Sprint objetivo:** v2.1  
**Esfuerzo:** Alto  
**Origen:** ROADMAP v2.1

**Contexto:**  
Una vez que existe el backend (A-05), el modelo freemium permite adquisición escalable con conversión a Pro. El tier gratuito actúa como canal de adquisición; el tier Pro ($9/mes o $79/año) es el modelo de negocio principal a largo plazo.

**Acciones requeridas:**
- Definir límites del tier gratuito: 1 cuenta, 50 trades, sin patrones avanzados, sin sync cloud
- Implementar lógica de límites en el frontend (gates de features)
- Crear página de upgrade dentro de la app
- Integrar Stripe o Gumroad Memberships para pagos recurrentes
- Implementar dashboard de afiliados (30% de comisión recurrente)

**Dependencia:** A-05 (backend Supabase) debe estar completo.

---

## 4. NUEVAS FUNCIONES

> Features de producto que agregan valor real al trader. Ninguna de estas entra en producción antes de que la sección 1 (Correcciones Críticas) esté resuelta.

---

### F-01 — Edición de trades ya guardados

**Valor:** 🔴 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Medio  
**Origen:** D2, OP1, Acción #7

**Justificación:**  
La ausencia de edición es la limitación funcional más citada en journals de trading de cualquier tipo. Desincentiva el registro inmediato — el trader espera tener todos los datos perfectos antes de registrar, o evita registrar trades problemáticos.

**Especificación:**
- Botón "Editar" en cada trade de la vista lista
- Abre el formulario pre-cargado con los datos del trade existente
- Al guardar: sobreescribir el trade por ID
- Campos editables: P&L, R:R, notas, libreta, estado emocional, regla rota, tags de setup, imagen
- Campos no editables: fecha, par, sesión, cuenta (para preservar integridad histórica)
- Agregar campos `edited: true` y `editedAt: ISO` al objeto trade
- Regla: editar una madre no debe alterar los datos de las hijas (solo edita el trade madre)

---

### F-02 — Función de edición de perfil y cuentas post-onboarding

**Valor:** 🔴 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Medio  
**Origen:** D5, D9, Acción #17

**Justificación:**  
Los traders cambian de cuenta regularmente. Cierran challenges, abren cuentas nuevas, cambian de prop firm. La instrucción actual ("resetear todo y empezar de cero") es inaceptable en producción. Es la limitación estructural más grave para la retención a largo plazo.

**Especificación:**
- En Ajustes: formularios para editar nombre, riesgo%, R:R mínimo, pares operados, sesiones
- Para cuentas: agregar cuenta nueva, editar tamaño/firma de cuenta existente, archivar cuenta
- Archivar cuenta: mantiene todos los trades y datos históricos, pero la excluye de los cálculos activos y de los selectores del formulario de trade
- No confundir con "resetear" — editar no borra datos

---

### F-03 — Estadísticas por par y por sesión

**Valor:** 🔴 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Bajo  
**Origen:** OP2, Acción #9

**Justificación:**  
"¿Soy rentable en XAU/USD? ¿En qué sesión gano más?" son las preguntas más frecuentes después de "¿cuál es mi WR?". Los datos ya existen en `trades[]`. Solo requiere lógica de agrupación y renderizado.

**Especificación:**
- Tabla de métricas cruzadas: WR, P&L promedio, R:R promedio, total de trades — agrupado por par
- La misma tabla agrupada por sesión (London, New York, Asia, Overlap)
- Ubicación: subsección en Análisis de Patrones o en Mis Trades
- Los datos deben respetar los filtros de cuenta activos si existen

---

### F-04 — Alerta de drawdown diario por cuenta

**Valor:** 🔴 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Bajo  
**Origen:** OP5, Acción #15

**Justificación:**  
Las prop firms tienen reglas de drawdown diario. Si el trader pierde más de X% en un día, viola las reglas y puede perder la cuenta fondeada. Esta feature puede literalmente salvar el activo principal del trader.

**Especificación:**
- Campo configurable por cuenta: "Pérdida máxima diaria (%)" — default: 5%
- Calcular el P&L de los trades del día actual agrupado por cuenta
- Si P&L del día supera el límite configurado: alerta en el dashboard y en el formulario de registro
- No bloquear — alertar con modal de confirmación antes de guardar el siguiente trade del día
- El trader puede ignorar la alerta y guardar de todas formas (autonomía del usuario)

---

### F-05 — Filtro por rango de fechas en Mis Trades

**Valor:** 🟠 Alto  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Bajo  
**Origen:** OP3, Acción #12

**Justificación:**  
Los traders revisan su rendimiento por períodos naturales: semana, mes, challenge. No poder filtrar por fecha obliga al scroll manual en historiales largos. Es la segunda limitación de navegación más frecuente.

**Especificación:**
- Dos inputs de fecha (desde / hasta) en la barra de filtros de Mis Trades
- El filtro temporal se aplica junto con los existentes (cuenta, par, resultado)
- Botones rápidos opcionales: "Esta semana", "Este mes", "Último mes", "Este challenge"
- La vista calendario ya filtra por mes — este filtro aplica a la vista lista

---

### F-06 — Diario de mercado como sección o subsección estructurada

**Valor:** 🟠 Alto  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Medio  
**Origen:** OP4

**Justificación:**  
El bias del día actual (campo de texto libre guardado por fecha) es un embrión de esta función. Un diario estructurado convierte a TradeOS de journal de trades a sistema operativo del trader — la feature con mayor impacto en retención a largo plazo y la que más alinea el producto con su eslogan.

**Especificación:**
- Sección por fecha con campos estructurados: sesgo HTF (texto), niveles clave (texto), plan de sesión (texto), reflexión post-sesión (texto)
- Guardado por fecha en localStorage (extendiendo o reemplazando `tt_bias_[fecha]`)
- El bias actual del checklist se convierte en el campo "sesgo HTF" de esta sección
- Vista de historial: lista de entradas pasadas con fecha y primer párrafo visible
- Incluido en el backup exportado (C-09 es prerequisito)

---

### F-07 — Campos de entrada, SL y TP por trade

**Valor:** 🟠 Alto  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** OP6, Acción #18

**Justificación:**  
Con precio de entrada, SL y TP es posible calcular el R:R planeado automáticamente y compararlo con el R:R real. La diferencia revela si el trader sale temprano de sus ganadores o mueve el SL — análisis de comportamiento de alto valor que los datos actuales no capturan.

**Especificación:**
- Tres campos numéricos opcionales en el formulario de trade: precio de entrada, SL, TP
- Calcular R:R planeado automáticamente desde estos campos al rellenarlos
- Mostrar en Análisis de Patrones: R:R planeado vs R:R real (promedio histórico y por trade)
- No obligatorio — los campos son opcionales para no aumentar fricción de registro

---

### F-08 — Notas post-sesión vinculadas al temporizador

**Valor:** 🟡 Medio  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** OP8

**Justificación:**  
Cuando el trader cierra la sesión, la app ya pregunta si tiene trades para registrar. Agregar un segundo paso "¿Qué aprendiste hoy?" con texto libre guardado por fecha separa a un trader que mejora de uno que solo registra. Fricción mínima, valor de retención alto.

**Especificación:**
- Al hacer click en "Cerrar sesión" en el checklist: flujo de dos pasos
  - Paso 1: "¿Tienes trades para registrar?" (ya existe)
  - Paso 2: "Reflexión de hoy" — campo de texto libre (opcional, se puede saltear)
- La reflexión se guarda por fecha y se integra con el Diario de Mercado (F-06) si está implementado

---

### F-09 — Historial de cumplimiento del checklist

**Valor:** 🟡 Medio  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** OP9

**Justificación:**  
Los datos del checklist ya se guardan en `checklists{}`. Renderizarlos como análisis de comportamiento es análisis de alto valor con costo de implementación bajo. El trader sabe cuántos días cumplió el 100% y qué ítem omite con mayor frecuencia.

**Especificación:**
- Vista mensual: días con check completo (verde), parcial (amarillo), sin check (gris)
- Estadística: porcentaje de días con checklist completo en el mes
- Ítem más frecuentemente omitido (los últimos 30 días)
- Ubicación: subsección en el Checklist Diario o en Análisis de Patrones

---

### F-10 — Análisis de patrones con estado informativo desde el primer uso

**Valor:** 🟡 Medio  
**Sprint objetivo:** v1.1  
**Esfuerzo:** Bajo  
**Origen:** D4

**Justificación:**  
En los primeros días, la sección de Patrones muestra una pantalla vacía. Un trader nuevo que explora el producto ve una sección sin contenido — primera impresión negativa antes de que el producto demuestre su valor.

**Especificación:**
- Con 0-4 trades: mostrar los 10 patrones que el sistema detectará con una descripción breve de cada uno
- Mostrar contador: "Necesitás X trades más para activar el análisis"
- No eliminar el requisito de 5 trades — mantener la calidad del análisis
- El cambio es de pantalla vacía a pantalla informativa

---

### F-11 — Score de Consistencia con ventana temporal configurable

**Valor:** 🟡 Medio  
**Sprint objetivo:** v1.2  
**Esfuerzo:** Bajo  
**Origen:** OP10

**Justificación:**  
El score actual se calcula sobre todos los trades históricos. Un trader con 6 meses de historial tiene su score "diluido" por trades viejos. Un score calculado sobre los últimos 30 días es más relevante para el estado actual del trader.

**Especificación:**
- Agregar selector en el dashboard: "Score de los últimos: 30 días / 60 días / Todo el historial"
- La fórmula (50% WR + 50% reglas cumplidas) no cambia — solo el conjunto de trades sobre el que se calcula
- Default: 30 días (cambio de comportamiento visible — comunicar en el changelog)
- La preferencia se guarda en `cfg`

---

## RESUMEN DE PRIORIDADES

### Secuencia de ejecución recomendada

```
BLOQUE A — Antes de cualquier distribución masiva
C-01, C-02, C-03, C-04, C-05   ← bugs críticos de datos

BLOQUE B — v1.1 completo
C-06, C-07, C-08, C-09, C-10, C-11
F-01, F-02, F-03, F-04, F-05, F-10
M-03 (card de fondeo — bajo esfuerzo, alto retorno)

BLOQUE C — Pre-escala / v1.2
A-01 (hosting dominio), A-02 (PWA), A-03 (versión visible)
M-01 (activar afiliados), M-02 (landing page)
F-06, F-07, F-08, F-09, F-11

PRE-v2.0
C-12, C-13, C-14, C-15
A-04 (normalizar esquema para Supabase)

v2.0 y más allá
A-05 (backend Supabase)
M-04 (licencias academias — evaluación)
M-05 (freemium)
```

### Conteo por categoría

| Categoría | Total ítems | Críticos (🔴) |
|-----------|------------|---------------|
| Correcciones Críticas | 15 | 5 |
| Arquitectura SaaS | 5 | 1 |
| Monetización | 5 | 2 |
| Nuevas Funciones | 11 | 3 |
| **Total** | **36** | **11** |

---

*Generado a partir de EXECUTIVE_REPORT.md — Junio 2026*  
*Todos los ítems tienen trazabilidad al reporte fuente (D#, RC#, ON#, OP#, DT#, Acción #)*  
*Próxima revisión: al cierre del sprint v1.1*
