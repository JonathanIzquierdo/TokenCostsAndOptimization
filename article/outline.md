# Outline del Artículo — Borrador v1

**Título tentativo:** "El dinero que se pierde en cada conversación con la IA"
**Subtítulo:** "Cómo los tokens se convirtieron en la nueva métrica de eficiencia de ingeniería — y qué hacer al respecto"

**Tono:** Storytelling técnico. Arrancar con un dato que sorprenda, llevar al lector por capas de conciencia, terminar con acciones concretas.
**Largo estimado:** 2.500–3.500 palabras
**Audiencia:** Desarrolladores y tech leads de Visma. No necesita explicar qué es un LLM ni qué es un token.

---

## Estructura propuesta

### Apertura — El dato que cambia la conversación
- Hook: el contexto re-enviado es el 62% de la factura (LeanOps 2026)
- La analogía: pagamos por el trabajo del modelo, pero en realidad pagamos por lo que le contamos antes de trabajar
- Transición: esto no es un problema de modelo — es un problema de arquitectura

### Sección 1 — El problema invisible: cómo crece el costo sin que nadie lo vea
- La sesión que empieza en 5K y llega a 200K tokens en el turno 50
- Output tokens cuestan 4–6x más que input — y nadie limita los outputs
- Extended thinking: pagás por el razonamiento invisible aunque uses `display: omitted`
- El efecto Copilot: el 10–15% de usuarios concentra el 60–70% del gasto

### Sección 2 — El cambio que hace esto urgente ahora
- GitHub Copilot usage-based billing desde junio 2026
- Ya no hay "plan flat" — cada token tiene precio directo
- El multiplicador agéntico: workflows complejos cuestan ~3.5x más
- Deloitte: AI consume hasta el 50% del gasto IT en algunas firmas

### Sección 3 — Las capas de optimización (de fácil a avanzado)

#### Capa 0 — Lo que podés hacer hoy sin cambiar nada
- Activar Auto Mode en Copilot (10% de descuento, zero config)
- Activar `chat.tools.compressOutput.enabled` en VSCode
- Tool search en VSCode (MCPs diferidos, hasta 20% ahorro)

#### Capa 1 — Arquitectura de contexto
- Separar contenido estático del dinámico
- Prompt caching: 90% de descuento en tokens repetidos
- No enviar historial completo — ventana deslizante + resumen
- Compaction API para conversaciones largas

#### Capa 2 — Selección de modelo correcta
- El mapa: qué modelo para qué tarea
- Haiku para clasificación, extracción, formateo
- Sonnet para el trabajo general
- Opus solo cuando el razonamiento complejo justifica el precio
- MCPs innecesarios activos = overhead en cada request

#### Capa 3 — Routing inteligente
- Del Auto Mode (cero config) al gateway enterprise (control total)
- RouteLLM: 95% calidad con 26% de llamadas al modelo fuerte
- LiteLLM / Portkey: budget controls por equipo

#### Capa 4 — Palancas de infraestructura
- Batch API: 50% de descuento para workloads asincrónicos
- Combinado con caching: hasta 95% de ahorro
- DeepSeek via Azure: el modelo más barato del mundo con compliance europeo

### Cierre — De práctica individual a cultura de equipo
- Gobernanza no es restricción — es visibilidad
- El objetivo no es gastar menos, es gastar bien
- Call to action: qué hacer esta semana

---

## Al final del artículo

Links a los archivos de research en este repositorio para quien quiera profundizar en cada capa.

---

## Notas editoriales

- Cada dato numérico debe tener su fuente — consultar `data/verified-metrics.md`
- El artículo no debe explicar qué es un token (la audiencia ya lo sabe)
- Evitar desvíos hacia prompt engineering o calidad de respuesta sin conexión directa al costo
- El tono: un colega que descubrió algo importante y lo comparte — no un manual corporativo
- Modelo para redacción: Claude Sonnet 4 (`claude-sonnet-4-20250514`)
