# PROJECT_RULES.md
## TradeOS — Reglas del Proyecto

**Versión:** 1.0 · **Junio 2026**  
**Autoridad:** Juan Pa — todas las reglas aquí son decisiones tomadas, no sugerencias.  
Cualquier agente de IA, colaborador o desarrollador que trabaje en este proyecto debe leer este archivo antes de tocar cualquier código, diseño o documento.

---

## 1. Reglas de Datos — NUNCA violar

1. **`isMother` e `isChild` son mutuamente excluyentes.** Un trade no puede ser madre e hijo al mismo tiempo. Si se replica un trade hijo, se rechaza la operación.

2. **La libreta (`notebook`) solo vive en el trade madre.** Nunca copiar el campo `notebook` a réplicas hijas al usar el replicador. Las réplicas se crean con `isChild: true, motherId: <id>` y sin `notebook`.

3. **El backup es la única fuente de verdad.** Al importar un JSON de backup, se restauran los datos exactamente como están — sin transformaciones, sin migración de esquema, sin normalización. Si el esquema cambia entre versiones, se documenta en CHANGELOG.md y se maneja con una función de migración explícita.

4. **Los 13 ítems predeterminados del checklist nunca se eliminan.** Al agregar ítems personalizados, los base siempre se incluyen. El array `defaultCL` es inmutable.

5. **La calculadora usa el balance real, no el tamaño original.** `balance_real = account.size + pnl_acumulado_negativo`. Nunca pasar `account.size` directamente si hay pérdidas registradas. Mostrar error antes que dar un número incorrecto.

6. **Los datos nunca salen del navegador del usuario.** Sin llamadas a APIs externas con datos del trader, sin analytics, sin telemetría, sin logging remoto. Cero. La única excepción son las fuentes de Google Fonts, que no reciben ningún dato del usuario.

---

## 2. Reglas de Código

7. **Siempre validar con Node.js después de cualquier cambio al script principal.**  
   ```bash
   node --check trader-os.html  # debe retornar sin errores
   ```
   Si el check falla, no se sube el cambio.

8. **Cero closing tags sin escapar dentro de strings JavaScript.**  
   `</script>` dentro de un string JS rompe el parser HTML. Siempre usar `<\/script>` y `<\/div>` dentro de template literals o strings.

9. **Todos los `getElementById` deben tener null check antes de operar.**  
   ```javascript
   // ❌ Incorrecto
   document.getElementById('mi-elem').textContent = valor;
   
   // ✅ Correcto
   const el = document.getElementById('mi-elem');
   if (el) el.textContent = valor;
   ```

10. **`sv()` es la única función que navega entre vistas.** No manipular `.view.on` directamente desde ningún otro lugar del código.

11. **`window.addEventListener('load', ...)` debe aparecer exactamente una vez.** Duplicarlo causa bugs difíciles de rastrear. Verificar en cada edición mayor.

12. **Todos los elementos dentro de `.main` deben tener `class="view"`.** Sin excepción. El div depth del body debe cerrar en 0 antes del script principal.

13. **El archivo es HTML vanilla.** Sin React, Vue, Angular, ni ningún framework de runtime. Sin npm. Sin proceso de build. Sin dependencias de node_modules. Si se necesita una librería, se evalúa si puede inlinarse o si realmente es necesaria.

14. **No editar el archivo con editores que auto-formateen o minifiquen HTML.** El archivo tiene estructura deliberada. Editores como Prettier con configuración agresiva pueden romper el parser JS interno. Usar VS Code con auto-format desactivado para este archivo.

---

## 3. Reglas de UI/UX

15. **El dorado `#c8a96e` es el color primario de marca.** No reemplazar, no "actualizar", no buscar alternativas. Es una decisión de identidad, no una preferencia.

16. **Verde musgo `#5a7a4a` = ganancia / positivo. Rojo ladrillo `#8a4a35` = pérdida / negativo.** No usar verde neón, verde lima, rojo brillante ni coral. Si el diseño necesita otro tono, se define como nueva variable CSS, no se reemplaza la existente.

17. **Todas las métricas numéricas van en DM Mono.** P&L, win rate, R:R, drawdown, lotaje, score — todos en fuente monospace. DM Sans es para UI y texto, no para números de trading.

18. **El sidebar tiene exactamente 9 secciones.** No agregar una sección décima sin rediseño aprobado del sidebar completo.

19. **El eslogan no cambia:** *"The operating system for serious traders."* en inglés, *"El sistema operativo para traders serios."* en español. En ninguna pieza de marketing, copy interno ni UI se modifica.

20. **La calculadora solo muestra resultado si es exacto.** Si el par no tiene valor por pip calculable con certeza (ej: USD/JPY con tipo de cambio variable), mostrar mensaje explicativo. Nunca dar un número aproximado sin etiquetarlo como tal.

---

## 4. Reglas de Documentación

21. **MASTER_CONTEXT.md es la fuente principal de verdad.** Cualquier decisión técnica, de diseño o de negocio debe estar reflejada ahí. Si un agente de IA o desarrollador tiene duda, lee ese archivo primero.

22. **CHANGELOG.md se actualiza con cada cambio que llega a producción.** No es opcional. Si se lanza una versión nueva sin entrada en el CHANGELOG, la versión no está documentada.

23. **TASKS.md refleja el estado real del trabajo.** Si una tarea está terminada, se marca `[x]` y se mueve a "Completado Recientemente". No dejar tareas cerradas en la columna de activas.

24. **Las reglas en este archivo son decisiones, no discusión.** Si algo debe cambiar, el cambio se propone, se evalúa, y se actualiza este documento. No se trabaja contra las reglas mientras están vigentes.

---

## 5. Reglas de Negocio

25. **El producto es para traders que ya saben operar.** El tono, el onboarding y la UX asumen conocimiento de conceptos como drawdown, R:R, lotaje, sesiones de mercado y prop firms. No hay modo "para principiantes" ni tooltips que expliquen qué es un pip.

26. **El precio de lanzamiento es $19 USD.** Cualquier promoción, descuento o cambio de precio se decide por Juan Pa explícitamente — no es una variable que se ajusta sola.

27. **Los afiliados reciben 30% de comisión.** Esta cifra no cambia sin decisión explícita. No hay negociación individual de porcentaje con afiliados.

28. **No hay publicidad dentro del producto.** TradeOS no muestra anuncios, no promociona prop firms de forma pagada, no tiene banners de sponsors. La experiencia es limpia.

29. **El soporte va a tradeossoporte@gmail.com.** Cualquier otra dirección que aparezca en el producto es un error.

---

## 6. Qué pasa cuando se rompe una regla

- Si se detecta una violación de las reglas de datos (§1): **rollback inmediato** al último backup funcional.
- Si se detecta una violación de las reglas de código (§2): **no se sube el cambio** hasta que pase el check de Node.js.
- Si se detecta una violación de UI (§3): **se revierte el elemento** al token correcto definido en UI_GUIDELINES.md.
- Si hay duda sobre si algo viola una regla: **no se hace** hasta tener claridad. La omisión es preferible al error.

---

*Para el contexto completo del proyecto, ver MASTER_CONTEXT.md.*  
*Para reglas de diseño con ejemplos visuales, ver UI_GUIDELINES.md.*
