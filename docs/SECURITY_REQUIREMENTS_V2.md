# SECURITY_REQUIREMENTS_V2.md
## TradeOS — Requisitos de Seguridad para Producción

**Versión objetivo:** 2.0  
**Fecha:** Junio 2026  
**Autor:** Security Auditor  
**Stack de referencia:** React + Vite + Supabase + Stripe + Resend + Vercel + Sentry  
**Fuentes:** MASTER_CONTEXT · SCHEMA · FUNCTIONS · LOCALSTORAGE_AUDIT · SECURITY_AUDIT · SAAS_ARCHITECTURE_PLAN

> Este documento es el checklist de seguridad que debe estar 100% completo antes del primer deploy a producción.
> Ningún requisito CRÍTICO puede estar pendiente en el momento del lanzamiento.
> Los requisitos ALTOS deben estar resueltos antes del lanzamiento o tener fecha de cierre firmada.
> Los MEDIOS son requisitos para el primer mes post-lanzamiento.

---

## LEYENDA

| Símbolo | Significado |
|---------|-------------|
| `[ ]` | Pendiente |
| `[x]` | Completo |
| 🔴 CRÍTICO | Bloquea el lanzamiento. Fix obligatorio antes de producción. |
| 🟠 ALTO | Debe cerrarse antes del lanzamiento o tener mitigación temporal documentada. |
| 🟡 MEDIO | Primer mes post-lanzamiento. Sin excepción. |

---

## 1. AUTENTICACIÓN

### 🔴 CRÍTICO

- [ ] **AUTH-C1** — Magic link configurado con expiración ≤ 15 minutos en Supabase Auth. Ningún link de acceso puede ser válido más allá de ese tiempo.
- [ ] **AUTH-C2** — Los magic links son de un solo uso. Después del primer click, el token se invalida independientemente de si el usuario completó el login.
- [ ] **AUTH-C3** — Las sesiones JWT tienen expiración configurada. El refresh token no puede ser indefinido. Configurar: access token 1 hora, refresh token 7 días.
- [ ] **AUTH-C4** — Todas las rutas de la aplicación están protegidas. Un usuario no autenticado que acceda a cualquier ruta de `/app/*` es redirigido a `/login` sin exponer datos.
- [ ] **AUTH-C5** — Google OAuth configurado con lista de permisos mínimos. Solo se solicita `email` y `profile`. No se solicitan permisos de Drive, Calendar ni ningún otro scope.
- [ ] **AUTH-C6** — El endpoint de login tiene rate limiting. Máximo 5 solicitudes de magic link por email por hora. El exceso retorna 429 sin revelar si el email existe.

### 🟠 ALTO

- [ ] **AUTH-A1** — Logout completo implementado. Al cerrar sesión: se invalida el JWT en Supabase, se limpia el store de Zustand, se limpia el caché de IndexedDB local, y se redirige a `/login`.
- [ ] **AUTH-A2** — Detección de sesión expirada en el cliente. Si el JWT expira durante el uso de la app, el usuario ve un modal de "Sesión expirada — reinicia sesión" en lugar de un error genérico o pantalla en blanco.
- [ ] **AUTH-A3** — La pantalla de login no revela si un email está registrado o no. El mensaje de confirmación es siempre el mismo: "Si ese email existe en nuestra plataforma, recibirás un link de acceso."
- [ ] **AUTH-A4** — Los tokens de magic link no se incluyen en logs del servidor ni en headers de referrer. Verificar configuración de Supabase para asegurar que los links no se loguean en texto plano.

### 🟡 MEDIO

- [ ] **AUTH-M1** — Página de dispositivos activos. El usuario puede ver desde qué dispositivos tiene sesión activa y puede cerrarlas remotamente.
- [ ] **AUTH-M2** — Email de notificación cuando se inicia sesión desde un dispositivo nuevo o una IP diferente a la habitual.

---

## 2. PERMISOS Y CONTROL DE ACCESO

### 🔴 CRÍTICO

- [ ] **PERM-C1** — Row Level Security (RLS) activado en **todas** las tablas de PostgreSQL: `profiles`, `accounts`, `trades`, `funds`, `checklists`, `phases`, `goals`, `custom_phases`. Sin excepción. Una tabla sin RLS es una filtración total de datos de todos los usuarios.
- [ ] **PERM-C2** — Políticas RLS verificadas para las 4 operaciones en cada tabla: SELECT, INSERT, UPDATE, DELETE. El patrón mínimo: `USING (auth.uid() = user_id)` en SELECT/UPDATE/DELETE, `WITH CHECK (auth.uid() = user_id)` en INSERT.
- [ ] **PERM-C3** — El bucket de Supabase Storage para screenshots tiene política de acceso por usuario. Un usuario autenticado solo puede leer, escribir y eliminar archivos dentro de su propio prefijo `/{user_id}/`.
- [ ] **PERM-C4** — Las Supabase Edge Functions validan el JWT del request antes de ejecutar cualquier operación. No existe ninguna Edge Function pública sin autenticación.
- [ ] **PERM-C5** — Feature gating por plan implementado en el backend, no solo en el frontend. La diferencia entre Free y Pro no puede bypassearse eliminando condiciones en el cliente. El servidor valida el campo `plan` del perfil antes de permitir operaciones que excedan los límites del plan gratuito.
- [ ] **PERM-C6** — La ruta `/admin` tiene autenticación separada. No basta con ser usuario autenticado — se requiere un rol `admin` verificado en el JWT o en la tabla `profiles`. Un usuario Pro no puede acceder al panel admin bajo ninguna circunstancia.

### 🟠 ALTO

- [ ] **PERM-A1** — Test de penetración básico antes del lanzamiento: intentar acceder a datos de otro usuario con JWT válido del propio usuario. El resultado debe ser 0 filas en todas las tablas.
- [ ] **PERM-A2** — Los límites del plan Free son validados por el servidor en cada INSERT: máximo 1 cuenta, máximo 50 trades. Si el cliente intenta insertar un trade número 51, el servidor retorna 403 con código de error específico, no un error genérico de base de datos.
- [ ] **PERM-A3** — No existe ninguna consulta SQL en el cliente que use `SELECT *` en tablas de usuarios. Cada consulta selecciona solo los campos que necesita.

### 🟡 MEDIO

- [ ] **PERM-M1** — Auditoría de políticas RLS ejecutada en entorno de staging con un usuario de prueba intentando operaciones cross-user. Resultado documentado y firmado antes del lanzamiento a producción.
- [ ] **PERM-M2** — Los webhooks de Stripe tienen validación de firma (`stripe.webhooks.constructEvent`). Un request falso a `/api/webhooks/stripe` no puede cambiar el plan de un usuario.

---

## 3. PROTECCIÓN DE DATOS

### 🔴 CRÍTICO

- [ ] **DATA-C1** — Los datos del trader en PostgreSQL no incluyen los campos muertos `balance`, `drawdown`, `pnl` del objeto `Account` de v1.x. El schema SQL de producción coincide exactamente con el schema documentado en SAAS_ARCHITECTURE_PLAN.md.
- [ ] **DATA-C2** — Ningún dato sensible del usuario (nombre, email, datos de trades) aparece en logs de Vercel, Supabase, Sentry o Resend en texto plano. Los logs contienen IDs y códigos de error, nunca contenido de usuario.
- [ ] **DATA-C3** — Las imágenes de trades NO se guardan en base64 en PostgreSQL. Van a Supabase Storage. La tabla `trades` guarda únicamente la URL del archivo. Este es un requisito de arquitectura: base64 en la BD degrada performance y viola el límite de tamaño de fila.
- [ ] **DATA-C4** — `saveAll()` (o su equivalente en v2.0) tiene manejo explícito de errores de persistencia. Si una operación de escritura falla (red, cuota, error de servidor), el usuario recibe un mensaje de error específico y el sistema no muestra confirmación de guardado.
- [ ] **DATA-C5** — Soft delete implementado en trades y fondos. Ningún dato se borra físicamente de la base de datos con operaciones de usuario. El campo `deleted_at` marca el registro como eliminado. El borrado físico solo ocurre por el proceso de retención de datos (ver RETENTION-C1).

### 🟠 ALTO

- [ ] **DATA-A1** — Validación de esquema en el servidor antes de cualquier INSERT o UPDATE. Si el cliente envía un objeto con campos inesperados o tipos incorrectos, el servidor retorna 400 con detalle del campo inválido. No se confía en la validación del frontend.
- [ ] **DATA-A2** — La función `importD()` / migración de v1.x valida la estructura del JSON antes de escribir cualquier dato. Si la validación falla en cualquier campo, se hace rollback completo. No existe sobreescritura parcial.
- [ ] **DATA-A3** — El campo `trade.ruleBroken` que en v1.x es un string delimitado por `, ` se almacena en PostgreSQL como `TEXT[]`. La conversión en `migrateFromV1Backup()` es la única transformación permitida — no se hacen transformaciones al importar backups de v2.x.
- [ ] **DATA-A4** — La relación madre-hija de trades tiene integridad referencial. `trade.mother_id` tiene `REFERENCES trades(id)`. Si se elimina una madre (soft delete), las hijas también se marcan con `deleted_at`. No existen hijas huérfanas.
- [ ] **DATA-A5** — Los IDs de todos los registros en v2.0 son UUIDs generados por el servidor. Ningún ID de negocio es `Date.now()` ni depende del cliente para ser único.

### 🟡 MEDIO

- [ ] **DATA-M1** — Política de privacidad publicada y enlazada desde la app antes del lanzamiento. Debe especificar: qué datos se guardan, dónde, por cuánto tiempo, y que nunca se venden ni se usan para analytics externos.
- [ ] **DATA-M2** — Vercel Analytics configurado en modo anonimizado. Sin IDs de usuario, sin IPs, solo métricas de uso agregadas. Verificar que el dashboard de Vercel no expone datos individuales.
- [ ] **DATA-M3** — Las variables de entorno de Sentry están configuradas para no capturar datos de usuario en los eventos de error. Configurar `beforeSend` para redactar emails y nombres antes de enviar a Sentry.

---

## 4. PAGOS (STRIPE)

### 🔴 CRÍTICO

- [ ] **PAY-C1** — TradeOS nunca almacena datos de tarjeta de crédito. Stripe maneja todo el procesamiento de pagos. La base de datos de TradeOS solo guarda: `stripe_customer_id`, `stripe_subscription_id`, `plan`, `plan_expires_at`.
- [ ] **PAY-C2** — Los webhooks de Stripe están validados con firma HMAC antes de procesar cualquier evento. Un request sin firma válida retorna 400 inmediatamente sin ejecutar ninguna lógica de negocio.
- [ ] **PAY-C3** — El cambio de plan de un usuario ocurre únicamente como resultado de un webhook de Stripe confirmado, nunca por una solicitud directa del cliente. La ruta `/api/upgrade` no existe — el upgrade se procesa en `/api/webhooks/stripe` al recibir `customer.subscription.updated`.
- [ ] **PAY-C4** — Los precios de los planes están definidos en Stripe Dashboard y referenciados por Price ID en el código. El monto nunca se envía desde el cliente. No existe ningún parámetro de `amount` en las solicitudes de checkout.
- [ ] **PAY-C5** — Las claves de Stripe están separadas por entorno: `STRIPE_SECRET_KEY` (live) solo existe en las variables de entorno de producción en Vercel. En staging y desarrollo se usan claves de test.

### 🟠 ALTO

- [ ] **PAY-A1** — El portal de clientes de Stripe está habilitado. El usuario puede gestionar su suscripción, cambiar método de pago, ver historial de facturas y cancelar, todo desde el portal de Stripe sin que TradeOS procese datos de pago.
- [ ] **PAY-A2** — El flujo de pago fallido está implementado. Si `invoice.payment_failed`, el usuario recibe email via Resend con link al portal de clientes. El plan no se revoca inmediatamente — Stripe tiene un período de gracia configurable (recomendado: 3 días).
- [ ] **PAY-A3** — El flujo de cancelación mantiene acceso Pro hasta la fecha de vencimiento del período pagado. Al cancelar, `plan_expires_at` se actualiza pero el plan sigue siendo `pro` hasta esa fecha.
- [ ] **PAY-A4** — Idempotencia en el procesamiento de webhooks. Si Stripe reenvía el mismo evento (comportamiento normal de Stripe en caso de timeout), el handler no procesa el mismo evento dos veces. Verificar el `event.id` contra una tabla de eventos procesados.

### 🟡 MEDIO

- [ ] **PAY-M1** — Notificación de renovación enviada 7 días antes del vencimiento del plan vía Resend.
- [ ] **PAY-M2** — El panel admin muestra: MRR actual, churn del mes, usuarios por plan. Sin acceso a datos de tarjeta ni datos personales de pago.
- [ ] **PAY-M3** — Reembolsos procesados solo manualmente desde el Dashboard de Stripe por Juan Pa. No existe endpoint de reembolso automático.

---

## 5. BACKUPS Y RECUPERACIÓN DE DATOS

### 🔴 CRÍTICO

- [ ] **BCK-C1** — Supabase tiene backups automáticos diarios habilitados (disponible desde el plan Pro de Supabase). El período de retención es mínimo 7 días. Antes del lanzamiento, verificar que la configuración de backup está activa y que existe al menos un backup de prueba exitoso.
- [ ] **BCK-C2** — La función de exportación de backup manual está disponible para todos los usuarios (Free y Pro). Un usuario puede descargar un JSON con todos sus datos en cualquier momento. Esta funcionalidad no puede estar detrás del paywall.
- [ ] **BCK-C3** — El backup JSON exportado en v2.0 incluye **todas** las claves sin excepción: trades, cuentas, fondos, fases, checklists, bias del día, metas. El bug de v1.x donde `checklists` se exportaba pero no se importaba está corregido y verificado.
- [ ] **BCK-C4** — El backup JSON incluye el campo `schema_version: '2.0'` para identificar la versión del schema y permitir migraciones futuras sin ambigüedad.

### 🟠 ALTO

- [ ] **BCK-A1** — El proceso de restauración desde backup JSON está implementado y probado end-to-end. La prueba mínima: exportar datos de un usuario de prueba → crear cuenta nueva → importar → verificar que los datos son idénticos.
- [ ] **BCK-A2** — La función `migrateFromV1Backup()` está implementada, documentada y probada con backups reales de usuarios de v1.x. La migración es transaccional: si falla en cualquier punto, no quedan datos parcialmente migrados.
- [ ] **BCK-A3** — Las imágenes en Supabase Storage están incluidas en el backup. El backup JSON contiene las URLs. La restauración descarga las imágenes desde Storage y las re-sube al bucket del usuario destino si el user_id cambia.
- [ ] **BCK-A4** — Existe un proceso documentado (runbook) para restaurar la base de datos completa desde un backup de Supabase. Juan Pa puede ejecutarlo sin asistencia técnica externa.

### 🟡 MEDIO

- [ ] **BCK-M1** — El backup automático para usuarios Pro se ejecuta semanalmente y se envía al email del usuario. El usuario tiene siempre una copia reciente fuera de TradeOS sin tener que recordar hacerlo manualmente.
- [ ] **BCK-M2** — El historial de bias del día (`tt_bias_*` en v1.x) está incluido en el backup de v2.0. Los usuarios que migren de v1.x no pierden ese historial si lo incluyeron en el JSON de exportación.

---

## 6. RECUPERACIÓN DE CUENTA

### 🔴 CRÍTICO

- [ ] **REC-C1** — La recuperación de cuenta es exclusivamente por email. El magic link enviado al email registrado es el único mecanismo de acceso. No existe "pregunta secreta" ni recovery code que pueda bypassear esto.
- [ ] **REC-C2** — Si el usuario pierde acceso a su email, el proceso de recuperación requiere verificación manual por Juan Pa. Este proceso está documentado en un runbook interno antes del lanzamiento. No existe recuperación automática sin acceso al email.
- [ ] **REC-C3** — Al registrar una cuenta, el usuario recibe un email de bienvenida que explica claramente que el email es su única llave de acceso y que debe mantenerlo accesible.

### 🟠 ALTO

- [ ] **REC-A1** — El proceso de cambio de email está implementado con doble confirmación: se envía un link de confirmación tanto al email actual como al email nuevo. El cambio solo ocurre si ambos confirman.
- [ ] **REC-A2** — La eliminación de cuenta está disponible en Ajustes. El proceso: confirmación por email → período de gracia de 30 días durante el cual el usuario puede cancelar → eliminación permanente de todos los datos. Se envía email de confirmación de eliminación definitiva.

### 🟡 MEDIO

- [ ] **REC-M1** — Los usuarios inactivos por 18 meses reciben un email de advertencia (ver SAAS_ARCHITECTURE_PLAN — DECISIÓN 11). Si no responden en 30 días, su cuenta y datos se eliminan definitivamente. Este proceso está automatizado vía cron en Supabase Edge Functions.

---

## 7. ALMACENAMIENTO DE SCREENSHOTS

### 🔴 CRÍTICO

- [ ] **IMG-C1** — Las imágenes de trades se almacenan en Supabase Storage, nunca en base64 en PostgreSQL. Este requisito no tiene excepción ni caso edge.
- [ ] **IMG-C2** — El bucket de Storage es privado. Las URLs de imágenes son URLs firmadas con expiración (signed URLs). Una URL de imagen de un usuario no es accesible por otro usuario ni por un visitante no autenticado, incluso si conoce la URL.
- [ ] **IMG-C3** — El prefijo de almacenamiento en el bucket es `/{user_id}/{trade_id}/screenshot.{ext}`. Un usuario no puede leer ni escribir fuera de su propio prefijo por política de RLS en Storage.
- [ ] **IMG-C4** — Límite de tamaño de imagen implementado en el cliente y en el servidor. Máximo 5MB por imagen. El cliente comprime antes de subir. El servidor rechaza uploads que excedan el límite con error descriptivo.
- [ ] **IMG-C5** — Validación de tipo de archivo en el servidor. Solo se aceptan `image/jpeg`, `image/png`, `image/webp`. Cualquier otro Content-Type es rechazado con 400.

### 🟠 ALTO

- [ ] **IMG-A1** — Las imágenes se comprimen antes del upload. Target: ≤ 500KB por imagen después de compresión. La calidad visual debe ser suficiente para ver el chart del trade.
- [ ] **IMG-A2** — Al eliminar un trade (soft delete), la imagen en Storage se mantiene hasta que se ejecute la limpieza de retención. Al eliminar definitivamente un trade o una cuenta, la imagen se elimina de Storage.
- [ ] **IMG-A3** — El storage de imágenes tiene cuota por usuario. Plan Free: máximo 50MB de imágenes. Plan Pro: máximo 2GB. El sistema muestra el uso actual y avisa al 80% de la cuota.

### 🟡 MEDIO

- [ ] **IMG-M1** — Las URLs firmadas de imágenes tienen expiración de 1 hora. Al renderizar la vista de trades, se generan URLs frescas. No se cachean URLs firmadas en el cliente por períodos largos.
- [ ] **IMG-M2** — Existe un proceso de limpieza de imágenes huérfanas en Storage (imágenes sin trade asociado). Se ejecuta como cron semanal.

---

## 8. PROTECCIÓN CONTRA PÉRDIDA DE DATOS

### 🔴 CRÍTICO

- [ ] **LOSS-C1** — Toda operación de escritura en Supabase tiene manejo de error explícito en el cliente. Si la operación falla, el usuario ve un mensaje de error específico. Nunca se muestra confirmación de guardado si la escritura no fue confirmada por el servidor.
- [ ] **LOSS-C2** — El modo offline-first con IndexedDB está implementado. Si el usuario pierde conexión durante una sesión, sus cambios se guardan localmente en IndexedDB y se sincronizan con Supabase cuando la conexión se restaura. El usuario ve indicador de estado de sincronización en la UI.
- [ ] **LOSS-C3** — Conflictos de sincronización se resuelven con política de "última escritura gana" con timestamp del servidor (no del cliente). El cliente no puede manipular el timestamp para ganar conflictos.
- [ ] **LOSS-C4** — La eliminación de trades y fondos requiere confirmación explícita del usuario. La acción de eliminar no es reversible desde la UI (aunque el registro permanece en soft delete en la BD). No existe "eliminar con un click accidental".

### 🟠 ALTO

- [ ] **LOSS-A1** — El formulario de registro de trade no se limpia hasta que el servidor confirma el guardado. Si el usuario pierde conexión después de llenar el formulario pero antes de confirmar, los datos del formulario persisten en IndexedDB y se pueden recuperar al reconectar.
- [ ] **LOSS-A2** — El sistema detecta modo incógnito (donde IndexedDB no persiste entre sesiones) y muestra una advertencia visible al usuario: "Estás en modo incógnito — tus datos no se guardarán localmente si pierdes conexión."
- [ ] **LOSS-A3** — Múltiples pestañas del mismo usuario se sincronizan via Supabase Realtime. Si el usuario abre dos pestañas, los cambios en una se reflejan en la otra sin sobreescritura silenciosa.

### 🟡 MEDIO

- [ ] **LOSS-M1** — El indicador de estado de sincronización es visible en el sidebar o header en todo momento: ✓ Sincronizado / ⟳ Sincronizando / ⚠ Sin conexión — guardado local.
- [ ] **LOSS-M2** — El usuario puede forzar una sincronización manual desde Ajustes si sospecha que sus datos no están sincronizados.

---

## 9. LOGS Y AUDITORÍA

### 🔴 CRÍTICO

- [ ] **LOG-C1** — Los logs del servidor no contienen datos de usuario en texto plano: sin emails, sin nombres, sin contenido de trades, sin notas. Los logs usan user_id (UUID) como identificador, no el email.
- [ ] **LOG-C2** — Los logs de Stripe Webhooks registran: event_id, event_type, user_id, timestamp, resultado (success/failure). No registran datos de tarjeta ni montos en exceso de lo necesario para debugging.
- [ ] **LOG-C3** — Sentry está configurado para redactar datos sensibles antes de enviar eventos. La configuración de `beforeSend` filtra: emails, nombres, contenido de campos de texto libre (notas de trades, bias del día).

### 🟠 ALTO

- [ ] **LOG-A1** — Los eventos de autenticación se loguean: intento de login, login exitoso, login fallido, logout, sesión expirada. Con user_id y timestamp. Sin contenido del magic link.
- [ ] **LOG-A2** — Las operaciones críticas de negocio se loguean: creación de trade, eliminación de trade, cambio de plan, migración de backup. Con user_id, operación, timestamp y resultado.
- [ ] **LOG-A3** — Los logs tienen retención definida: 30 días para logs de aplicación, 90 días para logs de seguridad (auth, cambios de plan). Los logs más antiguos se eliminan automáticamente.

### 🟡 MEDIO

- [ ] **LOG-M1** — Panel de actividad reciente disponible para Juan Pa en `/admin`: últimos registros, usuarios activos hoy, errores de las últimas 24h, webhooks de Stripe procesados.
- [ ] **LOG-M2** — Alertas configuradas en Sentry para: tasa de errores > 1% en 5 minutos, errores críticos en Edge Functions, fallos en webhooks de Stripe.

---

## 10. MONITOREO

### 🔴 CRÍTICO

- [ ] **MON-C1** — Sentry instalado y verificado en producción antes del lanzamiento. La verificación mínima: generar un error controlado en producción y confirmar que llega a Sentry con el contexto correcto.
- [ ] **MON-C2** — Uptime monitoring activo para `app.tradeos.io`. Alerta inmediata a Juan Pa si la app cae más de 2 minutos. Herramienta: Better Uptime (plan gratuito) o UptimeRobot.
- [ ] **MON-C3** — Alertas de Stripe configuradas para: pago fallido, disputa abierta (chargeback), webhook con fallo repetido (> 3 intentos). Las alertas llegan al email de Juan Pa en tiempo real.

### 🟠 ALTO

- [ ] **MON-A1** — Métricas de Supabase monitoreadas: uso de base de datos, uso de Storage, número de conexiones activas. Alerta si el uso de Storage supera el 80% del plan.
- [ ] **MON-A2** — Health check endpoint implementado en `/api/health` que verifica: conexión a Supabase, conexión a Stripe (ping). Retorna 200 si todo está bien, 503 con detalle si algo falla. El uptime monitor usa este endpoint.
- [ ] **MON-A3** — Dashboard operacional básico para Juan Pa: usuarios activos (DAU/MAU), trades registrados hoy, errores en las últimas 24h, MRR. Sin datos individuales de usuarios.

### 🟡 MEDIO

- [ ] **MON-M1** — Performance monitoring: tiempo de carga del dashboard, tiempo de respuesta de las queries más frecuentes. Alerta si el P95 supera 3 segundos.
- [ ] **MON-M2** — Monitoreo de cuota de Supabase: filas por tabla, tamaño de Storage por usuario. Alertas al 70% y 90% del límite del plan.

---

## 11. VARIABLES DE ENTORNO

### 🔴 CRÍTICO

- [ ] **ENV-C1** — Ninguna clave secreta está hardcodeada en el código fuente. Las siguientes variables **siempre** vienen de variables de entorno, nunca del código: `SUPABASE_SERVICE_ROLE_KEY`, `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `RESEND_API_KEY`, `SENTRY_DSN`.
- [ ] **ENV-C2** — Las variables de entorno están separadas por entorno: `.env.local` (desarrollo local, nunca en git), variables de Vercel para staging y producción con valores distintos. Las claves de producción nunca se usan en desarrollo.
- [ ] **ENV-C3** — El repositorio GitHub tiene `.env*` en `.gitignore` y branch protection configurado. Un push accidental de claves al repo dispara una alerta y la clave se revoca inmediatamente.
- [ ] **ENV-C4** — La `SUPABASE_SERVICE_ROLE_KEY` (que bypasea RLS) solo existe en Edge Functions del servidor. Nunca está disponible en el cliente. La clave pública de Supabase (`SUPABASE_ANON_KEY`) es la única que puede estar en el frontend.
- [ ] **ENV-C5** — Stripe usa claves separadas por entorno: `sk_test_*` en desarrollo/staging, `sk_live_*` solo en producción. El código verifica el entorno antes de procesar webhooks de Stripe.

### 🟠 ALTO

- [ ] **ENV-A1** — Lista completa de variables de entorno requeridas documentada en `README.md` con descripción de cada una. Un desarrollador nuevo puede levantar el entorno local sin preguntar.
- [ ] **ENV-A2** — Rotación de claves documentada como proceso: cuándo rotar, cómo, y quién tiene acceso. Las claves de Stripe, Resend y Sentry se rotan si hay cualquier sospecha de exposición.
- [ ] **ENV-A3** — El `SUPABASE_ANON_KEY` que se expone en el frontend está en el repositorio como variable pública (es el comportamiento esperado de Supabase), pero está documentado claramente que esta clave sin RLS activo sería una vulnerabilidad crítica. La seguridad depende de RLS, no de la secrecía de esta clave.

---

## 12. DESPLIEGUE

### 🔴 CRÍTICO

- [ ] **DEP-C1** — HTTPS obligatorio en todas las rutas. Vercel lo provee por defecto. Verificar que no existen redirecciones HTTP que filtren el JWT en headers.
- [ ] **DEP-C2** — Headers de seguridad configurados en `vercel.json`: `Content-Security-Policy`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy`.
- [ ] **DEP-C3** — Content Security Policy (CSP) configurada para bloquear scripts inline no autorizados y conexiones a dominios no whitelistados. Dominios permitidos: `supabase.co`, `stripe.com`, `fonts.googleapis.com`, `fonts.gstatic.com`. Ningún otro dominio externo.
- [ ] **DEP-C4** — Las migrations de base de datos se ejecutan en staging y se verifican antes de ejecutarlas en producción. No existe migración que se ejecute directamente en producción sin pasar por staging.
- [ ] **DEP-C5** — El dominio `app.tradeos.io` tiene SSL con renovación automática. Verificar que el certificado tiene al menos 60 días de vigencia en el momento del lanzamiento.
- [ ] **DEP-C6** — Branch protection en `main` configurado en GitHub: se requiere Pull Request + revisión antes de merge. Ningún commit va directo a `main`. Los deploys a producción son solo desde `main`.

### 🟠 ALTO

- [ ] **DEP-A1** — Pipeline de CI/CD configurado: en cada PR se ejecutan tests, lint y build. Solo si todos pasan se permite el merge. El deploy a producción es automático desde `main` solo si el build pasa.
- [ ] **DEP-A2** — Entorno de staging idéntico al de producción (`staging.tradeos.io` o preview de Vercel). Toda feature pasa por staging antes de producción. Los usuarios de prueba usan staging, no producción.
- [ ] **DEP-A3** — Rollback documentado: cómo revertir un deploy fallido en Vercel (botón "Redeploy" de la versión anterior) y cómo revertir una migración de base de datos. El proceso toma menos de 15 minutos.
- [ ] **DEP-A4** — El código del email de soporte (`tradeossoporte@gmail.com`) está en una variable de entorno, no hardcodeado. Permite cambiar el email de soporte sin modificar código.

### 🟡 MEDIO

- [ ] **DEP-M1** — Preview deployments de Vercel para cada PR. El equipo puede revisar cambios de UI antes del merge sin levantar entornos locales.
- [ ] **DEP-M2** — Dependencias del proyecto auditadas antes del lanzamiento con `npm audit`. Sin vulnerabilidades de severidad alta o crítica en dependencias directas.
- [ ] **DEP-M3** — Política de actualizaciones de dependencias: revisión mensual de `npm outdated`. Parches de seguridad se aplican en menos de 7 días desde su publicación.

---

## CHECKLIST DE LANZAMIENTO — RESUMEN EJECUTIVO

Antes de hacer cualquier deploy a producción verificar que:

### Todos los CRÍTICOS están en `[x]`

| Área | Requisitos CRÍTICOS |
|------|---------------------|
| Autenticación | AUTH-C1 al AUTH-C6 (6 requisitos) |
| Permisos | PERM-C1 al PERM-C6 (6 requisitos) |
| Datos | DATA-C1 al DATA-C5 (5 requisitos) |
| Pagos | PAY-C1 al PAY-C5 (5 requisitos) |
| Backups | BCK-C1 al BCK-C4 (4 requisitos) |
| Recuperación | REC-C1 al REC-C3 (3 requisitos) |
| Screenshots | IMG-C1 al IMG-C5 (5 requisitos) |
| Pérdida de datos | LOSS-C1 al LOSS-C4 (4 requisitos) |
| Logs | LOG-C1 al LOG-C3 (3 requisitos) |
| Monitoreo | MON-C1 al MON-C3 (3 requisitos) |
| Variables de entorno | ENV-C1 al ENV-C5 (5 requisitos) |
| Despliegue | DEP-C1 al DEP-C6 (6 requisitos) |
| **TOTAL CRÍTICOS** | **55 requisitos** |

**Un solo CRÍTICO pendiente = el lanzamiento no ocurre.**

---

### Todos los ALTOS tienen mitigación documentada o están en `[x]`

Si algún requisito ALTO no puede cerrarse antes del lanzamiento, debe existir:
1. Documentación del riesgo residual
2. Plan de cierre con fecha comprometida (máximo 2 semanas post-lanzamiento)
3. Aprobación explícita de Juan Pa

---

### Prueba de fuego mínima pre-lanzamiento

Los siguientes flujos deben ejecutarse manualmente en producción antes de abrir el registro:

1. **Registro completo:** email → magic link → onboarding → dashboard
2. **Registro de trade con imagen:** formulario → upload → guardado → vista en Mis Trades
3. **Replicador:** trade madre → réplicas hijas → verificar vínculos en BD
4. **Backup:** exportar JSON → crear cuenta nueva → importar → verificar que todos los datos coinciden
5. **Pago:** checkout Pro → webhook → plan actualizado → acceso a features Pro
6. **Cancelación:** cancelar plan → acceso hasta fin de período → degradación a Free
7. **Seguridad básica:** con JWT de usuario A, intentar leer datos del usuario B → debe retornar 0 filas en todas las tablas

---

*Documento generado por Security Auditor — TradeOS*  
*Fecha: Junio 2026*  
*Estado: Requisitos identificados — pendiente de implementación*  
*Próxima revisión: Al inicio de Fase 3 (Monetización) para verificar avance*  
*Para Juan Pa: los requisitos CRÍTICOS no son negociables. Los ALTOS tienen margen de negociación solo con mitigación documentada.*
