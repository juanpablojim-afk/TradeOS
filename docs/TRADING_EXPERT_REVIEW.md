# TRADING_EXPERT_REVIEW.md
## TradeOS — Evaluación desde la Perspectiva del Trader Real

**Autor:** Trading Expert  
**Fecha:** Junio 2026  
**Versión evaluada:** 1.0 Beta  
**Fuentes analizadas:** MASTER_CONTEXT.md · BUSINESS_VISION.md · SCHEMA.md · FUNCTIONS.md · LOCALSTORAGE_AUDIT.md · SECURITY_AUDIT.md · PRODUCT_RECOMMENDATIONS.md  
**Perfil del evaluador:** Trader rentable con experiencia en prop firms (Funding Pips, FTMO, E8), opera XAU/USD y NQ en London Open y New York, múltiples cuentas con replicador MT5.

> Este documento no es una revisión técnica — es una evaluación de valor real para el trader que usa esto todos los días. La pregunta que guía cada sección: **¿Esto me hace mejor trader o simplemente me da más trabajo?**

---

## 1. Funciones que Aportan Valor Real

Estas son las funciones que, como trader activo en prop firms, usaría todos los días sin que nadie me lo pidiera.

---

### 🥇 VALOR CRÍTICO — Diferencial real vs competidores

#### 1.1 Calculadora de Lotaje con Balance Real

Esta es la función más valiosa del producto. Y lo digo en serio.

La mayoría de journals calculates mal el lotaje. Usan el `account.size` original, no el balance real después de pérdidas. Si tengo una cuenta de $10,000 y perdí $400, mi balance real es $9,600. Si calculo el 1% de riesgo sobre $10,000 en lugar de $9,600, estoy arriesgando $100 cuando debería arriesgar $96. Parece insignificante. En un drawdown de $1,500, la diferencia empieza a importar y puede ser la diferencia entre pasar o reprobar un challenge.

TradeOS calcula sobre el **balance real = size + pnl acumulado negativo**. Eso es correcto. Eso es lo que hace un trader disciplinado.

La cobertura de futuros CME (NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL) con valores de tick exactos es otro diferencial. La mayoría de herramientas en español ni siquiera saben que NQ tiene un valor diferente a ES. Un trader de NQ que usa una calculadora genérica está arriesgando más de lo que cree.

**Mantener esto intacto. Es el núcleo.**

#### 1.2 Replicador de Trades (Madre/Hija)

Cualquier trader con 3+ cuentas activas en prop firms entiende el dolor de registrar el mismo trade 3 veces con P&Ls diferentes (porque los tamaños de cuenta son distintos). El concepto madre/hija es exactamente correcto.

La libreta solo en la madre también tiene sentido — el análisis del trade es uno, no uno por cuenta.

**Problema técnico confirmado (SECURITY_AUDIT A4-A5):** El `motherId` apunta al trade anterior, y eliminar una madre deja hijas huérfanas que contaminan las métricas. Esto es un bug crítico para un trader de prop firms con replicador porque **usa el replicador en casi cada trade**. Las métricas de drawdown y win rate son lo que determina si pasa o pierde una cuenta. Datos corruptos aquí = trader tomando decisiones basadas en números incorrectos.

**Prioridad de fix: máxima.**

#### 1.3 Seguimiento de Drawdown por Cuenta en Tiempo Real

La barra de drawdown por cuenta con colores (verde/amarillo/rojo) y la alerta cuando supera el 70% del máximo configurado es exactamente lo que un trader de prop firms necesita ver primero cada mañana.

Las prop firms tienen reglas de drawdown distintas: algunos son trailing (Funding Pips), otros son estáticos (FTMO en ciertos planes). Tener visible el % usado vs el máximo configurado por cuenta evita el error más caro que existe: reventar una cuenta por no llevar la cuenta del drawdown.

El soporte para múltiples cuentas de distintas firmas en el mismo lugar resuelve el problema real: hoy tengo Funding Pips $25K, E8 $50K, y FTMO $10K activas simultáneamente. Ninguna otra herramienta en español maneja esto bien.

#### 1.4 Estado Emocional por Trade

El campo de mood (Tranquilo/Confiado/Ansioso/Presionado/Cansado/Revenge) es una de las cosas más importantes que puede registrar un trader, y la mayoría lo ignora hasta que tiene una racha de pérdidas y no sabe por qué.

La detección automática de patrones que cruza mood con resultado es correcta en concepto. Si el sistema detecta "revenge trading en 2 de los últimos 5 trades", eso es exactamente la alerta que un trader necesita antes de destruir su cuenta.

**Hay un problema:** el campo mood actual tiene solo 6 opciones y no captura contexto suficiente. "Ansioso" puede ser por el mercado, por noticias económicas, por problemas personales, o porque el día anterior perdiste $800. El análisis de patrones necesita más granularidad para ser realmente útil. Pero el fundamento es correcto.

#### 1.5 Checklist Diario con Temporizador de Sesión

Los 13 ítems predeterminados cubren lo fundamental: sueño, drawdown, sesgo del día, noticias, setup definido, lotaje calculado. Un trader disciplinado revisa exactamente eso antes de operar.

El temporizador de sesión que pregunta al cerrar "¿hay trades para registrar?" es un detalle de UX que tiene valor real. Fuerza al trader a no evadir el journal.

El bias del día (campo libre por fecha) es útil para el análisis HTF pre-sesión. Un trader serio escribe su sesgo antes de ejecutar — "hoy espero continuación bajista en XAU después de barrida de liquidez en H4" — y después verifica si su lectura fue correcta.

**Problema:** el bias del día no se incluye en el backup (bug confirmado en LOCALSTORAGE_AUDIT). Eso es inaceptable. El historial de bias es el diario de análisis del trader. Perderlo al migrar de computador es perder meses de trabajo de reflexión.

---

### 🥈 VALOR SÓLIDO — Útil, bien pensado, con margen de mejora

#### 1.6 Journal de Trades con Libreta

El formulario de registro captura lo que importa: par, sesión, cuenta, resultado, P&L, R:R real, riesgo%, mood, reglas rotas, tags de setup, descripción y foto.

La libreta separada de la nota es correcta. La `note` es para el setup (entrada, SL, TP, razón). La `notebook` es para el análisis profundo post-trade. Son dos cosas distintas y merecen campos distintos.

Lo que falta y duele: **no se puede editar un trade ya guardado**. En la práctica, a veces registro el trade con R:R estimado y después quiero actualizar con el R:R real. O cometí un error en el P&L. Hoy tengo que borrar y volver a registrar. Eso interrumpe el flujo y desalienta el uso del journal.

#### 1.7 Análisis de Patrones

Los 10 patrones detectados automáticamente son correctos en concepto. Los más valiosos son:
- Revenge trading vs WR
- Reglas rotas vs pérdidas
- Mejor par y peor par
- Mejor sesión
- Peor día de la semana

La evolución del win rate semana a semana (últimas 12 semanas) es el gráfico más útil que puede ver un trader en proceso de fondeo. Le dice si está mejorando o deteriorando en el tiempo, que es exactamente la pregunta que se hace cada domingo.

**Problema real:** los patrones requieren 5+ trades. Un trader nuevo que compra el producto no ve nada en patrones por días. Eso genera abandono temprano. El umbral de 5 trades es muy alto para generar el primer engagement.

#### 1.8 Gestión de Fondos

Registrar payouts, challenges comprados, gastos y metas en el mismo lugar es útil para un trader que ya tiene ingresos del trading. La vinculación del payout a la cuenta específica y la firma es correcta.

**Problema:** los tipos de movimiento están incompletos para la realidad del trader. Falta `retiro_parcial` (cuando retiras parte del profit sin cerrar la cuenta), `bonificación` (algunas firmas dan bonos), y `pérdida_challenge` (cuando pierdes un challenge, que es diferente a un gasto operacional regular). El modelo actual de "challenge comprado = gasto" es correcto pero tosco.

#### 1.9 Fases del Plan

Las 6 fases (Disciplina, Challenge, Fondeado, Escalando, Capital Propio, Metas Cumplidas) reflejan la progresión real de un trader de prop firms. El orden es lógico.

Las condiciones en tiempo real para la fase Disciplina (racha de sesiones limpias) son una buena idea, pero hay un problema de calibración: **¿5 sesiones limpias consecutivas con WR ≥50% es suficiente para pasar a Challenge?** No lo es. Un trader que opera 1 sesión por semana pasa a Challenge en 5 semanas. Un trader que opera 5 días a la semana pasa en 1 semana. La métrica ignora la frecuencia y el volumen de trades.

Un umbral más realista para considerar que un trader está listo para un challenge: 20+ trades registrados, WR ≥50%, drawdown diario nunca roto en ninguna sesión, R:R promedio ≥ su mínimo configurado. Eso es lo que una prop firm realmente valora.

---

## 2. Funciones que Parecen Innecesarias o Cuestionables

Estas funciones existen pero su valor real es bajo, o tienen problemas de diseño que las hacen menos útiles de lo que deberían ser.

---

### 2.1 Resumen Semanal Solo en Sábado y Domingo

La función `renderWeeklySummary()` tiene un guard que la oculta de lunes a viernes. Esto no tiene ninguna lógica de trading.

Un trader que quiere revisar su semana lo hace el miércoles si tiene dudas, no solo el fin de semana. Ocultar un resumen de rendimiento durante 5 días de 7 es limitar arbitrariamente el acceso a información propia del usuario.

**Recomendación:** Mostrar el resumen semanal todos los días. Si el objetivo era "motivar la revisión del fin de semana", no funciona así — lo que hace es frustrar al trader que quiere revisarlo el jueves.

### 2.2 Score de Consistencia como Métrica Principal

El Score de Consistencia es el promedio entre win rate y cumplimiento de reglas. Entiendo la intención — combinar resultados con disciplina en un solo número. Pero en la práctica, esta métrica no le dice nada concreto al trader.

Un trader con WR de 60% y 100% de cumplimiento de reglas tiene Score de 80. Un trader con WR de 40% y 100% de cumplimiento también tiene 70. Pero el segundo puede ser rentable si su R:R es 2.5:1. El Score ignora el R:R, que es la variable más importante para la rentabilidad.

Una métrica de consistencia útil para prop firms debería incluir: WR, R:R promedio, days sin drawdown roto, y reglas cumplidas. El número actual es demasiado simplificado para que un trader lo use como guía real.

**No eliminar — reformular.** El concepto es bueno. La fórmula necesita trabajo.

### 2.3 Tags de Setup como Categorías Genéricas

Los tags actuales (Barrida de liquidez, Ruptura, Pullback, S&R, FVG, Order Block, News play) son categorías razonables pero demasiado generales para ser útiles en el análisis de patrones.

Un trader que opera ICT necesita más granularidad: ¿fue un OB en 15M con confluencia de FVG en 1H? ¿Fue una barrida de liquidez en sessión de Asia antes de London Open? Los tags actuales no capturan suficiente contexto como para que el sistema pueda decirle "tu mejor setup es X en Y condición".

**Solución:** Permitir tags personalizados adicionales, no solo los predeterminados. El trader define sus propios setups. Esto convierte los patrones de algo genérico en algo específico para ese trader.

### 2.4 Fases con Tareas de Checklist Redundantes

Las tareas por fase se marcan con checkbox, pero no se conectan con ningún dato real de la app. Si la tarea de la fase Challenge dice "Abrir cuenta en FTMO", el usuario la marca manualmente — no hay verificación automática de que realmente lo hizo.

Esto convierte las fases en una lista de tareas manuales que el trader podría mantener en Notion o papel. Para justificar su existencia dentro de TradeOS, las fases deberían conectarse con los datos reales: si el trader tiene una cuenta registrada con estado "challenge", la fase Challenge debería reflejar eso automáticamente.

**No eliminar — conectar con datos reales.** El concepto vale. La implementación actual es demasiado manual.

### 2.5 Sistema de Soporte por Gmail

Abrir Gmail con un email pre-completado no es soporte — es una dirección de correo que también podría estar en la página de Gumroad. Para un producto SaaS serio, el soporte necesita ser dentro del producto o al menos un formulario estructurado.

En v1.0 como archivo local es aceptable. En v2.0 con hosting y usuarios pagando suscripción mensual, esto no es suficiente.

---

## 3. Problemas Importantes del Trader sin Resolver

Estos son los gaps reales — cosas que como trader buscaría en una herramienta y TradeOS todavía no tiene.

---

### 3.1 🔴 No se Puede Editar un Trade Ya Guardado

Este es el problema de UX más frustrante del producto en la práctica diaria.

**Situación real:** Registro un trade a las 8:15 AM, justo después de entrar. El trade sigue abierto. Pongo el P&L estimado basado en mi TP. A las 9:30 AM el mercado no llega al TP y el precio se revierte. Cierro con P&L diferente. Tengo que ir al journal, borrar el trade, y volver a registrarlo desde cero.

Borrar y re-registrar tiene consecuencias: si el trade era madre, tengo que borrar también las hijas y re-replicar todo. Si tenía libreta escrita, la pierdo al borrar.

**Solución mínima viable:** Botón "Editar" en cada trade que abre el formulario pre-cargado con los datos del trade. Solo permite editar: P&L, R:R real, nota, libreta, mood. No permite cambiar fecha, par o cuenta (eso sí rompería la integridad de datos).

### 3.2 🔴 No hay Diario de Mercado Separado del Journal

El bias del día es un campo de texto libre en el checklist. Eso es insuficiente.

Un trader serio tiene un diario de mercado que incluye:
- Análisis HTF (Daily, 4H): sesgo direccional, niveles clave
- Contexto macroeconómico: qué eventos hay esta semana, cómo afectan al activo
- Plan de sesión: qué setup espera, en qué zona, con qué condición
- Post-sesión: qué pasó, si el setup se materializó, qué aprendió

Esto no es el journal de trades — es el análisis pre y post mercado. Son herramientas distintas. El bias del día en el checklist es un campo de texto de una línea. Un diario de mercado real necesita su propia sección con estructura de fecha, pares, sesión, y texto largo.

**Este es el gap más importante para traders serios.** Un trader disciplinado tiene esto en Notion o en un cuaderno. Si TradeOS lo incorpora bien, elimina una herramienta de la cadena de trabajo del trader.

### 3.3 🔴 No hay Precio de Entrada, SL ni TP Registrados por Trade

El trade registra P&L y R:R real, pero no los niveles de precio. Esto limita el análisis posterior.

¿Por qué importa? Porque un trader que revisa sus trades 6 meses después no puede reconstruir qué pasó sin la captura de pantalla. Y si no subió foto (que es el caso en la mayoría de los trades rápidos), no hay ninguna referencia de precio.

Con entrada, SL y TP registrados, el sistema puede calcular automáticamente el R:R planeado vs el R:R real obtenido. Esa diferencia es información valiosa: ¿el trader está saliendo temprano (menos R:R del planeado) o aguantando bien?

**Campos sugeridos:** Precio entrada, Precio SL, Precio TP (todos opcionales). El R:R planeado se calcula automáticamente como `(TP - entrada) / (entrada - SL)` y se compara con el R:R real.

### 3.4 🔴 No hay Alertas Activas de Reglas de la Prop Firm

La app calcula drawdown pero no advierte de forma activa cuando el trader está cerca de violar reglas de la firma.

**Situación real:** Son las 10 AM. Perdí dos trades seguidos en London Open. Mi daily loss en Funding Pips está en 3.8% de 5% permitido. La app muestra la barra, pero no me bloquea ni me alerta antes de que abra otro trade.

**Lo que debería existir:** Una alerta prominente (no un toast — algo que bloquee el acceso al formulario de registro) cuando el drawdown diario calculado supera el 80% del límite configurado. Y un aviso similar cuando el drawdown acumulado supera el 70%.

Esta función podría literalmente salvar cuentas. Una cuenta de $50,000 en Funding Pips vale entre $500-$1,000 pagada en challenge. Que la app te detenga antes de reventarla vale mucho más que los $19 del producto.

### 3.5 🟠 No hay Cálculo de Consistencia por Reglas de Prop Firm

Las prop firms tienen regla de consistencia: no más del 30-40% del profit total debe venir de un solo día (varía por firma). Un trader que gana $1,000 en un día y luego gana $200 en los siguientes 9 días puede violar la regla de consistencia de FTMO aunque esté en ganancia.

TradeOS no calcula esto. El Score de Consistencia mide otra cosa.

**Lo que debería existir:** Un campo configurable por cuenta para "máximo % del profit en un día" (con valor por defecto del 30-40%). El sistema debe advertir cuando un día de trading supera ese porcentaje del profit acumulado de esa cuenta.

### 3.6 🟠 El Análisis de Patrones Tiene Thresholds Fijos que no Reflejan la Realidad del Trader

El análisis detecta "revenge trading en 2 de los últimos 5 trades". Pero si un trader opera 1 trade por sesión, 2 trades con mood revenge puede ser el 40% de sus trades — gravísimo. Si opera 10 trades por sesión, 2 de 5 puede ser un outlier sin importancia.

Los patrones necesitan considerar el volumen de operaciones del trader para que las alertas tengan calibración correcta. Un umbral fijo para todos los traders es mejor que nada, pero no es suficientemente útil para un trader avanzado.

### 3.7 🟠 No hay Historial Accesible del Checklist con Métricas

El checklist guarda el estado por fecha, pero no hay forma de ver "en los últimos 30 días, ¿cuántos días completé el 100% del checklist?" o "¿cuántos días omití revisar el drawdown antes de operar?".

El historial existe en localStorage pero no se le muestra al usuario de ninguna forma útil. Eso convierte el checklist en un ritual sin retroalimentación — lo haces pero no sabes si te está ayudando.

### 3.8 🟡 No hay Notas Post-Sesión

El checklist tiene bias pre-sesión. El journal tiene notas por trade. Pero no hay un espacio para reflexionar sobre la sesión completa: "Hoy London Open fue choppiness total, no había liquidez clara. Debí no operar. Perdí 0.5R por impaciencia."

Ese tipo de reflexión — que no es un trade específico sino la sesión como un todo — es donde está el aprendizaje más importante. Un diario de sesión separado del diario de trades es una herramienta estándar en el proceso de cualquier trader profesional.

### 3.9 🟡 Falta Soporte para Más Tipos de Drawdown

Las prop firms tienen dos tipos principales de drawdown:
- **Trailing drawdown** (Funding Pips, Apex): el máximo permitido sube con el equity máximo. Si tu cuenta empezó en $25,000 y llegó a $26,000, el floor del trailing sube también.
- **Estático** (FTMO en ciertos planes, E8): el drawdown se calcula desde el balance inicial siempre.

TradeOS actualmente trata todos los drawdowns como estáticos (calcula desde `account.size`). Un trader en Funding Pips con trailing drawdown puede estar calculando mal su margen disponible.

Esto es un error de cálculo real que puede llevar a un trader a creer que tiene más margen del que realmente tiene.

---

## 4. Características que Justificarían una Suscripción Mensual

Para que un trader pague $9/mes en lugar de $19 de una vez, la propuesta de valor del plan de suscripción debe ser claramente superior. Estas son las features que crearían esa diferencia real.

---

### 4.1 💰 Sincronización Multi-Dispositivo con Backup Automático (CRÍTICO)

El mayor dolor del producto actual: los datos están en un solo navegador de una sola computadora. Si el disco falla, si cambias de computadora, si quieres ver tus stats en el celular antes de operar — no puedes.

Para un trader que opera con seriedad, **sus datos de trading son su activo más valioso después de su cuenta**. Un año de trades, patrones y journal no tiene precio. Perderlo es catastrófico.

La sincronización cloud automática resuelve este problema de forma definitiva. No requiere que el trader exporte manualmente. Simplemente existe y funciona.

**Esto solo ya justifica $9/mes.**

### 4.2 💰 Importación Automática de Trades desde MT5

Hoy el trader registra cada trade manualmente. Eso toma entre 2 y 5 minutos por trade. Un trader con replicador que opera en 4 cuentas puede tardar 15-20 minutos en registrar una sola sesión completa.

La integración via webhook o EA de MT5 que importa trades automáticamente (par, resultado, P&L, lotaje, hora) elimina la fricción más grande del uso diario. El trader solo agrega el contexto que el sistema no puede capturar: mood, setup, reglas rotas.

**Esto es transformacional para la retención.** Un trader que tiene que registrar manualmente eventualmente deja de hacerlo. Un trader con importación automática sigue usando el journal porque el trabajo ya está hecho.

### 4.3 💰 Alertas por WhatsApp o Email (Drawdown, Reglas, Metas)

Notificaciones activas cuando:
- El drawdown diario supera el 80% del límite configurado → WhatsApp inmediato
- Se detectan 2 trades consecutivos con mood "revenge" → Email de alerta
- Una cuenta llega al 100% de una meta → Notificación de celebración
- El trader lleva 3 días sin registrar trades → Recordatorio de disciplina

Las alertas que salen fuera de la app (al teléfono, al email) tienen valor porque el trader no siempre tiene TradeOS abierto cuando necesita la información.

### 4.4 💰 Análisis Semanal con IA (Resumen Generado Automáticamente)

Un resumen semanal en texto natural generado automáticamente:
> "Esta semana operaste 8 trades, WR de 62.5%, P&L de +$340. Tu mejor sesión fue London Open con 5 de 6 ganadores. El miércoles fue tu peor día — perdiste 2 trades consecutivos en la misma zona de precio sin esperar confirmación. Tu R:R real (1.8) estuvo por debajo de tu mínimo configurado (2.0) en 4 de los 8 trades."

Esto no es un dashboard — es un coach que habla en español y conoce la historia de ese trader específico. Para un trader en proceso de fondeo que no tiene mentor, esto tiene un valor enorme.

### 4.5 💰 Historial Ilimitado y Análisis de Largo Plazo

La versión gratuita (freemium) podría limitar a 50-100 trades y 1 cuenta. La versión Pro desbloquea historial ilimitado y análisis estadísticos de más largo plazo: rendimiento por trimestre, evolución del R:R en 6 meses, comparativa entre cuentas en el tiempo.

Un trader que lleva 1 año operando con TradeOS tiene datos suficientes para análisis que ninguna otra herramienta en español puede hacerle. Eso es retención real.

### 4.6 💰 Plan Teams para Academias y Grupos de Trading

$49/mes para hasta 10 traders que comparten un workspace. El líder del grupo puede ver las métricas agregadas (sin datos privados individuales), identificar patrones comunes de error en el grupo, y enviar alertas del grupo.

Para academias de trading que operan grupos de fondeo, esto es una herramienta de gestión que no existe en ningún producto de la categoría. El precio de $49/mes es bajo para el valor que aporta a una academia.

---

## 5. Mejoras Prioritarias desde la Perspectiva del Usuario Final

Ordenadas por impacto real en el día a día del trader, no por complejidad técnica.

---

### PRIORIDAD 1 — Fix de bugs que corrompen los datos

Antes de cualquier feature nueva, estos bugs me harían desinstalar el producto:

| Bug | Impacto real |
|-----|-------------|
| `motherId` apunta al trade incorrecto en réplicas | Mis métricas de win rate y P&L por cuenta son incorrectas si uso el replicador |
| Eliminar madre deja hijas huérfanas | Cada vez que borro un trade, mis estadísticas se contaminan permanentemente |
| `saveAll()` sin captura de `QuotaExceededError` | Creo que guardé un trade y no lo guardé. Esto destruye la confianza en el producto |
| Bias del día fuera del backup | Pierdo mi historial de análisis al cambiar de computadora |
| `importD()` no restaura checklists | El historial de mi disciplina diaria desaparece en cada migración |

**Ninguna feature nueva vale si los datos están corruptos o se pierden.**

---

### PRIORIDAD 2 — Editar trades registrados

Ya explicado en §3.1. Es la fricción más grande del flujo diario. Sin edición, el journal se convierte en un registro de un solo intento donde cualquier error requiere borrar y empezar de cero.

**Impacto:** Alta retención. Los traders que usan el journal diariamente son los que más valoran y recomiendan el producto.

---

### PRIORIDAD 3 — Alertas activas de drawdown diario por cuenta

Un modal o banner que bloquee el acceso al formulario de trade cuando el drawdown diario de cualquier cuenta supera el 80% del límite configurado.

No un toast de 3 segundos — algo que el trader tenga que confirmar activamente para ignorar. "Tu cuenta FTMO está al 87% del daily loss limit configurado. ¿Seguro que quieres registrar otro trade hoy?"

**Impacto:** Esta función podría salvar cuentas de $50,000+. El valor percibido es enorme y es algo que ningún competidor hace bien.

---

### PRIORIDAD 4 — Estadísticas por par y por sesión (tablas)

La pregunta que todo trader se hace cada semana: "¿En qué par gano más? ¿En qué sesión soy más consistente?"

TradeOS tiene los datos. Solo falta presentarlos en una tabla simple:

| Par | Trades | WR | P&L | R:R Prom |
|-----|--------|----|-----|----------|
| XAU/USD | 45 | 62% | +$1,240 | 2.1 |
| NQ | 12 | 50% | +$340 | 1.9 |
| GBP/USD | 8 | 37% | -$120 | 1.4 |

Con esta tabla, el trader sabe en 10 segundos que debería dejar de operar GBP/USD.

**Impacto:** Alta. Esta es la función que convierte el journal de "registro de trades" a "herramienta de mejora".

---

### PRIORIDAD 5 — Soporte para Drawdown Trailing

Agregar en la configuración de cada cuenta un selector: "Tipo de drawdown: Estático / Trailing". Si es trailing, el sistema calcula el floor del drawdown moviéndose con el equity máximo de la cuenta.

Esta es la diferencia entre calcular correctamente el margen disponible o no. Para un trader de Funding Pips (que usa trailing por defecto), el cálculo actual puede ser incorrecto.

**Impacto:** Técnico medio, pero el error de cálculo es grave para los traders afectados.

---

### PRIORIDAD 6 — Filtro por Rango de Fechas en Mis Trades

Poder ver "todos los trades de abril" o "todos los trades de esta semana" en la vista lista. Hoy solo hay filtros por cuenta, par y resultado. Ningún filtro temporal.

Para un trader que revisa su rendimiento mensual (que es la granularidad más común de revisión para alguien en proceso de fondeo), tener que hacer scroll manual por el historial completo es una pérdida de tiempo real.

---

### PRIORIDAD 7 — Diario de Mercado como Sección Independiente

Ya descrito en §3.2. Una sección con fecha, par operado, sesgo HTF, niveles clave, plan de sesión y reflexión post-sesión. Texto largo, guardado por fecha, exportable.

Esta sección sola convierte a TradeOS de "journal de trades" a "sistema operativo del trader". Elimina una herramienta más de la cadena de trabajo (Notion, cuaderno, notas de iPhone).

---

### PRIORIDAD 8 — Precio de Entrada, SL y TP por Trade

Campos opcionales en el formulario de registro. El R:R planeado se calcula automáticamente. El sistema puede entonces mostrar la diferencia entre R:R planeado y R:R real: ¿estás saliendo temprano de tus trades ganadores?

---

### PRIORIDAD 9 — Historial de Cumplimiento del Checklist

Una vista en la sección de checklist que muestre: en los últimos 30 días, ¿cuántos días tuviste 100% del checklist completado? ¿Cuáles ítems omites con más frecuencia?

Si el trader omite sistemáticamente el ítem "Revisé el drawdown de mis cuentas antes de operar", eso es información de comportamiento muy valiosa.

---

### PRIORIDAD 10 — Notas Post-Sesión

Un campo de texto libre que se abre al finalizar la sesión (cuando el trader hace click en "Cerrar sesión"), vinculado a la fecha de esa sesión.

"¿Qué aprendiste hoy?" es la pregunta más importante que un trader puede responder antes de cerrar el computador.

---

## Tabla de Síntesis — Priorización Final

| # | Mejora | Tipo | Impacto Trader | Complejidad | Versión |
|---|--------|------|---------------|-------------|---------|
| 1 | Fix replicador (motherId + huérfanas) | Bug crítico | ⭐⭐⭐⭐⭐ | Media | v1.1 |
| 2 | Fix saveAll() + compresión imágenes | Bug crítico | ⭐⭐⭐⭐⭐ | Media | v1.1 |
| 3 | Bias del día en backup | Bug | ⭐⭐⭐⭐ | Baja | v1.1 |
| 4 | Fix importD() checklist | Bug | ⭐⭐⭐⭐ | Muy baja | v1.1 |
| 5 | Editar trades registrados | Feature core | ⭐⭐⭐⭐⭐ | Media | v1.1 |
| 6 | Alerta activa de drawdown diario | Feature core | ⭐⭐⭐⭐⭐ | Baja | v1.1 |
| 7 | Stats por par y sesión (tabla) | Feature core | ⭐⭐⭐⭐⭐ | Baja | v1.1 |
| 8 | Drawdown trailing vs estático | Feature core | ⭐⭐⭐⭐ | Media | v1.1 |
| 9 | Filtro por fechas en Mis Trades | Feature UX | ⭐⭐⭐⭐ | Baja | v1.1 |
| 10 | Resumen semanal todos los días | Fix UX | ⭐⭐⭐ | Muy baja | v1.1 |
| 11 | Precio entrada + SL + TP por trade | Feature | ⭐⭐⭐⭐ | Baja | v1.2 |
| 12 | Diario de mercado (sección independiente) | Feature mayor | ⭐⭐⭐⭐⭐ | Alta | v1.2 |
| 13 | Notas post-sesión | Feature | ⭐⭐⭐⭐ | Baja | v1.2 |
| 14 | Historial de cumplimiento del checklist | Feature analítica | ⭐⭐⭐ | Media | v1.2 |
| 15 | Regla de consistencia por prop firm | Feature avanzada | ⭐⭐⭐⭐ | Media | v1.2 |
| 16 | Tags de setup personalizados | Feature | ⭐⭐⭐ | Baja | v1.2 |
| 17 | Importación automática desde MT5 | Feature Pro | ⭐⭐⭐⭐⭐ | Muy alta | v2.0 |
| 18 | Alertas WhatsApp/email | Feature Pro | ⭐⭐⭐⭐⭐ | Alta | v2.0 |
| 19 | Análisis semanal con IA | Feature Pro | ⭐⭐⭐⭐⭐ | Alta | v2.0 |
| 20 | Plan Teams para academias | Negocio | ⭐⭐⭐⭐⭐ | Muy alta | v2.1 |

---

## Decisiones Pendientes que Requieren Validación

| Pregunta | A quién | Urgencia |
|----------|---------|----------|
| ¿Cuánto tiempo conservar el historial de checklists? ¿90 días, 1 año, indefinido? | Trading Expert (este doc) + CTO | v1.1 |
| ¿El drawdown trailing es calculable sin datos externos (precio actual)? | CTO | v1.1 |
| ¿La alerta de drawdown diario bloquea el formulario o solo avisa? | Product Manager + Juan Pa | v1.1 |
| ¿La regla de consistencia se configura por cuenta o es un campo global? | Trading Expert | v1.2 |
| ¿Los campos de precio (entrada/SL/TP) deben ser obligatorios u opcionales? | Product Manager | v1.2 |
| ¿El Diario de Mercado es parte del checklist o sección independiente en el sidebar? | Product Manager | v1.2 |

---

## Respuestas a las Decisiones Pendientes de mi Competencia

**¿Cuánto tiempo conservar el historial de checklists?**  
Mínimo 1 año. Un trader que revisa si mejoró su disciplina de enero a diciembre necesita 12 meses de historial. Menos de eso no sirve para análisis real. Implementar limpieza automática de entradas de más de 18 meses para controlar el tamaño de localStorage.

**¿La alerta de drawdown bloquea o solo avisa?**  
Avisa con un modal que requiere confirmación activa, pero no bloquea. El trader adulto toma la decisión final. La app le informa; no le impide operar. Bloquear completamente sería paternalista y generaría fricción negativa.

**¿La regla de consistencia se configura por cuenta?**  
Sí, por cuenta. FTMO tiene una regla distinta a Funding Pips. El trader que tiene ambas necesita configurar cada una por separado. Campo sugerido: "% máximo de profit en un día" con valor por defecto de 35%.

**¿Los campos de precio son obligatorios u opcionales?**  
Opcionales siempre. Hay traders que no operan con TP fijo (escalan la salida). Forzarlos a poner un TP artificial solo para completar el formulario destruye la calidad del dato.

**¿Drawdown trailing es calculable sin precio actual?**  
No completamente. Para trailing drawdown exacto se necesita el equity máximo histórico de la cuenta, que el trader tendría que ingresar manualmente o la app calcularía aproximadamente desde los trades registrados. Solución práctica: pedir al usuario que ingrese manualmente el "equity máximo actual" de la cuenta fondeada y calcular el floor desde ahí. No es automático pero es correcto si el trader lo actualiza periódicamente.

---

## Conclusión

TradeOS tiene un núcleo sólido y una diferenciación real. La calculadora de lotaje exacta para futuros CME, el replicador madre/hija, y el seguimiento de drawdown multi-cuenta son ventajas genuinas que ningún competidor en español tiene resueltas correctamente.

El producto actual está a **2-3 semanas de desarrollo de ser algo que yo usaría todos los días**. Los bugs críticos (replicador, saveAll, importD) necesitan cerrarse primero. Después, edición de trades y estadísticas por par/sesión son las features que convierten el journal de registro a herramienta de mejora.

El camino a una suscripción de $9/mes está claro: sincronización cloud + importación MT5 + alertas activas. Sin esas tres cosas, el argumento para pagar mensual en lugar de $19 único no es lo suficientemente fuerte para el trader que ya compró la versión actual.

El diario de mercado como sección independiente es la feature que más diferencia puede hacer a largo plazo. Es el espacio que convierte a TradeOS de "journal de trades" a "sistema operativo del trader" — exactamente lo que promete el eslogan.

---

*Documento generado por Trading Expert — TradeOS*  
*Fecha: Junio 2026*  
*Para Product Manager: las prioridades 1-10 de la tabla de síntesis son el input recomendado para el sprint v1.1*  
*Para CTO: las decisiones técnicas pendientes requieren respuesta antes de implementar Prioridades 8, 12 y 15*  
*Para Juan Pa: las características de la sección 4 (suscripción mensual) son la base para el brief de v2.0*
