# PRODUCT_REQUIREMENTS_V2.md
## TradeOS Web — Definición de Producto para la Primera Versión Pública

**Autor:** Product Manager  
**Fecha:** Junio 2026  
**Fuentes:** MASTER_CONTEXT · EXECUTIVE_REPORT · SCHEMA · FUNCTIONS · LOCALSTORAGE_AUDIT · SAAS_ARCHITECTURE_PLAN · TRADING_EXPERT_REVIEW · BUSINESS_VISION  
**Audiencia:** Cualquier desarrollador que se incorpore al proyecto

> Este documento responde una pregunta: **¿qué es TradeOS Web v2 exactamente?**  
> No hay código aquí. No hay arquitectura técnica. Solo la definición del producto.  
> Si después de leer este documento alguien no entiende qué estamos construyendo, el documento falló.

---

## Glosario de Términos Clave

Antes de cualquier otra cosa, estos son los términos que se usan en todo el documento con significado específico.

| Término | Significado en TradeOS |
|---------|----------------------|
| **Trader** | El usuario de TradeOS. Profesional, no principiante. |
| **Prop firm** | Empresa que financia traders (Funding Pips, FTMO, E8, Apex). |
| **Challenge** | Fase de evaluación que un trader pasa para obtener fondeo de una prop firm. |
| **Cuenta fondeada** | Cuenta de capital real otorgada por una prop firm al trader que pasó el challenge. |
| **Drawdown** | Pérdida acumulada respecto al balance máximo. Las prop firms tienen límites de drawdown. |
| **Replicador** | Función que registra el mismo trade en múltiples cuentas simultáneamente. |
| **Trade madre** | El trade principal en el replicador. Contiene el análisis completo. |
| **Trade hija** | Réplica del trade madre en otra cuenta. Solo tiene P&L y datos operativos. |
| **R:R** | Risk:Reward. Relación entre riesgo y ganancia de un trade. |
| **Win Rate (WR)** | Porcentaje de trades ganadores sobre el total. |
| **Score de Consistencia** | Métrica de TradeOS: promedio entre Win Rate y cumplimiento de reglas. |
| **Plan Free** | Acceso gratuito con límites funcionales. |
| **Plan Pro** | Acceso completo por suscripción mensual. |
| **Onboarding** | Proceso de configuración inicial del perfil del trader. |
| **Libreta** | Campo de análisis profundo post-trade. Solo existe en el trade madre. |
| **Bias del día** | Nota de análisis de mercado del trader antes de la sesión. |
| **Sesión** | Período de trading: London Open, New York, Asia, Overlap. |

---

## 1. Qué es TradeOS Web v2

TradeOS Web v2 es la versión web de TradeOS — una aplicación que funciona como **el sistema operativo personal de un trader de prop firms**. Centraliza en un solo lugar todo lo que un trader necesita para operar con consistencia y disciplina:

- **Journal de trades** con historial completo, análisis y capturas de gráficos
- **Checklist diario** de pre-sesión y post-trade
- **Calculadora de lotaje exacta** para forex y futuros CME
- **Gestión de múltiples cuentas** con drawdown en tiempo real por cuenta
- **Replicador madre/hija** para traders con múltiples cuentas simultáneas
- **Análisis de patrones** automático que detecta comportamientos en el historial
- **Fases de carrera** que siguen el progreso del trader desde disciplina hasta fondeo

**La diferencia con v1.x:** TradeOS v1.x era un archivo HTML local que funcionaba sin internet. TradeOS Web v2 es una aplicación web en `app.tradeos.io` con cuentas de usuario, sincronización automática y acceso desde cualquier dispositivo. Los datos del trader viven en la nube, no en el navegador local.

**La propuesta de valor no cambia:** sin publicidad, los datos del trader son del trader, sin modo para principiantes, precio justo.

---

## 2. El Usuario de TradeOS Web v2

**Perfil principal:**
- Trader latinoamericano, 18-35 años
- Opera London Open y/o New York Open con MetaTrader 5
- Tiene entre 1 y 5 cuentas activas en prop firms (Funding Pips, FTMO, E8, Apex)
- Usa replicador de trades entre cuentas
- Opera XAU/USD, GBP/USD, EUR/USD, NQ, ES, MNQ
- Entiende drawdown, R:R, win rate, consistencia — no necesita explicaciones básicas
- Frustrado con herramientas en inglés que no entienden su realidad

**Lo que este usuario necesita que ninguna otra herramienta le da:**
- Calculadora de lotaje correcta (sobre balance real, no tamaño original)
- Replicador de trades nativo para múltiples cuentas
- Seguimiento de drawdown por cuenta de distintas prop firms
- Todo en español, adaptado a su realidad específica
- Sin pagar $30-50/mes por una herramienta genérica en inglés

---

## 3. Qué Tendrá TradeOS Web v2 en su Primera Versión Pública

La primera versión pública de TradeOS Web v2 incluye las siguientes funcionalidades, todas operativas y sin restricciones de tiempo de evaluación.

### 3.1 Sistema de Acceso

- Registro e inicio de sesión por **email sin contraseña** (magic link): el trader escribe su email, recibe un link de acceso, hace click y entra. Sin contraseña, sin fricción.
- Acceso alternativo con **Google OAuth** en un click.
- Sesión válida por **7 días** con renovación automática. El trader no se desloguea solo.
- Un solo email puede tener una sola cuenta.

### 3.2 Onboarding Personalizado (9 pasos)

El primer inicio activa un proceso de configuración que personaliza toda la experiencia. No es opcional — el producto no funciona sin este paso.

| Paso | Qué configura |
|------|--------------|
| 1 — Perfil | Nombre o alias, edad, país, años de experiencia, nivel (principiante a profesional) |
| 2 — Cuentas | Firma/broker, tamaño, tipo (propia o fondeada), instrumento (forex o futuros CME), drawdown máximo. Se pueden agregar hasta 5 cuentas. |
| 3 — Estilo | Pares operados, sesiones (London/NY/Asia/Overlap), estrategia, perfil de riesgo, % de riesgo por trade, R:R mínimo, máximo de trades por sesión |
| 4 — Reglas | Reglas personales del trader que no se pueden romper |
| 5 — Horario | Trabajo externo, horario de sueño, horas de operación |
| 6 — Checklist | Selección y personalización de los ítems del checklist diario |
| 7 — Metas | Objetivos financieros con nombre y monto objetivo |
| 8 — Fases | Selección de la fase actual en la carrera del trader |
| 9 — Psicología | Error más frecuente, condiciones en que toma peores decisiones |

Al completar el onboarding, la app está completamente configurada con los datos de ese trader específico. Todos los módulos reflejan sus cuentas, pares, sesiones y reglas.

### 3.3 Dashboard

La pantalla principal que el trader ve cada día al abrir la aplicación.

**Score de Consistencia:** Número grande en el centro del dashboard. Es el promedio entre el Win Rate global y el porcentaje de trades sin reglas rotas. Muestra un label de nivel (Excelente / Sólido / Regular / Mejorar) con color correspondiente (verde / amarillo / rojo).

**6 métricas rápidas:**
- Total de trades registrados
- Win Rate actual
- P&L neto total en $
- Racha limpia actual (trades consecutivos sin romper reglas)
- R:R promedio real
- Reglas rotas esta semana

**Curva de equity:** Gráfico de línea que muestra el P&L acumulado en el tiempo. Aparece desde el segundo trade registrado.

**Panel por cuenta:** Una tarjeta por cada cuenta configurada mostrando P&L de esa cuenta, barra de drawdown con colores (verde/amarillo/rojo según el porcentaje usado), y la fase activa.

**Alerta de drawdown:** Aviso visible cuando alguna cuenta supera el 70% de su drawdown máximo configurado.

**Metas financieras:** Barra de progreso por cada objetivo configurado en el onboarding.

**Resumen de la semana:** Panel automático con estadísticas de los trades de la semana actual (trades, WR semanal, mejor trade, peor trade). Visible todos los días, no solo el fin de semana.

**Últimos 5 trades:** Lista rápida con resultado y monto de los trades más recientes.

### 3.4 Checklist Diario

Una lista de verificación que el trader completa antes y después de cada sesión de trading.

**Estructura:** 13 ítems predeterminados organizados en 4 secciones:
- Antes de operar (estado físico y mental)
- Al abrir la plataforma (verificaciones técnicas)
- Al ver un setup (validaciones pre-entrada)
- Después del trade (registro y cierre)

Los 13 ítems predeterminados **nunca se pueden eliminar** — son la base del checklist. El trader puede agregar ítems personalizados en cualquier posición de la lista.

**Bias del día:** Campo de texto libre donde el trader escribe su análisis de mercado antes de la sesión (ej: "Espero continuación bajista en XAU después de barrida en H4"). Se guarda por fecha y se puede consultar históricamente.

**Temporizador de sesión:** Botón "Iniciar sesión" que activa un contador de tiempo visible. Al cerrarlo, pregunta si hay trades para registrar y redirige al formulario de registro.

**Historial:** El estado del checklist de cada día se guarda. El trader puede ver qué días tuvo el checklist completo y cuáles no.

### 3.5 Registro de Trades

Formulario para registrar cada operación. Es el módulo más usado del producto.

**Campos del formulario:**
- Par o instrumento (selector con los pares del onboarding + opción libre)
- Sesión (London / New York / Asia / Overlap)
- Cuenta (selector con las cuentas configuradas)
- Resultado (Ganador / Perdedor / Break Even) — selección visual con botones grandes
- P&L en $ (positivo o negativo)
- R:R real obtenido
- % de riesgo usado
- Estado emocional (Tranquilo / Confiado / Ansioso / Presionado / Cansado / Revenge trading)
- Reglas rotas (selección múltiple de las reglas configuradas — puede seleccionar varias)
- Tags de setup (Barrida de liquidez / Ruptura / Pullback / S&R / FVG / Order Block / News play)
- Descripción del setup (texto libre)
- Captura del gráfico (imagen del trade)
- Libreta (campo de análisis profundo — solo en Plan Pro)

**Calculadora de lotaje integrada:** Calcula el tamaño de posición exacto basado en el balance real de la cuenta (tamaño original más/menos el P&L acumulado), el porcentaje de riesgo y el stop loss en pips o ticks.

- Futuros CME con valores de tick exactos: NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL
- Forex con pip value exacto: EUR/USD, GBP/USD, AUD/USD, NZD/USD
- Instrumentos no soportados muestran un mensaje explicativo — nunca dan un número aproximado sin avisar

**Replicador madre/hija:** Botón que abre un panel donde el trader especifica los P&L de ese mismo trade en cada una de sus otras cuentas. Al guardar, crea el trade madre (con análisis completo) y las réplicas hijas (con solo los datos operativos por cuenta). La libreta solo existe en la madre — nunca se copia a las hijas.

**Edición de trades:** Todo trade guardado puede abrirse en modo edición para corregir cualquier campo. Se registra la fecha y hora de la edición. Un trade madre editado no altera los campos operativos de sus hijas.

### 3.6 Historial de Trades (Mis Trades)

Vista de todos los trades registrados con dos modos de visualización.

**Vista lista:**
- Filtros por cuenta, par, resultado y rango de fechas (desde / hasta)
- Botones rápidos de período: Esta semana / Este mes / Último mes
- Badge MADRE / HIJA por trade
- Libreta visible como panel colapsado si tiene contenido
- Reglas rotas marcadas en rojo
- Miniatura de la captura del gráfico, clicable para ver en tamaño completo
- Botones de editar y eliminar por trade

Al eliminar un trade madre, aparece un modal de confirmación que muestra cuántas réplicas hijas tiene. El trader elige eliminar todo el grupo o solo la madre.

**Vista calendario:**
- Navegación por mes (anterior / siguiente)
- Resumen mensual en la parte superior: total de trades, WR, P&L
- Días coloreados según resultado: verde (ganancia), rojo (pérdida), amarillo (break even)
- Puntos indicadores de cuántos trades hubo ese día
- Click en un día: panel con todos los trades de ese día con todos sus detalles

### 3.7 Fases del Plan

Seguimiento del progreso del trader a través de las etapas de su carrera.

**6 fases predeterminadas en orden:**
1. 🌱 Disciplina — desarrollar consistencia antes de arriesgar capital real
2. 🎯 Challenge — proceso de evaluación de la prop firm
3. 💰 Fondeado — cuenta real con capital de la firma
4. 🚀 Escalando — aumentando el tamaño de las cuentas
5. 🏦 Capital Propio — operando con capital personal
6. ❤️ Metas Cumplidas — objetivos financieros alcanzados

Cada fase tiene tareas específicas con checkbox de progreso. El trader marca las tareas a medida que las completa.

La fase "Disciplina" tiene condiciones en tiempo real: muestra la racha actual de sesiones limpias (sin romper reglas) y el Win Rate de las últimas sesiones.

El trader puede crear **fases personalizadas** con ícono, nombre, descripción y sus propias tareas.

Cuando el trader marca una fase como completada, queda registrada como hito histórico con la fecha de completado y las métricas de ese momento.

### 3.8 Gestión de Fondos

Registro de todos los movimientos de dinero relacionados con el trading.

**7 tipos de movimiento:**
- Payout de cuenta propia
- Payout de cuenta fondeada
- Salario / trabajo externo
- Meta alcanzada
- Challenge comprado (gasto)
- Gasto operacional (suscripciones, herramientas, etc.)
- Otro

Cada movimiento se vincula a una cuenta específica cuando aplica.

**Panel de totales:** Muestra ingresos totales, gastos totales y neto. Los challenges y gastos operacionales se clasifican como salidas; los payouts, salarios y metas como entradas.

### 3.9 Cuentas

Vista detallada de todas las cuentas configuradas.

Por cada cuenta:
- Nombre de la firma y tipo (propia / fondeada)
- Tamaño original de la cuenta
- Balance real actual (tamaño menos pérdidas acumuladas)
- P&L total en $ y en %
- Drawdown actual con barra visual de progreso (verde / amarillo / rojo)
- Drawdown máximo configurado
- Instrumento (forex o futuros CME)
- Fase actual (activa / challenge 1 / challenge 2 / fondeada / en pausa / en evaluación)

**Acciones sobre cuentas:**
- Editar todos los campos de una cuenta
- Agregar cuenta nueva
- Archivar cuenta (la cuenta queda inactiva y se excluye de los cálculos del dashboard, pero sus trades históricos se conservan)
- Eliminar cuenta (requiere confirmación — elimina todos los trades asociados)

**Alerta de drawdown diario:** Si se configura un porcentaje de pérdida máxima diaria por cuenta, la app muestra un aviso cuando el P&L del día en esa cuenta se acerca o supera ese límite.

### 3.10 Análisis de Patrones

Módulo que detecta automáticamente patrones de comportamiento en el historial del trader.

Requiere mínimo 5 trades registrados para activarse.

**10 patrones analizados:**
1. Revenge trading vs Win Rate normal — ¿rinde menos cuando opera después de una pérdida?
2. Reglas rotas vs pérdidas — ¿las pérdidas coinciden con trades donde rompió reglas?
3. Mejor y peor par por Win Rate
4. Mejor sesión por Win Rate
5. Efecto del sueño en el Win Rate — correlación entre horas de sueño y resultado
6. Patrón pérdida → regla rota — ¿tiende a romper reglas después de perder?
7. R:R real vs R:R planificado — ¿sale temprano de sus trades ganadores?
8. Peor día de la semana
9. Rachas de pérdidas consecutivas — duración y frecuencia
10. Break evens frecuentes — ¿saca el profit demasiado pronto por miedo?

**Estadísticas por par:** Tabla con WR, P&L promedio, R:R promedio y total de trades para cada instrumento operado.

**Estadísticas por sesión:** Misma tabla para London / New York / Asia / Overlap.

**Evolución del Win Rate:** Gráfico con el WR de las últimas 12 semanas. Permite ver si el trader está mejorando o deteriorando en el tiempo.

### 3.11 Calculadora de Lotaje (sección independiente)

Calculadora completa accesible sin necesidad de estar registrando un trade.

El trader ingresa:
- Cuenta (selector con sus cuentas configuradas)
- % de riesgo (con opciones predefinidas y campo libre)
- Stop Loss en pips (forex) o en ticks (futuros)

La calculadora devuelve:
- Lotaje en estándar, mini y micro (forex)
- Número de contratos y mini-contratos (futuros)
- Monto en riesgo en $

Siempre calcula sobre el **balance real** de la cuenta: tamaño original más o menos el P&L acumulado de todos los trades registrados. Si una cuenta de $10,000 tiene $400 en pérdidas, la calculadora usa $9,600, no $10,000. Esa es la diferencia entre arriesgar correctamente y sobre-arriesgar en drawdown.

### 3.12 Ajustes y Perfil

Panel donde el trader configura y edita todos los parámetros de su cuenta.

**Edición de perfil:** Nombre, país, experiencia, plataforma usada.

**Parámetros de trading:** % de riesgo por trade, R:R mínimo, máximo de trades por sesión, pares operados, sesiones.

**Reglas personales:** Agregar, editar y eliminar reglas.

**Checklist:** Personalizar los ítems del checklist (agregar, renombrar, reordenar). Los 13 ítems base nunca se pueden eliminar.

**Idioma:** Español o inglés. Toda la app cambia de idioma instantáneamente.

**Tema visual:** Modo oscuro (Obsidian Gold) o modo claro (Crema Dorado).

**Backup y datos:**
- Exportar todos los datos como archivo JSON descargable (backup completo)
- Importar datos desde un backup JSON previo
- Exportar historial de trades como CSV
- Importar datos desde TradeOS v1.x (migración desde el archivo HTML local)

**Peligroso:** Opción de eliminar todos los datos de la cuenta (requiere confirmación explícita escribiendo una frase).

### 3.13 Página de Upgrade

Página accesible desde cualquier lugar de la app cuando el usuario Free intenta usar una función Pro.

Muestra:
- Comparación clara Free vs Pro
- Precio ($9/mes o $79/año)
- Botón de pago directo a Stripe Checkout

---

## 4. Qué Queda Fuera de la Primera Versión

Estas funcionalidades están definidas para versiones posteriores. No se construyen en v2.0.

### Fuera de v2.0 — Planificado para v2.1

| Funcionalidad | Razón |
|--------------|-------|
| **Plan Teams** para academias y grupos de traders | Requiere sistema de invitaciones, roles y panel de mentor. Complejidad alta. |
| **Modo "Solo lectura"** para compartir journal con mentor | Depende de que Teams exista primero. |
| **Análisis semanal generado con IA** | Requiere integración con API de IA. Costo operacional variable. Se evalúa como feature Pro avanzada. |
| **Alertas por email o WhatsApp** cuando el drawdown supera umbrales | Requiere infraestructura de notificaciones push. No es offline-first. |

### Fuera de v2.0 — Planificado para v2.2 o posterior

| Funcionalidad | Razón |
|--------------|-------|
| **Importación automática de trades desde MT5** | Requiere desarrollo de webhook/plugin para MetaTrader. Complejidad muy alta. Es la feature más solicitada pero también la más costosa. |
| **App móvil nativa** (iOS / Android) | Se puede lograr con PWA primero. La app nativa requiere React Native y presupuesto de publicación en tiendas. |
| **Calculadora de lotaje vía bot de Telegram** | Canal de distribución adicional, no funcionalidad core. |
| **Modo "Competencia"** entre traders de un grupo | Feature social. Requiere Teams activo. |
| **Integración con TradingView** para capturas automáticas | Depende de APIs de TradingView que pueden cambiar. |
| **Diario de mercado como sección independiente** | Feature de alta prioridad para v2.1 — el bias del día en el checklist es el MVP de esto. |

### Decisiones pendientes que afectan el scope (requieren respuesta de Juan Pa antes de la Fase 3 de desarrollo)

| Pregunta | Impacto |
|---------|---------|
| ¿Los compradores de v1.x obtienen acceso Pro gratuito? ¿Por cuánto tiempo? | Afecta la propuesta de lanzamiento y la comunicación a compradores existentes. |
| ¿El archivo HTML v1.x sigue vendiéndose en paralelo a v2.0? | Afecta el modelo de negocio y la comunicación de marketing. |
| ¿El Plan Free requiere solo email o también tarjeta de crédito para registrarse? | Afecta la tasa de conversión del registro. |

---

## 5. Qué Ve el Usuario Después de Registrarse

### Primera vez — Usuario nuevo

```
1. Pantalla de bienvenida → botón "Comenzar configuración"
2. Onboarding: 9 pasos (ver §3.2)
   → El trader no puede saltar ni omitir el onboarding
   → Puede volver pasos anteriores para corregir
   → Si cierra el navegador a mitad: al volver retoma donde dejó
3. Al completar el onboarding: animación de bienvenida + redirige al Dashboard
4. Dashboard con los datos del trader configurados
   → Las métricas muestran cero (no hay trades aún)
   → El checklist del día está vacío
   → Aparece un tip contextual: "Empezá registrando tu primer trade →"
```

### Regreso — Usuario existente

```
1. El usuario abre app.tradeos.io
2. Si tiene sesión activa (menos de 7 días desde el último acceso): entra directo al Dashboard
3. Si la sesión expiró: pantalla de login → escribe email → magic link → Dashboard
4. Dashboard muestra el estado actual de sus datos
```

---

## 6. Módulos del Producto

TradeOS Web v2 tiene **10 módulos** accesibles desde el sidebar de navegación. El sidebar tiene exactamente estas secciones, en este orden:

| # | Módulo | Ícono | Descripción breve |
|---|--------|-------|------------------|
| 1 | Dashboard | 🏠 | Vista general del estado del trader |
| 2 | Checklist | ✅ | Lista de verificación diaria + bias del día |
| 3 | Registrar Trade | ➕ | Formulario de nuevo trade |
| 4 | Mis Trades | 📋 | Historial (lista + calendario) |
| 5 | Fases del Plan | 🗺️ | Progreso en la carrera del trader |
| 6 | Gestión de Fondos | 💰 | Payouts, gastos y metas |
| 7 | Cuentas | 🏦 | Detalle por cuenta con drawdown |
| 8 | Análisis | 📊 | Patrones, estadísticas por par y sesión |
| 9 | Calculadora | 🧮 | Calculadora de lotaje independiente |
| 10 | Ajustes | ⚙️ | Perfil, configuración, backup, plan |

El módulo "Upgrade" no tiene entrada en el sidebar — es una página a la que se llega desde los banners de limitación del Plan Free o desde el footer de Ajustes.

---

## 7. Qué Puede Hacer un Usuario Free

El Plan Free es gratuito, permanente, y no requiere tarjeta de crédito. Está diseñado para que el trader evalúe el producto durante el tiempo que necesite, con límites claros que justifican el upgrade.

### Lo que tiene el Free

| Módulo | Qué puede hacer |
|--------|----------------|
| Dashboard | Vista completa — Score, métricas, curva de equity, resumen semanal |
| Checklist | Completo — 13 ítems base, personalizables, bias del día, temporizador |
| Registrar Trade | Hasta **50 trades en total**. Sin imagen adjunta. Sin libreta. |
| Mis Trades | Ve todos sus trades (hasta 50). Filtros completos. Sin exportar CSV. |
| Fases del Plan | Completo — 6 fases + fases personalizadas |
| Gestión de Fondos | Completo — todos los tipos de movimiento sin límite |
| Cuentas | Máximo **1 cuenta** configurada |
| Análisis | ❌ No disponible — requiere Plan Pro |
| Calculadora | Completo — sin restricciones |
| Ajustes | Perfil, checklist, tema, idioma. Sin exportar CSV ni PDF. |
| Backup JSON | ✅ Puede exportar e importar backup — siempre, aunque no sea Pro |

### Lo que NO tiene el Free

| Función | Por qué es Pro |
|---------|---------------|
| Más de 50 trades | El valor del journal crece con el historial. 50 trades es suficiente para evaluar. |
| Más de 1 cuenta | El diferenciador de TradeOS para prop firm traders son las múltiples cuentas. |
| Screenshots de gráficos | Almacenamiento en la nube tiene costo. |
| Análisis de patrones | Requiere historial suficiente — el Free tiene el módulo bloqueado, no oculto. |
| Exportar CSV / PDF | Portabilidad de datos como feature Pro. |
| Libreta por trade | Feature de análisis avanzado. |
| Sincronización multi-dispositivo | El Free funciona en un dispositivo a la vez. |

### Cómo se presenta la limitación

Cuando el usuario Free toca una función Pro, no se bloquea de forma abrupta. Se muestra:

> "Esta función es parte del Plan Pro. Con Pro tenés trades ilimitados, múltiples cuentas, análisis de patrones y acceso desde cualquier dispositivo. **Ver Plan Pro →**"

El banner es informativo, no hostil. El trader puede ignorarlo y seguir usando las funciones Free.

Cuando el usuario Free llega a 45 trades (90% del límite), aparece un aviso proactivo en el dashboard:

> "Tenés 5 trades restantes en tu plan Free. Pasate a Pro para continuar registrando sin límite."

---

## 8. Qué Puede Hacer un Usuario Pro

El Plan Pro cuesta $9 USD/mes o $79 USD/año (equivalente a $6.58/mes). No tiene contrato de permanencia — se puede cancelar en cualquier momento desde el portal de billing.

### Todo lo del Free, más:

| Función | Detalle |
|---------|---------|
| Trades ilimitados | Sin tope de registros |
| Cuentas ilimitadas | Hasta 5 cuentas simultáneas (el límite del ICP real) |
| Screenshots por trade | Imagen del gráfico adjunta a cada trade |
| Análisis de patrones completo | 10 patrones + estadísticas por par y sesión |
| Exportar CSV | Descarga del historial completo con nombres de firma |
| Libreta por trade | Campo de análisis profundo en el trade madre |
| Backup automático | Los datos se sincronizan automáticamente en la nube |
| Multi-dispositivo | Acceso desde laptop, tablet y móvil con datos sincronizados |
| Soporte prioritario | Email de respuesta en 48hs (vs. 5 días para Free) |

### Al cancelar

El acceso Pro se mantiene hasta el fin del período pagado. Al vencer, la cuenta pasa a Free automáticamente. Los datos no se eliminan — solo se aplican los límites Free. Si el trader tiene más de 50 trades, puede ver todos los anteriores pero no puede registrar nuevos hasta que se suscriba de nuevo o elimine trades hasta llegar a 50.

---

## 9. Páginas del Producto

Estas son todas las páginas (rutas) de la aplicación y lo que contienen.

### Páginas públicas (sin login)

| Ruta | Qué muestra | Redirección si tiene sesión |
|------|-------------|---------------------------|
| `/` | Redirige a `/login` | Redirige a `/dashboard` |
| `/login` | Formulario de email para magic link + botón de Google | Redirige a `/dashboard` |

### Páginas privadas (requieren login + onboarding completado)

| Ruta | Módulo | Contenido principal |
|------|--------|-------------------|
| `/onboarding` | Onboarding | 9 pasos de configuración inicial. Solo accesible si `onboarding_done = false`. |
| `/dashboard` | Dashboard | Score de Consistencia, métricas, curva de equity, paneles por cuenta, metas. |
| `/checklist` | Checklist | Lista diaria, bias del día, temporizador de sesión. |
| `/trades/new` | Registrar Trade | Formulario de nuevo trade con calculadora y replicador. |
| `/trades` | Mis Trades | Historial en vista lista. Filtros por cuenta, par, resultado, fechas. |
| `/trades/calendar` | Mis Trades — Calendario | Vista calendario mensual navegable. |
| `/trades/:id/edit` | Editar Trade | Formulario pre-cargado con los datos del trade seleccionado. |
| `/phases` | Fases del Plan | Cards de fases con tareas y progreso. |
| `/funds` | Gestión de Fondos | Lista de movimientos y totales. |
| `/accounts` | Cuentas | Tarjetas por cuenta con drawdown y acciones. |
| `/patterns` | Análisis | Patrones detectados, estadísticas por par/sesión, evolución del WR. *(Pro)* |
| `/calculator` | Calculadora | Calculadora de lotaje standalone. |
| `/settings` | Ajustes | Perfil, parámetros, checklist, tema, idioma, backup, plan. |
| `/upgrade` | Upgrade | Comparación de planes y botón de pago. |

### Páginas de administración (solo Juan Pa y equipo interno)

| Ruta | Acceso | Contenido |
|------|--------|-----------|
| `/admin` | Solo rol admin | Dashboard con métricas de negocio en tiempo real |
| `/admin/users` | Solo rol admin | Lista de usuarios, búsqueda por email, cambio de plan |
| `/admin/subscriptions` | Solo rol admin | Suscripciones activas, filtros por plan y estado |

---

## 10. Acciones por Página

### `/login`
- Escribir email y solicitar magic link
- Hacer login con Google
- Ver estado del envío del email ("Revisá tu bandeja de entrada")
- Reenviar magic link si no llegó

### `/onboarding`
- Navegar entre los 9 pasos (siguiente / anterior)
- Completar campos de texto y selecciones
- Agregar cuentas (hasta 5) con todos sus parámetros
- Seleccionar chips (sesiones, pares, estrategia, estados emocionales)
- Agregar reglas personalizadas
- Agregar metas con nombre y monto
- Agregar ítems personalizados al checklist
- Finalizar y guardar el perfil completo

### `/dashboard`
- Ver Score de Consistencia y métricas
- Click en una cuenta → ir a `/accounts` con esa cuenta en foco
- Click en "Nuevo trade" → ir a `/trades/new`
- Click en una meta → ir a `/funds`
- Ver alerta de drawdown (si aplica)
- Abrir panel de resumen semanal

### `/checklist`
- Marcar / desmarcar cada ítem
- Escribir el bias del día
- Iniciar temporizador de sesión
- Cerrar sesión (pregunta si hay trades para registrar)
- Agregar ítem personalizado
- Renombrar ítem existente
- Reordenar ítems
- Restaurar checklist a los 13 ítems base
- Resetear checks del día (sin borrar historial)
- Ver historial de días anteriores

### `/trades/new`
- Completar todos los campos del formulario
- Seleccionar resultado con botones grandes (Ganador / Perdedor / BE)
- Seleccionar reglas rotas (multi-selección)
- Seleccionar tags de setup (multi-selección)
- Adjuntar imagen del gráfico *(Pro)*
- Escribir en la libreta *(Pro)*
- Usar la calculadora integrada
- Replicar a otras cuentas (abrir panel del replicador)
- Guardar trade
- Limpiar formulario sin guardar

### `/trades`
- Cambiar entre vista lista y vista calendario
- Filtrar por cuenta / par / resultado
- Filtrar por rango de fechas (desde - hasta)
- Usar botones rápidos de período (Esta semana / Este mes / Último mes)
- Ver detalle de un trade (expandir)
- Editar trade → ir a `/trades/:id/edit`
- Eliminar trade (con confirmación si tiene réplicas)
- Ver imagen del gráfico en tamaño completo
- Ver libreta si el trade tiene contenido

### `/trades/calendar`
- Navegar entre meses (anterior / siguiente)
- Click en un día → panel lateral con trades de ese día
- Ver resumen mensual

### `/trades/:id/edit`
- Editar todos los campos del trade
- Actualizar imagen *(Pro)*
- Guardar cambios
- Cancelar sin guardar
- Ver historial de ediciones (fecha del cambio)

### `/phases`
- Ver todas las fases en tarjetas
- Marcar tareas como completadas
- Expandir / colapsar el detalle de cada fase
- Marcar una fase como completada (con confirmación)
- Crear fase personalizada (ícono, nombre, descripción, tareas)
- Editar fase personalizada
- Eliminar fase personalizada
- Ver hitos históricos de fases completadas

### `/funds`
- Ver listado de movimientos con totales
- Registrar nuevo movimiento (tipo, monto, cuenta, nota)
- Eliminar movimiento (con confirmación)
- Ver totales: ingresos / gastos / neto

### `/accounts`
- Ver tarjeta detallada por cuenta
- Editar cualquier campo de una cuenta
- Agregar cuenta nueva *(Free: máximo 1; Pro: ilimitadas)*
- Archivar cuenta (pasa a inactiva, conserva historial)
- Eliminar cuenta (con confirmación — elimina todos los trades asociados)
- Ver trades de esa cuenta → enlace a `/trades` con filtro aplicado

### `/patterns` *(Solo Pro)*
- Ver los 10 patrones detectados con datos reales
- Ver tabla de estadísticas por par (WR, P&L, R:R, total de trades)
- Ver tabla de estadísticas por sesión
- Ver gráfico de evolución del WR (últimas 12 semanas)

### `/calculator`
- Seleccionar cuenta
- Ingresar % de riesgo
- Ingresar stop loss en pips o ticks
- Ver resultado: lotaje / contratos / monto en riesgo
- Cambiar entre los instrumentos soportados
- Ver mensaje explicativo para instrumentos no soportados

### `/settings`
- Editar nombre, país, experiencia
- Cambiar % de riesgo, R:R mínimo, máximo de trades
- Agregar / editar / eliminar pares operados
- Agregar / editar / eliminar sesiones
- Agregar / editar / eliminar reglas personales
- Personalizar checklist (ver `/checklist` para detalle)
- Cambiar idioma (ES / EN)
- Cambiar tema (oscuro / claro)
- Exportar backup JSON
- Importar backup JSON (con confirmación — sobreescribe datos actuales)
- Exportar trades como CSV *(Pro)*
- Importar datos desde TradeOS v1.x (backup JSON del archivo HTML)
- Ver número de versión de la app
- Ver estado del plan y fecha de renovación
- Ir a Portal de Billing de Stripe (cancelar, actualizar tarjeta, ver facturas)
- Eliminar cuenta completa (confirmación explícita escribiendo una frase)

### `/upgrade`
- Ver comparación Free vs Pro
- Seleccionar plan mensual o anual
- Click en "Suscribirme" → redirige a Stripe Checkout
- Ver testimonios de traders Pro (si los hay)

---

## 11. Funcionalidades Críticas que Nunca Deben Romperse

Estas son las invariantes del producto. Si alguna de estas falla, el producto falla. Cualquier cambio de código debe verificar que estas condiciones se mantienen.

### Invariantes de datos

**1. El balance real de la calculadora siempre es correcto.**  
La calculadora usa `tamaño_original + suma_de_pnl_de_todos_los_trades_de_esa_cuenta`. Nunca el tamaño original a secas si hay pérdidas o ganancias registradas.

**2. Los trades madre nunca se convierten en hijos, y viceversa.**  
Un trade es madre (`is_mother = true`) o es hijo (`is_child = true`). Nunca los dos. Nunca ninguno (todo trade tiene uno de los dos en true).

**3. La libreta solo existe en el trade madre.**  
El campo notebook nunca se copia ni se muestra en réplicas hijas. Si un usuario edita un trade madre, el campo notebook de las hijas no se toca.

**4. Eliminar un trade madre elimina todas sus hijas.**  
Cuando se elimina un trade madre, todas las réplicas hijas de ese mismo trade se eliminan también en la misma operación. No quedan huérfanas en el historial.

**5. El backup incluye todos los datos del usuario.**  
El archivo JSON exportado incluye: perfil, cuentas, trades, fases, fondos, historial de checklist, bias de todos los días, metas. Al importar ese mismo archivo, todos esos datos se restauran exactamente. Nada se pierde en el ciclo export → import.

**6. Los datos del usuario nunca se comparten con otros usuarios.**  
Ningún usuario puede leer, escribir ni ver los datos de otro usuario, en ningún escenario, incluso con acceso técnico al sistema.

**7. Un fallo al guardar siempre es visible para el usuario.**  
Si una operación de guardado falla (sea por conectividad, límite de plan o error del servidor), el usuario recibe un aviso claro y específico. La app nunca confirma un guardado que no ocurrió.

### Invariantes de UX

**8. Los 13 ítems base del checklist no se pueden eliminar.**  
El usuario puede agregar ítems personalizados. Puede reordenar. Pero los 13 ítems base siempre están presentes.

**9. La calculadora solo da un número si ese número es exacto.**  
Para instrumentos no soportados o con pip value variable (XAU/USD, índices CFD, pares de JPY), la calculadora muestra un mensaje explicativo en lugar de un número aproximado sin advertencia.

**10. El idioma del usuario se aplica a toda la app instantáneamente.**  
Al cambiar de ES a EN (o viceversa), todos los textos de la interfaz cambian. No hay secciones que queden en el idioma anterior.

**11. El tema visual (oscuro / claro) se aplica a toda la app.**  
El dorado `#c8a96e` es el color de identidad. El verde musgo `#5a7a4a` es ganancia. El rojo ladrillo `#8a4a35` es pérdida. Estos colores no se alteran bajo ningún tema.

**12. Un usuario Free nunca pierde datos al llegar al límite.**  
Al llegar a 50 trades, el usuario Free no puede registrar trades nuevos, pero puede ver, editar y exportar todos sus trades existentes. El límite bloquea el registro, no el acceso a los datos.

---

## 12. Flujo Completo del Usuario: Desde el Registro Hasta el Uso Diario

### Registro (una sola vez)

```
1. El trader entra a app.tradeos.io
2. Ve la pantalla de login
3. Escribe su email → click en "Enviar acceso"
4. Recibe un email con un botón "Entrar a TradeOS"
5. Hace click en el email → abre la app → está autenticado
```

*Alternativa:* Click en "Continuar con Google" → autoriza en Google → entra directo.

---

### Onboarding (una sola vez, después del registro)

```
1. La app muestra la pantalla de bienvenida con el botón "Empezar"
2. PASO 1 — Perfil:
   → Escribe nombre o alias
   → Selecciona país, edad, nivel de experiencia
   → Click en "Siguiente"

3. PASO 2 — Cuentas:
   → Click en "Agregar cuenta"
   → Escribe el nombre de la firma (ej: "Funding Pips")
   → Ingresa el tamaño de la cuenta ($25,000)
   → Selecciona tipo: Fondeada
   → Selecciona instrumento: Forex o Futuros CME
   → Ingresa el drawdown máximo permitido (10%)
   → Click en "Agregar cuenta" de nuevo para la segunda cuenta (si la tiene)
   → Click en "Siguiente"

4. PASO 3 — Estilo:
   → Selecciona pares (chips): XAU/USD, GBP/USD, EUR/USD
   → Selecciona sesiones: London Open, New York
   → Selecciona estrategia: SMC / ICT
   → Ingresa % de riesgo por trade: 1%
   → Ingresa R:R mínimo: 2.0
   → Click en "Siguiente"

5. PASO 4 — Reglas:
   → Escribe sus reglas ("No opero si hay noticias en 15 minutos")
   → Click en "Siguiente"

6. PASO 5 — Horario:
   → Indica si tiene trabajo externo
   → Ingresa hora de dormir y hora de despertar
   → Click en "Siguiente"

7. PASO 6 — Checklist:
   → Ve los 13 ítems predeterminados activos
   → Puede agregar un ítem personalizado ("Revisé el calendario económico semanal")
   → Click en "Siguiente"

8. PASO 7 — Metas:
   → Click en "Agregar meta"
   → Nombre: "Computador nuevo" → Monto: $800
   → Click en "Siguiente"

9. PASO 8 — Fases:
   → Selecciona su fase actual: "🎯 Challenge"
   → Click en "Siguiente"

10. PASO 9 — Psicología:
    → Selecciona su error más frecuente: Revenge trading
    → Selecciona sus peores condiciones: Cansado / Presionado
    → Click en "Finalizar"

11. Animación de bienvenida → Dashboard
```

---

### Uso Diario (rutina real del trader)

**Mañana antes de operar — ~5 minutos**

```
1. Entra a app.tradeos.io (sesión activa, entra directo)
2. Dashboard → revisa:
   → Score de Consistencia y métricas de la semana
   → Estado de drawdown de cada cuenta (¿alguna en rojo?)
   → Resumen de la semana anterior
3. Va a Checklist
4. Marca los ítems que aplican al momento:
   → ✅ Dormí suficientes horas
   → ✅ Sin ansiedad por recuperar pérdidas
   → ✅ Revisé el bias del mercado
   → (deja sin marcar los que dependen del setup)
5. Escribe el bias del día:
   "XAU en rango en D1. Espero ruptura bajista después de manipulación en London.
   Niveles clave: 2310 resistencia, 2285 soporte. No opero antes de las 8AM EST."
6. Click en "Iniciar sesión" → activa el temporizador
```

**Durante la sesión — al ver un setup**

```
7. Ve un setup en el gráfico
8. Antes de ejecutar, completa el checklist:
   → ✅ El setup es A+ — no estoy forzando
   → ✅ El R:R mínimo está cumplido (tiene al menos 2.0R)
9. Va a la Calculadora:
   → Selecciona su cuenta de Funding Pips ($25,000)
   → Riesgo: 1%
   → SL: 25 pips en GBP/USD
   → Resultado: 0.10 lotes estándar ($250 en riesgo)
10. Ejecuta el trade en MetaTrader 5
```

**Después del trade — registro inmediato**

```
11. Cierra la plataforma o pone los stops
12. Va a "Registrar Trade" en la app
13. Completa el formulario:
    → Par: GBP/USD
    → Sesión: London Open
    → Cuenta: Funding Pips $25K
    → Resultado: Ganador ✅
    → P&L: $480
    → R:R real: 1.9
    → Riesgo %: 1
    → Estado emocional: Tranquilo
    → Reglas rotas: Ninguna
    → Tags: FVG, Order Block
    → Descripción: "Barrida de sellside en H1, FVG dejado en M15, entrada en 50% del imbalance"
14. Adjunta una captura del gráfico *(Pro)*
15. Tiene otra cuenta en E8 $50K con replicador activo:
    → Click en "Replicar a otras cuentas"
    → E8 $50K: P&L $960 (tamaño doble, mismo porcentaje de riesgo)
    → Click en "Guardar todo"
16. La app guarda:
    → Trade madre en Funding Pips con todos los datos
    → Trade hija en E8 con P&L de esa cuenta
17. Completa los ítems finales del checklist:
    → ✅ Calculé el lotaje exacto
    → ✅ Cerré la plataforma después del trade
    → ✅ Registré el trade en el journal
18. Click en "Cerrar sesión" en el checklist
    → El temporizador se detiene → guarda duración de la sesión
```

**Fin de semana — revisión semanal**

```
19. Entra al Dashboard
20. Ve el Resumen Semanal:
    → Esta semana: 8 trades, 75% WR, +$2,100 P&L
    → Racha limpia: 6 trades sin romper reglas
21. Va a Análisis de Patrones *(Pro)*:
    → Mejor par: GBP/USD → 80% WR
    → Mejor sesión: London Open → 78% WR
    → Patrón detectado: Win Rate 20% más alto cuando duerme +7 horas
22. Va a Mis Trades → filtra "Esta semana"
23. Revisa los 2 trades perdedores:
    → Click en "Editar" en el trade con P&L incorrecto → corrige el monto
24. Va a Fases del Plan:
    → Marca como completada la tarea "Primera semana sin romper ninguna regla"
25. Va a Gestión de Fondos:
    → Registra el payout recibido esta semana: "Payout Funding Pips" $800
```

---

## Apéndice: Principios de Producto No Negociables

Estos principios no están sujetos a debate en ninguna decisión de diseño o desarrollo.

1. **Sin publicidad, jamás.** Ningún banner, ningún contenido patrocinado, ningún producto de terceros promovido dentro de la app.

2. **Los datos del trader son del trader.** Los datos no se venden, no se usan para analytics de negocio sin opt-in explícito, no se comparten con terceros. El trader puede exportar todo y eliminar todo en cualquier momento.

3. **Sin modo para principiantes.** El usuario sabe qué es un pip, un drawdown y un R:R. La app no explica estos conceptos.

4. **El precio es justo.** $9/mes Pro es el precio correcto. No se sube arbitrariamente por presión de ingresos. Si el producto vale más, se añaden features que justifiquen el precio, no se sube el precio de lo que ya existe.

5. **La calidad genera la distribución.** El producto tiene que ser tan bueno que los traders lo recomienden sin incentivo. La retención es el mejor argumento de marketing.

---

*Documento generado por Product Manager — TradeOS*  
*Fecha: Junio 2026*  
*Próxima revisión: al inicio de Fase 2 de implementación o cuando se tomen las decisiones pendientes de §4*  
*Para el equipo de desarrollo: este documento es el contrato del producto — cualquier desviación debe consultarse con PM antes de implementar*
