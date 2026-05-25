# Tokens, factura, y una decisión de presupuesto (versión short-align)

## El stake

**El 62% de la factura de IA es contexto repetido**, el mismo system prompt y el mismo historial viajando ida y vuelta hasta el modelo. El 1 de junio GitHub Copilot, uno de los vendors más adoptados en Visma, pasa de flat fee a usage-based billing, y eso convierte ese 62% en algo facturable que antes no se veía. **Para los patrones de uso más intensivo (agentic, sesiones largas), el costo puede llegar hasta ~3.5x del flat fee que pagábamos hasta ahora**, mientras que para uso normal de autocomplete probablemente quede dentro de los AI Credits del seat.

Este no es un artículo sobre apretarle el cinturón a la IA. **Es sobre que cada euro que se pone trabaje, no caliente aire. Spend well, not spend less.**

---

## Las 3 acciones que mueven la aguja este trimestre

Una línea de qué, una de por qué, una de cómo empezar. El desarrollo completo está en (Link al documento técnico).

**1. Activar prompt caching.** Anthropic cobra el contenido cacheado al 10% del precio normal, descuento de 90% en cada read. Es la optimización individual de mayor impacto: una app con system prompt estable y volumen alto puede ahorrar 88 a 95% sobre el precio sin cache. *Cómo empezar:* si usás Claude Code o Copilot en VSCode, ya lo hacen por vos, no toques nada (cache reuse rate documentado del 93% en VSCode). Si tu equipo tiene una feature de producto que llama a la API directa, activá `cache_control` automatic en el system prompt esta semana.

**2. Asignar el modelo correcto por tarea.** El flagship (Claude Opus, GPT-5.5) cuesta hasta 100x más que los modelos chicos (Haiku, GPT-5 Nano). Para clasificación, extracción, navegación de archivos o Q&A simple, Haiku rinde lo mismo a 1/5 del precio. *Cómo empezar:* en Copilot activá **Auto Mode** (10% de descuento automático, el modelo se elige solo). En features con API directa, mantené un diccionario `MODEL_ASSIGNMENT` que rutee por tipo de tarea.

**3. Configurar spending limits ANTES del 1 de junio.** Ningún proveedor serio te explota la factura por default, pero el tope por default puede no coincidir con tu presupuesto real. *Cómo empezar:* entrá al admin console de cada herramienta que use tu equipo (Copilot, Cursor, Anthropic, OpenAI, Azure, Bedrock), revisá cuánto está configurado hoy, y poné alertas al 50%, 80% y 100% del presupuesto del mes. Si nadie está designado para mirar la trayectoria, asignalo.

---

## Qué significa esto para vos, según rol

### Si sos developer

**Por qué te importa:** la factura ya no es ajena, parte de ella se atribuye a sesiones individuales. Tus sesiones largas, los agentes que dejaste corriendo el fin de semana, el thinking activado "por las dudas", todo eso pasa a tener un costo medible. Y al mismo tiempo, **medir tu propio consumo te lleva diez minutos** y no requiere permisos.

**Esta semana:**
- Lunes: activá Auto Mode + compressOutput + tool search en VSCode. 15 a 25% de ahorro sin tocar workflow. Detalle exacto en sección 4 del técnico (Link al documento técnico).
- Martes a la tarde: armate tu dashboard personal con el script Python local que está en sección 9.6 del técnico (Link al documento técnico). Lee el archivo JSONL que Copilot exporta, genera un HTML estático con gráficos, todo en tu máquina, sin subir métricas a ningún lado.
- Resto de la semana: con el dashboard ya andando, mirá una sesión larga tuya y observá cuánto contexto repetido estás re-enviando, qué modelos usaste, cuántos tokens fueron output. La intuición se construye así.

### Si sos tech lead / engineering manager

**Por qué te importa:** los heavy users (10 a 15% del equipo) típicamente generan el 60 a 70% del spend. Sin visibilidad por equipo, esa concentración es invisible y se atribuye a "más uso de IA en general". Cuando el budget pase a usage-based, vos sos el que va a tener que explicar qué movió la factura este mes. Sin observabilidad, no hay respuesta concreta.

**El patrón a establecer en tu equipo:**
- **Observabilidad básica:** que cada dev tenga su dashboard personal (te toma 10 min en tu standup explicarlo, después cada uno lo arma solo). La gobernanza colectiva se construye desde abajo, no pidiendo reportes a las BUs.
- **Una decisión de arquitectura por feature LLM:** dónde vive cada caso de uso en el espectro de tres niveles (productos cerrados / herramientas con configuración / API directa). Las features con uso programático masivo deberían migrar a API directa con caching y batch; el resto puede quedarse donde está.
- **Un responsable nombrado** de mirar la trayectoria mensual del consumo de tu equipo. Sin nombre y apellido, no se mira.

**Esta semana:** identificá si tu equipo tiene alguna feature de producto que llame a un modelo sin caching, sin OTel (OpenTelemetry, el estándar de observabilidad), o sin model assignment por tarea. Si la respuesta es sí a cualquiera, esa es la primera conversación que conviene tener.

### Si tomás decisiones de presupuesto en una BU

**Por qué te importa:** la línea "AI tools" de tu BU se vuelve variable a partir del 1 de junio, en una parte significativa. La pregunta de Finance en julio no va a ser "¿gastamos más?", va a ser "¿por qué subió?". La respuesta correcta no es "porque usamos más IA", es: "porque tales features pasaron a producción y tales no tienen caching activado".

**Unit economics que conviene tener mapeados antes de fin de mes:**
- Costo actual por seat de Copilot, multiplicado por tu población activa.
- Proyección de costo para los heavy users (~10 a 15% del equipo) bajo el nuevo billing. Si tu BU tiene heavy users con uso agéntico, la línea puede subir hasta ~3.5x para ese segmento. Vale la pena conocerlo antes que reaccionar.
- Línea separada para las features de producto que llaman a APIs directas. Esa línea es la que escala con uso de cliente, no con headcount, y se gobierna con prácticas de ingeniería (caching, batch, routing), no con licencias.

**La decisión de presupuesto:** no es subir o bajar el line item. Es separar el budget en dos categorías (uso de developer = scaling con headcount; uso programático = scaling con producto) y asignar a cada una una práctica de gobierno distinta. Si las dos están en el mismo bucket, cualquier conversación de costo se contamina.

---

## Los números que importan

Datos verificados que sostienen las decisiones de arriba. Las fuentes están en el documento técnico (Link al documento técnico), sección de referencia.

- **62% de la factura típica de IA = contexto re-enviado**, no información nueva. Medido en 30 equipos relevados a comienzos de 2026.
- **Output cuesta 4 a 6x más que input.** Limitar `max_tokens` por tipo de tarea tiene impacto desproporcionado.
- **Cache reads = 10% del precio normal**, descuento de 90%. Break-even al primer hit.
- **VSCode prompt caching reuse rate = 93%** dentro del editor.
- **Auto Mode en Copilot = 10% de descuento automático.** Settings sin sacrificar calidad para tareas estándar.
- **EA + MACC discount = 23 a 28%**, pero Azure tiene overhead de 15 a 40% en TCO. **El descuento no es ahorro automático**: en escenarios típicos compensa el overhead pero no lo supera. El EA se justifica por compliance y procurement, no por precio.
- **Heavy users (10 a 15% del equipo) = 60 a 70% del spend.** La distribución es siempre asimétrica.
- **Uso agéntico bajo usage-based billing = hasta ~3.5x el flat fee** que pagábamos antes para ese patrón específico. Para uso normal de autocomplete, mucho menos.

---

## El momento

Tuvimos un par de años muy generosos. Aparecieron herramientas nuevas todos los meses, los modelos eran cada vez mejores, los precios bajaban, y los planos flat nos dejaban experimentar sin pensar dos veces. Si activaste extended thinking "por las dudas", si abriste 40 sesiones esta semana, si dejaste un agente corriendo todo el fin de semana, el costo de aprender de eso fue bajo.

Ese período se está terminando. No de golpe, no para todos, no el 1 de junio. Pero se está terminando. Y la ventana para aprender barato, para experimentar, medir, equivocarse, ajustar, sin que duela, **es exactamente ahora**.

Los que aprovechen esta ventana van a salir del otro lado entendiendo cómo se gobierna el consumo de IA: qué setting mueve qué número, qué patrón de equipo escala, qué conversación llevar a Finance. Van a ser los que en seis meses, cuando esto sea política de empresa, ya tengan la respuesta. **Los que la dejen pasar van a tener que aprender lo mismo, pero con la factura corriendo y menos margen para experimentar.**

---

## Si querés ir más a fondo

El detalle técnico de todo lo de arriba, con snippets, settings exactos, el script Python del dashboard personal completo y la matriz de decisión de procurement, está en el documento técnico del proyecto (Link al documento técnico). La versión extendida del artículo, con las objeciones más frecuentes y el paralelismo con la transición a cloud, está en (Link al artículo extendido).

Si tu equipo o BU identifica un caso donde optimización de consumo de tokens puede mover la factura de forma medible, **el Accelerator interno (en fase piloto) puede acompañar la implementación**. Es un programa que ayuda a equipos de ingeniería a modernizar sus prácticas y volverse AI-native, y en los próximos meses se está incorporando un módulo específico sobre optimización de consumo de tokens. También se están preparando **teaching modules self-service** para que los equipos puedan distribuir el conocimiento internamente sin depender de sesiones one-to-one.

El programa está en piloto, **sin sign-up oficial todavía**: la capacidad se asigna por impacto y por encaje con el caso, no por demanda. Si tu equipo tiene un caso candidato, contactá al lead del Accelerator para sumarse a la waitlist del piloto.

---

**Spend well, not spend less.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: developers que entienden lo que consumen, tech leads que tienen un patrón para su equipo, BUs que separan el budget de seats del de features programáticas. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La traen las personas que se animan a entender lo que están consumiendo.
