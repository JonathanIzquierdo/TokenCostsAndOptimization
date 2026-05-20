# El dinero que se pierde en cada conversación con la IA

*Cómo los tokens se convirtieron en la nueva métrica de eficiencia de ingeniería — y qué hacer al respecto*

---

## Apertura

Hay un número que, cuando lo escuchás por primera vez, no parece real.

El **62% de lo que tu empresa paga en AI** no es por el trabajo que hace el modelo. Es por contexto que ya procesó antes, que le estás enviando de nuevo, en cada request, en cada turno de conversación, en cada llamada de agente. No porque sea necesario. Sino porque nadie lo configuró para que no lo hiciera.

Esto no lo estimó nadie. Lo midió LeanOps en una auditoría de 30 equipos de ingeniería corriendo IA agéntica en producción entre marzo y mayo de 2026. Más de la mitad de la factura es ruido repetido.

La intuición de la mayoría del equipo es que pagamos por lo que el modelo *produce*. Por las respuestas, el código generado, los análisis. Esa intuición es incorrecta. Pagamos por todo lo que le *decimos* antes de que produzca algo — y en la mayoría de los sistemas en producción, le decimos lo mismo una y otra vez.

Esto no es un problema de qué modelo usás. Es un problema de arquitectura. Y tiene solución.

---

## Sección 1 — El problema invisible: cómo crece el costo sin que nadie lo vea

### La conversación que parece plana pero no lo es

Imaginá una sesión de trabajo con un agente. Empieza simple: le das contexto, le pedís algo, te responde. Diez minutos después le preguntás algo relacionado. Veinte minutos después, algo más. A la hora, seguís en la misma sesión.

Lo que no ves es lo que pasa por abajo. Cada turno nuevo incluye todo el historial anterior. No solo el último mensaje — todo. El agente necesita ese historial para mantener coherencia. Pero vos pagás por cada token de ese historial en cada request.

Una conversación de 20 turnos puede consumir entre 5.000 y 10.000 tokens cuando bastarían 500 a 1.000 tokens de contexto reciente. El resto es historial que el modelo ya procesó, que ya "sabe", y que le estás pagando para que procese de nuevo.

En sesiones cortas, esto es manejable. Pero en workflows agénticos — donde un agente trabaja durante horas, llama herramientas, recibe resultados, itera — el efecto se compone exponencialmente. Una sesión que empieza con 5.000 tokens por llamada puede llegar a 200.000 tokens por llamada en el turno 50. El trabajo del modelo no cambió. El costo, sí.

### El multiplicador que nadie configura

Hay otro factor que hace todo peor: los output tokens cuestan entre 4 y 6 veces más que los input tokens.

En modelos como Claude Sonnet 4.6, el input cuesta $3 por millón de tokens. El output cuesta $15 por millón. No es una diferencia marginal — es un multiplicador de 5x.

La mayoría de los equipos setea prompts cuidadosamente. Muy pocos setean `max_tokens` explícitamente en sus workflows. Cuando no lo hacés, el modelo genera lo que considera apropiado para la tarea. En tareas simples — una clasificación, una extracción, una respuesta de sí/no — eso puede ser 10 a 50 veces más tokens de los necesarios.

Una app que no tiene `max_tokens` configurado, donde el modelo responde con 800 tokens cuando necesitaba 200, tiene una factura 4 veces más alta en outputs. Por el mismo resultado.

### El costo oculto del extended thinking

Los modelos con razonamiento extendido — Claude Opus con adaptive thinking, los modelos o1/o3 de OpenAI — tienen un comportamiento que muchos no conocen en detalle: sus tokens de *thinking* se facturan como output tokens.

Cuando Claude Opus 4.7 resuelve algo complejo, puede generar miles de tokens de razonamiento interno antes de producir la respuesta. Esos tokens se facturan a $25 por millón.

El error más común: usar `display: "omitted"` creyendo que eso reduce el costo. No lo hace. Omitir el display del thinking reduce la latencia percibida, pero los tokens se siguen generando y se siguen facturando. Un response con 500 tokens de output visible y 2.000 tokens de thinking cuesta 5 veces más que el mismo response sin thinking habilitado.

Esto no significa que el extended thinking sea malo. Para análisis arquitectural o debugging complejo, puede ser exactamente lo que necesitás. Pero aplicarlo a clasificaciones simples o completions de boilerplate es un multiplicador de costo que pasa completamente desapercibido si no sabés buscarlo.

### El efecto Copilot: el 10% que mueve el 60%

En deployments enterprise de Copilot, hay un patrón consistente: el top 10 al 15% de usuarios concentra entre el 60 y el 70% del consumo de features avanzadas.

Estos son los desarrolladores que usan chat intensivamente, que corren sesiones agénticas, que hacen code review con Copilot. Son también los más productivos del equipo — y con el nuevo modelo de billing, son los que van a agotar los créditos incluidos primero.

Para equipos con workflows agénticos intensivos, las estimaciones apuntan a costos de alrededor de £52 por developer por mes — aproximadamente 3.5 veces el flat fee anterior. No para todo el equipo: para el cohort de usuarios intensivos. Pero si ese cohort representa el 15% de un equipo de 200 personas, el impacto en la factura total es mayor que el de todo el resto del equipo junto.

---

## Sección 2 — El cambio que hace todo esto urgente ahora

Durante años, el costo de AI en herramientas de desarrollo fue predecible. Pagabas un flat fee por seat. El desarrollador más intensivo costaba lo mismo que el que apenas la tocaba. Los tokens eran un detalle de implementación, no una métrica de negocio.

Eso cambió el 1 de junio de 2026.

### Copilot ya no es una suscripción plana

GitHub Copilot migró a usage-based billing. Cada plan incluye créditos equivalentes al precio del seat — $19 en AI Credits para Business, $39 para Enterprise. Un AI Credit vale $0.01. Cuando se agotan los créditos incluidos, cada token adicional tiene un precio directo.

Los créditos son pooled a nivel enterprise. Si alguien no los usa, otros los consumen automáticamente. Lo que suena bien en teoría — eficiencia colectiva — significa en práctica que el cohort intensivo del 10-15% puede consumir el pool completo antes de que el resto del equipo tenga chance de usarlo.

GitHub está ofreciendo un período de transición con créditos adicionales durante los primeros tres meses: $30 extra para Business, $70 extra para Enterprise. Esto amortigua el impacto inicial. Pero a partir de septiembre, la economía real entra en vigor.

### Lo que sigue siendo gratuito (y lo que no)

Las completions inline y las sugerencias de código no consumen AI Credits. Permanecen ilimitadas en todos los planes. Si tu equipo usa Copilot principalmente para autocompletado, el impacto es menor.

Lo que sí consume créditos: chat en el editor, chat en GitHub.com, Copilot Workspace, Cloud Agents, y code review agéntico. Y este último tiene un detalle importante: corre sobre GitHub Actions, lo que significa que consume tanto AI Credits como GitHub Actions minutes. Doble facturación que muchos equipos no anticipan.

### El multiplicador agéntico

El cambio de billing coincide con un cambio en cómo se usa Copilot. El modelo evolucionó de asistente de autocompletado a plataforma agéntica que puede manejar sesiones largas, multi-step, iterando sobre repositorios enteros. Ese uso agéntico es el default para los usuarios más avanzados — y trae demandas de compute significativamente más altas.

Un desarrollador que pasa el día haciendo completions de código genera un consumo predecible y relativamente bajo. El mismo desarrollador usando Copilot Workspace para planificar features, refactorizar módulos, y generar tests automáticos puede generar 10 a 20 veces más tokens en el mismo período. Sin que haya hecho nada "mal" — simplemente está usando la herramienta para lo que fue diseñada.

### La escala del problema

Deloitte documentó en 2026 que en algunas firmas enterprise, AI consume hasta el 50% del gasto total de IT. IDC predice que las empresas Global 1000 subestimarán sus costos de infraestructura AI en un 30% hacia 2027.

Estos no son números de startups experimentando con APIs. Son organizaciones con procesos maduros, con presupuestos controlados, que de todos modos se encontraron con facturas de AI que no anticiparon. La razón es siempre la misma: los tokens son invisibles hasta que aparecen en el billing.

La buena noticia es que hay capas concretas de optimización disponibles hoy. Algunas toman cinco minutos. Otras requieren cambios de arquitectura. Pero todas mueven la factura.

---

## Sección 3 — Antes de las capas: en qué relación estás con el modelo

Hay una decisión que es anterior a cualquier técnica de optimización, y que determina qué técnicas tenés disponibles. No es sobre qué modelo elegís. Es sobre **cómo lo estás consumiendo**.

### Producto cerrado vs API key

Si tu equipo consume IA solamente desde **Claude.ai, ChatGPT, Copilot, Cursor** — productos terminados — está en el primer grupo: usuarios de caja cerrada. Las palancas que tenés disponibles son reales pero limitadas. Podés activar Auto Mode en Copilot (10% de descuento), podés configurar compressOutput y tool search en VSCode (hasta 20% de ahorro), podés auditar qué MCPs tenés activos. Todo eso suma. Pero no podés activar prompt caching, no podés elegir batch processing, no podés controlar el extended thinking, no podés setear `max_tokens`. Esas palancas no están expuestas.

Si tu equipo construye con **API key contra Anthropic, OpenAI, DeepSeek o el proveedor que sea**, está en el segundo grupo: constructores. Y acá es donde aparecen los números grandes. El 90% de descuento del prompt caching. El 50% del batch API. El 95% combinado. El 25x de diferencia entre Haiku y Opus. Las palancas que mueven la factura en serio viven en código de aplicación.

Es la diferencia entre usar Excel y construir una hoja de cálculo personalizada. Ambas resuelven problemas. La segunda los resuelve a tu medida.

### Si vamos a construir, ¿dónde compramos la API?

Una vez que aceptás que querés construir con API, aparece la pregunta de procurement: vendor directo o vía hyperscaler. La intuición común de los equipos enterprise es que el hyperscaler conviene por el descuento EA. Los datos cuentan otra historia.

Para Claude y para GPT, el precio nominal por token es **idéntico** entre el vendor directo y el hyperscaler. Anthropic mantiene pricing parity en Bedrock, Vertex y Foundry. OpenAI lo mantiene en Azure OpenAI. El descuento EA no es sobre el token — es sobre la factura total de Microsoft.

Lo que hace al hyperscaler más caro no es el precio del token. Es lo que se acumula alrededor. En Azure OpenAI, support plans mandatorios, data egress, networking, fine-tuned model hosting que se cobra aun sin uso. Tres fuentes independientes ([TokenMix](https://tokenmix.ai/blog/azure-openai-cost), [Inference.net](https://inference.net/content/azure-openai-pricing-explained/), [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/)) coinciden: **el TCO efectivo de Azure OpenAI corre 15–40% por encima de OpenAI directo**, con el cliente promedio en +22%. En Bedrock para Claude pasa algo parecido — el precio per-token es idéntico, pero entre cross-region endpoints (+10%), data transfer, y billing de errores HTTP 500, el TCO efectivo termina **20–35% por encima del directo**. ([TokenMix Bedrock](https://tokenmix.ai/blog/aws-bedrock-pricing))

Hacé la cuenta con cualquier escenario de descuento EA y vas a ver lo mismo: el descuento del 15–25% (o 23–28% con MACC) **compensa pero no supera** el overhead. La factura final queda entre -14% y +12% vs el vendor directo. En muchos casos, ir vía hyperscaler termina costando lo mismo o un poco más.

Y hay una trampa específica que pocas empresas conocen hasta que les llega: **los créditos Azure / Founders Hub / MACC no se aplican a Claude en Microsoft Foundry**. Claude se factura ahí como Marketplace de terceros, directo a tarjeta de crédito. ([AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/), [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352)). Una startup acumuló $13.000 USD en un mes pensando que sus credits de Foundry cubrían Claude. No los cubrieron. Si tu organización está cerrando un acuerdo con Microsoft y planea consumir Claude vía Foundry asumiendo que el MACC lo absorbe, ese es exactamente el momento de pedir aclaración escrita.

### Entonces, ¿por qué iría al hyperscaler?

Por una razón que no es el precio: **compliance**. Si tus datos son europeos y caen bajo GDPR, si manejás información de salud o financiera regulada, si tu organización tiene mandato corporativo de que todo corra en Azure o AWS — el overhead deja de ser overhead y pasa a ser prima de compliance. Estás pagando por el contrato, no por el modelo.

El caso más claro es DeepSeek. El modelo en sí es ultra-barato (~15x más barato que Sonnet input) pero la API directa rutea los datos por servidores chinos, lo que lo hace inviable para cualquier empresa europea con datos de clientes. DeepSeek vía Azure cuesta 20–35% más caro que directo, pero los datos quedan en la región Azure que elegiste, el contrato es con Microsoft, y el compliance europeo está resuelto. Aún con el markup, sigue siendo dramáticamente más barato que correr el mismo workload contra Sonnet. ([DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing))

| Escenario | Costo directo | Costo Azure | Costo Bedrock | Neto vs directo |
|-----------|---------------|--------------|----------------|------------------|
| GPT-5.4, sin compliance, sin descuento | 100 | 115–140 | n/a | **+15% a +40%** |
| GPT-5.4, con EA -20% | 100 | 92–112 | n/a | **-8% a +12%** |
| GPT-5.4, con EA+MACC -25% | 100 | 86–105 | n/a | **-14% a +5%** |
| Claude Sonnet, sin compliance | 100 | n/a (Foundry sin desc.) | 120–135 | **+20% a +35% Bedrock** |
| DeepSeek con GDPR EU | inviable | 120–135 | n/a | **prima de compliance** |

La línea para llevarse: el hyperscaler no se elige por descuento — se elige por compliance, por mandato corporativo, o porque el modelo lo necesita para ser usable en enterprise. Si ninguno de los tres aplica, el vendor directo gana en precio neto y en velocidad de features (las betas y nuevas capacidades salen primero en la API del fabricante).

Si tu organización está negociando acuerdos con vendors en este momento, este es el dato que conviene llevar a la mesa: el descuento que te ofrecen no se compara contra el precio de lista del hyperscaler. Se compara contra el precio del vendor directo más el overhead del hyperscaler. Y ese cálculo cambia completamente lo que vale el descuento.

---

## Sección 4 — Las capas de optimización

No existe una sola palanca que resuelva el problema. Existe una secuencia de capas, cada una más técnica que la anterior, cada una con mayor impacto potencial. Podés aplicar solo las primeras y ya notás diferencia. Podés aplicar todas y cambiar radicalmente la economía de tu uso de AI.

---

### Capa 0 — Lo que podés hacer hoy, en cinco minutos, sin cambiar nada de arquitectura

Estas son configuraciones. No requieren código nuevo, no requieren aprobaciones de arquitectura, no requieren sprints. Las activás y funcionan.

**Activar Auto Mode en Copilot**

En el selector de modelo de Copilot en VSCode, hay una opción "Auto" que la mayoría del equipo ignora porque suena ambigua. No lo es: con Auto activado, Copilot selecciona dinámicamente el modelo más eficiente para cada tarea — ruteando entre GPT-5.4, GPT-5.3-Codex, Claude Sonnet 4.6 y Haiku 4.5 según la complejidad del request.

El beneficio inmediato es un 10% de descuento en el multiplicador del modelo. El beneficio estructural es que tareas simples van a modelos simples, y solo las tareas complejas usan los modelos caros. Cero configuración adicional.

**Activar `chat.tools.compressOutput.enabled` en VSCode**

Agregá esto a tu `settings.json`:

```json
{
  "chat.tools.compressOutput.enabled": true
}
```

Esta feature de VSCode 1.120 post-procesa el output de comandos de terminal antes de enviarlo al modelo. Colapsa hunks sin cambios en diffs, descarta lockfile diffs completos, reduce `ls -l` a nombres de archivo, elimina barras de progreso de `npm install` y advertencias de deprecación.

El output de un `git diff` en un repositorio activo puede tener miles de líneas. Sin compresión, todo eso va al modelo. Con compresión, solo van los cambios relevantes.

**Tool search — MCPs diferidos por defecto**

VSCode 1.118 introdujo una separación del toolset del agente: aproximadamente 30 herramientas core están siempre disponibles (cubren el 88% de los casos reales), y el resto se carga bajo demanda.

Para modelos Anthropic esto está activo por defecto. Para modelos GPT en Copilot:

```json
{
  "github.copilot.chat.responsesApi.toolSearchTool.enabled": true
}
```

Impacto: hasta 20% de ahorro en tokens por request.

---

### Capa 1 — Arquitectura de contexto

**Prompt caching: el 90% de descuento que casi nadie usa**

El contenido estático de tus prompts — system prompt, tool definitions, knowledge base — se puede cachear. Las cache reads cuestan el 10% del precio base de input. Una app RAG con 50.000 tokens en el system prompt, consultada 1.000 veces al día, ahorra entre el 88% y el 95% en esa porción del costo simplemente activando caching.

Regla: todo lo que no cambia entre el 90% de los requests va en el cache. El input del usuario, el historial reciente, y los resultados de tools del turno actual nunca van en el cache.

**Historial de conversación: ventana deslizante más resumen**

No enviar la conversación completa en cada turno. Mantener los últimos N turnos relevantes más un resumen comprimido de lo anterior. Para sistemas agénticos, la Compaction API de Anthropic (beta enero 2026) automatiza esto completamente — resume el historial con comprensión semántica cuando el contexto se acerca al límite, y es elegible para Zero Data Retention.

**RAG en lugar de context stuffing**

Recuperar solo los fragmentos específicamente relevantes para la query actual. RAG puede reducir el uso de tokens de contexto en hasta un 70% comparado con enviar documentos completos.

---

### Capa 2 — Selección de modelo correcta

La diferencia de precio entre Haiku y Opus es de 25x. La mayoría de los workflows agénticos usan Sonnet u Opus para tareas que Haiku resuelve igual.

El mapa correcto:
- **Haiku:** clasificación, extracción, formateo, navegación de archivos, pattern matching
- **Sonnet:** generación de código, code review, summarización, chat, trabajo general
- **Opus:** análisis arquitectural complejo, razonamiento multi-step crítico, coordinación de agentes

Cada MCP activo innecesariamente agrega 500 tokens de schema al contexto en cada request. Con 10 MCPs activos que no usás: $1.500/mes de overhead puro en 100K requests a Sonnet.

---

### Capa 3 — Routing inteligente

En lugar de decidir manualmente qué modelo usar, un sistema decide por request.

- **Auto Mode Copilot:** cero configuración, 10% de descuento automático
- **OpenRouter Auto Router:** routing entre 300+ modelos, powered by NotDiamond, sin markup
- **RouteLLM (open source, UC Berkeley/LMSYS):** 95% de calidad de GPT-4 con 26% de llamadas. Drop-in replacement del cliente de OpenAI
- **LiteLLM/Portkey:** gateways enterprise con budget controls, observabilidad, y protección contra loops descontrolados

---

### Capa 4 — Palancas de infraestructura

**Batch API:** 50% de descuento para workloads sin necesidad de respuesta en tiempo real. Combinado con caching: hasta 95% de ahorro vs un request estándar.

**DeepSeek via Azure:** el modelo más barato del mundo (~$0.19/MTok via Azure) con compliance europeo. 15x más barato que Sonnet para tareas de alta volumetría. El acceso directo a DeepSeek no es viable por data residency. Azure agrega 20–35% de markup sobre el directo, pero ese markup es prima de compliance — y aun pagándolo, sigue siendo dramáticamente más barato que cualquier alternativa enterprise. Es el ejemplo más limpio de "el hyperscaler vale la pena cuando compra compliance, no descuento".

**Negociación del EA:** el descuento en Copilot via Enterprise Agreement es negociado, no publicado. El rango documentado: 15–25% con EA solo, 23–28% con EA + Azure MACC. Importante: como vimos en la sección anterior, este descuento típicamente compensa pero no supera el overhead Azure. La decisión real de proveedor se gana o se pierde por compliance, no por descuento. Si la organización está cerrando acuerdos con vendors en este momento, conviene llevar el dato del overhead a la mesa de negociación — porque define el verdadero break-even del descuento que te ofrecen.

---

## Cierre — De práctica individual a cultura de equipo

Todo lo que describimos hasta acá puede aplicarlo una persona sola esta semana. El Auto Mode, el compressOutput, el prompt caching — son decisiones individuales que tienen impacto inmediato.

Pero el problema de fondo no se resuelve con configuraciones individuales. Se resuelve con visibilidad colectiva.

El 10-15% de usuarios que concentra el 60-70% del gasto no está haciendo nada malo. Está usando las herramientas para lo que fueron diseñadas. El problema es que nadie ve ese consumo hasta que llega la factura. Y cuando llega la factura, la conversación se vuelve reactiva: recortes, restricciones, limitaciones.

La alternativa es construir gobernanza proactiva: observabilidad de token consumption por usuario y por equipo, alertas en 50%, 80% y 100% del crédito mensual, políticas de selección de modelo por tipo de tarea, y presupuestos por equipo con controles automáticos.

Las empresas que construyan esta infraestructura ahora van a reutilizarla para cada nuevo servicio de AI que adopten. Las que lo traten como un cambio de billing puntual van a reconstruirlo desde cero cada vez.

El objetivo no es gastar menos. Es gastar bien.

---

## Qué hacer esta semana

**Hoy (5 minutos cada uno):**
- Cambiar el selector de modelo de Copilot a "Auto" en VSCode
- Agregar `chat.tools.compressOutput.enabled: true` a `settings.json`
- Verificar que estás en VSCode 1.118 o superior

**Esta semana:**
- Revisar qué MCPs tenés activos — desactivar los que no usás
- Identificar qué tareas van a Sonnet/Opus que podrían ir a Haiku
- Activar el preview de billing en GitHub Billing Overview

**Este mes:**
- Implementar prompt caching en sistemas con system prompts largos
- Evaluar Batch API para pipelines que no necesitan respuesta en tiempo real
- Mapear los top consumers del equipo y entender qué workflows generan ese consumo

**A mediano plazo:**
- Evaluar RouteLLM o un AI gateway para sistemas agénticos en producción
- Negociar el EA de Microsoft en el próximo renewal incluyendo Copilot en el contexto del MACC — con el dato del overhead Azure encima de la mesa
- Explorar DeepSeek via Azure para workloads de alta volumetría con datos EU

---

*Para profundizar en cada tema con detalles técnicos, ejemplos de código, y guías de implementación, ver `PROD/technical-docs-draft-v1.md`*
