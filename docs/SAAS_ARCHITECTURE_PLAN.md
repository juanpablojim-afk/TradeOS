# SAAS_ARCHITECTURE_PLAN.md
## TradeOS — Plan de Arquitectura SaaS

**Versión:** 1.0  
**Fecha:** Junio 2026  
**Autor:** CTO  
**Basado en:** MASTER_CONTEXT · SCHEMA · FUNCTIONS · LOCALSTORAGE_AUDIT · SECURITY_AUDIT · BUSINESS_VISION · EXECUTIVE_REPORT

> Este documento es la arquitectura de decisiones. No hay una línea de código aquí.
> Todo lo que se construya en v2.0 debe consultarse aquí primero.
> Las decisiones marcadas como **CERRADA** no se reabren sin aprobación de Juan Pa + CTO.

---

## PRINCIPIOS DE DISEÑO NO NEGOCIABLES

Antes de cualquier decisión técnica, estos principios gobiernan la arquitectura completa:

**1. Los datos del trader son del trader.**
El modelo de datos en la nube replica la filosofía local: nada se comparte entre usuarios, nada se vende, nada se usa para analytics de negocio sin opt-in explícito.

**2. La experiencia offline-first no desaparece.**
El frontend debe funcionar sin conexión y sincronizar cuando hay red. Un trader en London Open a las 2AM no puede depender de la latencia de un servidor para registrar un trade.

**3. La migración de v1.x a v2.0 no puede pedir al trader que empiece de cero.**
Cada usuario con datos en localStorage debe poder importarlos a la nube en un click. La migración es parte del producto, no una nota al pie.

**4. Complejidad solo donde agrega valor.**
No se introduce infraestructura por sofisticación técnica. Cada componente nuevo debe justificar su costo operacional, su costo de mantenimiento y su valor para el usuario.

**5. El MVP de SaaS debe funcionar con una persona operándolo.**
Juan Pa solo. La arquitectura debe ser operable sin un equipo de DevOps.

---

## VISIÓN GENERAL DE LA ARQUITECTURA

```
┌─────────────────────────────────────────────────────────────────┐
│                        USUARIOS                                  │
│         Trader Free │ Trader Pro │ Trader Teams                  │
└──────────────────────────────┬──────────────────────────────────┘
                               │ HTTPS
┌──────────────────────────────▼──────────────────────────────────┐
│                      FRONTEND (SPA)                              │
│          app.tradeos.io — React + Vite — Vercel                  │
│    (Lógica de negocio actual migrada de vanilla JS a React)      │
└──────────────────────────────┬──────────────────────────────────┘
                               │ HTTPS / WebSocket
┌──────────────────────────────▼──────────────────────────────────┐
│                   BACKEND AS A SERVICE                           │
│                    Supabase (PostgreSQL)                         │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────────────────┐   │
│  │    Auth     │ │  Database   │ │        Storage           │   │
│  │ Magic Link  │ │ Row Level   │ │  Trade screenshots       │   │
│  │ Google OAuth│ │ Security    │ │  Supabase Storage        │   │
│  └─────────────┘ └─────────────┘ └──────────────────────────┘   │
└──────────────────────────────┬──────────────────────────────────┘
                               │
┌──────────────────────────────▼──────────────────────────────────┐
│                    SERVICIOS EXTERNOS                            │
│  ┌─────────────┐ ┌─────────────┐ ┌────────────┐                 │
│  │   Stripe    │ │  Resend     │ │  Sentry    │                 │
│  │  Pagos y    │ │  Emails     │ │  Error     │                 │
│  │ suscripción │ │transaccional│ │ monitoring │                 │
│  └─────────────┘ └─────────────┘ └────────────┘                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## DECISIÓN 1 — STACK TECNOLÓGICO

**CERRADA.**

### Frontend

| Componente | Tecnología | Justificación |
|------------|-----------|---------------|
| Framework | **React 18 + Vite** | La lógica de negocio de TradeOS ya está estructurada en funciones puras. Migrar a React es mapear esas funciones a componentes. Vite da HMR rápido y build optimizado. No hay razón para un framework más pesado. |
| Estilo | **CSS Variables + Tailwind (utilidades)** | Las variables CSS actuales de TradeOS (`--bg`, `--accent`, `--win`, `--loss`) se mantienen intactas como design tokens. Tailwind solo para spacing y layout — nunca para colores ni tipografía del sistema de diseño. |
| Estado | **Zustand** | Estado global simple, sin boilerplate. Reemplaza las variables globales actuales (`cfg`, `trades[]`, etc.). Un store por dominio. |
| Persistencia offline | **IndexedDB via Dexie.js** | Reemplaza localStorage para datos locales. Sin límite de 5-10MB. Soporte para imágenes binarias sin base64. Dexie es una capa simple sobre IndexedDB. |
| Sync cloud | **Supabase Realtime + React Query** | React Query maneja el caché, la revalidación y el estado de sincronización. Supabase Realtime para actualizaciones en tiempo real en Teams. |
| Routing | **React Router v6** | Rutas declarativas. Protección de rutas por plan. |
| Testing | **Vitest + Testing Library** | Co-ubicado con Vite. |
| Deploy | **Vercel** | Deploy automático desde GitHub. CDN global. Preview por PR. Gratis para el volumen inicial. |

### Backend

| Componente | Tecnología | Justificación |
|------------|-----------|---------------|
| Base de datos | **Supabase (PostgreSQL)** | PostgreSQL managed. RLS (Row Level Security) nativo — cada usuario ve solo sus datos por política en la BD, no por lógica en el cliente. Realtime incluido. Auth incluido. Storage incluido. No requiere servidor propio. Plan gratuito para el MVP. |
| Auth | **Supabase Auth** | Magic link por email (sin contraseña). Google OAuth como opción adicional. JWT incluido. No hay que construir ni mantener auth. |
| Storage de imágenes | **Supabase Storage** | Bucket privado por usuario para screenshots de trades. CDN incluido. Políticas de acceso por RLS. Reemplaza base64 en la base de datos. |
| Edge functions | **Supabase Edge Functions (Deno)** | Solo para lógica que no puede estar en el cliente: webhooks de Stripe, envío de emails via Resend. No se usa para lógica de negocio que ya funciona en el frontend. |

### Servicios externos

| Servicio | Uso | Alternativa descartada |
|----------|-----|----------------------|
| **Stripe** | Suscripciones Pro y Teams, pagos únicos, portal de clientes | Gumroad Memberships — menos control, sin portal de billing en app |
| **Resend** | Emails transaccionales (magic link, recibo de pago, alerta de renovación) | SendGrid — más caro, más complejo para el volumen inicial |
| **Sentry** | Error monitoring en producción | Datadog — excesivo para el volumen inicial |
| **Vercel Analytics** | Métricas de uso anonimizadas (sin datos del trader) | Google Analytics — viola el principio de privacidad del producto |

---

## DECISIÓN 2 — BASE DE DATOS (SCHEMA SQL)

**CERRADA.**

Migración directa del schema de localStorage documentado en SCHEMA.md a PostgreSQL. Cada campo muerto del objeto `Account` (balance, drawdown, pnl) se elimina. Los tipos se normalizan.

### Convenciones

- Todas las tablas tienen `user_id UUID REFERENCES auth.users(id)` con RLS.
- IDs: `UUID` generados por el servidor (reemplaza `Date.now()`).
- Timestamps: `TIMESTAMPTZ` en UTC.
- Soft delete en trades y fondos: `deleted_at TIMESTAMPTZ NULL` (nunca borrado físico).
- Las imágenes NO van en la base de datos. Van en Supabase Storage. La tabla guarda solo la URL.

---

### Tabla: `profiles`

Perfil del trader. Extiende `auth.users` de Supabase.

```sql
CREATE TABLE profiles (
  id              UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  
  -- Identidad
  name            TEXT NOT NULL DEFAULT 'Trader',
  alias           TEXT,
  age             SMALLINT,
  country         TEXT,
  
  -- Estilo de trading
  experience      TEXT CHECK (experience IN ('beginner','inter','advanced','pro')),
  risk_profile    TEXT CHECK (risk_profile IN ('conservative','moderate','aggressive')),
  risk_pct        NUMERIC(4,2) DEFAULT 1.0,
  rr_min          NUMERIC(4,2) DEFAULT 1.8,
  max_trades      TEXT DEFAULT '2',
  strategy        TEXT,
  platform        TEXT DEFAULT 'MT5',
  sessions        TEXT[],
  pairs           TEXT[],
  rules           TEXT[],
  
  -- Horario
  has_job         BOOLEAN DEFAULT false,
  job_hours       TEXT,
  days_off        TEXT,
  sleep_time      TEXT,
  wake_time       TEXT,
  
  -- Finanzas
  monthly_expenses NUMERIC(10,2) DEFAULT 0,
  
  -- Psicología
  main_error      TEXT,
  bad_conditions  TEXT[],
  motivation      TEXT,
  
  -- Checklist
  cl_items        TEXT[],
  custom_cl_items TEXT[],
  
  -- Sistema
  lang            TEXT DEFAULT 'es' CHECK (lang IN ('es','en')),
  plan            TEXT DEFAULT 'free' CHECK (plan IN ('free','pro','teams')),
  plan_expires_at TIMESTAMPTZ,
  onboarding_done BOOLEAN DEFAULT false,
  
  created_at      TIMESTAMPTZ DEFAULT NOW(),
  updated_at      TIMESTAMPTZ DEFAULT NOW()
);

-- RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_profile" ON profiles
  USING (auth.uid() = id);
```

---

### Tabla: `accounts`

Cuentas de trading del usuario.

```sql
CREATE TABLE accounts (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  
  firm        TEXT NOT NULL,
  type        TEXT NOT NULL CHECK (type IN ('own','funded')),
  instrument  TEXT NOT NULL DEFAULT 'forex' CHECK (instrument IN ('forex','futures')),
  size        NUMERIC(12,2) NOT NULL,
  max_dd      NUMERIC(5,2) NOT NULL DEFAULT 10,
  phase       TEXT DEFAULT 'active' CHECK (phase IN (
                'active','challenge1','challenge2','funded','paused','evaluation')),
  
  -- Drawdown diario opcional (feature v1.1)
  max_daily_dd NUMERIC(5,2),
  
  is_archived BOOLEAN DEFAULT false,
  sort_order  SMALLINT DEFAULT 0,
  
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

-- Índice para queries frecuentes
CREATE INDEX accounts_user_id ON accounts(user_id);

-- RLS
ALTER TABLE accounts ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_accounts" ON accounts
  USING (auth.uid() = user_id);
```

**Nota:** Los campos `balance`, `drawdown`, `pnl` del objeto Account original se eliminan. Son campos muertos en v1.x — el P&L se calcula en runtime agregando trades.

---

### Tabla: `trades`

Trade individual. Corazón del sistema.

```sql
CREATE TABLE trades (
  id           UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id      UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  account_id   UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
  
  -- Datos del trade
  traded_at    TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  pair         TEXT NOT NULL,
  session      TEXT CHECK (session IN ('London','New York','Asia','Overlap')),
  result       TEXT NOT NULL CHECK (result IN ('win','loss','be')),
  pnl          NUMERIC(10,2) NOT NULL DEFAULT 0,
  rr_real      NUMERIC(6,2) DEFAULT 0,
  risk_pct     NUMERIC(4,2),
  mood         TEXT CHECK (mood IN ('calm','confident','anxious','pressure','tired','revenge')),
  
  -- Campos opcionales (v1.2)
  entry_price  NUMERIC(12,5),
  stop_loss    NUMERIC(12,5),
  take_profit  NUMERIC(12,5),
  rr_planned   NUMERIC(6,2),  -- calculado: (TP - entry) / (entry - SL)
  
  -- Texto libre
  note         TEXT,
  notebook     TEXT,          -- Solo en trades madre
  rules_broken TEXT[],        -- Array de reglas, no string separado por comas
  tags         TEXT[],
  
  -- Imagen
  screenshot_url TEXT,        -- URL de Supabase Storage. NULL si no tiene imagen.
  
  -- Replicador
  is_mother    BOOLEAN DEFAULT true,
  is_child     BOOLEAN DEFAULT false,
  mother_id    UUID REFERENCES trades(id) ON DELETE CASCADE,
  -- ON DELETE CASCADE: al borrar la madre, se borran las hijas automáticamente (DB, no código)
  
  -- Auditoría
  deleted_at   TIMESTAMPTZ,   -- Soft delete. NULL = activo.
  edited_at    TIMESTAMPTZ,   -- NULL si nunca fue editado.
  created_at   TIMESTAMPTZ DEFAULT NOW()
);

-- Índices para las queries más frecuentes
CREATE INDEX trades_user_id ON trades(user_id);
CREATE INDEX trades_account_id ON trades(account_id);
CREATE INDEX trades_traded_at ON trades(traded_at DESC);
CREATE INDEX trades_mother_id ON trades(mother_id) WHERE mother_id IS NOT NULL;
CREATE INDEX trades_active ON trades(user_id, deleted_at) WHERE deleted_at IS NULL;

-- RLS
ALTER TABLE trades ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_trades" ON trades
  USING (auth.uid() = user_id);
```

**Nota crítica:** `mother_id` con `ON DELETE CASCADE` en la base de datos resuelve el bug BL-4 del EXECUTIVE_REPORT a nivel de infraestructura. Al borrar una madre, PostgreSQL borra las hijas automáticamente. No se necesita lógica en el cliente.

**Nota sobre `rules_broken`:** Se migra de string `'Regla 1, Regla 2'` a `TEXT[]`. Las queries de patrones se vuelven triviales con `ANY(rules_broken)`.

---

### Tabla: `funds`

Movimientos de fondos del trader.

```sql
CREATE TABLE funds (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  account_id  UUID REFERENCES accounts(id) ON DELETE SET NULL,
  
  type        TEXT NOT NULL CHECK (type IN (
                'own_payout','funded_payout','salary','goal',
                'challenge','expense','other')),
  amount      NUMERIC(10,2) NOT NULL,
  note        TEXT,
  
  transacted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at  TIMESTAMPTZ,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX funds_user_id ON funds(user_id);

ALTER TABLE funds ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_funds" ON funds
  USING (auth.uid() = user_id);
```

---

### Tabla: `goals`

Metas financieras del trader.

```sql
CREATE TABLE goals (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  
  name        TEXT NOT NULL,
  target      NUMERIC(10,2) NOT NULL,
  saved       NUMERIC(10,2) DEFAULT 0,
  
  sort_order  SMALLINT DEFAULT 0,
  is_active   BOOLEAN DEFAULT true,
  
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE goals ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_goals" ON goals
  USING (auth.uid() = user_id);
```

---

### Tabla: `phases`

Estado de las fases del trader.

```sql
CREATE TABLE phases (
  id            UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id       UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  
  phase_key     TEXT NOT NULL,  -- 'discipline','challenge','funded','scaling','own_capital','goals' o custom
  is_custom     BOOLEAN DEFAULT false,
  
  -- Solo para fases custom
  icon          TEXT,
  name          TEXT,
  subtitle      TEXT,
  description   TEXT,
  tasks         TEXT[],
  
  -- Estado
  task_progress BOOLEAN[],      -- Array parallel a tasks[], true = completada
  is_completed  BOOLEAN DEFAULT false,
  completed_at  TIMESTAMPTZ,
  sort_order    SMALLINT DEFAULT 0,
  
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  updated_at    TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(user_id, phase_key)
);

ALTER TABLE phases ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_phases" ON phases
  USING (auth.uid() = user_id);
```

---

### Tabla: `checklists`

Estado del checklist diario. Una fila por día por usuario.

```sql
CREATE TABLE checklists (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id     UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  
  date        DATE NOT NULL,
  items       JSONB NOT NULL DEFAULT '{}',
  -- Ejemplo: {"sleep_c": true, "no_rev_c": true, "bias_c": false}
  bias_note   TEXT,
  session_duration_minutes INTEGER,  -- duración de la sesión si se usó el timer
  post_session_note TEXT,            -- nota post-sesión (feature v1.2)
  
  created_at  TIMESTAMPTZ DEFAULT NOW(),
  updated_at  TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(user_id, date)
);

CREATE INDEX checklists_user_date ON checklists(user_id, date DESC);

ALTER TABLE checklists ENABLE ROW LEVEL SECURITY;
CREATE POLICY "users_own_checklists" ON checklists
  USING (auth.uid() = user_id);
```

**Nota:** `bias_note` y `session_duration_minutes` se incorporan directamente aquí. Resuelve el bug de bias no incluido en backup y la fragmentación de claves `tt_bias_[fecha]` en localStorage.

---

### Tabla: `subscriptions`

Estado de suscripción del usuario. Sincronizada desde Stripe via webhook.

```sql
CREATE TABLE subscriptions (
  id                  UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id             UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  
  stripe_customer_id  TEXT UNIQUE,
  stripe_sub_id       TEXT UNIQUE,
  plan                TEXT NOT NULL DEFAULT 'free' CHECK (plan IN ('free','pro','teams')),
  status              TEXT NOT NULL DEFAULT 'active' 
                        CHECK (status IN ('active','trialing','past_due','canceled','incomplete')),
  
  current_period_start TIMESTAMPTZ,
  current_period_end   TIMESTAMPTZ,
  cancel_at_period_end BOOLEAN DEFAULT false,
  
  -- Para plan Teams
  team_id             UUID,  -- referencia a teams (tabla futura)
  seats               SMALLINT DEFAULT 1,
  
  created_at          TIMESTAMPTZ DEFAULT NOW(),
  updated_at          TIMESTAMPTZ DEFAULT NOW(),
  
  UNIQUE(user_id)
);

-- Esta tabla NO tiene RLS del usuario — solo se escribe desde Edge Functions.
-- El frontend la lee via una función RPC que retorna solo los datos del usuario autenticado.
```

---

### Tabla: `teams` (v2.1)

Para el plan Teams. No se construye en v2.0.

```sql
-- Placeholder — diseño en v2.1
CREATE TABLE teams (
  id          UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  owner_id    UUID NOT NULL REFERENCES profiles(id),
  name        TEXT NOT NULL,
  slug        TEXT UNIQUE NOT NULL,
  max_seats   SMALLINT DEFAULT 10,
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE team_members (
  team_id     UUID REFERENCES teams(id) ON DELETE CASCADE,
  user_id     UUID REFERENCES profiles(id) ON DELETE CASCADE,
  role        TEXT DEFAULT 'member' CHECK (role IN ('owner','admin','member')),
  joined_at   TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (team_id, user_id)
);
```

---

## DECISIÓN 3 — AUTENTICACIÓN

**CERRADA.**

### Método principal: Magic Link por email

Sin contraseña. El trader escribe su email, recibe un link de acceso válido por 1 hora, hace click y está dentro. La sesión dura 7 días con refresh automático.

**Por qué no contraseña:** El ICP no es un desarrollador. "Olvidé mi contraseña" es fricción innecesaria para un producto que se usa a las 2AM antes de London Open. Magic link elimina ese problema por completo.

### Método secundario: Google OAuth

Un click. Recomendado en el onboarding. La mayoría de los traders tienen Gmail.

### Flujo completo de auth

```
REGISTRO / LOGIN (mismo flujo):
─────────────────────────────────
1. Usuario escribe email en /login
2. Frontend llama supabase.auth.signInWithOtp({ email })
3. Supabase envía email via Resend con magic link
4. Usuario hace click en el link
5. Supabase verifica el token, crea sesión, retorna JWT
6. Frontend recibe el JWT, lo almacena en memoria (no localStorage)
7. Si es primera vez: redirigir a /onboarding
8. Si tiene cuenta: redirigir a /dashboard

SESIÓN:
─────────
- JWT almacenado en memoria (no localStorage, no cookies sin Secure+HttpOnly)
- Supabase SDK maneja refresh automático del JWT
- Sesión expira después de 7 días de inactividad
- En refresh de página: Supabase recupera la sesión del cookie HttpOnly que maneja automáticamente

LOGOUT:
─────────
- supabase.auth.signOut()
- JWT invalidado en servidor
- Estado local limpiado
- Redirigir a /login
```

### Protección de rutas

```
/login          → Público (redirige a /dashboard si tiene sesión)
/onboarding     → Requiere auth, redirige si onboarding_done = true
/dashboard      → Requiere auth + onboarding completo
/settings       → Requiere auth
/upgrade        → Requiere auth
/admin/*        → Requiere auth + role = 'admin' (verificado en servidor)
```

---

## DECISIÓN 4 — ALMACENAMIENTO DE IMÁGENES

**CERRADA.**

### Supabase Storage — Bucket privado por usuario

Las imágenes de screenshots de trades abandonan base64 en localStorage y van a Supabase Storage.

```
Estructura del bucket:
─────────────────────
tradeos-screenshots/          ← bucket privado
  └── {user_id}/
        └── {trade_id}.jpg    ← imagen comprimida
```

**Políticas del bucket:**
- Solo el usuario dueño puede leer/escribir sus imágenes (RLS en Storage)
- Tamaño máximo por archivo: **500 KB** (se comprime en el cliente antes de subir)
- Formatos aceptados: JPEG, PNG, WebP
- Retención: indefinida mientras el usuario tenga cuenta activa
- Al eliminar un trade: la imagen en Storage se elimina via Edge Function

**Flujo de upload:**
```
1. Usuario selecciona imagen en el formulario de trade
2. Frontend comprime a máximo 1200px ancho, calidad 0.7 JPEG (≤ 500KB target)
3. Frontend sube directamente a Supabase Storage con el JWT del usuario
4. Supabase retorna la URL pública con token temporal
5. Frontend guarda la URL en el campo screenshot_url del trade
6. NO se almacena base64 en ningún lugar
```

**Plan de usuario Free:**
- Máximo 50 screenshots totales (feature gating por plan)
- Límite 5 MB de storage total
- Al llegar al límite: avisar y ofrecer upgrade a Pro

**Plan Pro:**
- Screenshots ilimitadas
- 2 GB de storage total (suficiente para miles de imágenes comprimidas)

---

## DECISIÓN 5 — PAGOS Y SUSCRIPCIONES

**CERRADA.**

### Stripe como procesador de pagos

**Por qué Stripe sobre alternativas:**
- El ICP paga con tarjeta de crédito o débito. Stripe tiene cobertura completa en Latinoamérica.
- Portal de clientes de Stripe (gestión de billing) = cero código de UI para cancelar, actualizar tarjeta, ver facturas.
- Webhooks confiables con reintentos automáticos.
- Stripe Tax maneja IVA si se necesita en el futuro.

### Planes y precios (según BUSINESS_VISION.md)

| Plan | Precio | Facturación | stripe_price_id |
|------|--------|-------------|-----------------|
| Free | $0 | — | — |
| Pro Mensual | $9 USD | Mensual | price_pro_monthly |
| Pro Anual | $79 USD | Anual | price_pro_yearly |
| Teams | $49 USD | Mensual | price_teams_monthly |
| One-time (legacy) | $19 USD | Único | price_onetime |

### Flujo de suscripción

```
UPGRADE FREE → PRO:
───────────────────
1. Usuario hace click en "Upgrade to Pro" en la app
2. Frontend llama a Edge Function /create-checkout-session
3. Edge Function crea Stripe Checkout Session con metadata: { user_id }
4. Frontend redirige a Stripe Checkout (hosted)
5. Usuario paga
6. Stripe envía webhook payment_intent.succeeded o checkout.session.completed
7. Edge Function /stripe-webhook recibe el evento
8. Edge Function actualiza tabla subscriptions: plan='pro', status='active'
9. Edge Function actualiza profiles: plan='pro'
10. Usuario es redirigido de vuelta a la app con plan activo

RENOVACIÓN AUTOMÁTICA:
──────────────────────
1. Stripe cobra automáticamente al vencimiento
2. Stripe envía webhook invoice.payment_succeeded
3. Edge Function actualiza current_period_end en subscriptions

CANCELACIÓN:
────────────
1. Usuario abre Portal de Clientes de Stripe (link generado por Edge Function)
2. Cancela desde el portal de Stripe
3. Stripe envía webhook customer.subscription.updated con cancel_at_period_end=true
4. Edge Function actualiza subscriptions
5. El acceso Pro dura hasta current_period_end
```

### Feature gating por plan

Las restricciones se validan en dos lugares: UI (para experiencia) y RLS de Supabase (para seguridad real).

| Feature | Free | Pro | Teams |
|---------|------|-----|-------|
| Cuentas de trading | 1 | Ilimitadas | Ilimitadas |
| Trades registrados | 50 total | Ilimitados | Ilimitados |
| Screenshots por trade | No | Sí | Sí |
| Análisis de patrones | No | Sí | Sí |
| Exportar CSV/PDF | No | Sí | Sí |
| Múltiples dispositivos | No | Sí | Sí |
| Backup automático | No | Sí | Sí |
| Acceso equipo | No | No | Sí (hasta 10) |

**Implementación del gating en el cliente:**
```javascript
// Zustand store de auth
const canAccessFeature = (feature) => {
  const { plan, tradeCount, accountCount } = useAuthStore();
  
  if (plan === 'pro' || plan === 'teams') return true;
  
  // Plan Free
  if (feature === 'patterns') return false;
  if (feature === 'screenshots') return false;
  if (feature === 'add_trade') return tradeCount < 50;
  if (feature === 'add_account') return accountCount < 1;
  
  return true;
};
```

**El gating real está en RLS de Supabase:**
```sql
-- Política que limita inserts en trades para usuarios Free
CREATE POLICY "free_plan_trade_limit" ON trades
  FOR INSERT
  WITH CHECK (
    (SELECT plan FROM profiles WHERE id = auth.uid()) != 'free'
    OR
    (SELECT COUNT(*) FROM trades WHERE user_id = auth.uid() AND deleted_at IS NULL) < 50
  );
```

---

## DECISIÓN 6 — ARQUITECTURA DEL FRONTEND

**CERRADA.**

### Estructura de carpetas

```
tradeos-web/
├── public/
│   ├── favicon.ico
│   └── robots.txt
├── src/
│   ├── main.jsx              ← Entry point
│   ├── App.jsx               ← Router + Auth provider
│   │
│   ├── components/           ← Componentes reutilizables
│   │   ├── ui/               ← Design system (Button, Card, Modal, Badge...)
│   │   ├── charts/           ← EquityCurve, WeeklyEvolution
│   │   └── layout/           ← Sidebar, TopBar, PageHeader
│   │
│   ├── pages/                ← Una carpeta por vista/sección
│   │   ├── auth/
│   │   │   ├── LoginPage.jsx
│   │   │   └── OnboardingPage.jsx
│   │   ├── dashboard/
│   │   │   └── DashboardPage.jsx
│   │   ├── checklist/
│   │   ├── trades/
│   │   │   ├── TradesPage.jsx
│   │   │   ├── TradeForm.jsx        ← Nuevo + Editar
│   │   │   └── TradeCalendar.jsx
│   │   ├── phases/
│   │   ├── funds/
│   │   ├── accounts/
│   │   ├── patterns/
│   │   ├── settings/
│   │   └── upgrade/
│   │
│   ├── stores/               ← Estado global (Zustand)
│   │   ├── authStore.js      ← user, plan, session
│   │   ├── tradesStore.js    ← trades[], filters, pagination
│   │   ├── profileStore.js   ← cfg (migrado de localStorage)
│   │   └── uiStore.js        ← sidebar, modals, theme
│   │
│   ├── hooks/                ← Custom hooks con React Query
│   │   ├── useTrades.js
│   │   ├── useAccounts.js
│   │   ├── usePatterns.js    ← Lógica de análisis de patrones
│   │   └── useSync.js        ← Sincronización offline/online
│   │
│   ├── lib/
│   │   ├── supabase.js       ← Cliente Supabase configurado
│   │   ├── calculator.js     ← calcLot() migrado — función pura
│   │   ├── patterns.js       ← renderPatterns() migrado — función pura
│   │   ├── consistency.js    ← Score de Consistencia — función pura
│   │   └── migration.js      ← Importar datos desde backup v1.x JSON
│   │
│   ├── i18n/
│   │   ├── es.js             ← LG.es migrado
│   │   └── en.js             ← LG.en migrado
│   │
│   └── styles/
│       ├── tokens.css        ← Variables CSS del design system actual
│       ├── global.css
│       └── themes.css        ← dark / light
│
├── supabase/
│   ├── migrations/           ← SQL migrations versionadas
│   │   └── 001_initial.sql
│   └── functions/            ← Edge Functions
│       ├── create-checkout-session/
│       ├── stripe-webhook/
│       ├── send-email/
│       └── delete-trade-screenshot/
│
├── package.json
├── vite.config.js
└── .env.example
```

### Estrategia de sincronización offline

```
ESTADO DE CONECTIVIDAD:
─────────────────────────
Online  → Leer/escribir en Supabase directamente via React Query
Offline → Leer desde caché de React Query (datos del último fetch)
          Escribir en cola local (IndexedDB via Dexie)
          Al reconectar → procesar cola y sincronizar con Supabase

COLA DE SINCRONIZACIÓN (sync queue en IndexedDB):
──────────────────────────────────────────────────
{
  id: uuid,
  operation: 'INSERT' | 'UPDATE' | 'DELETE',
  table: 'trades' | 'checklists' | 'funds',
  payload: { ... },
  created_at: timestamp,
  retries: 0
}

RESOLUCIÓN DE CONFLICTOS:
──────────────────────────
- Estrategia: last-write-wins con timestamp del servidor
- Si el servidor tiene un registro más nuevo que el cliente: el servidor gana
- Nunca se pierden datos del cliente — se mueve a un log de conflictos resueltos
- En v2.0: no hay múltiples dispositivos editando simultáneamente (la complejidad es baja)
```

---

## DECISIÓN 7 — HOSTING Y DEVOPS

**CERRADA.**

### Hosting del frontend: Vercel

- Plan Hobby gratuito suficiente para el MVP (100GB de bandwidth/mes)
- Deploy automático desde `main` branch
- Preview deployments automáticos por PR
- Variables de entorno en el dashboard de Vercel
- CDN global incluido

**Variables de entorno requeridas:**
```bash
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_STRIPE_PUBLISHABLE_KEY=
VITE_APP_URL=https://app.tradeos.io
```

### Dominio: app.tradeos.io

- `tradeos.io` → Landing page de marketing (estático, puede ser en Webflow o HTML simple)
- `app.tradeos.io` → La aplicación React (Vercel)
- Ambos con HTTPS via Let's Encrypt (Vercel maneja automáticamente)

### Backend: Supabase

- Plan Free suficiente hasta 50,000 filas y 500MB de storage
- Plan Pro ($25/mes) cuando: más de 500MB de storage O más de 2GB de ancho de banda/mes
- Migración automática entre planes — sin downtime

### CI/CD

```
GitHub → GitHub Actions
──────────────────────────
Push a feature/* → Tests + lint
PR a main        → Preview deploy en Vercel + tests
Merge a main     → Deploy automático a producción
Tag v*.*.* →       Crear release en GitHub + notificar en Discord interno
```

**Pipeline de CI:**
```yaml
# .github/workflows/ci.yml
jobs:
  test:
    - npm install
    - npm run lint
    - npm run test        # Vitest
    - npm run build       # Verificar que buildea sin errores
  
  # Vercel maneja el deploy automáticamente desde GitHub
```

### Entornos

| Entorno | URL | Branch | Base de datos |
|---------|-----|---------|---------------|
| Local | localhost:5173 | cualquiera | Supabase local via CLI |
| Preview | *.vercel.app | cualquier PR | Supabase staging |
| Producción | app.tradeos.io | main | Supabase producción |

---

## DECISIÓN 8 — PANEL ADMINISTRATIVO

**CERRADA.**

El panel admin es interno. Solo Juan Pa (y futuros colaboradores de soporte) lo usan. No es un producto — es una herramienta operacional.

### Arquitectura del admin

**No se construye un admin custom desde cero.** Se usa Supabase Studio (el dashboard de Supabase) para el 80% de las necesidades operativas, más un conjunto mínimo de vistas React accesibles en `/admin/*` para las operaciones frecuentes.

### Qué cubre Supabase Studio (sin código adicional)

- Ver y editar registros de cualquier tabla
- Ejecutar SQL directamente
- Ver logs de auth (accesos, registros, errores)
- Ver logs de Edge Functions
- Ver uso de Storage
- Ver métricas de base de datos

### Qué requiere vistas custom en `/admin`

Solo las operaciones que ocurren frecuentemente y que Supabase Studio hace incómodo:

```
/admin/dashboard
  → Métricas clave en tiempo real:
    - Usuarios registrados hoy / esta semana / total
    - Conversiones Free → Pro
    - MRR actual
    - Trades registrados hoy (indicador de engagement)
    - Errores en Sentry (últimas 24h)

/admin/users
  → Lista de usuarios con: email, plan, fecha de registro, último acceso, trade count
  → Buscar por email
  → Acciones: cambiar plan manualmente, enviar email, suspender cuenta

/admin/subscriptions
  → Lista de suscripciones activas con estado de Stripe
  → Filtro por plan, por estado (activa, cancelada, past_due)
  → Link directo al dashboard de Stripe de ese cliente

/admin/support
  → Lista de usuarios que enviaron email de soporte (si se integra un formulario)
  → Estado: abierto / resuelto
```

### Protección del admin

```
Acceso solo con:
1. JWT válido de Supabase (usuario autenticado)
2. Campo role = 'admin' en la tabla profiles
3. Middleware en el router que verifica ambas condiciones
4. Si falla cualquiera: redirigir a /dashboard (no a /login — no revelar que existe /admin)
```

```sql
-- Columna en profiles para el rol admin
ALTER TABLE profiles ADD COLUMN role TEXT DEFAULT 'user' 
  CHECK (role IN ('user','admin'));

-- Solo los admin pueden ver la tabla subscriptions completa
CREATE POLICY "admin_only_subscriptions" ON subscriptions
  FOR SELECT
  USING (
    (SELECT role FROM profiles WHERE id = auth.uid()) = 'admin'
  );
```

### Métricas del admin — queries base

```sql
-- Usuarios activos esta semana
SELECT COUNT(DISTINCT user_id) 
FROM trades 
WHERE created_at > NOW() - INTERVAL '7 days';

-- MRR actual
SELECT SUM(
  CASE plan 
    WHEN 'pro' THEN 9
    WHEN 'teams' THEN 49
    ELSE 0
  END
) as mrr
FROM subscriptions
WHERE status = 'active';

-- Conversión Free → Pro (últimos 30 días)
SELECT COUNT(*) 
FROM subscriptions 
WHERE plan = 'pro' 
  AND status = 'active'
  AND created_at > NOW() - INTERVAL '30 days';

-- Distribución de usuarios por plan
SELECT plan, COUNT(*) 
FROM profiles 
GROUP BY plan;
```

---

## DECISIÓN 9 — SEGURIDAD

**CERRADA.**

### Modelo de seguridad: Defense in Depth

Tres capas. Si una falla, las otras contienen el daño.

```
CAPA 1: Red Level
  → HTTPS en todos los endpoints (Vercel + Supabase — automático)
  → HSTS headers
  → No hay servidor propio que proteger — Vercel y Supabase son responsables

CAPA 2: Auth Level
  → JWT verificado en cada request a Supabase
  → Magic link con expiración de 1 hora
  → Sessions con expiración de 7 días + refresh automático
  → Rate limiting en magic link (5 intentos por hora por email — Supabase lo maneja)

CAPA 3: Database Level (la más importante)
  → Row Level Security en TODAS las tablas
  → Ningún usuario puede leer datos de otro usuario, incluso con JWT válido
  → Las Edge Functions usan la service role key (nunca expuesta al cliente)
  → El cliente nunca tiene acceso a la service role key
```

### Variables de entorno — separación estricta

```bash
# En el cliente (VITE_ prefix — expuesto en el bundle):
VITE_SUPABASE_URL           # Solo la URL pública
VITE_SUPABASE_ANON_KEY      # Clave anónima — no es un secreto real (está protegida por RLS)
VITE_STRIPE_PUBLISHABLE_KEY # Clave pública de Stripe

# En Edge Functions ÚNICAMENTE (nunca al cliente):
SUPABASE_SERVICE_ROLE_KEY   # Acceso total a la BD — solo para Edge Functions
STRIPE_SECRET_KEY           # Clave secreta de Stripe
STRIPE_WEBHOOK_SECRET       # Para verificar autenticidad de webhooks
RESEND_API_KEY              # Para enviar emails
```

### Reglas de seguridad para el código

1. **Nunca la service role key en el frontend.** Si algo requiere service role, va en una Edge Function.
2. **RLS es la seguridad real.** El feature gating en el cliente es UX, no seguridad. La base de datos aplica las reglas.
3. **Validar en el servidor, no solo en el cliente.** Cualquier límite de plan (50 trades en Free) se valida en RLS, no solo en JS.
4. **Los webhooks de Stripe se verifican con la firma.** Ningún webhook se procesa sin verificar `stripe.webhooks.constructEvent()`.
5. **Sin logging de datos personales del trader.** Sentry no recibe trades, P&L ni datos del perfil. Solo errores de código con stack trace.
6. **CORS configurado explícitamente.** Solo `app.tradeos.io` puede hacer requests a las Edge Functions.

### Gestión de datos y privacidad

```
GDPR / Privacidad:
──────────────────
- Derecho al olvido: DELETE en profiles + CASCADE elimina todos los datos
- Exportar datos: el botón de backup descarga todos los datos del usuario en JSON
- No hay terceros que reciban datos del trader (Stripe solo recibe email para billing)
- Los screenshots en Storage se eliminan con el trade o con la cuenta
- No hay analytics de comportamiento dentro del producto (solo Vercel Analytics por página, sin datos de usuario)

RETENTION:
──────────
- Cuentas inactivas (+18 meses sin login): email de aviso → 30 días para reactivar → eliminación
- Checklists: sin TTL en la BD (la BD no tiene límite como localStorage). Limpiar en cliente si >500 entradas.
```

---

## DECISIÓN 10 — MIGRACIÓN DE v1.x A v2.0

**CERRADA.**

Este es el momento más crítico del lanzamiento de v2.0. Un usuario con 6 meses de trades en un archivo HTML local debe poder migrar en menos de 60 segundos.

### Flujo de migración para el usuario

```
1. Usuario registra cuenta en app.tradeos.io (magic link)
2. Completa onboarding rápido (puede importar su cfg del backup)
3. Ve botón prominente: "Tenés una versión anterior de TradeOS — Importar mis datos"
4. Usuario sube su backup JSON de v1.x
5. El sistema valida el JSON (versión, schema)
6. Muestra preview: "Se importarán: 234 trades, 3 cuentas, 89 días de checklist"
7. Usuario confirma
8. El sistema hace el mapeo y crea los registros en Supabase:
   - cfg → profiles + accounts + goals + phases
   - trades[] → tabla trades (UUID generados, motherId resuelto)
   - checklists{} → tabla checklists (fecha parseada, bias incluido)
   - funds[] → tabla funds
9. Mensaje de éxito con resumen de lo importado
10. Redirigir al dashboard — todo está ahí
```

### Función de migración — responsabilidades

```javascript
// src/lib/migration.js
async function migrateFromV1Backup(jsonData, supabase, userId) {
  
  // 1. Validar estructura mínima
  validateV1Schema(jsonData); // lanza si falta cfg.name o trades no es array
  
  // 2. Mapear cuentas (number id → UUID)
  const accountIdMap = {}; // { oldId: newUUID }
  const accounts = jsonData.cfg.accounts.map(a => ({
    user_id: userId,
    firm: a.firm,
    type: a.type,
    instrument: a.instrument || 'forex',
    size: a.size,
    max_dd: a.maxDD,
    phase: a.phase || 'active',
  }));
  // INSERT y mapear IDs viejos a UUIDs nuevos
  
  // 3. Mapear trades
  // trade.id (timestamp) → UUID
  // trade.account (string del número) → UUID via accountIdMap
  // trade.ruleBroken (string) → TEXT[] (split por ', ')
  // trade.motherId (timestamp) → UUID del padre mapeado
  // trade.img (base64) → subir a Storage, guardar URL
  
  // 4. Mapear checklists
  // { "Mon Jun 02 2026": { "sleep_c": true } } → { user_id, date: '2026-06-02', items: {...} }
  // bias guardado en tt_bias_* se intenta recuperar del JSON si fue incluido
  
  // 5. Mapear fases y fondos
  
  // 6. Confirmar todo en una transaction (o via batch insert)
  
  return { imported: { trades: N, accounts: M, checklists: K } };
}
```

### Compatibilidad de versiones

El backup JSON de v1.x no tiene campo de versión. Se infiere por la presencia de campos clave:
- Tiene `cfg.name` y `trades` array → es v1.x → migrar con `migrateFromV1Backup()`
- En v2.0 el backup incluirá `schema_version: '2.0'` para importaciones futuras

---

## DECISIÓN 11 — EMAILS TRANSACCIONALES

**CERRADA.**

Todos los emails van via Resend. Templates HTML simples que respetan la paleta visual de TradeOS.

| Email | Trigger | Contenido |
|-------|---------|-----------|
| Magic link | Login intent | Link de acceso (generado por Supabase) |
| Bienvenida | Registro exitoso | "Tu cuenta está lista. Empieza tu onboarding." |
| Upgrade confirmado | Stripe webhook | "Sos Pro. Estas son tus nuevas funciones." |
| Renovación próxima | 7 días antes del vencimiento | "Tu suscripción renueva en 7 días." |
| Pago fallido | Stripe past_due | "Hubo un problema con tu pago. Actualizá tu tarjeta." |
| Cancelación confirmada | Stripe canceled | "Cancelaste tu plan Pro. Acceso hasta [fecha]." |
| Cuenta inactiva | 18 meses sin login | "¿Todavía usás TradeOS? Tus datos se eliminarán en 30 días." |

---

## HOJA DE RUTA DE IMPLEMENTACIÓN

### Fase 0 — Preparación (2-3 semanas, en paralelo a v1.1)

Sin escribir código de v2.0 todavía. Solo preparación:

- [ ] Cerrar todos los bugs críticos en v1.x (EXECUTIVE_REPORT acciones #1-#5)
- [ ] Registrar dominio tradeos.io y app.tradeos.io
- [ ] Crear cuenta Supabase (plan Free)
- [ ] Crear cuenta Stripe (modo test)
- [ ] Crear cuenta Vercel
- [ ] Crear cuenta Resend
- [ ] Crear cuenta Sentry
- [ ] Crear repositorio GitHub privado
- [ ] Configurar entornos: local / staging / producción
- [ ] Ejecutar las migrations SQL en Supabase staging y verificar que el schema es correcto

### Fase 1 — Esqueleto (3-4 semanas)

El objetivo es tener la app funcionando con auth y datos reales, sin features completas.

- [ ] Setup React + Vite + Tailwind + variables CSS del design system actual
- [ ] Componentes UI base (Button, Card, Modal, Badge, Input) — migrados del CSS actual
- [ ] Supabase Auth: magic link funcionando, protección de rutas
- [ ] Página de login y onboarding (migrado de v1.x)
- [ ] `saveAll()` / `loadAll()` migrados a hooks de React Query con Supabase
- [ ] Dashboard básico funcionando con datos reales de Supabase
- [ ] Deploy en Vercel con dominio app.tradeos.io

### Fase 2 — Feature completo (4-6 semanas)

Migrar cada módulo de v1.x a React con datos de Supabase:

- [ ] Checklist diario (con sync en tiempo real)
- [ ] Formulario de registro de trade + replicador
- [ ] Mis Trades (lista + calendario)
- [ ] Calculadora de lotaje (migración directa de `calcLot()`)
- [ ] Fases del plan
- [ ] Gestión de fondos
- [ ] Análisis de patrones (migración directa de `renderPatterns()`)
- [ ] Perfil y ajustes (con edición real de perfil — corrige D9 del EXECUTIVE_REPORT)
- [ ] Upload de screenshots a Supabase Storage

### Fase 3 — Monetización (2-3 semanas)

- [ ] Stripe integrado: checkout, webhooks, portal de clientes
- [ ] Feature gating por plan (Free vs Pro)
- [ ] Página de upgrade dentro de la app
- [ ] Emails transaccionales via Resend

### Fase 4 — Migración y lanzamiento (1-2 semanas)

- [ ] Función de migración de backup v1.x a v2.0
- [ ] Email a compradores de Gumroad anunciando v2.0 con instrucciones
- [ ] Panel admin básico en /admin
- [ ] Testing end-to-end del flujo completo (registro → onboarding → trade → billing)
- [ ] Lanzamiento de v2.0

**Estimación total: 12-16 semanas desde el inicio de Fase 1.**

---

## DECISIONES PENDIENTES (requieren validación de Juan Pa)

| Decisión | Opciones | Impacto | Urgencia |
|----------|---------|---------|----------|
| ¿El plan One-time de $19 sigue existiendo en v2.0? | A) Sí, con acceso limitado (como Free+) B) No, solo Free y Pro | Modelo de negocio completo | Alta — antes de Fase 3 |
| ¿Los compradores de v1.x obtienen Pro gratis? | A) Sí por 6 meses B) Sí de por vida C) No, se paga v2.0 aparte | Fidelidad vs ingresos | Alta — antes del lanzamiento |
| ¿Cuándo se depreca el archivo HTML local? | A) Nunca (sigue vendiéndose en paralelo) B) Con el lanzamiento de v2.0 C) 6 meses post v2.0 | Operacional | Media |
| ¿El plan Free es realmente gratuito o requiere tarjeta? | A) Email solo (sin tarjeta) B) Requiere tarjeta para empezar | Conversión | Media — antes de Fase 3 |
| ¿El panel admin es solo Juan Pa o habrá equipo de soporte? | A) Solo Juan Pa B) Hasta 3 personas de soporte | Scope del admin | Baja — puede decidirse en Fase 4 |

---

## COSTOS OPERACIONALES ESTIMADOS

### MVP (hasta 500 usuarios activos)

| Servicio | Plan | Costo/mes |
|---------|------|-----------|
| Vercel | Hobby (gratis hasta 100GB bandwidth) | $0 |
| Supabase | Free (hasta 50K filas, 500MB storage) | $0 |
| Stripe | 2.9% + $0.30 por transacción | Variable |
| Resend | Free (3,000 emails/mes) | $0 |
| Sentry | Free (5K errores/mes) | $0 |
| Dominio tradeos.io | — | ~$1.5/mes |
| **TOTAL fijo** | | **~$1.5/mes** |

### Crecimiento (500-2000 usuarios activos)

| Servicio | Plan | Costo/mes |
|---------|------|-----------|
| Vercel | Pro ($20/mes) | $20 |
| Supabase | Pro ($25/mes) | $25 |
| Stripe | 2.9% + $0.30 | Variable (~2.9% del MRR) |
| Resend | Starter ($20/mes, 50K emails) | $20 |
| Sentry | Team ($26/mes) | $26 |
| **TOTAL fijo** | | **~$91/mes** |

Con 200 usuarios Pro a $9/mes = $1,800 MRR → margen operacional ~95% después de costos fijos.

---

## RESUMEN DE DECISIONES

| # | Decisión | Tecnología elegida | Estado |
|---|---------|-------------------|--------|
| 1 | Stack | React + Vite + Zustand + Supabase | CERRADA |
| 2 | Base de datos | PostgreSQL en Supabase con RLS | CERRADA |
| 3 | Auth | Supabase Auth — Magic Link + Google OAuth | CERRADA |
| 4 | Storage de imágenes | Supabase Storage — bucket privado | CERRADA |
| 5 | Pagos | Stripe — suscripciones + portal de clientes | CERRADA |
| 6 | Arquitectura frontend | SPA React con offline-first via IndexedDB | CERRADA |
| 7 | Hosting | Vercel (frontend) + Supabase (backend) | CERRADA |
| 8 | Panel admin | Supabase Studio + vistas custom mínimas en /admin | CERRADA |
| 9 | Seguridad | RLS como capa principal + Defense in Depth | CERRADA |
| 10 | Migración v1→v2 | Función `migrateFromV1Backup()` + UI en app | CERRADA |
| 11 | Emails | Resend con templates HTML simples | CERRADA |

---

*Documento generado por CTO — TradeOS*
*Fecha: Junio 2026*
*Próxima revisión: al inicio de Fase 1 de implementación*
*Para Juan Pa: revisar "Decisiones Pendientes" antes del inicio de Fase 3*
*Para el equipo: nada de código de v2.0 antes de cerrar todos los bugs de v1.1*
