# TASKS.md
## TradeOS — Tareas Activas

**Actualizado:** Junio 2026  
**Sistema:** GitHub Issues / Notion (según flujo del equipo)

---

## 🔴 Crítico / Bloqueante

- [ ] **[BUG]** Safari iOS bloquea localStorage en archivos locales — impide uso en iPhone/iPad
  - Solución temporal: hospedar en Netlify o GitHub Pages
  - Prioridad: Alta — afecta a traders móviles

- [ ] **[BUG]** Imágenes base64 de trades pueden saturar localStorage (~5-10MB límite)
  - Solución: comprimir imagen antes de guardar (canvas + toDataURL con calidad reducida)

---

## 🟡 Alta Prioridad

- [ ] **[FEATURE]** Barra de uso de localStorage en Ajustes
  - Mostrar: usado / disponible estimado
  - Disparar advertencia visual si supera 80%

- [ ] **[FEATURE]** Selector de idioma persistente en el onboarding
  - Actualmente el idioma se puede perder al recargar en edge cases

- [ ] **[MEJORA]** Mejorar rendimiento del calendario con más de 200 trades
  - Virtualizar renderizado de días o paginar por mes lazy

- [ ] **[MEJORA]** Validación en formulario de trade con mensajes de error claros por campo

---

## 🟢 Media Prioridad

- [ ] **[FEATURE]** Exportar trades a PDF además de CSV
- [ ] **[FEATURE]** Filtro por rango de fechas en "Mis Trades"
- [ ] **[FEATURE]** Vista de estadísticas por par (win rate de XAU/USD vs GBP/USD, etc.)
- [ ] **[FEATURE]** Vista de estadísticas por sesión (London vs NY)
- [ ] **[MEJORA]** Animación de entrada al dashboard post-onboarding
- [ ] **[MEJORA]** Tooltip en métricas del dashboard explicando cómo se calcula cada una

---

## 🔵 Baja Prioridad / Backlog

- [ ] **[FEATURE]** Modo "Sólo lectura" para compartir el journal con mentor
- [ ] **[FEATURE]** Widget de cuenta por separado (vista simplificada por cuenta)
- [ ] **[REFACTOR]** Extraer lógica de calculadora de lotaje a función pura testeable
- [ ] **[REFACTOR]** Separar CSS en variables más consistentes con sistema de diseño
- [ ] **[DOC]** Guía de usuario en PDF (cómo usar TradeOS, paso a paso)
- [ ] **[DOC]** Video tutorial de onboarding (YouTube / Loom)

---

## ✅ Completado Recientemente

- [x] Score de Consistencia reemplaza sistema de XP
- [x] Libreta por trade (solo en trade madre, no en réplicas)
- [x] Sistema de replicación madre/hija con P&L por cuenta
- [x] Checklist diario con temporizador de sesión
- [x] Modo oscuro Obsidian Gold + modo claro Crema Dorado
- [x] Calculadora de lotaje exacta para futuros CME y forex majors
- [x] Fases personalizadas con condiciones en tiempo real
- [x] Exportación/importación de backup en JSON
- [x] Sistema bilingüe ES/EN completo

---

## Convenciones

| Prefijo | Significado |
|---------|-------------|
| `[BUG]` | Comportamiento incorrecto ya existente |
| `[FEATURE]` | Funcionalidad nueva |
| `[MEJORA]` | Mejora sobre algo que ya existe |
| `[REFACTOR]` | Cambio interno sin impacto visible |
| `[DOC]` | Documentación o materiales de soporte |
