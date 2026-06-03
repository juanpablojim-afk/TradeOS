# EXECUTIVE_REPORT.md
## TradeOS — Reporte Ejecutivo de Estado del Producto

**Versión analizada:** 1.0 Beta  
**Fecha:** Junio 2026  
**Autor:** CTO — basado en auditoría directa de código + todos los documentos del proyecto  
**Fuentes:** tradeOS_base.html · MASTER_CONTEXT · SCHEMA · FUNCTIONS · LOCALSTORAGE_AUDIT · SECURITY_AUDIT · PRODUCT_RECOMMENDATIONS · TRADING_EXPERT_REVIEW · ROADMAP · TASKS · BUSINESS_VISION

---

## ESTADO ACTUAL DEL PRODUCTO

TradeOS es una aplicación web de archivo único (225 KB, HTML/CSS/JS vanilla, sin backend) distribuida a $19 USD en Gumroad. Funciona completamente offline, persiste todos los datos en localStorage del navegador, y no requiere cuenta ni registro.

**En una sola frase:** es el mejor journal de trading en español para traders de prop firms que existe hoy, pero no está listo para distribución masiva.

### Métricas técnicas verificadas en el código

| Métrica | Estado real |
|---------|-------------|
| Tamaño del archivo | 225 KB |
| Líneas de código | 3.206 |
| Funciones JS nombradas | 73 |
| Funciones duplicadas en el código | 6 (resetAll, openM, closeM, showN, toggleTheme, loadTheme) |
| Claves de localStorage | 9 distintas |
| Bugs de pérdida silenciosa de datos | 2 críticos confirmados |
| Bugs de corrupción de datos | 2 altos confirmados |
| Inconsistencias de tipo en datos | 3 identificadas |
| Código muerto (funciones no utilizadas) | 3+ funciones |
| Node.js syntax check | PASS |
| window.addEventListener('load') | 1 — no duplicado ✅ |
| Cierre de tags en JS strings | 0 sin escapar ✅ |
| Privacidad — llamadas externas con datos | 0 ✅ |

### Módulos funcionales (lo que realmente existe en el código)

| Módulo | Estado | Notas |
|--------|--------|-------|
| Onboarding (9 pasos) | ✅ Funcional | Construye cfg completo |
| Dashboard + Score de Consistencia | ✅ Funcional | Fórmula: 50% WR + 50% reglas cumplidas |
| Checklist diario + Temporizador | ✅ Funcional | 13 ítems base, personalizables |
| Registro de trade | ✅ Funcional | 12 campos capturados |
| Replicador madre/hija | ⚠️ Funcional con bug | motherId puede ser incorrecto |
| Calculadora de lotaje | ✅ Funcional | 4 pares forex + 9 futuros CME exactos |
| Mis Trades (lista + calendario) | ✅ Funcional | Resumen semanal solo en fines de semana |
| Fases del Plan | ✅ Funcional | Bug de doble prefijo en custom_ |
| Gestión de Fondos | ✅ Funcional | 7 tipos de movimiento |
| Análisis de Patrones | ✅ Funcional | 10 patrones, requiere 5+ trades |
| Vista de Cuentas | ✅ Funcional | P&L y drawdown en tiempo real |
| Backup JSON export/import | ⚠️ Funcional con bug | importD no restaura checklists |
| Export CSV de trades | ⚠️ Funcional con bug | Muestra ID de cuenta, no nombre |
| Sistema bilingüe ES/EN | ✅ Funcional | |
| Modo oscuro + modo claro | ✅ Funcional | |

---

## FORTALEZAS

### F1 — Diferenciación técnica real en el mercado

La calculadora de lotaje exacta usando el **balance real** (no el tamaño original) para cuentas con pérdidas acumuladas es la implementación correcta. Ningún competidor en español la hace así. Los valores de tick exactos para 9 futuros CME (NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL) son verificados y correctos. Para un trader que opera futuros CME desde Latinoamérica, esta calculadora no tiene sustituto accesible en español.

### F2 — Único producto que maneja la realidad de las prop firms latinoamericanas

Múltiples cuentas simultáneas de distintas firmas, drawdown en tiempo real por cuenta, replicador madre/hija integrado, fases de challenge/fondeo/escalado. Ningún competidor genérico (Tradezella, Edgewonk) modela este caso de uso. Para el ICP exacto del producto, no hay alternativa equivalente.

### F3 — Privacidad y autonomía absolutas

Cero llamadas externas con datos del usuario. Cero tracking. Cero telemetría. Funciona completamente offline después de la carga inicial de fuentes. El trader es el único dueño de sus datos. En un mercado donde los traders son habitualmente paranoicos con compartir resultados, esto es una ventaja de posicionamiento poderosa.

### F4 — Arquitectura que escala sin necesitar refactors completos

El estado global centralizado en 6 variables (`cfg`, `trades`, `phases`, `completedPh`, `funds`, `checklists`) con persistencia unificada en `saveAll()` es limpio y predecible. La función `sv()` como único punto de navegación es correcta. Agregar features nuevas al sistema actual tiene fricción baja.

### F5 — Onboarding de 9 pasos que personaliza toda la experiencia

El onboarding captura 25+ variables del trader y las usa para personalizar el checklist, los selectores de pares, las sesiones, la calculadora y las alertas. Ningún competidor hace personalización de esta profundidad sin crear una cuenta. Es un argumento de conversión fuerte.

### F6 — Precio de entrada sin fricción

$19 USD, pago único, descarga inmediata, sin registro, sin suscripción. Para el ICP (trader latinoamericano frustrado con herramientas de $30-50/mes en inglés), esta propuesta es obvia. El modelo freemium futuro tiene una base sólida de compradores satisfechos para convertir.

### F7 — Documentación del proyecto a un nivel poco común para un producto en beta

Todos los roles críticos tienen documentación estructurada: MASTER_CONTEXT, PROJECT_RULES, UI_GUIDELINES, SCHEMA, FUNCTIONS, LOCALSTORAGE_AUDIT, SECURITY_AUDIT, PRODUCT_RECOMMENDATIONS, TRADING_EXPERT_REVIEW. Esto permite incorporar colaboradores y agentes de IA sin pérdida de contexto.

---

## DEBILIDADES

### D1 — Dos bugs críticos de pérdida silenciosa de datos en producción

`saveAll()` no captura `QuotaExceededError`. Cuando localStorage se llena (inevitable con uso normal de imágenes), el trade "se guarda" con toast de confirmación pero no persiste. El usuario no lo sabe. Este es el escenario más dañino para la reputación del producto.

`importD()` no restaura el historial de checklist (`tt_cl`). La línea simplemente no existe en el código. Todo usuario que migra de dispositivo o restaura un backup pierde su historial de disciplina diaria.

### D2 — No se pueden editar trades ya guardados

Una de las quejas más frecuentes de traders en cualquier journal. Si el trader registra un P&L equivocado o quiere actualizar el R:R real después de revisar, debe borrar el trade y volver a registrarlo. Esto rompe el flujo de uso diario y desincentiva el registro inmediato.

### D3 — Resumen semanal solo visible sábados y domingos

Un guard deliberado en `renderWeeklySummary()` oculta el panel de lunes a viernes. El resumen semanal es exactamente la información que el trader necesita revisar el lunes antes de operar — no solo el fin de semana. Este es un fallo de UX con impacto en la utilidad diaria.

### D4 — El análisis de patrones requiere 5+ trades para mostrar algo

En los primeros días de uso, la sección de patrones está vacía. Un trader nuevo que compra el producto y explora todas las secciones ve una pantalla vacía en Patrones. Primera impresión negativa que puede generar abandono temprano antes de que el producto demuestre su valor.

### D5 — Sin función de edición de cuentas post-onboarding

Para editar los datos de una cuenta (tamaño, drawdown máximo, nombre de la firma), el manual de usuario dice "resetear todo y empezar de cero". Esto es inaceptable en producción. Los traders cambian de cuenta regularmente — cierran challenges, abren cuentas nuevas, cambian de prop firm.

### D6 — El CSV exportado muestra IDs de cuenta, no nombres

La columna "Cuenta" del CSV exportado contiene `"1"`, `"2"`, `"3"` en lugar de `"Funding Pips"`, `"FTMO"`, `"E8"`. Un archivo de datos que requiere otra tabla para interpretarse no es un archivo de datos — es basura para el análisis externo.

### D7 — Crecimiento ilimitado de `checklists` en localStorage

Cada día de uso agrega una entrada nueva. No hay limpieza automática. A 73 KB/año, en 3-4 años de uso diario contribuye significativamente al límite de localStorage. No es crítico hoy pero debe planificarse antes de v2.0.

### D8 — 6 funciones declaradas dos veces en el código

`resetAll()`, `openM()`, `closeM()`, `showN()`, `toggleTheme()`, `loadTheme()` están declaradas dos veces. La segunda sobrescribe a la primera en runtime sin error. No causa bugs hoy, pero es deuda de mantenimiento que confunde al leer el código.

### D9 — No existe función de edición de perfil

Para cambiar el nombre, el riesgo por trade, o los pares operados después del onboarding, el camino actual es Ajustes → Resetear todo. El trader pierde todos sus datos para cambiar un campo. La única edición post-onboarding habilitada es el nombre (en el modal de zona del dashboard) y la fase/DD de cuentas.

---

## RIESGOS CRÍTICOS

### RC1 — Pérdida masiva de datos en el usuario que usa imágenes

**Probabilidad:** Alta. **Impacto:** Crítico para el negocio.

El flujo es: trader sube 30+ capturas de gráficos (uso esperado del producto) → localStorage se satura → `saveAll()` falla sin manejo → trades se pierden con toast de "guardado ✓" → trader descubre la pérdida días después → el relato en redes sociales destruye la reputación del producto antes de que escale.

En un modelo de distribución basado en recomendaciones orgánicas de traders, un solo caso documentado públicamente de "TradeOS me borró 3 semanas de trades" puede costar decenas de ventas futuras. El mercado es pequeño y habla mucho.

### RC2 — Safari iOS bloqueado

El 40-60% del tráfico móvil en Latinoamérica usa iOS. Safari en iOS bloquea localStorage para archivos locales. La app no funciona en iPhone/iPad en su modo de distribución actual (archivo HTML descargable). Esto excluye a una parte significativa del ICP hasta que se resuelva el hosting.

### RC3 — Eliminación de madre deja réplicas huérfanas que corrompen métricas

Un trader con replicador activo que borra un trade malo contamina sus propias estadísticas. Win rate, P&L, drawdown por cuenta y análisis de patrones muestran números incorrectos de forma acumulativa. Las decisiones de riesgo del trader están basadas en datos corruptos. Este escenario ocurre en el caso de uso central del producto.

### RC4 — Sin mecanismo de actualización para compradores de Gumroad

Cuando se lance v1.1 con los bugs críticos corregidos, no hay forma automatizada de notificar a los compradores de v1.0 que existe una versión más segura. Cada comprador tiene que saber buscar la actualización manualmente. Los usuarios con bugs no corregidos siguen en riesgo de pérdida de datos indefinidamente.

### RC5 — Un solo archivo HTML editable — riesgo de corrupción irreversible

El código fuente completo en un solo archivo sin versionado integrado (sin Git, sin proceso de build) significa que un error de edición sin backup puede destruir el producto. No hay rollback automático ni separación de preocupaciones. El riesgo aumenta con cada persona que toca el archivo.

---

## DEUDA TÉCNICA

### DT1 — Crítica: `saveAll()` sin manejo de error
Una función llamada decenas de veces por sesión de usuario, que escribe el activo más valioso del producto (los datos del trader), sin ningún manejo de excepciones. Costo de resolver: bajo. Riesgo de no resolver: máximo.

### DT2 — Alta: Funciones duplicadas en el código
Seis funciones declaradas dos veces ocupan espacio y generan confusión. Cualquier modificación a una de ellas debe hacerse en dos lugares, con riesgo de que una versión quede desincronizada. Costo de resolver: trivial.

### DT3 — Alta: Tipos inconsistentes en datos
`Account.id` es `number`. `trade.account` es `string`. Se comparan con `String(t.account) === String(a.id)`. Funciona hoy, pero en una futura migración a Supabase (donde los tipos son estrictos) esto requiere una migración de datos explícita. Normalizar antes de v2.0 es obligatorio.

### DT4 — Alta: Campos muertos en el objeto Account
`Account.balance`, `Account.drawdown` y `Account.pnl` se inicializan en el onboarding y nunca se actualizan. El P&L real se calcula en runtime. Estos tres campos ocupan espacio en localStorage, confunden a quien lee el esquema, y son una trampa para futuras features que podrían leer el valor almacenado (incorrecto) en lugar de calcularlo.

### DT5 — Media: `checklists` crece sin límite
Sin TTL ni limpieza automática, el historial de checklist crece 73 KB por año de uso. En v2.0 con backend, esto es irrelevante. En v1.x con localStorage, debe implementarse una limpieza de entradas de más de 18 meses.

### DT6 — Media: Bias del día fuera del sistema de backup
Las claves `tt_bias_[fecha]` son claves independientes de localStorage que no pasan por `saveAll()` ni se incluyen en el backup. El historial de notas de análisis pre-sesión del trader se pierde en cualquier migración. Incluirlo en el backup requiere incluir todas las claves `tt_bias_*` en `exportD()`.

### DT7 — Media: ID de fases personalizadas con doble prefijo `custom_`
El bug de `'ph_custom_custom_[timestamp]'` no causa errores visibles hoy, pero si se construyen features que navegan por ID de fases (profundización de analytics, exportación de progreso), el doble prefijo creará inconsistencias. Limpiar antes de v1.1.

### DT8 — Baja: Código muerto en scope global
`buildRuleSelect()` usa `<select id="t-rule">` pero la UI actual usa chips `#t-rule-chips`. La primera versión de `toggleTag(tag, btn)` fue reemplazada por la versión actual. `toggleReplicator()` es un panel inline que el flujo actual nunca activa. Tres funciones que nunca se llaman en uso normal.

### DT9 — Baja: `motherId` puede apuntar al trade anterior
En `applyReplicator()`, el `motherId` se asigna como `trades[0]?.id` antes de que `saveTrade()` guarde el trade madre. En ese momento, `trades[0]` es el penúltimo trade, no el madre actual. No hay features que naveguen de hija a madre hoy, pero cuando se construya esa navegación, el bug se manifestará.

---

## OPORTUNIDADES DE NEGOCIO

### ON1 — Hosting en dominio propio: desbloquea iOS y legitima el producto

Hospedar en `app.tradeos.io` (costo: ~$10-15/año de dominio + Netlify gratis) resuelve Safari iOS, permite compartir un link en lugar de un archivo HTML, hace el producto más profesional y credible para afiliados, y es el primer paso hacia la versión web/PWA. ROI inmediato y muy alto para costo casi nulo.

### ON2 — Mecanismo de actualización para compradores Gumroad

Gumroad permite enviar emails a todos los compradores de un producto. Implementar un flujo de "versión actualizada disponible" para cada release relevante convierte a los compradores de $19 en una base de usuarios recurrentes que pueden actualizar a Pro cuando exista el plan. Sin este mecanismo, esa base se fragmenta en versiones distintas y el canal de upgrades se pierde.

### ON3 — Afiliados activos como canal principal — la infraestructura está lista

El programa de afiliados (30% de comisión) está documentado en PROJECT_RULES y BUSINESS_VISION. Lo que no está es la infraestructura mínima: un formulario de aplicación, un dashboard simple de seguimiento de referidos, y un proceso de pago mensual. Formalizar esto en Gumroad (que ya tiene programa de afiliados integrado) puede activar el canal en días, no semanas.

### ON4 — Landing page con demo en video antes de escalar distribución

El modelo de distribución depende de creadores de contenido que muestran el producto. Hoy no existe una landing page con capturas reales, video demo y testimonios. Antes de activar afiliados en escala, necesita un destino al que mandar tráfico que convierta de forma independiente al creador.

### ON5 — El momento de fondeo como contenido viral orgánico

Cuando un trader pasa de Challenge a Fondeado es el momento emocional más compartible de su carrera reciente. Si la app captura ese hito con sus métricas exactas y genera una card compartible ("Pasé mi challenge con 67% de WR y sin romper una sola regla — TradeOS"), cada trader fondeado que comparte esa card es publicidad orgánica de costo cero.

### ON6 — Licencias para academias antes de v2.0

Hay academias de trading que tienen 50-300 alumnos a los que recomiendan herramientas. Una licencia de academia (ej: $99 para 20 licencias) es un canal B2B que no requiere backend — solo un modelo de distribución de archivos y facturación directa. Puede probarse antes de que exista el plan Teams en v2.0.

---

## OPORTUNIDADES DE PRODUCTO

### OP1 — Edición de trades ya guardados

La ausencia de edición es la limitación funcional más citada en journals de trading de cualquier tipo. Un modal de edición que permita modificar P&L, R:R, notas y libreta de un trade existente convierte el journal de "registro inmutable" a "herramienta de reflexión". Impacto en retención: alto. Complejidad: media.

### OP2 — Estadísticas por par y por sesión

El trader quiere saber: "¿Soy rentable en XAU/USD? ¿En qué sesión gano más?". Los datos ya están en `trades[]`. Construir una tabla de métricas cruzadas (par × sesión × resultado) es análisis que el trader haría manualmente en Excel — si la app lo da directamente, elimina una tarea de su flujo semanal. Complejidad: baja.

### OP3 — Filtro por rango de fechas en Mis Trades

El trader revisa su mes. El trader revisa su challenge. El trader revisa "esta semana". El calendario ya existe pero no puede filtrar la vista lista. Un filtro de fecha desde/hasta en Mis Trades es la segunda pregunta más frecuente después de "¿cuál es mi WR?". Complejidad: baja.

### OP4 — Diario de mercado como sección independiente

Un campo de texto largo estructurado (sesgo HTF, niveles clave, plan de sesión, reflexión post-sesión) guardado por fecha convierte a TradeOS de journal de trades a sistema operativo del trader. Es la feature que más alinea el producto con su eslogan y la que más diferencia puede hacer en retención a largo plazo. El bias del día actual es un embrión de esto.

### OP5 — Alerta de drawdown diario

Las prop firms tienen reglas de drawdown diario (no solo total). Si el trader pierde más de X% en un día, viola las reglas. TradeOS puede calcular el P&L del día actual y alertar cuando se acerca al límite configurado. Esta sola feature puede evitar que un trader pierda una cuenta fondeada — ese valor es inmediato y concreto.

### OP6 — Precio de entrada, SL y TP por trade

Campos opcionales que permiten calcular el R:R planeado y compararlo con el R:R real. La diferencia entre R:R planeado y R:R real muestra si el trader sale temprano de sus ganadores o mueve el SL. Es análisis de comportamiento de alto valor que los datos actuales no capturan. Complejidad: baja.

### OP7 — Hito de fondeo con card compartible

Al completar la fase Challenge y marcar como Fondeado, la app muestra una card con las métricas del período: trades, WR, P&L, reglas cumplidas, drawdown máximo usado. La card se puede descargar como imagen. Contenido orgánico integrado al flujo natural del producto.

### OP8 — Notas post-sesión vinculadas al temporizador

Cuando el trader cierra la sesión (click en "Cerrar sesión" en el checklist), actualmente se le pregunta si tiene trades para registrar. Agregar un segundo paso: "¿Qué aprendiste hoy?" con un campo de texto libre guardado por fecha. Esta reflexión diaria es lo que separa a un trader que mejora de uno que repite errores.

### OP9 — Historial de cumplimiento del checklist

Una vista mensual que muestra: ¿cuántos días cumpliste el 100% del checklist? ¿Qué ítem omitiste con más frecuencia? Los datos ya se guardan en `checklists{}`. Renderizarlos como análisis de comportamiento es análisis de alto valor con costo de implementación bajo.

### OP10 — Score de Consistencia con ventana temporal configurable

Actualmente el score se calcula sobre todos los trades históricos. Un trader con 6 meses de historia tiene su score "diluido" por trades viejos. Un score calculado sobre los últimos 30 días es más relevante para el estado actual. La fórmula (50% WR + 50% reglas cumplidas) ya está documentada — solo necesita parámetro de ventana.

---

## TOP 20 ACCIONES RECOMENDADAS

Criterios de priorización: impacto en confianza del usuario > impacto en negocio > complejidad de implementación.

---

### #1 — Corregir `saveAll()` con manejo de `QuotaExceededError`

**Tipo:** Bug crítico · **Sprint:** v1.1 Bloque A · **Esfuerzo:** Medio

Envolver `saveAll()` en try/catch. Si falla, mostrar modal bloqueante (no toast) con instrucciones claras: "Tu almacenamiento está lleno. Exportá un backup y eliminá algunas fotos de trades para continuar." Agregar barra de uso de localStorage en Ajustes con alerta proactiva al 80%.

**Por qué es #1:** Es el único punto del sistema donde la app miente activamente al usuario. Un trader que pierde datos sin saberlo y lo descubre públicamente destruye el canal de recomendaciones orgánicas.

---

### #2 — Comprimir imágenes antes de guardar en `prevImg()`

**Tipo:** Bug crítico · **Sprint:** v1.1 Bloque A · **Esfuerzo:** Bajo

Usar `canvas.toDataURL('image/jpeg', 0.6)` para comprimir antes de asignar a `imgB64`. Limitar el ancho máximo a 1200px antes de comprimir. Mostrar el tamaño estimado de la imagen en el formulario. Con compresión, pasar de 200 KB promedio a ~30-50 KB — multiplicando por 4-6 la capacidad de imágenes antes de saturar localStorage.

**Por qué es #2:** Resuelve la causa raíz de RC1. Sin compresión, el manejo de error en `saveAll()` (#1) se activará en el uso normal del producto.

---

### #3 — Corregir `importD()` — restaurar `tt_cl` (checklists)

**Tipo:** Bug confirmado · **Sprint:** v1.1 Bloque A · **Esfuerzo:** Trivial

Agregar una línea en `importD()`:
```javascript
if(d.checklists) localStorage.setItem('tt_cl', JSON.stringify(d.checklists));
```
Sin esta línea, todo usuario que migra de dispositivo pierde su historial de disciplina. Es un bug confirmado que ya existe en el código.

**Por qué es #3:** Una línea de código, impacto máximo en confianza del usuario.

---

### #4 — Validación de esquema y rollback en `importD()`

**Tipo:** Bug crítico · **Sprint:** v1.1 Bloque A · **Esfuerzo:** Medio

Antes de escribir: snapshot del estado actual en memoria. Validar que `d.cfg` sea objeto con `d.cfg.name` como string, que `d.trades` sea array, que `d.funds` sea array. Si alguna validación falla: no escribir nada + mostrar error específico + mantener snapshot. Si el proceso falla a mitad: rollback automático al snapshot.

**Por qué es #4:** La herramienta de recuperación de datos no puede ser ella misma un punto de destrucción de datos.

---

### #5 — Corregir `delTrade()` — eliminar réplicas huérfanas

**Tipo:** Bug crítico · **Sprint:** v1.1 Bloque A · **Esfuerzo:** Bajo

Al eliminar una madre (`isMother: true`), buscar todos los trades con `motherId === id` y eliminarlos en la misma operación. Mostrar modal de confirmación: "Este trade tiene X réplicas en otras cuentas. ¿Eliminar todas?" Dar opción de eliminar solo la madre (dejando hijas como independientes con `isChild: false`) o eliminar todo el grupo.

**Por qué es #5:** Afecta directamente al caso de uso más diferenciador del producto (replicador multi-cuenta). Métricas corruptas en el dashboard son inaceptables.

---

### #6 — Hospedar en dominio propio (app.tradeos.io)

**Tipo:** Infraestructura · **Sprint:** v1.2 · **Esfuerzo:** Muy bajo

Registrar dominio + configurar Netlify/Vercel con el archivo HTML. Resolver Safari iOS instantáneamente. Eliminar la fricción de "descargar un archivo HTML" del proceso de ventas. Hacer el producto compartible por link. Habilitar PWA en el siguiente paso.

**Por qué es #6:** Desbloquea iOS (40-60% del mercado móvil latinoamericano), profesionaliza el producto, y es el prerequisito para todas las features web futuras. Costo casi nulo.

---

### #7 — Implementar edición de trades guardados

**Tipo:** Feature core · **Sprint:** v1.1 · **Esfuerzo:** Medio

Botón "Editar" en cada trade de la vista lista. Abre el formulario pre-cargado con los datos del trade. Al guardar, sobreescribe el trade por id. Validar que editar una madre no altere la libreta de las hijas. Registrar que fue editado (campo `edited: true`, `editedAt: ISO`).

**Por qué es #7:** Es la limitación funcional más citada en cualquier journal de trading. Su ausencia desincentiva el registro inmediato (el trader espera tener todos los datos perfectos antes de registrar).

---

### #8 — Mecanismo de actualizaciones para compradores Gumroad

**Tipo:** Negocio · **Sprint:** Inmediato · **Esfuerzo:** Muy bajo

Documentar el proceso de notificación en Gumroad (email a compradores existentes con link a versión actualizada). Implementar número de versión visible en la UI (`v1.0.x` en el sidebar o en Ajustes). Crear política de actualizaciones gratuitas para compradores de v1.x.

**Por qué es #8:** Sin este mecanismo, los compradores de v1.0 con bugs críticos no saben que existe una versión segura. La reputación del producto depende de que los usuarios activos tengan la versión más estable.

---

### #9 — Estadísticas por par y por sesión

**Tipo:** Feature core · **Sprint:** v1.1 · **Esfuerzo:** Bajo

En la sección Patrones o en una subsección de Mis Trades, agregar tabla de métricas cruzadas: WR, P&L promedio, R:R promedio y total de trades por par. Lo mismo por sesión. Los datos ya están en `trades[]`. Solo requiere lógica de agrupación y renderizado.

**Por qué es #9:** Es la segunda pregunta que todo trader se hace después de "¿cuál es mi WR global?". El trader de prop firms quiere saber si su edge funciona en XAU/USD de London o solo en NY. Esta información existe pero no se muestra.

---

### #10 — Corregir CSV: mostrar nombre de firma, no ID de cuenta

**Tipo:** Bug menor · **Sprint:** v1.1 Bloque B · **Esfuerzo:** Trivial

En `exportTradesCSV()`, cruzar `t.account` con `cfg.accounts` para obtener `a.firm` antes de incluirlo en el CSV. Una línea de lógica. El CSV pasa de ser inutilizable a ser un archivo de análisis standalone.

**Por qué es #10:** El CSV es la promesa de portabilidad de datos del producto. Que no funcione correctamente es una señal de falta de cuidado que el trader percibe y menciona.

---

### #11 — Resumen semanal visible todos los días (no solo fines de semana)

**Tipo:** Fix UX · **Sprint:** v1.1 · **Esfuerzo:** Trivial

Eliminar el guard `if(now.getDay()!==0&&now.getDay()!==6)` en `renderWeeklySummary()`. El trader que revisa su estado el lunes por la mañana necesita ver el resumen de la semana anterior, no un panel oculto.

**Por qué es #11:** Una línea de código elimina una fricción cotidiana que afecta a todos los usuarios activos de lunes a viernes.

---

### #12 — Filtro por rango de fechas en Mis Trades

**Tipo:** Feature UX · **Sprint:** v1.1 · **Esfuerzo:** Bajo

Agregar dos inputs de fecha (desde / hasta) en la barra de filtros de Mis Trades. Aplicar el filtro temporal junto con los existentes (cuenta, par, resultado). Opcionalmente, botones rápidos: "Esta semana", "Este mes", "Último mes".

**Por qué es #12:** Los traders revisan su rendimiento por períodos naturales (semana, mes, challenge). No poder filtrar por fecha obliga al scroll manual en historiales largos.

---

### #13 — Incluir bias del día en backup exportado

**Tipo:** Bug / Feature · **Sprint:** v1.1 · **Esfuerzo:** Bajo

En `exportD()`, iterar sobre todas las claves de localStorage que empiecen con `tt_bias_` e incluirlas en el JSON de backup. En `importD()`, restaurar esas claves. El historial de notas de análisis del trader deja de perderse en migraciones.

**Por qué es #13:** El bias del día es el diario de análisis del trader. Perderlo al cambiar de computadora es perder meses de reflexión. Es un bug de diseño que tiene solución directa.

---

### #14 — Detección de modo incógnito con advertencia al cargar

**Tipo:** Prevención · **Sprint:** v1.1 Bloque B · **Esfuerzo:** Bajo

Al cargar la app, intentar escribir un byte en localStorage. Si falla o si la cuota disponible es cero, mostrar un banner prominente: "Estás en modo incógnito. Los datos no se guardarán al cerrar esta ventana." No bloquear el uso — informar y dejar la decisión al trader.

**Por qué es #14:** Un trader que registra una sesión entera en incógnito y cierra el navegador pierde todo. No es frecuente, pero es devastador cuando ocurre.

---

### #15 — Alerta de drawdown diario por cuenta

**Tipo:** Feature core · **Sprint:** v1.1 · **Esfuerzo:** Bajo

Agregar campo configurable por cuenta: "Pérdida máxima diaria (%)" con valor default de 5%. Calcular el P&L de los trades del día actual por cuenta. Si supera el límite configurado, mostrar alerta en el dashboard y en el formulario de registro. No bloquear — alertar con modal de confirmación antes de guardar el siguiente trade.

**Por qué es #15:** Las prop firms tienen reglas de drawdown diario. Es el error más caro que puede cometer un trader fondeado. Esta feature puede literalmente salvar una cuenta fondeada.

---

### #16 — Eliminar funciones duplicadas y código muerto

**Tipo:** Deuda técnica · **Sprint:** v1.1 Bloque C · **Esfuerzo:** Trivial

Eliminar la segunda declaración de `resetAll()`, `openM()`, `closeM()`, `showN()`, `toggleTheme()`, `loadTheme()`. Eliminar o deprecar `buildRuleSelect()`, la primera versión de `toggleTag()`, y `toggleReplicator()`. Reducir el archivo en ~2-3 KB y eliminar confusión de mantenimiento.

**Por qué es #16:** El costo es nulo. La deuda acumulada de código muerto y duplicado hace cada edición futura más riesgosa.

---

### #17 — Función de edición de perfil y cuentas post-onboarding

**Tipo:** Feature core · **Sprint:** v1.1 · **Esfuerzo:** Medio

En la sección Ajustes, agregar formularios para editar: nombre, riesgo%, R:R mínimo, pares operados, sesiones. Para cuentas: agregar cuenta nueva, editar tamaño/firma de cuenta existente, archivar cuenta (mantener datos pero excluir de cálculos activos). Sin esto, el trader que cambia de prop firm o cierra una cuenta no tiene un flujo correcto.

**Por qué es #17:** La imposibilidad de editar el perfil sin resetear todo es la limitación estructural más grave para la retención a largo plazo.

---

### #18 — Precio de entrada, SL y TP como campos opcionales en el trade

**Tipo:** Feature · **Sprint:** v1.2 · **Esfuerzo:** Bajo

Agregar tres campos numéricos opcionales al formulario: precio de entrada, SL y TP. Calcular automáticamente el R:R planeado desde estos campos. Mostrar en el análisis de patrones la diferencia entre R:R planeado y R:R real. El análisis de si el trader sale temprano de sus ganadores es valioso y no está disponible hoy.

**Por qué es #18:** Convierte el journal de registro a herramienta de diagnóstico de comportamiento. Datos que el trader no tiene en ningún otro lugar.

---

### #19 — Landing page profesional con video demo

**Tipo:** Marketing / Negocio · **Sprint:** Pre-escala · **Esfuerzo:** Medio

Página web standalone (no la app) con: propuesta de valor clara, capturas reales del producto, video demo de 3-5 minutos mostrando el flujo real de un trader (onboarding → checklist → registrar trade → dashboard), testimonios de los primeros compradores, botón de compra directo a Gumroad. Es el destino al que los afiliados envían su audiencia.

**Por qué es #19:** El modelo de distribución depende de afiliados. Sin una landing page que convierta de forma independiente, el afiliado está vendiendo con sus propias palabras un producto que la audiencia no puede ver antes de comprar.

---

### #20 — Preparar esquema de datos para migración a Supabase

**Tipo:** Deuda técnica estratégica · **Sprint:** Pre-v2.0 · **Esfuerzo:** Alto

Normalizar los tipos antes de v2.0: `Account.id` de number a string UUID, `trade.account` ya es string (OK), eliminar campos muertos (`balance`, `drawdown`, `pnl` en Account), centralizar `tt_bias_*` como parte de `checklists` o estructura separada, documentar el esquema SQL equivalente para Supabase. Sin este trabajo previo, la migración a cloud requerirá transformaciones de datos en producción sobre datos reales de usuarios — el escenario más riesgoso posible.

**Por qué es #20:** La migración a Supabase es el step más importante del roadmap (habilita el modelo SaaS de $9/mes). Hacer el trabajo sucio ahora, en v1.1, significa que v2.0 es un cambio de capa de persistencia, no una reescritura.

---

## SÍNTESIS EJECUTIVA

TradeOS tiene un producto real con diferenciación genuina. La calculadora de lotaje exacta para futuros CME, el replicador madre/hija y el seguimiento de drawdown multi-cuenta son ventajas que ningún competidor en español tiene resueltas. El ICP está definido con precisión y el precio de entrada es correcto.

El problema no es el producto — es la estabilidad de los datos. Dos bugs críticos de pérdida silenciosa están activos en producción. Si un trader pierde datos sin saberlo y lo documenta públicamente, el canal de distribución basado en recomendaciones orgánicas se daña en un mercado pequeño y vocal.

**La distancia entre el producto actual y un producto distribuible con confianza son 7 bugs y fixes (acciones #1-#5, #10, #11). Estimación: 2-3 semanas de desarrollo.**

**La distancia entre el producto distribuible y un producto que justifica $9/mes de suscripción son 5 features (edición de trades, stats por par/sesión, alerta de drawdown diario, diario de mercado, importación MT5). Estimación: 2-3 meses de desarrollo.**

**La distancia entre el producto actual y $40,000 USD/mes en MRR (meta de visión a 3 años) son: hospedar en dominio propio + mecanismo de updates + landing page + 5 afiliados activos + lanzar v2.0 con Supabase. Estimación: 12-18 meses de ejecución consistente.**

El camino está claro. La ejecución es lo que determina el resultado.

---

*Reporte generado por CTO — TradeOS*  
*Fecha: Junio 2026*  
*Próxima revisión: al cierre del sprint v1.1*  
*Para Juan Pa: las acciones #1-#5 son no negociables antes de cualquier campaña de distribución*  
*Para el equipo: empezar por los bugs, no por las features*
