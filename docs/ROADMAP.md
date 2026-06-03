# ROADMAP.md
## TradeOS — Hoja de Ruta del Producto

**Versión actual:** 1.0 Beta  
**Última actualización:** Junio 2026  
**Mantenedor:** Juan Pa · tradeossoporte@gmail.com

---

## Estado Actual — v1.0 Beta ✅

Producto funcional como archivo HTML local. Distribuido en Gumroad a $19 USD.

---

## v1.1 — Pulido y Estabilidad
**Objetivo:** Cerrar bugs conocidos y mejorar experiencia del usuario actual.

- [ ] Comprimir imágenes base64 antes de guardar en localStorage (evitar saturación)
- [ ] Límite visual de almacenamiento en Ajustes (barra de uso de localStorage)
- [ ] Mensaje de advertencia cuando el almacenamiento supera el 80%
- [ ] Validación de formulario de trade con feedback visual claro
- [ ] Mejorar rendimiento del calendario con muchos trades (>200)
- [ ] Fix: checklist no recuerda estado tras recargar en algunos navegadores

---

## v1.2 — Hosting y Accesibilidad
**Objetivo:** Hacer que la app funcione en iOS y sin archivos locales.

- [ ] Hospedar versión web en dominio propio (ej. `app.tradeos.io`)
- [ ] Soporte para Safari iOS (actualmente bloqueado por política de localStorage en archivos locales)
- [ ] PWA — instalable en móvil sin App Store
- [ ] URL persistente para compartir entre dispositivos del mismo usuario (pre-cloud)

---

## v2.0 — Cloud y Multi-dispositivo
**Objetivo:** Migrar de localStorage a backend real. Mayor retención y valor percibido.

- [ ] Backend con Supabase (plan gratuito — PostgreSQL + Auth)
- [ ] Autenticación por magic link (sin contraseña)
- [ ] Sincronización automática entre dispositivos
- [ ] Backup automático en la nube (reemplaza exportación manual en JSON)
- [ ] Historial de versiones del perfil del trader

---

## v2.1 — Plan Freemium
**Objetivo:** Modelo de adquisición escalable con conversión a Pro.

- [ ] Tier gratuito: 1 cuenta, 50 trades, sin patrones avanzados
- [ ] Tier Pro ($9/mes o $79/año): ilimitado + sync cloud + todas las funciones
- [ ] Página de upgrade dentro de la app
- [ ] Integración de pagos (Stripe o Gumroad Memberships)
- [ ] Dashboard de afiliados (30% de comisión recurrente)

---

## v3.0 — SaaS Completo
**Objetivo:** Producto B2C maduro con posibilidad de expansión B2B.

- [ ] App móvil nativa (React Native)
- [ ] Webhooks MT5/MT4 para importar trades automáticamente
- [ ] Alertas por email/WhatsApp (drawdown, racha rota, meta alcanzada)
- [ ] Análisis de patrones con IA (resumen semanal generado automáticamente)
- [ ] Plan Teams para academias y grupos de traders ($49/mes hasta 10 usuarios)
- [ ] Licencias institucionales para prop firms

---

## Ideas en Evaluación (sin fecha)

- Integración con TradingView para importar screenshots automáticamente
- Modo "Competencia" entre traders de un mismo grupo
- Calculadora de lotaje vía bot de Telegram

---

*Ver IDEAS.md para propuestas detalladas sin prioridad asignada aún.*
