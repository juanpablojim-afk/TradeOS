# BUILD_ORDER_V2.md
## TradeOS Web v2 — Orden Exacto de Construcción

**Versión:** 1.0  
**Fecha:** Junio 2026  
**Autor:** CTO  
**Audiencia:** Claude Code — agente de construcción paso a paso  
**Fuentes:** SAAS_ARCHITECTURE_PLAN · PRODUCT_REQUIREMENTS_V2 · SECURITY_REQUIREMENTS_V2 · SCHEMA

---

## INSTRUCCIONES PARA CLAUDE CODE

1. **Ejecutar las etapas en el orden numérico exacto.** Ninguna etapa puede iniciarse si la anterior no cumple su criterio de finalización.
2. **Verificar el criterio de finalización de cada etapa antes de avanzar.** Si el criterio no se cumple, terminar la etapa antes de continuar.
3. **No agregar funcionalidades no listadas.** Este documento es el scope completo.
4. **Cada etapa tiene sus archivos afectados listados.** Solo modificar esos archivos en esa etapa.
5. **Las dependencias de cada etapa son los artefactos producidos por etapas anteriores**, no documentación.
6. **Al final de cada etapa, el proyecto debe buildear sin errores** (`npm run build` pasa).

---

## MAPA DE DEPENDENCIAS

```
E01 → E02 → E03 → E04 → E05 → E06 → E07 → E08
                    ↓
                   E09 → E10 → E11 → E12
                                ↓
                               E13 → E14 → E15
                                            ↓
                                           E16 → E17 → E18 → E19 → E20
```

---

## ETAPA 01 — Scaffolding y Configuración del Proyecto

**Objetivo:** Crear la estructura base del repositorio con todas las herramientas configuradas. Nada de lógica de negocio. Solo el andamiaje sobre el que se construye todo lo demás.

**Dependencias:** Ninguna. Es la primera etapa.

**Archivos a crear:**
```
tradeos-web/
├── package.json
├── vite.config.js
├── index.html
├── .env.example
├── .env.local              ← no se commitea
├── .gitignore
├── vercel.json
├── README.md
├── src/
│   ├── main.jsx
│   └── App.jsx             ← solo retorna <div>TradeOS</div> por ahora
├── supabase/
│   ├── config.toml
│   └── migrations/         ← carpeta vacía por ahora
└── .github/
    └── workflows/
        └── ci.yml
```

**Comandos a ejecutar:**
```bash
npm create vite@latest tradeos-web -- --template react
cd tradeos-web
npm install
npm install @supabase/supabase-js
npm install zustand
npm install react-router-dom
npm install @tanstack/react-query
npm install dexie
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

**Contenido de `vercel.json`:**
```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" }
      ]
    }
  ],
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

**Contenido de `.env.example`:**
```bash
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key
VITE_STRIPE_PUBLISHABLE_KEY=pk_test_your-key
VITE_APP_URL=http://localhost:5173
VITE_SUPPORT_EMAIL=tradeossoporte@gmail.com
```

**Contenido de `.gitignore`:**
```
node_modules/
dist/
.env
.env.local
.env.production
*.local
```

**Contenido de `ci.yml`:**
```yaml
name: CI
on: [push, pull_request]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run lint
      - run: npm run build
```

**Criterio de finalización:**
- [ ] `npm run dev` levanta sin errores en `localhost:5173`
- [ ] `npm run build` genera carpeta `dist/` sin errores
- [ ] `.env.local` existe con valores reales de un proyecto Supabase de staging
- [ ] `.env.local` está en `.gitignore`
- [ ] `vercel.json` tiene los headers de seguridad DEP-C2

---

## ETAPA 02 — Design System y Variables CSS

**Objetivo:** Establecer la identidad visual completa de TradeOS en CSS antes de construir ningún componente. Todo componente futuro usa estas variables — nunca valores hardcodeados.

**Dependencias:** E01 completada.

**Archivos a crear:**
```
src/
└── styles/
    ├── tokens.css          ← variables CSS del design system
    ├── global.css          ← reset y estilos base del body
    └── themes.css          ← data-theme="light"
```

**Modificar:**
```
src/main.jsx                ← importar los tres CSS
```

**Contenido exacto de `tokens.css`:**
```css
:root {
  /* Fondos */
  --bg: #07070f;
  --bg2: #0d0d1a;
  --bg3: #131320;
  --bg4: #1a1a2e;

  /* Bordes */
  --border: rgba(200, 169, 110, 0.08);
  --border2: rgba(200, 169, 110, 0.04);

  /* Texto */
  --text: #e8e8f5;
  --text2: #72729a;
  --text3: #3a3a5a;

  /* Marca — NUNCA cambiar estos valores */
  --accent: #c8a96e;
  --accent2: #e8c98e;
  --accent3: #a07840;
  --highlight: #d4af7a;
  --highlight2: #f0d090;

  /* Semánticos */
  --win: #5a7a4a;
  --loss: #8a4a35;
  --warn: #c49a2a;

  /* Tipografía */
  --font-display: 'Bebas Neue', sans-serif;
  --font-ui: 'DM Sans', sans-serif;
  --font-mono: 'DM Mono', monospace;

  /* Radios */
  --r: 14px;
  --r2: 9px;
}
```

**Contenido de `themes.css`:**
```css
[data-theme="light"] {
  --bg: #f5f0e8;
  --bg2: #ede8dc;
  --bg3: #e5ded0;
  --bg4: #dbd4c4;
  --border: rgba(139, 101, 32, 0.15);
  --border2: rgba(139, 101, 32, 0.07);
  --text: #1a1510;
  --text2: #6b5c40;
  --text3: #b0a080;
  --accent: #8b6520;
  --accent2: #a07830;
  --accent3: #c09040;
  --win: #3a5a2a;
  --loss: #6a3020;
  --warn: #a07010;
}
```

**Criterio de finalización:**
- [ ] Las tres fuentes de Google Fonts están importadas en `index.html` (Bebas Neue, DM Sans, DM Mono)
- [ ] Las variables CSS están disponibles globalmente (verificar en DevTools)
- [ ] El fondo del body es `#07070f` al cargar

---

## ETAPA 03 — Cliente Supabase y Tipos Base

**Objetivo:** Crear el cliente de Supabase configurado y los tipos TypeScript base que corresponden exactamente al schema SQL documentado en SAAS_ARCHITECTURE_PLAN.md. Estos tipos son la fuente de verdad del frontend.

**Dependencias:** E01 completada.

**Archivos a crear:**
```
src/lib/
├── supabase.js             ← cliente configurado
└── types.js                ← JSDoc types de todos los objetos de negocio
```

**Contenido de `supabase.js`:**
```javascript
import { createClient } from '@supabase/supabase-js'

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Supabase URL and anon key are required')
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
  auth: {
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: true,
  },
})
```

**Contenido de `types.js` — JSDoc que refleja el schema SQL exacto:**
```javascript
/**
 * @typedef {Object} Profile
 * @property {string} id - UUID
 * @property {string} name
 * @property {string|null} alias
 * @property {number|null} age
 * @property {string|null} country
 * @property {string} experience - 'beginner'|'inter'|'advanced'|'pro'
 * @property {string} risk_profile - 'conservative'|'moderate'|'aggressive'
 * @property {number} risk_pct
 * @property {number} rr_min
 * @property {string} max_trades
 * @property {string|null} strategy
 * @property {string} platform
 * @property {string[]} sessions
 * @property {string[]} pairs
 * @property {string[]} rules
 * @property {boolean} has_job
 * @property {string|null} job_hours
 * @property {string|null} days_off
 * @property {string|null} sleep_time
 * @property {string|null} wake_time
 * @property {number} monthly_expenses
 * @property {string|null} main_error
 * @property {string[]} bad_conditions
 * @property {string|null} motivation
 * @property {string[]} cl_items
 * @property {string[]} custom_cl_items
 * @property {string} lang - 'es'|'en'
 * @property {string} plan - 'free'|'pro'|'teams'
 * @property {string|null} plan_expires_at
 * @property {boolean} onboarding_done
 */

/**
 * @typedef {Object} Account
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string} firm
 * @property {string} type - 'own'|'funded'
 * @property {string} instrument - 'forex'|'futures'
 * @property {number} size
 * @property {number} max_dd
 * @property {string} phase - 'active'|'challenge1'|'challenge2'|'funded'|'paused'|'evaluation'
 * @property {number|null} max_daily_dd
 * @property {boolean} is_archived
 * @property {number} sort_order
 */

/**
 * @typedef {Object} Trade
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string} account_id
 * @property {string} traded_at - ISO 8601
 * @property {string} pair
 * @property {string} session - 'London'|'New York'|'Asia'|'Overlap'
 * @property {string} result - 'win'|'loss'|'be'
 * @property {number} pnl
 * @property {number} rr_real
 * @property {number|null} risk_pct
 * @property {string} mood - 'calm'|'confident'|'anxious'|'pressure'|'tired'|'revenge'
 * @property {string|null} entry_price
 * @property {string|null} stop_loss
 * @property {string|null} take_profit
 * @property {number|null} rr_planned
 * @property {string|null} note
 * @property {string|null} notebook
 * @property {string[]} rules_broken
 * @property {string[]} tags
 * @property {string|null} screenshot_url
 * @property {boolean} is_mother
 * @property {boolean} is_child
 * @property {string|null} mother_id
 * @property {string|null} deleted_at
 * @property {string|null} edited_at
 * @property {string} created_at
 */

/**
 * @typedef {Object} Fund
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string|null} account_id
 * @property {string} type - 'own_payout'|'funded_payout'|'salary'|'goal'|'challenge'|'expense'|'other'
 * @property {number} amount
 * @property {string|null} note
 * @property {string} transacted_at
 * @property {string|null} deleted_at
 */

/**
 * @typedef {Object} Goal
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string} name
 * @property {number} target
 * @property {number} saved
 * @property {number} sort_order
 * @property {boolean} is_active
 */

/**
 * @typedef {Object} Checklist
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string} date - DATE 'YYYY-MM-DD'
 * @property {Object} items - { [itemId: string]: boolean }
 * @property {string|null} bias_note
 * @property {number|null} session_duration_minutes
 */

/**
 * @typedef {Object} Phase
 * @property {string} id - UUID
 * @property {string} user_id
 * @property {string} phase_key
 * @property {boolean} is_custom
 * @property {string|null} icon
 * @property {string|null} name
 * @property {string|null} subtitle
 * @property {string|null} description
 * @property {string[]|null} tasks
 * @property {boolean[]|null} task_progress
 * @property {boolean} is_completed
 * @property {string|null} completed_at
 * @property {number} sort_order
 */
```

**Criterio de finalización:**
- [ ] `import { supabase } from './lib/supabase'` funciona sin error
- [ ] Si las variables de entorno no existen, la app lanza un error descriptivo (no silencioso)
- [ ] Los tipos en `types.js` coinciden campo a campo con el schema SQL de SAAS_ARCHITECTURE_PLAN §DECISIÓN 2

---

## ETAPA 04 — Migrations SQL en Supabase

**Objetivo:** Ejecutar el schema completo de la base de datos en el proyecto de Supabase de staging. Este es el contrato de datos del que depende todo el resto.

**Dependencias:** E03 completada. Proyecto Supabase de staging creado.

**Archivos a crear:**
```
supabase/migrations/
└── 001_initial_schema.sql
```

**Contenido de `001_initial_schema.sql`:**

El archivo contiene en este orden exacto:
1. Tabla `profiles` con todas sus columnas, constraints y política RLS
2. Tabla `accounts` con índice y política RLS
3. Tabla `trades` con todos sus índices (especialmente `mother_id` con `ON DELETE CASCADE`) y política RLS
4. Tabla `funds` con índice y política RLS
5. Tabla `goals` con política RLS
6. Tabla `phases` con constraint `UNIQUE(user_id, phase_key)` y política RLS
7. Tabla `checklists` con índice compuesto `(user_id, date DESC)` y política RLS
8. Tabla `subscriptions` sin RLS de usuario (solo admin)
9. Función `handle_new_user()` — trigger que crea fila en `profiles` cuando Supabase Auth crea un usuario
10. Trigger `on_auth_user_created` que ejecuta `handle_new_user()`

**El schema SQL completo está especificado en SAAS_ARCHITECTURE_PLAN.md §DECISIÓN 2. Copiar exactamente — no modificar tipos ni nombres de columnas.**

**Políticas RLS que deben existir en cada tabla:**

Para `profiles`, `accounts`, `funds`, `goals`, `phases`, `checklists`:
```sql
-- Habilitar RLS
ALTER TABLE [tabla] ENABLE ROW LEVEL SECURITY;

-- SELECT: solo los propios datos
CREATE POLICY "[tabla]_select_own" ON [tabla]
  FOR SELECT USING (auth.uid() = user_id);

-- INSERT: solo para el propio user_id
CREATE POLICY "[tabla]_insert_own" ON [tabla]
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- UPDATE: solo los propios datos
CREATE POLICY "[tabla]_update_own" ON [tabla]
  FOR UPDATE USING (auth.uid() = user_id);

-- DELETE: solo los propios datos
CREATE POLICY "[tabla]_delete_own" ON [tabla]
  FOR DELETE USING (auth.uid() = user_id);
```

Para `trades`, agregar además:
```sql
-- Soft delete: los trades con deleted_at no aparecen en SELECT normal
CREATE POLICY "trades_select_active" ON trades
  FOR SELECT USING (auth.uid() = user_id AND deleted_at IS NULL);
```

**Política de Storage:**
```sql
-- Bucket: tradeos-screenshots (crearlo en Supabase Dashboard como privado)
-- Política de Storage para el bucket:
CREATE POLICY "screenshots_user_prefix" ON storage.objects
  FOR ALL USING (
    bucket_id = 'tradeos-screenshots'
    AND auth.uid()::text = (storage.foldername(name))[1]
  );
```

**Límites del plan Free en RLS:**
```sql
-- Máximo 1 cuenta para Free
CREATE POLICY "accounts_free_limit" ON accounts
  FOR INSERT WITH CHECK (
    (SELECT plan FROM profiles WHERE id = auth.uid()) != 'free'
    OR (SELECT COUNT(*) FROM accounts WHERE user_id = auth.uid() AND NOT is_archived) < 1
  );

-- Máximo 50 trades para Free
CREATE POLICY "trades_free_limit" ON trades
  FOR INSERT WITH CHECK (
    (SELECT plan FROM profiles WHERE id = auth.uid()) != 'free'
    OR (SELECT COUNT(*) FROM trades WHERE user_id = auth.uid() AND deleted_at IS NULL) < 50
  );
```

**Comandos para aplicar:**
```bash
npx supabase login
npx supabase link --project-ref [staging-project-ref]
npx supabase db push
```

**Criterio de finalización:**
- [ ] Todas las tablas existen en Supabase staging (verificar en Supabase Studio)
- [ ] RLS está habilitado en todas las tablas (verificar con `SELECT tablename, rowsecurity FROM pg_tables WHERE schemaname = 'public'`)
- [ ] El trigger `on_auth_user_created` existe en Supabase Studio → Database → Triggers
- [ ] El bucket `tradeos-screenshots` existe como privado
- [ ] Prueba manual PERM-A1: con JWT de usuario A intentar SELECT en tablas de usuario B → 0 filas

---

## ETAPA 05 — Store de Autenticación (Zustand)

**Objetivo:** Crear el store de autenticación que gestiona el estado del usuario en toda la app. Sin UI todavía — solo el estado y las acciones.

**Dependencias:** E03 completada (cliente Supabase disponible).

**Archivos a crear:**
```
src/stores/
└── authStore.js
```

**El store debe contener:**

Estado:
- `user` — objeto del usuario de Supabase Auth (null si no autenticado)
- `profile` — objeto Profile de la tabla profiles (null si no autenticado)
- `session` — sesión JWT activa
- `isLoading` — boolean mientras se verifica la sesión inicial
- `isAuthenticated` — computed: `!!user`
- `plan` — computed desde `profile.plan`: `'free'|'pro'|'teams'`

Acciones:
- `initialize()` — llama `supabase.auth.getSession()` y suscribe a `onAuthStateChange`. Se llama una sola vez en `App.jsx`.
- `signInWithMagicLink(email)` — llama `supabase.auth.signInWithOtp({ email })`
- `signInWithGoogle()` — llama `supabase.auth.signInWithOAuth({ provider: 'google' })`
- `signOut()` — llama `supabase.auth.signOut()`, limpia el store, limpia IndexedDB
- `fetchProfile()` — SELECT en tabla profiles donde id = user.id. Guarda en `profile`.
- `canUseFeature(feature)` — retorna boolean según el plan actual

**`canUseFeature` debe implementar exactamente:**
```javascript
canUseFeature: (feature) => {
  const { profile } = get()
  const plan = profile?.plan ?? 'free'
  if (plan === 'pro' || plan === 'teams') return true
  // Límites Free:
  const freeLimits = {
    'screenshots': false,
    'patterns': false,
    'csv_export': false,
    'notebook': false,
    'multi_account': false,  // más de 1 cuenta
  }
  if (feature in freeLimits) return freeLimits[feature]
  return true  // resto de features disponibles en Free
}
```

**Criterio de finalización:**
- [ ] `useAuthStore()` exportado correctamente
- [ ] `initialize()` suscribe a `onAuthStateChange` exactamente una vez (no más)
- [ ] El store no lanza errores si `user` es null
- [ ] `canUseFeature('screenshots')` retorna false para plan free
- [ ] `canUseFeature('screenshots')` retorna true para plan pro

---

## ETAPA 06 — Router y Protección de Rutas

**Objetivo:** Configurar todas las rutas de la aplicación con protección según estado de autenticación y estado de onboarding.

**Dependencias:** E05 completada (authStore disponible).

**Archivos a modificar/crear:**
```
src/App.jsx                 ← configurar RouterProvider
src/router.jsx              ← definición de todas las rutas
src/components/
└── guards/
    ├── AuthGuard.jsx       ← redirige a /login si no autenticado
    └── OnboardingGuard.jsx ← redirige a /onboarding si onboarding_done = false
```

**Rutas a configurar (exactamente estas, en este orden):**
```
/                    → redirige a /login (público)
/login               → LoginPage (público — redirige a /dashboard si tiene sesión)
/auth/callback       → AuthCallbackPage (maneja el redirect de magic link y OAuth)
/onboarding          → OnboardingPage (requiere auth, redirige si onboarding ya completo)
/dashboard           → DashboardPage (requiere auth + onboarding completo)
/checklist           → ChecklistPage (requiere auth + onboarding completo)
/trades/new          → TradeFormPage (requiere auth + onboarding completo)
/trades              → TradesPage (requiere auth + onboarding completo)
/phases              → PhasesPage (requiere auth + onboarding completo)
/funds               → FundsPage (requiere auth + onboarding completo)
/accounts            → AccountsPage (requiere auth + onboarding completo)
/patterns            → PatternsPage (requiere auth + onboarding completo)
/calculator          → CalculatorPage (requiere auth + onboarding completo)
/settings            → SettingsPage (requiere auth + onboarding completo)
/upgrade             → UpgradePage (requiere auth)
/admin               → AdminPage (requiere auth + role = 'admin')
*                    → redirige a /dashboard (si auth) o /login (si no)
```

**Lógica de `AuthGuard`:**
```jsx
// Si isLoading: mostrar spinner
// Si no isAuthenticated: <Navigate to="/login" replace />
// Si isAuthenticated: <Outlet />
```

**Lógica de `OnboardingGuard`:**
```jsx
// Si profile.onboarding_done === false: <Navigate to="/onboarding" replace />
// Si profile.onboarding_done === true: <Outlet />
```

**Importante:** Todas las páginas listadas arriba deben existir como archivos .jsx que retornan `<div>Página: [nombre]</div>` — son placeholders que se completan en etapas siguientes. El router debe funcionar sin errores con páginas vacías.

**Criterio de finalización:**
- [ ] Todas las rutas están definidas en `router.jsx`
- [ ] Navegar a `/dashboard` sin sesión redirige a `/login`
- [ ] Navegar a `/login` con sesión activa redirige a `/dashboard`
- [ ] Todas las páginas placeholder existen y el build pasa sin errores
- [ ] La ruta `/admin` no es accesible por un usuario sin role='admin'

---

## ETAPA 07 — Páginas de Autenticación

**Objetivo:** Implementar las páginas de login y callback de auth. Es la puerta de entrada al producto.

**Dependencias:** E06 completada.

**Archivos a completar:**
```
src/pages/auth/
├── LoginPage.jsx
└── AuthCallbackPage.jsx
```

**`LoginPage.jsx` debe contener:**
- Campo de email (input type="email")
- Botón "Continuar con email" que llama `signInWithMagicLink(email)`
- Botón "Continuar con Google" que llama `signInWithGoogle()`
- Estado visual: loading mientras se envía el magic link
- Estado de éxito: mensaje "Revisá tu email — enviamos un link de acceso" (AUTH-A3: el mismo mensaje siempre, no revela si el email existe)
- El mensaje de confirmación NO dice "Si ese email está registrado..." — dice simplemente "Revisá tu email"
- Logo TradeOS: `Trade<span style="color:var(--accent)">OS</span>` en Bebas Neue
- Eslogan debajo del logo en DM Mono

**`AuthCallbackPage.jsx` debe:**
- Al montar, llamar `supabase.auth.exchangeCodeForSession()` si hay `code` en la URL
- Si la sesión se establece correctamente: redirigir a `/onboarding` (si onboarding_done = false) o `/dashboard`
- Si hay error: mostrar mensaje y link para volver a `/login`

**Criterio de finalización:**
- [ ] El formulario de email llama a Supabase correctamente
- [ ] El botón de Google redirige al flujo de OAuth de Google
- [ ] Después del click en el magic link el usuario llega al dashboard (probar end-to-end)
- [ ] AUTH-A3: el mensaje de confirmación es el mismo independientemente de si el email existe
- [ ] AUTH-C4: un usuario no autenticado que accede a /dashboard es redirigido a /login

---

## ETAPA 08 — Layout Principal y Sidebar

**Objetivo:** Crear el layout de la aplicación con sidebar de navegación. Es el shell que envuelve todos los módulos.

**Dependencias:** E07 completada. E02 completada (variables CSS disponibles).

**Archivos a crear:**
```
src/components/layout/
├── AppLayout.jsx           ← wrapper con sidebar + área de contenido
├── Sidebar.jsx             ← navegación con las 10 secciones
└── SyncIndicator.jsx       ← indicador de estado de sync (LOSS-M1)
```

**`Sidebar.jsx` debe contener exactamente estas 10 secciones en este orden:**
1. Dashboard → `/dashboard`
2. Checklist → `/checklist`
3. Registrar Trade → `/trades/new`
4. Mis Trades → `/trades`
5. Fases del Plan → `/phases`
6. Gestión de Fondos → `/funds`
7. Cuentas → `/accounts`
8. Análisis → `/patterns`
9. Calculadora → `/calculator`
10. Ajustes → `/settings`

**El sidebar también contiene:**
- Logo TradeOS en la parte superior
- Avatar con inicial del nombre del usuario
- Nombre del usuario
- Label "TRADER ACTIVO" en DM Mono
- Botón de idioma (ES / EN) en la parte inferior
- `SyncIndicator` en la parte inferior

**`SyncIndicator`** muestra uno de tres estados:
- `✓ Sincronizado` (verde) — cuando todos los datos están en Supabase
- `⟳ Sincronizando...` (dorado) — cuando hay operaciones en curso
- `⚠ Sin conexión` (amarillo) — cuando no hay red, con nota "guardado local"

**Criterio de finalización:**
- [ ] El layout renderiza correctamente en todas las rutas protegidas
- [ ] El ítem activo del sidebar tiene borde izquierdo dorado (`border-left: 2px solid var(--accent)`)
- [ ] El nombre del usuario se muestra con la inicial en el avatar
- [ ] Navegar entre secciones funciona sin reload de página
- [ ] El sidebar es responsive: en mobile se oculta con un menú hamburguesa

---

## ETAPA 09 — IndexedDB (Offline Storage)

**Objetivo:** Configurar Dexie.js como capa de persistencia local para el modo offline-first. Todos los datos se guardan aquí primero, luego se sincronizan con Supabase.

**Dependencias:** E03 completada.

**Archivos a crear:**
```
src/lib/
├── db.js                   ← configuración de Dexie
└── syncQueue.js            ← cola de operaciones pendientes para sync
```

**`db.js` — schema de IndexedDB:**
```javascript
import Dexie from 'dexie'

export const db = new Dexie('TradeOS')

db.version(1).stores({
  profiles:   'id',
  accounts:   'id, user_id',
  trades:     'id, user_id, account_id, traded_at, deleted_at',
  funds:      'id, user_id',
  goals:      'id, user_id',
  phases:     'id, user_id, phase_key',
  checklists: 'id, user_id, date',
  syncQueue:  '++id, table_name, operation, created_at',
})
```

**`syncQueue.js` — lógica de cola:**

Funciones a implementar:
- `enqueue(operation, tableName, payload)` — agrega operación a la cola en IndexedDB
- `processQueue(supabase)` — procesa todas las operaciones pendientes. Para cada operación: ejecutar en Supabase, si éxito: eliminar de la cola. Si falla: incrementar `retries`, si `retries >= 3`: mover a cola de errores.
- `getQueueLength()` — retorna número de operaciones pendientes (usado por SyncIndicator)
- `clearQueue()` — limpia toda la cola (llamado en logout)

**Criterio de finalización:**
- [ ] `db.trades.add(tradeObj)` funciona sin error
- [ ] La cola de sync se puede enqueue y procesar
- [ ] Al desconectar la red en DevTools, los writes van a IndexedDB sin error
- [ ] `clearQueue()` es llamado en `authStore.signOut()`

---

## ETAPA 10 — Onboarding (9 pasos)

**Objetivo:** Implementar el flujo de onboarding completo. Es el primer uso del producto — donde el trader configura toda su experiencia. Al completarlo, se crea el profile completo en Supabase.

**Dependencias:** E04 (tablas en BD), E05 (authStore), E08 (layout), E09 (IndexedDB).

**Archivos a completar/crear:**
```
src/pages/auth/
└── OnboardingPage.jsx      ← orquesta los 9 pasos

src/pages/onboarding/
├── Step1Profile.jsx
├── Step2Accounts.jsx
├── Step3Style.jsx
├── Step4Rules.jsx
├── Step5Schedule.jsx
├── Step6Checklist.jsx
├── Step7Goals.jsx
├── Step8Phases.jsx
└── Step9Psychology.jsx

src/stores/
└── onboardingStore.js      ← estado parcial del onboarding
```

**`onboardingStore.js` debe:**
- Guardar el estado parcial en IndexedDB a medida que el usuario avanza
- Al recargar la página, recuperar el estado guardado para retomar donde se dejó
- Al completar el paso 9: llamar `finishOnboarding()` que hace INSERT/UPDATE en `profiles` + INSERT en `accounts` y `goals`

**Cada paso del onboarding:**
- Tiene un botón "Siguiente →" y un botón "← Volver" (excepto paso 1)
- Valida sus campos antes de avanzar — si hay error muestra mensaje inline
- Muestra barra de progreso con "Paso X de 9"
- No puede saltarse

**Paso 2 (Cuentas) — detalles críticos:**
- Permite agregar entre 1 y 5 cuentas
- Por cada cuenta: firma/broker, tipo (own/funded), instrumento (forex/futures), tamaño ($), drawdown máximo (%)
- Para el drawdown: chips de 5%, 6%, 8%, 10%, 12% + opción "Otro" con input
- La primera cuenta no se puede eliminar — siempre debe haber al menos 1

**Paso 6 (Checklist) — detalles críticos:**
- Los 13 ítems predeterminados están siempre presentes y marcados
- El trader puede agregar ítems personalizados
- Los 13 ítems base NUNCA se pueden eliminar (están hardcodeados, no en estado editable)

**`finishOnboarding()` debe:**
1. Construir el objeto profile completo desde el estado de todos los pasos
2. Hacer UPSERT en tabla `profiles` con `onboarding_done: true`
3. Hacer INSERT en tabla `accounts` por cada cuenta configurada
4. Hacer INSERT en tabla `goals` por cada meta configurada
5. Crear las 6 fases predeterminadas en tabla `phases`
6. Mostrar animación de loading mientras se procesa
7. Al completar: redirigir a `/dashboard`
8. Si falla: mostrar error específico, NO limpiar el formulario

**Criterio de finalización:**
- [ ] El flujo completo de 9 pasos funciona sin errores
- [ ] Si el trader cierra el navegador en el paso 4 y vuelve, retoma desde el paso 4
- [ ] Al completar el paso 9, los datos están en Supabase (verificar en Supabase Studio)
- [ ] `profiles.onboarding_done` = true después de completar
- [ ] Las cuentas configuradas aparecen en la tabla `accounts`
- [ ] Las 6 fases predeterminadas aparecen en la tabla `phases`
- [ ] `authStore.profile` se actualiza después del onboarding

---

## ETAPA 11 — Hooks de Datos (React Query + Supabase)

**Objetivo:** Crear todos los custom hooks que encapsulan las operaciones de datos. Estos hooks son la única forma en que los componentes leen y escriben datos — nunca llaman a Supabase directamente.

**Dependencias:** E04 (schema BD), E05 (authStore), E09 (IndexedDB).

**Archivos a crear:**
```
src/hooks/
├── useProfile.js           ← leer/actualizar profile
├── useAccounts.js          ← CRUD de cuentas
├── useTrades.js            ← CRUD de trades con filtros
├── useFunds.js             ← CRUD de movimientos de fondos
├── useGoals.js             ← CRUD de metas
├── usePhases.js            ← estado de fases y tareas
├── useChecklists.js        ← checklist del día + historial
└── useSync.js              ← estado de conectividad y sincronización
```

**Patrón que cada hook debe seguir:**
```javascript
// Ejemplo: useTrades.js
export function useTrades(filters = {}) {
  const { user } = useAuthStore()
  
  // READ — con React Query
  const { data: trades, isLoading, error } = useQuery({
    queryKey: ['trades', user?.id, filters],
    queryFn: async () => {
      // 1. Intentar leer de Supabase
      // 2. Si offline: leer de IndexedDB
      // 3. Retornar datos
    },
    enabled: !!user,
  })
  
  // WRITE — con useMutation
  const { mutate: saveTrade } = useMutation({
    mutationFn: async (tradeData) => {
      // 1. Validar datos
      // 2. Guardar en IndexedDB inmediatamente
      // 3. Intentar guardar en Supabase
      // 4. Si offline: encolar en syncQueue
      // 5. Si error de Supabase: mostrar error, NO limpiar el formulario
    },
    onSuccess: () => {
      queryClient.invalidateQueries(['trades'])
      queryClient.invalidateQueries(['dashboard'])
    },
    onError: (error) => {
      // Mostrar error específico al usuario
      // NUNCA mostrar "guardado" si Supabase falló
    }
  })
  
  return { trades, isLoading, error, saveTrade, ... }
}
```

**Operaciones que debe soportar cada hook:**

`useAccounts`:
- `accounts` — lista de cuentas no archivadas del usuario
- `addAccount(accountData)` — INSERT validando límite Free (1 cuenta)
- `updateAccount(id, changes)` — UPDATE de cualquier campo
- `archiveAccount(id)` — soft-archive: `is_archived = true`
- `deleteAccount(id)` — DELETE con confirmación (elimina trades asociados via CASCADE)

`useTrades`:
- `trades` — lista de trades activos (`deleted_at IS NULL`) con filtros aplicados
- `saveTrade(tradeData)` — INSERT del trade madre
- `saveChildTrades(motherTrade, childData[])` — INSERT de trades hijos (replicador)
- `updateTrade(id, changes)` — UPDATE con `edited_at = NOW()`. Un madre editado no toca las hijas.
- `deleteTrade(id)` — soft delete (`deleted_at = NOW()`). Si `is_mother`: soft delete todas las hijas también.
- `getTradesByAccount(accountId)` — usado por calculadora para balance real

`useChecklists`:
- `todayChecklist` — checklist del día actual (fecha UTC)
- `toggleItem(itemId)` — toggle de un ítem, guarda inmediatamente
- `saveBias(text)` — guarda el bias del día
- `startSession()` — guarda `session_start_at` en el store local
- `endSession()` — calcula duración y guarda `session_duration_minutes`
- `getHistoryForMonth(year, month)` — para análisis de cumplimiento

**Criterio de finalización:**
- [ ] `useTrades().saveTrade(data)` crea un trade en Supabase y en IndexedDB
- [ ] `useTrades().deleteTrade(motherId)` hace soft delete del madre Y de todas sus hijas
- [ ] Si Supabase retorna error en un save, el hook lanza el error (no lo silencia)
- [ ] React Query cachea los datos y no hace refetch innecesario
- [ ] `useSync.js` detecta cambio de `navigator.onLine` y dispara `processQueue()`

---

## ETAPA 12 — Lógica de Negocio (Funciones Puras)

**Objetivo:** Migrar toda la lógica de cálculo de v1.x a funciones puras y testeables. Sin efectos secundarios, sin llamadas a Supabase. Solo reciben datos y retornan resultados.

**Dependencias:** E03 (tipos disponibles).

**Archivos a crear:**
```
src/lib/
├── calculator.js           ← calcLot() migrado exactamente
├── patterns.js             ← detección de los 10 patrones
├── consistency.js          ← Score de Consistencia
├── metrics.js              ← todas las métricas del dashboard
└── migration.js            ← migrateFromV1Backup()
```

**`calculator.js` — reglas exactas de v1.x:**

Los pares con pip value exacto (copiar exactamente los arrays de v1.x):
- `forexExact = ['EUR/USD', 'GBP/USD', 'AUD/USD', 'NZD/USD']` → $10/pip/lote estándar
- `futuresData` con los 9 contratos CME y sus tick sizes/values exactos (NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL)
- `noCalc` — pares que muestran mensaje explicativo (nunca número aproximado)

Fórmula de balance real:
```javascript
const realBalance = account.size + Math.min(0, accumulatedPnl)
// accumulatedPnl = suma de pnl de todos los trades de esa cuenta
```

**`consistency.js`:**
```javascript
// Score = (winRate * 0.5) + (ruleCompliance * 0.5)
// winRate = wins / totalTrades * 100
// ruleCompliance = tradesWithNoRulesBroken / totalTrades * 100
// Ambos sobre TODOS los trades del usuario (sin ventana temporal por ahora)
export function calcConsistencyScore(trades) { ... }
```

**`metrics.js`** — todas las métricas del dashboard:
- `calcGlobalMetrics(trades)` → { totalTrades, winRate, netPnl, cleanStreak, avgRR, brokenRulesThisWeek }
- `calcAccountPnl(trades, accountId)` → number
- `calcDrawdownPct(account, trades)` → number (0-100)
- `calcWeeklySummary(trades)` → { thisWeek: Stats, lastWeek: Stats }
- `detectPattern(trades)` → string | null (mismo algoritmo de v1.x `detectMain()`)

**`migration.js` — migración de v1.x:**
```javascript
// Valida el JSON de backup v1.x
// Retorna el preview: { tradesCount, accountsCount, checklistDays }
export function validateV1Backup(json) { ... }

// Convierte los datos de v1.x al formato de v2.0
// No escribe en Supabase — solo transforma los datos
export function transformV1ToV2(json, userId) {
  // Mapear accounts: id numérico → UUID generado
  // Mapear trades: id timestamp → UUID, account id numérico → UUID
  // Mapear checklists: { "Mon Jun 02 2026": {...} } → { date: '2026-06-02', items: {...} }
  // Mapear ruleBroken: string "R1, R2" → array ['R1', 'R2']
  // Resolver motherId: el timestamp viejo → el UUID nuevo del padre
  // Retornar { profiles, accounts, trades, funds, goals, phases, checklists }
}
```

**Tests requeridos para esta etapa** (usar Vitest):
```
src/lib/__tests__/
├── calculator.test.js      ← probar calcLot con valores conocidos
├── consistency.test.js     ← probar el score con trades mock
└── migration.test.js       ← probar que un backup v1.x válido se transforma correctamente
```

**Criterio de finalización:**
- [ ] `npm run test` pasa con 0 failures
- [ ] `calcLot` para NQ con stop 20 puntos y 1% de riesgo en cuenta de $10,000 retorna 1 contrato
- [ ] `calcLot` para USD/JPY retorna mensaje explicativo, no número
- [ ] `calcConsistencyScore` con 10 trades, 6 ganadores, 2 reglas rotas = score correcto
- [ ] `transformV1ToV2` convierte un backup de muestra sin perder datos
- [ ] Ninguna función en estos archivos llama a Supabase ni modifica estado global

---

## ETAPA 13 — Dashboard

**Objetivo:** Implementar el dashboard completo. Es la primera página que el trader ve cada día.

**Dependencias:** E11 (hooks de datos), E12 (lógica de negocio), E08 (layout).

**Archivos a completar:**
```
src/pages/dashboard/
├── DashboardPage.jsx
└── components/
    ├── ConsistencyScore.jsx
    ├── MetricsGrid.jsx
    ├── EquityCurve.jsx
    ├── AccountPanels.jsx
    ├── DrawdownAlert.jsx
    ├── GoalsBars.jsx
    ├── WeeklySummary.jsx
    └── RecentTrades.jsx
```

**Cada componente consume datos de los hooks de E11 y funciones de E12.**

**`ConsistencyScore`:**
- Número grande (DM Mono) con el score en porcentaje
- Color: verde (≥70%), amarillo (50-69%), rojo (<50%)
- Label: Excelente (≥80%) / Sólido (≥65%) / Regular (≥50%) / Mejorar (<50%)
- Barra de progreso del ancho del score

**`MetricsGrid`:**
- 6 tarjetas en grid: Total trades, Win Rate, P&L neto, Racha limpia, R:R promedio, Reglas rotas esta semana
- Todos los números en DM Mono
- P&L: verde si positivo, rojo si negativo

**`EquityCurve`:**
- SVG generado desde `metrics.js`
- Visible solo si hay ≥ 2 trades
- Línea verde si P&L final positivo, roja si negativo
- Área bajo la curva con 12% de opacidad del color de la línea

**`AccountPanels`:**
- Una tarjeta por cuenta activa (no archivadas)
- Muestra: nombre de firma, P&L de la cuenta, barra de drawdown
- Color de barra: verde (<50% del max_dd), amarillo (50-80%), rojo (>80%)
- `DrawdownAlert` aparece como banner encima de los paneles cuando alguna cuenta supera el 70% de su drawdown máximo

**`WeeklySummary`:**
- Siempre visible — no solo en fines de semana (corrige bug de v1.x)
- Oculto si no hay trades de la semana actual
- Muestra: trades de la semana, WR semanal, P&L semanal
- Comparativa vs semana anterior si hay datos

**`RecentTrades`:**
- Últimos 5 trades en orden cronológico inverso
- Por cada trade: par, resultado (color), P&L, sesión

**Criterio de finalización:**
- [ ] El dashboard carga en menos de 2 segundos con datos reales
- [ ] Todas las métricas se calculan usando las funciones de `metrics.js` (no lógica inline)
- [ ] La curva de equity aparece con 2+ trades y desaparece con menos de 2
- [ ] El resumen semanal es visible de lunes a domingo (no solo fines de semana)
- [ ] La alerta de drawdown aparece cuando una cuenta supera el 70% de su DD máximo
- [ ] Con 0 trades, el dashboard muestra empty states descriptivos con CTA a registrar trade

---

## ETAPA 14 — Checklist Diario

**Objetivo:** Implementar el checklist diario completo con temporizador de sesión y bias del día.

**Dependencias:** E11 (useChecklists), E08 (layout).

**Archivos a completar:**
```
src/pages/checklist/
├── ChecklistPage.jsx
└── components/
    ├── ChecklistSection.jsx ← agrupa ítems por sección
    ├── ChecklistItem.jsx    ← ítem individual con toggle
    ├── BiasNote.jsx         ← campo de texto para el bias del día
    └── SessionTimer.jsx     ← temporizador con botones iniciar/cerrar
```

**Los 13 ítems predeterminados, agrupados en 4 secciones (hardcodeados, no editables desde aquí):**

Sección 1 — Antes de operar: sleep_c, no_rev_c, bias_c, risk_c, mental_c  
Sección 2 — Al abrir la plataforma: pending_c, dd_c, news_c  
Sección 3 — Al ver un setup: setup_c, rr_c, lot_c  
Sección 4 — Después del trade: close_c, journal_c (+ ítems personalizados al final)

**`BiasNote`:**
- Textarea libre asociada a la fecha de hoy
- Se guarda automáticamente con debounce de 500ms (no requiere botón)
- Al cargar: recupera el bias guardado para hoy
- Texto de label configurable por idioma

**`SessionTimer`:**
- Botón "▶ Iniciar sesión" visible por defecto
- Al hacer click: el botón desaparece, aparece el contador `HH:MM:SS` en DM Mono
- Botón "✕ Cerrar sesión" que abre un `confirm()` nativo: "Sesión duró X minutos. ¿Tenés trades para registrar?"
- Si confirma: redirigir a `/trades/new`
- La duración de sesión se guarda en `checklists.session_duration_minutes`

**La personalización del checklist (agregar, renombrar, reordenar ítems) va en `/settings`, no aquí.**

**Criterio de finalización:**
- [ ] Hacer toggle en un ítem lo persiste inmediatamente (verificar en Supabase Studio)
- [ ] Recargar la página mantiene el estado de los checkboxes del día
- [ ] El bias del día se guarda automáticamente sin botón
- [ ] El temporizador cuenta correctamente desde 00:00:00
- [ ] Al cerrar sesión y confirmar, redirige a `/trades/new`
- [ ] Los ítems personalizados del perfil aparecen al final de la sección 4

---

## ETAPA 15 — Formulario de Registro de Trade

**Objetivo:** Implementar el formulario de registro de trade completo incluyendo el replicador y la calculadora integrada.

**Dependencias:** E11 (useTrades, useAccounts), E12 (calculator.js), E08 (layout).

**Archivos a completar:**
```
src/pages/trades/
├── TradeFormPage.jsx       ← página de nuevo trade Y edición
└── components/
    ├── ResultSelector.jsx   ← botones Ganador/Perdedor/BE
    ├── RulesBrokenChips.jsx ← chips multi-selección de reglas
    ├── TagsSelector.jsx     ← chips de setup tags
    ├── ScreenshotUpload.jsx ← upload de imagen a Storage
    ├── NotebookPanel.jsx    ← libreta (solo Pro)
    ├── LotCalculator.jsx    ← calculadora integrada
    └── ReplicatorModal.jsx  ← modal del replicador
```

**Campos del formulario (exactamente en este orden):**
1. Par / Instrumento (select con pares del perfil)
2. Sesión (select: London / New York / Asia / Overlap)
3. Cuenta (select con cuentas no archivadas)
4. Resultado (3 botones: Ganador / Perdedor / Break Even)
5. P&L en $ (número, positivo o negativo)
6. R:R real (número decimal)
7. Riesgo % (select con [0.1, 0.25, 0.35, 0.5, 0.75, 1, 1.5, 2, 3, 5])
8. Estado emocional (select: Tranquilo/Confiado/Ansioso/Presionado/Cansado/Revenge)
9. Reglas rotas (chips multi-selección — chip "✓ Ninguna" activo por defecto)
10. Tags de setup (chips: Barrida de liquidez / Ruptura / Pullback / S&R / FVG / Order Block / News play)
11. Descripción del setup (textarea)
12. Captura del gráfico (upload — solo Pro, mostrar banner si Free)
13. Libreta (toggle + textarea expandible — solo Pro)
14. Calculadora de lotaje integrada (siempre visible)

**`ScreenshotUpload`:**
- Solo renderiza si `canUseFeature('screenshots')` es true
- Si Free: muestra el campo como locked con mensaje "Disponible en Plan Pro →"
- Proceso de upload: seleccionar imagen → comprimir a ≤500KB con Canvas API → upload a Supabase Storage en `/{user_id}/{tradeId}.jpg` → guardar URL en el trade
- Botón ✕ para eliminar la imagen seleccionada antes de guardar

**`LotCalculator`** — la misma lógica de `calculator.js` de E12:
- Lee automáticamente el par y la cuenta del formulario principal
- Inputs adicionales: stop loss (pips o puntos) y riesgo %
- Resultado en tiempo real conforme el usuario escribe

**`ReplicatorModal`:**
- Se abre al hacer click en "⊕ Replicar" antes de guardar
- Muestra todas las cuentas EXCEPTO la seleccionada en el formulario
- Por cada cuenta adicional: un input de P&L editable pre-cargado con el P&L del formulario principal
- Al confirmar: llama `useTrades().saveChildTrades()` que crea los trades hijos
- Los trades hijos tienen: `is_child: true`, `is_mother: false`, `mother_id: [UUID del madre]`, `notebook: null`

**Modo edición:**
- `TradeFormPage.jsx` recibe un prop `tradeId` (opcional)
- Si `tradeId` existe: carga los datos del trade, pre-llena el formulario, el submit hace UPDATE en lugar de INSERT
- Al editar una madre, NO se tocan los datos de las hijas (solo los campos de la madre)
- Se guarda `edited_at: NOW()` en el trade editado

**Validación antes de guardar:**
- Resultado es obligatorio
- P&L debe ser un número válido (puede ser 0 o negativo)
- Cuenta es obligatoria
- Par es obligatorio

**Criterio de finalización:**
- [ ] Guardar un trade crea un registro en Supabase con todos los campos correctos
- [ ] El replicador crea los trades hijos con `mother_id` correcto (el UUID del madre que se acaba de crear)
- [ ] Editar un trade actualiza el registro existente y guarda `edited_at`
- [ ] La imagen se sube a Supabase Storage y la URL queda en `trades.screenshot_url` (no base64 en BD)
- [ ] DATA-C3: verificar que no existe base64 en la tabla trades en Supabase Studio
- [ ] Un usuario Free que tiene 50 trades ve el formulario pero recibe error al intentar guardar el trade 51
- [ ] La libreta no se copia a los trades hijos del replicador

---

## ETAPA 16 — Historial de Trades (Mis Trades)

**Objetivo:** Implementar la vista de historial de trades con modo lista y modo calendario.

**Dependencias:** E11 (useTrades), E15 (TradeFormPage existe para edición).

**Archivos a completar:**
```
src/pages/trades/
├── TradesPage.jsx
└── components/
    ├── TradesFilters.jsx    ← filtros por cuenta, par, resultado, fechas
    ├── TradesList.jsx       ← vista lista
    ├── TradeCard.jsx        ← tarjeta individual de trade
    ├── TradesCalendar.jsx   ← vista calendario
    └── DeleteTradeModal.jsx ← modal de confirmación de eliminación
```

**`TradesFilters`:**
- Selector de cuenta
- Selector de par (poblado desde trades existentes)
- Selector de resultado (Todos / Ganadores / Pérdidas / Break Even)
- Inputs de fecha: "Desde" y "Hasta"
- Botones rápidos: "Esta semana" / "Este mes" / "Último mes"
- Botón ✕ para limpiar todos los filtros

**`TradeCard`:**
- Borde izquierdo con color según resultado (verde/rojo/amarillo)
- Badge MADRE / HIJA
- Par y sesión
- Fecha y estado emocional
- P&L en DM Mono con color
- Miniatura de screenshot clicable (si tiene)
- Reglas rotas en rojo (si tiene)
- Nota del setup (colapsada por defecto si > 2 líneas)
- Libreta como panel colapsado con borde dorado (si tiene y es Pro)
- Botones: Editar → `/trades/{id}/edit` | Eliminar → `DeleteTradeModal`

**`DeleteTradeModal`:**
- Si el trade tiene `is_mother: true` y tiene hijas: mostrar "Este trade tiene X réplicas hijas. ¿Qué querés hacer?"
  - Opción A: "Eliminar solo este trade" — soft delete solo la madre, las hijas quedan con `mother_id: null`
  - Opción B: "Eliminar todo el grupo (madre + X hijas)" — soft delete la madre y todas las hijas
- Si no tiene hijas: confirmación simple "¿Eliminar este trade?"
- Ninguna eliminación es instantánea — siempre pasa por este modal

**`TradesCalendar`:**
- Grid de 7 columnas (lun-dom)
- Cada día: número del día, P&L del día, puntos de color (uno por trade)
- Colores del día según P&L: verde (positivo), rojo (negativo), amarillo (cero con trades)
- El día de hoy tiene borde dorado
- Click en día con trades: abre modal con los trades de ese día (colapsables si > 2)
- Navegación prev/next de mes

**Criterio de finalización:**
- [ ] Los filtros funcionan y se combinan correctamente
- [ ] El filtro por rango de fechas funciona con el botón "Este mes"
- [ ] Eliminar una madre con hijas muestra el modal con las dos opciones
- [ ] Eliminar solo la madre deja las hijas con `mother_id: null`
- [ ] Eliminar el grupo hace soft delete de madre y todas las hijas
- [ ] El calendario muestra correctamente los días con/sin trades
- [ ] Click en un día con trades abre el modal con detalles

---

## ETAPA 17 — Módulos Secundarios (Fases, Fondos, Cuentas, Calculadora)

**Objetivo:** Implementar los cuatro módulos secundarios. Todos comparten la misma estructura: lista con estado + formulario de alta + acciones por ítem.

**Dependencias:** E11 (todos los hooks), E12 (lógica de negocio), E08 (layout).

**Archivos a completar:**
```
src/pages/
├── phases/
│   ├── PhasesPage.jsx
│   └── components/
│       ├── PhaseCard.jsx
│       └── CustomPhaseModal.jsx
├── funds/
│   └── FundsPage.jsx
├── accounts/
│   └── AccountsPage.jsx
└── calculator/
    └── CalculatorPage.jsx
```

**`PhasesPage`:**
- Las 6 fases predeterminadas en orden + fases personalizadas al final
- Cada `PhaseCard` muestra: ícono, nombre, descripción, lista de tareas con checkboxes, barra de progreso
- La fase Disciplina muestra condiciones en tiempo real (racha de sesiones limpias + WR)
- Botón "Marcar como completada" activo solo cuando todas las tareas están cumplidas (o las condiciones en tiempo real para Disciplina)
- `CustomPhaseModal` para crear fases personalizadas con: ícono, nombre, subtítulo, descripción, tareas (una por línea)
- Botón "Resetear progreso" por fase (borra el progreso de tareas, no elimina la fase)

**`FundsPage`:**
- Panel de totales: Ingresos totales / Gastos totales / Neto
- Formulario de alta: tipo de movimiento (7 opciones), cuenta asociada (opcional), monto, nota
- Lista del historial con ícono por tipo, monto en verde (ingresos) o rojo (gastos), botón eliminar
- Los challenges y gastos se muestran en rojo; el resto en verde

**`AccountsPage`:**
- Una tarjeta por cuenta con: nombre, tipo, instrumento, tamaño, balance real, P&L total, barra de drawdown
- Botón "Editar" → modal con todos los campos editables
- Botón "Archivar" → `is_archived: true`, la cuenta desaparece del dashboard pero sus trades se conservan
- Botón "Agregar cuenta" → modal de alta con los mismos campos del onboarding paso 2
- Para usuarios Free: solo 1 cuenta — el botón "Agregar" muestra el banner Pro

**`CalculatorPage`:**
- Misma lógica que `LotCalculator` de E15 pero como página standalone
- El selector de cuenta incluye todas las cuentas activas del usuario
- El par se selecciona libremente (no depende del formulario de trade)

**Criterio de finalización:**
- [ ] Marcar una tarea en una fase persiste inmediatamente
- [ ] Completar una fase guarda `is_completed: true` y `completed_at: NOW()` en la tabla
- [ ] Agregar un movimiento de fondos aparece inmediatamente en el historial
- [ ] El balance real de una cuenta en AccountsPage es: `size + sum(negative pnl of trades)`
- [ ] La calculadora funciona idéntico a como funciona integrada en el formulario de trade
- [ ] Un usuario Free no puede agregar más de 1 cuenta (bloqueado en UI y en BD por RLS)

---

## ETAPA 18 — Análisis de Patrones

**Objetivo:** Implementar el módulo de análisis con los 10 patrones, estadísticas por par y sesión, y evolución semanal de WR.

**Dependencias:** E11 (useTrades), E12 (patterns.js), E08 (layout).

**Archivos a completar:**
```
src/pages/patterns/
├── PatternsPage.jsx
└── components/
    ├── PatternCard.jsx
    ├── WeeklyEvolution.jsx  ← gráfico de WR semanal (últimas 12 semanas)
    ├── StatsByPair.jsx      ← tabla: par → WR, P&L prom, R:R prom, trades
    └── StatsBySession.jsx   ← misma tabla para sesiones
```

**El módulo está bloqueado para usuarios Free:**
- Si `!canUseFeature('patterns')`: mostrar página locked con descripción de los 10 patrones y CTA a upgrade
- No ocultar — mostrar el módulo bloqueado con preview de lo que verían

**Los 10 patrones detectados por `patterns.js`:**
1. Revenge trading vs WR normal (compara WR en trades después de una pérdida vs el global)
2. Reglas rotas vs pérdidas (% de trades con reglas rotas que son pérdidas vs el global)
3. Mejor y peor par por WR (tabla ordenada)
4. Mejor sesión por WR (London vs NY vs Asia vs Overlap)
5. Efecto del sueño en WR (si el usuario registra mood 'tired': WR de trades cansado vs normal)
6. Patrón pérdida → regla rota (trades con reglas rotas inmediatamente después de una pérdida)
7. R:R real vs R:R mínimo configurado (% de trades donde el R:R real quedó bajo el mínimo del perfil)
8. Peor día de la semana por WR (lun-vie)
9. Rachas de pérdidas consecutivas (máxima racha, número de veces con ≥3 pérdidas seguidas)
10. Break evens frecuentes (si >15% de los trades son BE, alerta de posible cierre prematuro)

**Requiere mínimo 5 trades para mostrar patrones. Con menos de 5: empty state explicativo.**

**`WeeklyEvolution`:**
- SVG inline (no canvas)
- Puntos por semana, línea conectora
- Línea de referencia horizontal en 50%
- Colores: punto verde si WR ≥ 50%, rojo si < 50%
- Solo las últimas 12 semanas con al menos 1 trade

**Criterio de finalización:**
- [ ] Con menos de 5 trades: empty state (no error)
- [ ] Con 5+ trades: todos los patrones calculan correctamente
- [ ] Un usuario Free ve el módulo bloqueado con preview (no 404 ni error)
- [ ] El gráfico de evolución semanal no rompe con semanas sin trades (simplemente no tiene punto)
- [ ] Las tablas de par y sesión muestran los datos correctos

---

## ETAPA 19 — Ajustes, Backup y Migración v1.x

**Objetivo:** Implementar la página de ajustes completa, el backup JSON, y la migración desde v1.x.

**Dependencias:** E11 (useProfile), E12 (migration.js), E08 (layout).

**Archivos a completar:**
```
src/pages/settings/
├── SettingsPage.jsx
└── components/
    ├── ProfileEditor.jsx    ← edición de todos los campos del perfil
    ├── ChecklistEditor.jsx  ← gestión de ítems del checklist
    ├── BackupPanel.jsx      ← export/import JSON y CSV
    ├── MigrationPanel.jsx   ← importar desde v1.x
    └── DangerZone.jsx       ← eliminar cuenta
```

**`ProfileEditor`:**
- Nombre/alias, país, experiencia, plataforma
- % de riesgo, R:R mínimo, máximo de trades por sesión
- Pares operados (chips editables — agregar/eliminar)
- Sesiones (chips editables)
- Reglas personales (lista editable — agregar/renombrar/eliminar)
- Toggle de idioma (ES/EN)
- Toggle de tema (Oscuro/Claro)
- Guardar todo con un botón → UPDATE en tabla profiles

**`ChecklistEditor`:**
- Lista de los ítems activos del checklist
- Los 13 ítems base se muestran con label "(predeterminado)" — no tienen botón eliminar
- Los ítems personalizados tienen botón renombrar y eliminar
- Botón "Agregar ítem" → input para escribir el ítem + selector de posición (antes de cuál ítem)
- Botón "Restaurar predeterminados" → elimina customizaciones y deja los 13 base

**`BackupPanel`:**

Export JSON:
- Botón "Exportar backup completo"
- Incluye: `{ schema_version: '2.0', cfg: profile, accounts, trades, funds, goals, phases, checklists }` (BCK-C3, BCK-C4)
- Nombre del archivo: `tradeos-backup-YYYY-MM-DD.json`

Import JSON (v2.0):
- Input de archivo `.json`
- Valida que `schema_version` es `'2.0'`
- Muestra preview: "Se restaurarán X trades, Y cuentas, Z días de checklist"
- Requiere confirmación antes de sobreescribir
- Si falla en cualquier punto: rollback completo (DATA-A2)

Export CSV de trades:
- Solo disponible en Plan Pro
- Columnas: Fecha, Par, Sesión, Cuenta (nombre de firma, no ID), Resultado, P&L, R:R, Riesgo%, Mood, Reglas Rotas, Tags, Nota
- Nombre: `trades-YYYY-MM-DD.csv`

**`MigrationPanel`:**
- Sección separada: "Importar desde TradeOS v1.x"
- Input de archivo `.json` (backup del archivo HTML v1.x)
- Llama `validateV1Backup()` → muestra preview
- Si válido: llama `transformV1ToV2()` → INSERT batch en Supabase
- Si falla: rollback, mensaje de error específico
- Éxito: "Se importaron X trades, Y cuentas, Z días de checklist"

**`DangerZone`:**
- Botón rojo "Eliminar todos mis trades" (confirmación: escribir "ELIMINAR")
- Botón rojo "Eliminar mi cuenta permanentemente" (REC-A2: link a email de soporte para proceso manual)
- Los eliminados de cuenta van por email — no por botón automático en v2.0

**Criterio de finalización:**
- [ ] Editar el nombre del perfil y guardar actualiza `profiles.name` en Supabase
- [ ] El CSV exportado tiene nombres de firma reales, no IDs
- [ ] El backup JSON incluye `schema_version: '2.0'` y todos los datos
- [ ] Importar un backup de v1.x válido crea todos los datos en Supabase sin errores
- [ ] Si se importa un JSON inválido, no se modifica ningún dato existente (rollback)
- [ ] Los 13 ítems base del checklist no tienen botón eliminar en `ChecklistEditor`

---

## ETAPA 20 — Monetización (Stripe) y Panel Admin

**Objetivo:** Implementar el flujo de pago con Stripe, las Edge Functions de webhook, y el panel admin básico.

**Dependencias:** Todas las etapas anteriores completadas. Cuenta Stripe configurada con los Price IDs.

**Archivos a crear:**
```
src/pages/
├── upgrade/
│   └── UpgradePage.jsx     ← comparación Free vs Pro + botón de pago

supabase/functions/
├── create-checkout-session/
│   └── index.ts
├── stripe-webhook/
│   └── index.ts
├── customer-portal/
│   └── index.ts
└── health/
    └── index.ts

src/pages/admin/
└── AdminPage.jsx            ← panel admin básico
```

**`UpgradePage`:**
- Tabla comparativa Free vs Pro con las funciones listadas en PRODUCT_REQUIREMENTS_V2 §7 y §8
- Precio mensual ($9) y anual ($79 = $6.58/mes)
- Botón "Empezar Pro mensual" y "Empezar Pro anual"
- Ambos llaman a `create-checkout-session` Edge Function con el Price ID correspondiente
- Al retornar de Stripe: la página muestra el nuevo plan activo

**`create-checkout-session` Edge Function:**
```typescript
// Recibe: { priceId: string }
// Verifica JWT del request
// Crea o recupera Stripe Customer para este user_id
// Crea Stripe Checkout Session con:
//   - price_id del plan
//   - metadata: { user_id }
//   - success_url: VITE_APP_URL + '/settings?upgraded=true'
//   - cancel_url: VITE_APP_URL + '/upgrade'
// Retorna: { url: checkoutUrl }
```

**`stripe-webhook` Edge Function:**
```typescript
// Verifica firma HMAC de Stripe (PAY-C2)
// Maneja estos eventos:
//   - checkout.session.completed → crear/actualizar subscriptions
//   - customer.subscription.updated → actualizar plan y fechas
//   - customer.subscription.deleted → degradar a Free
//   - invoice.payment_failed → marcar status como 'past_due'
// En cada evento: UPDATE en profiles.plan + UPDATE en subscriptions
// Idempotencia: verificar event.id antes de procesar (PAY-A4)
```

**`customer-portal` Edge Function:**
```typescript
// Crea una sesión del portal de clientes de Stripe
// Retorna la URL del portal
// El cliente redirige al usuario a esa URL
```

**`health` Edge Function:**
```typescript
// GET /health
// Verifica conexión a Supabase (SELECT 1)
// Retorna: { status: 'ok', timestamp } o 503 si falla
```

**`AdminPage`** — solo accesible con `role = 'admin'` en profiles:
- Métricas: usuarios registrados total, usuarios activos esta semana, MRR actual, usuarios por plan
- Lista de últimos 20 usuarios registrados (email, plan, fecha de registro)
- Buscador por email
- Acción: cambiar plan de un usuario manualmente (para casos de soporte)
- Los queries de admin usan la service role key via Edge Function — nunca el anon key

**Variables de entorno adicionales para esta etapa (solo en Edge Functions):**
```bash
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRO_MONTHLY_PRICE_ID=price_...
STRIPE_PRO_YEARLY_PRICE_ID=price_...
```

**Criterio de finalización:**
- [ ] PAY-C2: el webhook verifica la firma antes de procesar
- [ ] PAY-C3: el cambio de plan ocurre SOLO desde el webhook, no desde el cliente
- [ ] Un usuario Free puede hacer checkout y después de pagar tiene `plan = 'pro'` en su perfil
- [ ] El portal de Stripe abre correctamente desde Ajustes con el historial de facturas
- [ ] El endpoint `/health` retorna 200 cuando Supabase está disponible
- [ ] El panel `/admin` no es accesible por usuarios sin `role = 'admin'`
- [ ] PERM-C6: intentar acceder a `/admin` con un usuario Pro redirige a `/dashboard`

---

## CHECKLIST DE LANZAMIENTO A PRODUCCIÓN

Ejecutar este checklist después de completar E20 y antes del primer deploy a producción.

### Seguridad crítica (SECURITY_REQUIREMENTS_V2 — todos los CRÍTICOS)
- [ ] AUTH-C1 al AUTH-C6 verificados
- [ ] PERM-C1 al PERM-C6 verificados
- [ ] DATA-C1 al DATA-C5 verificados
- [ ] PAY-C1 al PAY-C5 verificados
- [ ] BCK-C1 al BCK-C4 verificados
- [ ] REC-C1 al REC-C3 verificados
- [ ] IMG-C1 al IMG-C5 verificados
- [ ] LOSS-C1 al LOSS-C4 verificados
- [ ] LOG-C1 al LOG-C3 verificados
- [ ] MON-C1 al MON-C3 verificados
- [ ] ENV-C1 al ENV-C5 verificados
- [ ] DEP-C1 al DEP-C6 verificados

### Prueba de fuego (ejecutar manualmente en producción)
- [ ] Flujo completo: email → magic link → onboarding → dashboard
- [ ] Registro de trade con imagen: formulario → upload → guardado → vista en Mis Trades
- [ ] Replicador: trade madre → réplicas hijas → verificar mother_id correcto en BD
- [ ] Backup: exportar JSON → crear cuenta nueva → importar → todos los datos coinciden
- [ ] Migración v1.x: importar backup real de un usuario de v1.x
- [ ] Pago: checkout Pro → webhook → plan actualizado → acceso a features Pro
- [ ] Cancelación: cancelar plan → acceso hasta fin de período → degradación a Free
- [ ] Seguridad: con JWT de usuario A, SELECT en cualquier tabla → 0 filas de usuario B
- [ ] Health check: GET /health → 200

### Build y deploy
- [ ] `npm run build` sin warnings ni errores
- [ ] `npm run test` con 0 failures
- [ ] Variables de entorno de producción configuradas en Vercel (distintas de staging)
- [ ] Migrations aplicadas en BD de producción (no solo staging)
- [ ] Sentry instalado y con un error de prueba llegando correctamente
- [ ] UptimeRobot configurado para `app.tradeos.io`
- [ ] Dominio `app.tradeos.io` apunta a Vercel con SSL activo

---

*Documento generado por CTO — TradeOS*  
*Fecha: Junio 2026*  
*Para Claude Code: ejecutar las etapas en orden numérico estricto. No avanzar si el criterio de finalización no se cumple al 100%.*
