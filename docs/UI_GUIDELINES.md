# UI_GUIDELINES.md
## TradeOS — Guía de Diseño e Interfaz

**Versión:** 1.0 · **Junio 2026**  
Estas reglas son invariantes. Ningún cambio de UI debe violarlas sin aprobación explícita.

---

## 1. Identidad Visual

### Nombre y Eslogan

- **Nombre:** TradeOS (siempre con mayúscula T y OS)
- **Logotipo textual:** `Trade<span color=#c8a96e>OS</span>`
- **Eslogan EN:** *"The operating system for serious traders."*
- **Eslogan ES:** *"El sistema operativo para traders serios."*
- El eslogan **nunca cambia**, en ninguna versión del producto.

---

## 2. Paleta de Colores

### Modo Oscuro — Obsidian Gold (default)

| Token | Valor | Uso |
|-------|-------|-----|
| `--bg` | `#07070f` | Fondo principal |
| `--bg2` | `#0e0e1a` | Fondo de cards y modales |
| `--bg3` | `#16161f` | Hover states, inputs |
| `--gold` | `#c8a96e` | **Color de identidad de marca** — primario |
| `--text` | `#e8e8f5` | Texto principal |
| `--text2` | `#9090a8` | Texto secundario |
| `--text3` | `#52526a` | Texto terciario / placeholder |
| `--win` | `#5a7a4a` | Verde musgo — ganancia / positivo |
| `--loss` | `#8a4a35` | Rojo ladrillo — pérdida / negativo |
| `--be` | `#b8a060` | Amarillo apagado — break even |
| `--border` | `rgba(200,169,110,0.15)` | Bordes de cards |

### Modo Claro — Crema Dorado

| Token | Valor | Uso |
|-------|-------|-----|
| `--bg` | `#f5f0e8` | Fondo principal |
| `--bg2` | `#ede8e0` | Fondo de cards |
| `--bg3` | `#e5e0d8` | Hover states |
| `--gold` | `#a07840` | Dorado oscuro (contraste sobre claro) |
| `--text` | `#1a1a2e` | Texto principal |
| `--text2` | `#4a4a6a` | Texto secundario |
| `--text3` | `#8a8aaa` | Placeholder |

### Reglas de color — NUNCA violar

- El dorado `#c8a96e` es el color primario de marca. No reemplazar por ningún otro tono.
- El verde de ganancia es **musgo** (`#5a7a4a`). No usar verde limón, esmeralda ni neón.
- El rojo de pérdida es **ladrillo** (`#8a4a35`). No usar rojo brillante (#f00) ni coral.
- No introducir colores fuera de la paleta sin definirlos como variables CSS.

---

## 3. Tipografía

| Familia | Uso | Fallback |
|---------|-----|---------|
| **Bebas Neue** | Títulos de display, nombres de secciones grandes | `sans-serif` |
| **DM Sans** | UI general, labels, botones, texto de párrafo | `sans-serif` |
| **DM Mono** | **TODAS las métricas numéricas** — P&L, %, R:R, lotaje | `monospace` |

### Regla crítica
> Todos los números que representen métricas de trading (P&L, win rate, drawdown, R:R, lotaje, score) van en **DM Mono**. Nunca en DM Sans.

---

## 4. Componentes

### Botones

```css
.btn-p   /* Primario — fondo dorado, texto oscuro */
.btn-g   /* Secundario — fondo gris oscuro, texto claro */
.btn-d   /* Destructivo — borde rojo, texto rojo */
```

- Los botones primarios usan `--gold` como fondo.
- Los botones destructivos (eliminar, resetear) siempre tienen confirmación antes de ejecutar.

### Cards

- Fondo: `--bg2`
- Borde: `1px solid var(--border)`
- Border radius: `12px` para cards grandes, `8px` para cards pequeñas
- Sin sombras externas — el contraste de fondo hace la separación visual

### Badges / Pills

```
.badge.bp   /* Completado — verde musgo */
.badge.ba   /* Activo — dorado */
.badge.bs   /* Bloqueado — gris */
```

### Métricas (dashboard)

- Número grande en **DM Mono**, tamaño `22-28px`
- Label pequeño en **DM Sans**, tamaño `10-11px`, color `--text3`
- P&L positivo: color `--win`
- P&L negativo: color `--loss`

---

## 5. Layout y Navegación

### Sidebar

- El sidebar tiene exactamente **9 secciones fijas**:
  1. Dashboard
  2. Checklist
  3. Registrar Trade
  4. Mis Trades
  5. Fases del Plan
  6. Gestión de Fondos
  7. Calculadora
  8. Patrones
  9. Ajustes

- **No agregar ni quitar secciones** sin un rediseño completo aprobado.
- El ítem activo tiene borde izquierdo dorado (`border-left: 2px solid var(--gold)`).

### Vistas

- Toda vista dentro de `.main` debe tener `class="view"`.
- La función `sv()` es la única que navega entre vistas. No manipular `.view.on` directamente.
- Las vistas no usan `display:block/none` directo — usan la clase `.on`.

### Modales

- Fondo overlay: `rgba(0,0,0,0.7)` con `backdrop-filter: blur(4px)`
- El modal se cierra al hacer click en el overlay (no solo en botón X)
- Siempre tener un botón de cierre visible (X en esquina superior derecha o "Cancelar")

---

## 6. Feedback Visual

### Notificaciones (toast)

- Aparecen en la esquina inferior derecha o superior central
- Duración: 2.5 segundos
- Éxito: borde izquierdo `--win`
- Error: borde izquierdo `--loss`
- Texto corto — máximo 5 palabras

### Estados vacíos

- Siempre mostrar un mensaje cuando una lista está vacía.
- Incluir una acción clara ("Registrá tu primer trade →").
- Nunca mostrar una sección en blanco sin contexto.

### Loading states

- Para operaciones asíncronas futuras: usar skeleton placeholders, no spinners.

---

## 7. Reglas Generales de UX

1. El trader es un profesional — el tono de la UI es directo, no condescendiente.
2. Las confirmaciones destructivas siempre usan `confirm()` o modal propio — nunca acción inmediata.
3. Los formularios guardan estado temporalmente — si el trader cierra por error, no pierde lo que escribió.
4. Las métricas se muestran siempre con contexto (ej: "Win Rate" no aparece solo el número "63%").
5. El sistema bilingüe ES/EN es completo — ningún texto hardcodeado en español sin su equivalente en inglés.
6. El modo oscuro es el default — el modo claro es una opción, no el estándar.

---

## 8. Qué NO hacer

- ❌ Usar colores fuera de la paleta sin definirlos como variables CSS
- ❌ Usar números en DM Sans (siempre DM Mono para métricas)
- ❌ Verde limón o rojo brillante para resultados de trades
- ❌ Agregar secciones al sidebar sin aprobación
- ❌ Enviar datos a servidores externos (la app es 100% local)
- ❌ Usar `display:none` directo en vistas — usar clase `.on`
- ❌ Manipular `.view.on` fuera de la función `sv()`
- ❌ Texto hardcodeado en un solo idioma sin su par traducido

---

*Ver MASTER_CONTEXT.md §12 para las reglas invariantes completas del proyecto.*
