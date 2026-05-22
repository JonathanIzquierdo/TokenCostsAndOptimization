# Si no cambiás nada, en junio pagás 3x

Un artículo opinado sobre el costo real de la IA en el día a día del equipo. Como tal, tomalo con una pizca de sal. 🧂

---

## Executive Summary

El 1 de junio cambia cómo se factura GitHub Copilot. No es un ajuste de precio, es un cambio de modelo: pasamos de plano a por consumo. La factura enterprise sube aproximadamente 3x si no movés nada. La buena noticia es que casi todo lo que la hace subir es contexto que mandamos de vuelta sin pensarlo, llamadas que podrían rutearse a modelos baratos, y descuentos del 90% que están ahí esperando que alguien los active. Este artículo es para que nadie del equipo se entere de lo segundo después de ver lo primero en el dashboard.

Gastar bien, no gastar menos.

---

## Call to Action

- Mirá tu propia sesión de Copilot de los últimos 3 días. Si tenés conversaciones de más de 20 turnos, ya estás pagando por contexto que se repite.
- Activá Copilot Auto Mode. 10% de descuento por un click. ([fuente: GitHub Changelog](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/))
- Activá `chat.tools.compressOutput.enabled` en VSCode. 10 segundos, ahorra ~15-20%. ([fuente: VS Magazine](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx))
- Si tenés un workflow batch (reportes, clasificación, embeddings), revisalo. Batch API tiene 50% de descuento sin diferencia de calidad. ([fuente: Anthropic](https://www.finout.io/blog/anthropic-api-pricing))
- Compartí este artículo con tu tech lead. La factura es de toda Visma, no de tu seat.
- Si después de leer querés profundizar, la documentación técnica del proyecto tiene el detalle por capa.

---

## La factura ya está rota, solo que todavía no la viste

Hay un dato que me sigue dando vueltas. En 30 equipos relevados entre marzo y mayo de 2026, el **62% de la factura de IA no es por trabajo del modelo**. Es por mandar de vuelta el mismo contexto, una y otra vez. ([fuente: LeanOps](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/))

Pensá un segundo qué significa eso. De cada 100 dólares que tu organización pone en IA, 62 son literalmente el mismo system prompt, la misma documentación de proyecto, el mismo historial, viajando ida y vuelta hasta el modelo. Es como si un courier te cobrara cada vez que mueve una caja, y vos lo hicieras pasar 50 veces por la misma esquina porque te olvidaste de pedirle que la dejara en el destino.

Lo concreto: una sesión típica empieza en 5.000 tokens. Para el turno 50 está en **200.000**. ([fuente: Redblink](https://redblink.com/ai-token-cost-optimization/)) Nadie lo ve. No hay un cartel que diga "estás a 40x del costo de cuando empezaste". La factura llega a fin de mes y se atribuye a "más uso de IA".

Hasta ahora eso no nos pegaba directo porque pagábamos plano. Copilot Business: $19/seat/mes. Copilot Enterprise: $39/seat/mes. Fijo, predecible, una línea en el presupuesto. ([fuente: GitHub Docs](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises))

El 1 de junio eso se termina.

---

## Qué cambia exactamente

Cada seat sigue costando lo mismo, pero ahora incluye "AI Credits" equivalentes al precio del seat. Por encima de eso, **se paga por consumo**. Un AI Credit son $0.01. Los créditos del seat son pooled a nivel enterprise: si vos no los usás, los usa otra persona del equipo. ([fuente: GitHub Docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing))

Suena razonable hasta que mirás los números reales.

Los workflows agénticos intensivos —que es donde vamos como industria— están midiéndose en torno a **£52 por dev por mes, ~3.5x el flat fee anterior**. ([fuente: Synapx](https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/))

Y no se distribuye parejo. En las organizaciones que ya están bajo usage-based, el **top 10-15% de usuarios concentra el 60-70% del gasto**. ([fuente: Synapx](https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/)) Si vos sos uno de los devs que más usa AI —y si estás leyendo esto, probablemente lo seas—, sos el que mueve la aguja para todos los demás.

Esto no es una predicción. IDC estima que las **Global 1000 están subestimando sus costos de AI en un 30% hacia 2027**. ([fuente: IDC via InfoWorld](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing-signaling-new-cost-model-for-enterprise-ai-tools.html)) En algunas firmas, la IA ya es **hasta el 50% del gasto total de IT**. ([fuente: Deloitte 2026 via Redblink](https://redblink.com/ai-token-cost-optimization/))

Si tu reacción es "bueno, pero a mí me sigue costando $19", releé el párrafo anterior. La factura es de Visma, no de tu seat.

---

## Las objeciones (que ya escuché tres veces esta semana)

Liderar la conversación sobre esto en Visma me dejó una colección de respuestas. Vale la pena visitarlas.

### "Yo uso Copilot, no veo costo. ¿Qué cambia?"

Cambia que hoy estás dentro del flat fee. A partir del 1 de junio, cada interacción suma o resta créditos del pool. La diferencia entre un dev que activó Auto Mode y caching, y otro que no, **no es 5% mensual, es 3x al año**. No estás eligiendo entre "ahorrar" o "no ahorrar". Estás eligiendo entre "pagar lo justo" y "subsidiar a quien todavía no aprendió a usar la herramienta".

### "Prompt caching suena a optimización prematura"

Knuth dijo que la optimización prematura es la raíz de todos los males. Tenía razón sobre CPU en los 70s. No tenía razón sobre tokens en 2026, porque acá la "optimización" no es reescribir un loop —es activar un descuento del **90%** que Anthropic puso ahí para que lo uses. ([fuente: TokenOptimize.dev](https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies))

El break-even del cache es **1 solo hit**. ([fuente: MetaCTO](https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration)) Cualquier app con system prompt estable o RAG está dejando dinero en la mesa si no lo activa. Una app RAG típica (50K tokens de contexto, 1000 queries diarias) ahorra **entre 88% y 95%** con caching. ([fuente: Finout](https://www.finout.io/blog/anthropic-api-pricing))

VSCode ya implementó esto silenciosamente: el **prompt caching reuse rate dentro del editor es del 93%**. ([fuente: VS Magazine](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)) No es teoría, es lo que ya está pasando en tu IDE sin que lo veas.

### "Mi sesión no es tan larga"

Eso es lo que pensaba yo también. Después miré.

Una conversación normal de 20 turnos arrastra **entre 5.000 y 10.000 tokens innecesarios** que se podrían haber dejado afuera. ([fuente: Redis 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)) Eso es por conversación. Multiplicalo por la cantidad de conversaciones por día. Multiplicalo por el equipo.

Y acá hay un detalle técnico que se subestima: **el output cuesta 4-6x más que el input**. ([fuente: Redis 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)) No es un detalle menor. Cuando le pedís a un modelo que te "resuma todo lo que hablamos hasta acá", el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

### "Pero el extended thinking lo tengo configurado en `display: omitted`. No se ve, no cuenta"

Cuenta. Cuenta como **output token**. ([fuente: CheckThat.ai](https://checkthat.ai/brands/anthropic/pricing)) Que no se muestre no significa que no se facture. La opción `omitted` te ahorra latencia perceptual, no dinero.

Una llamada con 2.000 tokens de thinking más 500 de output cuesta **5x más** que una llamada sin thinking. ([fuente: PECollective](https://pecollective.com/tools/anthropic-api-pricing/)) Si activaste extended thinking porque "siempre da mejores respuestas" y nunca lo apagaste, esto te toca. Pensar es caro. Decidir cuándo vale la pena es trabajo del que prompt-ea, no del modelo.

### "Tenemos Microsoft Enterprise Agreement, ya tenemos descuento"

Lo tenemos. Es del **15-25%**. ([fuente: Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing)) Con MACC sube a **23-28%**. ([misma fuente](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing)) Pero ese descuento no se aplica sobre la API directa, se aplica sobre Azure. Y Azure sobre OpenAI directo tiene un **overhead de 15-40% en TCO**. ([fuente: TokenMix](https://tokenmix.ai/blog/azure-openai-cost))

Hagamos la cuenta. Descuento del 25% sobre un servicio que tiene 30% de overhead sobre la alternativa directa. El "descuento" desaparece. No es que el EA no sirva —sirve, pero no como ahorro automático, sirve como mecanismo de compliance, data residency y consolidación de factura.

DeepSeek vía Azure es el caso más claro: hasta **+35% de markup sobre el precio directo**. ([fuente: DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing)) DeepSeek directo es **~15x más barato que Claude Sonnet en input**. ([fuente: TechJack](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/)) Pero los servidores están en China. ([fuente: TechJack](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/)) Para Visma —empresa europea, GDPR— no hay debate posible sobre dónde correrlo. El "descuento" de Azure ahí compra compliance, no ahorro.

### "Esto se va a resolver solo con modelos más baratos"

Probablemente sí, en parte. Pero ya hay una técnica que da el **95% de la calidad de GPT-4 con el 26% de las llamadas al modelo fuerte**. Se llama RouteLLM. ([fuente: LMSYS ICLR 2025](https://www.lmsys.org/blog/2024-07-01-routellm/)) Con data augmentation, baja al **14% de llamadas al modelo fuerte y 75% de ahorro**. ([misma fuente](https://www.lmsys.org/blog/2024-07-01-routellm/))

El **37% de las enterprises ya corre 5+ modelos en producción**. ([fuente: Swfte AI](https://www.swfte.com/blog/intelligent-llm-routing-multi-model-ai)) IDC proyecta **70% para 2028**. ([fuente: IDC FutureScape 2026](https://blogs.idc.com/2025/11/17/the-future-of-ai-is-model-routing/)) El ahorro promedio por model routing inteligente es **20-80% del OpEx**. ([fuente: IDC FutureScape 2026](https://www.einpresswire.com/article/903464074/2026-agentic-ai-era-why-multi-model-routing-has-become-a-must-have-not-a-nice-to-have))

Esto ya está. No es futuro. La pregunta no es "¿llegarán modelos más baratos?". Es "¿estamos usando los baratos que ya existen para las tareas que no necesitan los caros?".

---

## El paralelismo que me ayuda a pensarlo

En los 90s, cuando AWS recién aparecía, hubo un patrón que se repitió en muchas empresas: alguien levantaba una instancia EC2 para probar algo, se olvidaba de bajarla, y a fin de mes alguien preguntaba "¿qué es esto de $4.000 en una región que ni usamos?". Las primeras facturas de cloud eran un shock no porque el cloud fuera caro, sino porque la unidad de costo había cambiado y nadie había actualizado los hábitos.

Compute pasó de "compré un servidor, lo amortizo en 3 años" a "pago por hora-CPU mientras esté prendida". El mismo equipo, con los mismos workloads, podía gastar 5x más o 5x menos según si alguien apagaba la instancia el viernes.

Tokens están en esa fase ahora. Pagábamos planos por seat, ahora pagamos por uso. La factura va a tener varianzas grandes según hábito, no según workload. Y como en el caso de AWS, la diferencia entre los que ven el cambio venir y los que lo ven después en el dashboard no es 10%, es múltiplos.

La parte buena: ya sabemos cómo termina esta película. Las empresas que sobrevivieron a la transición de cloud no fueron las que volvieron a on-prem. Fueron las que aprendieron a tagear instancias, a apagar lo que no usaban, a elegir el tier correcto para cada workload. **Gastar bien, no gastar menos.**

---

## Qué se puede hacer ya, sin proyecto, sin permisos, sin reunión

Esto es lo que cualquier dev puede hacer esta semana sin escalar nada.

**Auto Mode en Copilot.** Activación: un click. Descuento: **10% en el multiplicador**. ([fuente: GitHub Changelog](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/)) No hay decisión técnica acá, es plata gratis.

**`chat.tools.compressOutput.enabled` en VSCode.** Setting nuevo desde 1.120. Post-procesa el output del terminal antes de mandarlo al modelo. Es por eso que VSCode pudo "anticiparse" al cambio de billing —ya empezó a comprimir antes de que la factura llegue a los usuarios. ([fuente: VS Magazine](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx))

**Tool search en VSCode.** Ahorro de hasta **20%** solo por activarlo. ([misma fuente](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx))

**Prompt caching cuando uses la API directo.** Si tu app tiene system prompt estable, cache reads cuestan **10% del precio normal**. ([fuente: TokenOptimize.dev](https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies)) Cache write TTL 5 minutos cuesta **1.25x** y break-even es **un hit**. ([fuente: MetaCTO](https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration))

**Batch API para todo lo que no necesita respuesta inmediata.** Reportes, clasificaciones nocturnas, embeddings, análisis offline. **50% de descuento sin diferencia de calidad**. ([fuente: Anthropic](https://www.finout.io/blog/anthropic-api-pricing)) Combinado con caching, **hasta 95% de ahorro contra el endpoint estándar**. ([fuente: PECollective](https://pecollective.com/tools/anthropic-api-pricing/))

**Apagar extended thinking en tareas que no lo necesitan.** "Always on" sale **5x** más. ([fuente: PECollective](https://pecollective.com/tools/anthropic-api-pricing/)) Reservalo para problemas reales, no para responder qué día es hoy.

Ninguna de estas cosas requiere refactor, ni squad nuevo, ni budget aparte. Son configuraciones, decisiones de prompt o elección de endpoint.

---

## Lo que no te puede arreglar Anthropic, Microsoft ni GitHub

Estas tres cosas dependen del equipo. Las pongo en orden de impacto.

**1. Decidir qué tarea va a qué modelo.** Si todo va a Opus, todo cuesta como Opus. La factura no la fija el proveedor, la fija el routing. Una clasificación de tickets a Haiku, un boilerplate a DeepSeek directo (si no hay tema de datos), una decisión técnica compleja a Sonnet, una revisión arquitectural a Opus. El ahorro promedio de hacer esto bien es del **20-80% del OpEx**. ([fuente: IDC FutureScape 2026](https://www.einpresswire.com/article/903464074/2026-agentic-ai-era-why-multi-model-routing-has-become-a-must-have-not-a-nice-to-have))

**2. Cortar contexto.** El **62% de basura en la factura** ([fuente: LeanOps](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/)) no se va con un setting. Se va decidiendo qué información mandar y qué no, qué historia comprimir y qué dejar afuera de la próxima llamada, cuándo abrir una sesión nueva en vez de seguir la del lunes. Context engineering reduce costos **entre 60% y 80%**. ([fuente: Maxim AI](https://www.getmaxim.ai/articles/context-engineering-for-ai-agents-production-optimization-strategies/)) Esa es una habilidad, no un click.

**3. Mirar la factura.** Esto suena trivial y es el más importante. Mirá la propia, mirá la del equipo, mirá la trayectoria mensual. El **30% de subestimación que IDC proyecta para las Global 1000** ([fuente](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing-signaling-new-cost-model-for-enterprise-ai-tools.html)) es exactamente lo que pasa cuando nadie mira. Si no hay alguien con la pregunta "¿qué la está moviendo este mes?", la respuesta siempre va a ser "más uso de IA", y eso no es accionable.

---

## Qué hacer esta semana

Tres cosas. En orden.

1. **Lunes.** Activá Auto Mode en Copilot y `chat.tools.compressOutput.enabled` en VSCode. 5 minutos. Es el mínimo común denominador del equipo.

2. **Miércoles.** Mirá una de tus sesiones largas. Cualquiera. Mirá cuánto contexto está arrastrando en el turno 30 que ya no usás. Si es la primera vez que mirás eso, te va a sorprender.

3. **Viernes.** Si tu equipo tiene alguna app que llame a una API de modelo directo (no Copilot, sino API propia para alguna feature), revisá si tiene prompt caching activado. Si no lo tiene, ese es el ticket más rentable que vas a abrir este mes.

Y después, si querés profundizar: la documentación técnica del proyecto en el repo tiene el detalle por capa, con ejemplos de código, multiplicadores, casos de no-aplicación. Pero antes de eso, los tres pasos de arriba. **Gastar bien, no gastar menos.**

La factura ya está cambiando. La diferencia entre verla cambiar en junio y entender por qué cambia es leer este artículo dos veces y hacer una llamada de 15 minutos con tu tech lead. La parte de Visma que vea el cambio venir va a salir bien parada. La parte que no, va a explicar en julio por qué la línea de AI tools subió 3x.

Mejor estar del lado que entendió.
