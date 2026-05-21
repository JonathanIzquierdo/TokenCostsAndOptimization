# 03 — Reglas innegociables

Estas reglas no se discuten. Romperlas arruina el entregable.

---

## Regla 1 — Ningún número sin fuente verificada

**Qué significa:** Ningún dato numérico, porcentaje, multiplicador, precio o métrica puede aparecer en el proyecto sin una URL de fuente real en `data/verified-metrics.md`.

**Esto incluye:**
- El cuerpo del artículo
- La documentación técnica
- Los visuales (gráficos, diagramas, tablas con números)
- Los captions de figuras
- Los ejemplos

**Si tenés que usar un número y no lo tenés con fuente:**
1. Marcalo en el texto como `[VERIFICAR: descripción del dato]`
2. Avisale al usuario que falta verificar
3. Hacé web search para encontrar la fuente
4. Si no la encontrás: **eliminá el número del texto** y reescribí la frase sin el dato

**Lo que NO está permitido:**
- ❌ "Aproximadamente un 30%..."
- ❌ "Suele rondar el 50%..."
- ❌ Promediar dos fuentes para inventar un "valor medio"
- ❌ Convertir un rango (15-28%) en un número fijo ("24%")
- ❌ Usar un número que viste en otro archivo del repo si ese archivo no tiene fuente

**Lo que SÍ está permitido:**
- ✅ Citar el número EXACTAMENTE como está en `verified-metrics.md`
- ✅ Usar el rango completo si es un rango (no convertirlo a punto)
- ✅ Citar al lado: "(fuente: Anthropic docs)" con el link disponible en `verified-metrics.md`

---

## Regla 2 — No resumir documentación técnica

**Qué significa:** La documentación técnica (`PROD/technical-docs-*.md`) es exhaustiva por diseño.

**Cita textual del dueño del proyecto:**
> "Si hay mucha data no me molesta, no trates de resumir."

**Esto incluye:**
- No comprimir secciones para que entren en menos páginas
- No eliminar ejemplos de código por "redundancia"
- No quitar tablas comparativas por extensión
- No simplificar explicaciones técnicas porque "son largas"

**El artículo SÍ se sintetiza.** La documentación NO.

Si el usuario te pide "resumí la documentación", confirmá: *"La regla del proyecto es no resumir la doc técnica. ¿Querés que igual lo haga (y lo guardamos como variante), o preferís un nuevo entregable tipo 'resumen ejecutivo'?"*

---

## Regla 3 — Foco en costo y gobernanza

**Qué significa:** Cada tema, sección, dato o ejemplo debe responder a una de estas dos preguntas:

1. **¿Esto mueve la factura?** (impacto en costo)
2. **¿Es accionable para el equipo?** (control/gobernanza)

**Si la respuesta a ambas es "no" o "indirectamente":** no va en el proyecto.

**Ejemplo de tema descartado por esta regla:** "Lost in the middle" problem. Es un tema interesante de accuracy/calidad, pero el vínculo con el costo es indirecto. **Descartado.**

**Ejemplos de temas que pasan el filtro:**
- Prompt caching → 90% descuento → mueve factura ✅
- Output vs input pricing → impacta cada llamada → mueve factura ✅
- Auto Mode Copilot → activación 1 click → accionable ✅
- Extended thinking display:omitted → costo oculto → mueve factura ✅

---

## Regla 4 — No explicar lo básico

**Qué significa:** La audiencia ya sabe qué es un LLM, un token, un modelo, un prompt, una API, un repo. **No explicarles eso.**

**Lo que SÍ va explicado:**
- Diferencias específicas entre tiers de modelos en pricing
- Detalles del nuevo Copilot billing (porque acaba de cambiar)
- Funcionamiento técnico de prompt caching
- Cómo funciona RouteLLM internamente
- Setup de LiteLLM o Portkey

**Lo que NO va explicado:**
- ❌ "Un token es la unidad básica que procesa un LLM..."
- ❌ "GitHub Copilot es una herramienta de asistencia de código..."
- ❌ "Anthropic es la empresa que creó Claude..."
- ❌ "Cuando enviás un prompt al modelo..."

---

## Regla 5 (operacional) — Actualizar `02-state.md` al cerrar sesión

Si trabajaste en el proyecto y hiciste cambios:

1. **Antes de cerrar la sesión**, actualizá `.context/02-state.md`
2. Mencioná: fecha, modelo usado, qué se hizo, próximo paso
3. Si hay decisiones nuevas: agregalas a `.context/07-decisions.md`
4. Commit con mensaje claro

**Si no actualizás el state**, la próxima sesión va a empezar desinformada y va a romper algo.

---

## Regla 6 (operacional) — Confirmar antes de tocar `PROD/`

Los archivos en `PROD/` son entregables. Antes de tocarlos:

1. Confirmá al usuario qué vas a cambiar
2. Confirmá en qué archivo exacto
3. Confirmá que entendiste la intención del cambio
4. **Después** hacé el cambio

Las modificaciones a `research/`, `data/` o `.context/` son más livianas pero igual: si vas a borrar algo, confirmá.
