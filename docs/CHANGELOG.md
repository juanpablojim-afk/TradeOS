# CHANGELOG.md
## TradeOS — Historial de Cambios

Formato: [Versión] — Fecha · Descripción breve  
Convención: [Semver](https://semver.org/) — MAJOR.MINOR.PATCH

---

## [1.0.0] — Junio 2026 · Lanzamiento Beta

### Añadido
- Onboarding personalizado en 9 pasos (perfil, cuentas, estilo, reglas, horario, checklist, metas, fases, psicología)
- Dashboard con Score de Consistencia, 6 métricas clave y curva de equity SVG
- Paneles mini por cuenta con barra de drawdown y alerta visual al 70%
- Checklist diario con 13 ítems predeterminados, temporizador de sesión y campo de bias del día
- Formulario de registro de trade con: par, sesión, cuenta, resultado, P&L, R:R, riesgo %, estado emocional, reglas rotas (multi-selección), tags de setup, captura de imagen
- Libreta de análisis por trade (solo en trade madre, nunca en réplicas hijas)
- Sistema de replicación de trades (madre + hijos) con P&L editable por cuenta
- Calculadora de lotaje exacta: forex majors (EUR/USD, GBP/USD, AUD/USD, NZD/USD) y futuros CME (NQ, MNQ, ES, MES, YM, MYM, RTY, GC, CL)
- Vista "Mis Trades" con filtros por cuenta, par y resultado + badges MADRE/HIJA
- Vista calendario con días coloreados (verde/rojo/amarillo), dots de trades y modal por día
- Sección de Fases con 6 fases predeterminadas, condiciones en tiempo real para Disciplina, y creador de fases personalizadas
- Gestión de fondos: depósitos, retiros, payouts, comisiones, profit splits por cuenta
- Análisis de patrones con evolución semanal de win rate
- Sistema bilingüe completo ES/EN con toggle en sidebar
- Modo oscuro (Obsidian Gold) y modo claro (Crema Dorado) con toggle persistente
- Exportación de backup en JSON e importación con restauración exacta
- Exportación de trades a CSV
- Sistema de metas financieras con barras de progreso
- Splash de beta para colaboradores exclusivos (`jft_beta` en localStorage)
- Validación de sintaxis con Node.js: PASS
- Archivo único HTML (225KB), 100% offline, sin backend, sin APIs externas

### Limitaciones conocidas en v1.0
- No funciona en Safari iOS con archivos locales (localStorage bloqueado)
- Imágenes base64 pueden saturar localStorage con uso intensivo
- No hay sync entre dispositivos (localStorage es local al navegador)

---

## [0.9.0] — Mayo 2026 · Beta Interna

### Añadido
- Primera versión funcional con onboarding, journal y dashboard básico
- Score de Consistencia (reemplazó sistema de XP experimental)
- Soporte inicial para futuros CME en calculadora

### Corregido
- Script tag partido por `</script>` en strings JS — separado en 2 `<script>` independientes
- Duplicación de `window.addEventListener('load')` eliminada
- `.fi-ic` renombrado a `.cfg-cell` por conflicto de estilos CSS

---

## Próximos cambios planeados

Ver ROADMAP.md para versiones futuras (v1.1, v1.2, v2.0...)
