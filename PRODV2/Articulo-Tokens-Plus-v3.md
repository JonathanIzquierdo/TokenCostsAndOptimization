# Tokens, factura, y una conversación que se nos venía

El 1 de junio GitHub Copilot cambia su modelo de cobro. En Visma es uno de los vendors de IA más adoptados, así que para muchas BUs este es el primer mes en el que la pregunta "¿cuánto consumimos en tokens?" deja de ser teórica. Un artículo para entender el cambio, el impacto, las estrategias para optimizar consumo, y cómo monitorearlo en serio —empezando por vos mismo.

---

## Por qué este artículo, y por qué ahora

El 1 de junio GitHub Copilot deja de cobrarse plano y pasa a usage-based billing. Cada seat sigue costando lo mismo, pero ahora trae AI Credits incluidos: mientras estás dentro de esos créditos no se cobra extra; cuando se acaban, se paga por consumo. Es un cambio que afecta directo a Visma porque Copilot es uno de los vendors de IA más adoptados internamente: una porción importante de nuestro consumo de IA hoy pasa por ahí.

Ese es el disparador. Pero conviene decirlo desde el principio: **el resto del stack ya funcionaba así**. Cursor cobra por planes con cuota y overage desde hace meses. Claude (Desktop, Code, Cowork) lo mismo. Las APIs directas de Anthropic, OpenAI, Azure OpenAI, Bedrock —siempre fueron pay-per-token desde el día uno, nunca tuvieron flat fee. Lo único que cambia el 1 de junio es Copilot. Lo demás ya estaba.

Y ahí está la cosa incómoda. La pregunta "¿cuánto nos cuesta cada token que consumimos?" debería haber estado dando vueltas en nuestras cabezas desde hace bastante. Si no estaba, fue porque el flat fee de Copilot nos tapaba la vista en la parte del stack donde más consumimos, y porque en el resto —Claude, Cursor, APIs— el consumo era todavía chico o estaba sumido dentro de presupuestos más grandes que nadie auditaba.

Ese período se termina. Y se termina de la peor manera posible: con el cambio del vendor más usado, justo cuando nadie en Visma está particularmente entrenado en pensar consumo de tokens como una capacidad del usuario. Porque eso es lo que es. **Usar IA bien no es solo prompt engineering, es saber consumir menos tokens para obtener el mismo resultado.** Y esa capacidad, igual que el FinOps de cloud hace 10 años, se construye con hábito, no con una herramienta mágica.

Visto así, **este evento es la excusa correcta para una conversación que ya nos debíamos.** El 1 de junio no inventa el problema, lo hace visible. Y mientras el problema fue invisible, nadie tenía razones para construir la capacidad. Ahora sí.

Este artículo es para empezar a construir ese hábito en Visma. Va a hablar del impacto concreto que tiene el cambio de Copilot. Va a traer tips y estrategias prácticas para reducir consumo y costo, aplicables tanto si usás Copilot como si usás Cursor o Claude. Va a poner sobre la mesa una distinción conceptual que cambia cómo se piensa todo lo demás: **en qué punto del espectro entre caja negra y API estás parado.** Y, sobre todo, va a mostrar **cómo cada uno se puede medir a sí mismo** — porque la gobernanza individual es la base de toda gobernanza colectiva.

Hay una idea que va a aparecer varias veces: gastar bien, no gastar menos. No estamos buscando achicar la IA. Estamos buscando que cada euro que cada empresa de Visma pone en IA esté trabajando, no calentando aire.

---

## Lo que está pasando con el dinero (y nadie lo está mirando)

Antes de hablar de qué hacer, vale mostrar dónde está el problema.

En 30 equipos relevados entre marzo y mayo de 2026, **el 62% de la factura de IA no es por trabajo del modelo**. Es por mandar el mismo contexto, una y otra vez. De cada 100 euros que una empresa pone en IA, 62 son literalmente el mismo system prompt, la misma documentación de proyecto, el mismo historial, viajando ida y vuelta hasta el modelo. Es como si un courier te cobrara cada vez que mueve una caja y vos lo hicieras pasar 50 veces por la misma esquina porque te olvidaste de pedirle que la dejara en el destino.

Lo concreto: una sesión típica empieza en 5.000 tokens. Para el turno 50 está en 200.000. Nadie lo ve. No hay un cartel que diga "estás a 40x del costo de cuando empezaste". La factura llega a fin de mes y se atribuye a "más uso de IA".

Hasta ahora esto no nos pegaba directo porque en Copilot pagábamos plano —Copilot Business $19/seat, Enterprise $39/seat— y en el resto de las herramientas el consumo todavía no era el suficiente como para que alguien levantara la mano. Eso cambia. Los workflows agénticos intensivos, que es donde la industria va, se están midiendo en torno a **3.5x el flat fee anterior**. Y no se distribuye parejo: el **top 10-15% de usuarios concentra el 60-70% del gasto**. Si tu BU tiene devs que usan IA todo el día —y son los que la usan bien y producen más—, son ellos los que mueven la aguja para todos los demás.

Esto importa de forma distinta según cómo esté cada empresa. Visma no es una sola empresa con una factura unificada de IA: es un grupo donde cada empresa tiene sus finanzas, su selección de tecnología y sus propios presupuestos. Lo que se comparte son comunidad, reglas, y —en algunos casos— acuerdos corporativos opcionales a los que las empresas pueden adherirse.

Esos acuerdos son una palanca real. Hay acuerdos vigentes con vendors importantes de IA que permiten a las empresas adheridas acceder a pricing diferenciado: descuentos por volumen agregado que individualmente no alcanzarían, waivers de seat fees cuando el grupo cruza ciertos umbrales de compromiso anual, escalones de descuento sobre el consumo que aplican a todas las empresas que se suman. El consumo se trackea y se reinvoicea por empresa, pero **el volumen consolidado del grupo es lo que dispara los descuentos que cada empresa individual no podría conseguir sola**.

Por eso este artículo importa para cada empresa por separado **y** para Visma como conjunto. Para cada empresa, porque su factura individual se va a mover el 1 de junio. Para Visma como conjunto, porque la capacidad de gobernar tokens —y el aprendizaje sobre cómo hacerlo— son algo que se puede compartir entre las empresas que decidan organizarse así.

---

## El espectro: caja negra opaca, caja gris configurable, API Key

Antes de seguir con tips concretos, una pausa conceptual que vale para gente de todos los niveles técnicos. Es la distinción más importante del artículo.

Hay una idea muy difundida que dice que el mundo de la IA se divide en dos: las herramientas (caja negra) y las APIs (transparentes). Esa división era cierta hace año y medio. Hoy no. Lo que tenemos es un espectro de tres niveles, y entender dónde está parada cada cosa cambia bastante el análisis de costo y de control.

| Nivel | Qué es | Qué ves y controlás | Ejemplos |
|---|---|---|---|
| **Caja negra opaca** | Producto consumer sin instrumentación pensada para el usuario | Casi nada: el chat, la respuesta, alguna cuota mensual del plan | ChatGPT versión consumer, Gemini versión consumer, Claude.ai en planes Free/Pro |
| **Caja gris configurable** | Herramienta enterprise o IDE con visibilidad, model picker, telemetría OTel y settings ricos | Modelo elegible (model picker), uso (usage panel), métricas vía OTel, controles de qué se exporta y a qué backend | Copilot en VSCode/Visual Studio, Claude Code, Claude Cowork en Team/Enterprise, Cursor (parcial — OTel no nativo) |
| **API Key** | Vos escribís el código que llama al modelo | Todo: prompt exacto, contexto, modelo, parámetros, métricas por llamada, costos por token, cache control | Anthropic API, OpenAI API, Azure OpenAI, AWS Bedrock, Google Vertex |

La diferencia entre los tres niveles no es solo cuánto ves. Es **qué optimizaciones tenés disponibles**:

- **Caja negra opaca:** podés cambiar de plan, podés activar lo poco que el vendor expone (típicamente nada). Punto.
- **Caja gris configurable:** podés elegir modelo, podés activar OpenTelemetry y armarte tu dashboard, podés cambiar settings que mueven el costo (Auto Mode en Copilot, compressOutput en VSCode, tamaño de CLAUDE.md, etc). El caching lo hace la herramienta por vos.
- **API Key:** todo lo de arriba más controlar exactamente cuándo cachear, cuándo usar Batch API (50% off), cuándo usar extended thinking, cuándo no.

Una BU puede estar en los tres niveles a la vez: caja gris para que los devs usen Copilot, API para una feature de producto que llama LLM por debajo, y quizás algún equipo informal usando ChatGPT consumer. Eso no es problema —es la realidad. Lo que importa es saber **dónde está cada caso** y no asumir que todo lo que no es API es caja negra opaca.

La regla simple para diseñar nuevas implementaciones: **caja gris para uso humano** (un dev escribiendo código, alguien explorando ideas), **API para uso programático** (una feature de producto que llama un modelo sin humano del otro lado). Las cajas negras opacas son un problema sobre todo cuando se usan para casos que ameritan otra cosa: si tu equipo está usando ChatGPT consumer para tareas que requieren observabilidad, gobernanza o repetibilidad, ahí hay algo para revisar.

---

## Las objeciones que ya escuché tres veces esta semana

Liderar esta conversación dentro de Visma me dejó una colección de respuestas. Vale la pena visitarlas, porque las preguntas son las mismas en todas las BUs.

### "Yo uso Cursor o Claude Code, esto no me toca"

Te toca igual, aunque por una razón distinta. El 1 de junio no cambia ni Cursor ni Claude —esos ya estaban en planes con cuota y overage desde antes. Lo que cambia es que **antes nadie miraba el consumo en ninguna parte del stack**, porque Copilot era flat y el resto era todavía chico. Ahora que la línea más gorda del presupuesto (Copilot) pasa a usage-based, las BUs van a empezar a mirar todo. Y cuando empiecen a mirar Cursor o Claude se van a encontrar exactamente los mismos vicios: sesiones eternas, contexto repetido, modelos caros para tareas chiquitas.

Lo concreto para vos: las técnicas que mueven la aguja son las mismas. Mantené sesiones cortas, abrí una nueva cuando la conversación cambió de tema, sacá lo que el modelo no necesita ver, elegí el modelo más chico que sirva para la tarea. En Cursor eso se traduce en usar Auto mode y no forzar Claude Opus para todo. En Claude Code, en cuidar el `CLAUDE.md` y no dejar que el contexto crezca sin freno.

### "¿Cómo funciona prompt caching y cómo lo activo?"

Esta es una pregunta más útil que la objeción que solía aparecer en su lugar ("suena a optimización prematura"). Vamos directo al cómo.

**El concepto.** Anthropic cobra los tokens cacheados a un 10% del precio normal —es decir, **descuento del 90% en cache reads**. La contrapartida: el primer envío (cache write) cuesta **25% más** que un input normal. La cuenta sale a favor rápido: con que el mismo contenido se lea del cache una sola vez, ya ahorrás respecto a no cachear. Más allá de un hit, el ahorro se multiplica. El TTL por defecto es **5 minutos**. Hay una opción de **1 hora** disponible a costo extra.

**Cómo se activa, según dónde estés:**

*Si usás Claude Code o Claude Desktop (caja gris):* no tenés que activar nada. La herramienta cachea por vos. Tu CLAUDE.md, tu system prompt, los archivos que cargás —todo va al cache automáticamente. Lo que sí podés hacer es **medirlo**: si tenés OpenTelemetry activado (sección "Gobernanza personal" más abajo), vas a ver dos métricas separadas, `cache_read` y `cache_creation`. La proporción entre las dos es tu hit rate efectivo. Si tu cache_read es alto comparado con input fresco, vas bien. Si no, probablemente estás cambiando demasiado el prompt entre llamadas, lo que invalida el cache.

*Si usás Copilot en VSCode (caja gris):* mismo principio, la herramienta cachea sola. VSCode ha publicado que su prompt caching reuse rate es del **93%** dentro del editor, así que confiá: está pasando aunque no lo veas. Lo que vos podés hacer para favorecer el cache: **mantené estable la estructura de tus prompts y de tus archivos abiertos**. Cada vez que abrís y cerrás archivos, o cambiás el orden de los tabs, podés estar invalidando porciones del cache. Una conversación larga en un mismo contexto cachea bien; saltar entre proyectos resetea la cuenta.

*Si construís con API directa de Anthropic:* acá sí lo activás vos, y hay dos modos. El más simple es **automatic caching**: agregás un único campo `"cache_control": {"type": "ephemeral"}` a nivel top-level del request y Anthropic ubica el breakpoint automáticamente en el último bloque cacheable. El otro modo es **explicit caching**, donde ponés `cache_control` en bloques específicos (system prompt, tool definitions, contenido pesado). Hasta 4 breakpoints por request. Con explicit caching tenés control fino sobre qué cachear y qué no; con automatic empezás en treinta segundos. Cualquier app que mande el mismo system prompt o el mismo set de tools en cada turno debería tener esto activado.

*Si vas vía AWS Bedrock o Google Vertex:* la sintaxis es la misma —Bedrock soporta `cache_control: {"type": "ephemeral"}` igual que la API directa. Documentación de cada hyperscaler tiene el detalle por modelo, porque no todos los modelos cachean todo.

**Cómo verificás que está funcionando.** En la respuesta de cualquier llamada API de Anthropic, el objeto `usage` te trae:

```
"usage": {
  "input_tokens": 21,
  "cache_creation_input_tokens": 188086,
  "cache_read_input_tokens": 0,
  "output_tokens": 393
}
```

En la primera llamada vas a ver `cache_creation` alto y `cache_read` en cero (estás escribiendo el cache). En la segunda llamada con el mismo prefijo, ves `cache_creation` en cero y `cache_read` alto. Esa segunda llamada está pagando 10% del precio para el contenido cacheado. Si nunca ves `cache_read` distinto de cero, algo está mal: lo más común es que el contenido cacheable esté cambiando entre requests y por eso se reescribe siempre.

### "Mi sesión no es tan larga"

Eso es lo que pensaba yo también. Después miré.

Una conversación normal de 20 turnos arrastra entre **5.000 y 10.000 tokens innecesarios** que se podrían haber dejado afuera. Eso es por conversación. Multiplicalo por la cantidad de conversaciones por día. Multiplicalo por el equipo. Multiplicalo por las empresas de Visma.

Y un detalle técnico que se subestima mucho: **el output cuesta 4-6x más que el input**. No es un detalle menor. Cuando le pedís a un modelo que te resuma todo lo que hablamos hasta acá, el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

### "Tengo extended thinking en `display: omitted`. No se ve, no cuenta"

Cuenta. Cuenta como output token. Que no se muestre no significa que no se facture. La opción `omitted` te ahorra latencia perceptual, no dinero.

Una llamada con 2.000 tokens de thinking y 500 de output cuesta **5x más** que una sin thinking. Si activaste extended thinking porque "siempre da mejores respuestas" y nunca lo apagaste, esto te toca. Pensar es caro. Decidir cuándo vale la pena es trabajo del que prompt-ea, no del modelo.

### "Tenemos Microsoft Enterprise Agreement, ya tenemos descuento"

Lo tenemos. Es del **15-25%**, y con MACC sube a **23-28%**. Pero ese descuento aplica sobre el consumo Azure, no sobre la API directa. Y Azure sobre OpenAI directo tiene un **overhead de 15-40% en TCO**.

Hagamos la cuenta. Descuento del 25% sobre un servicio que tiene 30% de overhead sobre la alternativa directa. El "descuento" desaparece. No es que el EA no sirva —sirve, pero no como ahorro automático. Sirve como compliance, data residency y consolidación de factura. Y eso, para empresas europeas con GDPR encima como las de Visma, **es exactamente lo que estamos comprando.** Pagamos hyperscaler por seguridad jurídica, no por precio.

El caso extremo es DeepSeek vía Azure: **+35% de markup** sobre el precio directo, pero el precio directo no es opción porque los servidores están en China. Acá el descuento del EA compra compliance, no ahorro. Está bien que así sea, **pero hay que llamarlo por su nombre.**

### "Esto se va a resolver solo con modelos más baratos"

En parte, sí. Pero ya hay una técnica que da el **95% de la calidad de GPT-4 con el 26% de las llamadas al modelo fuerte**. Se llama RouteLLM. Con data augmentation, baja al **14% de llamadas al modelo fuerte y 75% de ahorro**.

El **37% de las enterprises ya corre 5+ modelos en producción**. IDC proyecta **70% para 2028**. El ahorro promedio por model routing inteligente es **20-80% del OpEx**.

Esto ya está. No es futuro. La pregunta no es "¿llegarán modelos más baratos?", es "¿estamos usando los baratos que ya existen para las tareas que no necesitan los caros?".

---

## "Pero antes de cobrarme, ¿no me avisa? ¿No se bloquea solo?"

Esta es la pregunta más importante del artículo, y la que más gente me hizo. La respuesta corta es: **depende. Y la mayoría de las veces, no se bloquea solo.**

Vale la pena explicarlo herramienta por herramienta, porque cada una funciona distinto. No es para que sigas una receta uniforme —cada BU de Visma tiene su mix de proveedores, sus planes, sus convenios con corporate. El objetivo de esta tabla es que sepas **dónde mirar** en cada caso para conocer tu propia situación:

| Herramienta | Hay stopper por default | Dónde se configura el tope |
|---|---|---|
| **GitHub Copilot (nuevo billing)** | No. Por encima de los AI Credits del seat, el overage se factura sin límite | Admin organización: setear "spending limit" en cero o en un monto X en billing settings |
| **Cursor** | Hay tope por plan, pero permite "additional usage" si lo activás | Settings de la org: desactivar overage |
| **Claude Desktop / Code (planes Pro/Team/Enterprise)** | Sí, te corta cuando se acaba la cuota del plan | Subir de plan o esperar reset mensual |
| **Anthropic API directa** | No. Cobra hasta que llegue el límite de tu tarjeta o tu workspace limit | Consola Anthropic → Limits → Workspace spend limit |
| **OpenAI API directa** | No. Mismo modelo que Anthropic | Dashboard OpenAI → Settings → Limits → Monthly budget |
| **Azure OpenAI / Bedrock** | No. Va a la factura cloud sin tope salvo que se ponga budget alert | Azure Cost Management / AWS Budgets, alertas en X% del presupuesto |

Lo que hay que hacer con esto no es una receta. Es un ejercicio de **revisar y configurar**.

**Revisá tu situación específica.** ¿Qué herramientas usás en tu BU? ¿Bajo qué plan? ¿El convenio con Visma corporate cubre todo o algunos seats están fuera? ¿El billing está centralizado o cada equipo paga aparte? Esa información determina dónde tenés que ir a poner un límite, y a veces revela que no tenés permisos para hacerlo vos —que la conversación es con IT o con el admin de la organización.

**Configurá lo que corresponda a tu situación.** Si la herramienta soporta spending limit y aplica a tu caso, ponelo. Si soporta alertas de budget, configuralas. Si no soporta ninguna de las dos, asegurate de que alguien esté mirando la trayectoria del consumo manualmente. Lo importante no es que todos hagan exactamente lo mismo —es que **nadie esté operando con default = "sin tope"** sin saberlo.

**Revisá de nuevo en un mes.** Las herramientas cambian settings, los planes se actualizan, los contratos se renegocian. Lo que configuraste en mayo puede haber dejado de aplicarse en junio. El stopper no es un evento, es una práctica.

Una nota sobre cómo leer esto: **el stopper no es una solución, es una alarma de incendio.** Si tu factura llega al tope, algo se rompió antes. La pregunta no es "¿cuándo nos cortan?", es "¿quién está mirando la trayectoria y avisando antes de que el tope se acerque?". Si nadie tiene ese rol explícito en tu BU, ese es el primer puesto que hay que crear o asignar. No es un rol full-time, son 2 horas al mes mirando un dashboard. Pero tiene que ser de alguien.

---

## Gobernanza personal: cómo te medís vos mismo

Acá llega la parte concreta del artículo. Porque toda esta conversación sobre factura, BUs y empresas arranca en algo mucho más chico: **cada persona que usa IA todos los días puede —y debería— ver su propio consumo.** No el del equipo. No el de la BU. El propio.

No te estoy proponiendo esto como obligación ni como única forma. Te lo propongo como **la forma más rápida de empezar a entender lo que consumís**.

Si vos no sabés cuántas sesiones abriste esta semana, cuánto contexto repetido mandaste, qué modelos disparaste, cuántos tokens de input contra tokens de output —que cuestan 4-6x más—, entonces no estás haciendo gobernanza, estás adivinando. Y la buena noticia es que **medirse a uno mismo en 2026 lleva diez minutos de setup y cero euros**, si usás las herramientas correctas.

Vamos por partes.

### Lo que tenés que actualizar primero: caja gris ya no significa "ciego"

Antes de la sección técnica, una corrección importante a algo que se dice mucho. Las herramientas IDE no son cajas negras opacas. Son **cajas grises configurables**: pagás por plan, no por token, pero exponen OpenTelemetry de fábrica. Lo que cambia es **quién instrumenta**: vos en lugar del vendor.

Herramientas que **sí** exponen OpenTelemetry hoy:

- **Claude Code:** soporte oficial. Activás una variable de entorno y empieza a exportar métricas y eventos a cualquier backend OTLP.
- **Claude Cowork (Claude Desktop)** en planes Team y Enterprise, desde la versión 1.1.4173 en adelante. Stream de eventos con prompts, invocaciones de tools y MCP, latencias.
- **GitHub Copilot Chat:** settings nativos en VSCode (`github.copilot.chat.otel.enabled`). Exporta traces, métricas y eventos de cada llamada LLM, tool execution y sesión de agente. Atributos siguiendo GenAI Semantic Conventions (`gen_ai.request.model`, `gen_ai.provider.name`, token counts, durations).
- **Codex CLI (OpenAI):** exporta logs estructurados y métricas OTel.

La que **todavía no** lo expone de forma nativa: **Cursor**. Tiene Admin Dashboard con API propia y exports CSV en planes Enterprise, pero no OTel out-of-the-box. Si te urge, hay hooks de comunidad (cursor-otel-hook, CursorLens como proxy) que cierran el gap.

**La regla actualizada:** antes de adoptar o profundizar en una herramienta de IA, preguntá si expone OTel. Si lo hace, podés tener observabilidad de grado enterprise sin necesidad de migrar nada a API directa. Si no lo hace, sabés que tu visibilidad va a depender del dashboard del vendor.

### Tu dashboard personal de Claude Code en diez minutos

Si usás Claude Code (que es el caso mejor documentado y el que más data exporta), esto es lo que vas a hacer.

Claude Code emite seis tipos de métricas por OpenTelemetry. Los nombres son textuales, no inventados:

- `claude_code_token_usage` — input, output, **cache creation** y **cache reads** desglosados. Esta es la métrica que te dice cuánto contexto estás repitiendo: si tus `cache_read` son altos relativo a tus `input` frescos, vas bien. Si no, estás pagando precio completo por cosas que ya enviaste.
- `claude_code_cost_usage` — costo en USD por API call.
- `claude_code_session_count` — número de sesiones que abriste.
- `claude_code_active_time_total` — tiempo activo de Claude Code.
- `claude_code_lines_of_code_count` — líneas agregadas y removidas.
- `claude_code_code_edit_tool_decision` — accept/reject de sugerencias, desglosado por lenguaje.

Para activarlo, abrís tu `~/.zshrc` o `~/.bashrc` y agregás estas variables:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

Eso solo no te da dashboard, te da el grifo abierto. Te falta el backend. Para uso personal hay dos caminos limpios:

**Camino A — Grafana Cloud (sin instalar nada en tu máquina, free tier).** Te abrís cuenta gratis, te dan un endpoint OTLP, lo ponés en `OTEL_EXPORTER_OTLP_ENDPOINT`, y ya estás mandando métricas a un Grafana que vive en la nube. Mirás tu dashboard desde el browser. Cero infraestructura local.

**Camino B — Aspire Dashboard local (un solo container Docker).** Si preferís que nada salga de tu máquina, levantás el container de Aspire Dashboard que Microsoft publica gratis. Te da un trace viewer con un endpoint OTLP built-in. Apuntás Claude Code ahí y listo. Sin cuenta cloud, sin enviar nada afuera.

Las dos opciones están documentadas y son de tres comandos cada una. No hace falta IT, no hace falta budget, no hace falta una reunión con tu tech lead. Es algo que armás vos un martes a la tarde.

Una vez que el dashboard está vivo, las preguntas que podés contestar son exactamente las que importan para gobernanza personal:

- **¿Cuántas sesiones abrí esta semana?** `claude_code_session_count` agrupado por día.
- **¿Cuánto estoy reenviando de contexto repetido?** Ratio entre `cache_read` y `input` fresco. Cuanto más alto el cache_read, mejor —significa que tu prompt caching está funcionando.
- **¿Qué modelos estoy usando, en qué proporción?** Las métricas vienen con label de modelo. Si veis que 80% de tus llamadas son a Opus y solo 20% a Sonnet, ahí tenés un ahorro inmediato.
- **¿Cuántos tokens input vs output?** El output cuesta 4-6x más. Ver la proporción te dice si tus pedidos están sesgados a generación pesada (reescribir archivos enteros, resúmenes largos) o a interacciones más livianas.
- **¿Cuánto me cuesta cada día?** `claude_code_cost_usage` por día. Si tenés un día atípico, mirá qué pasó. Si tu costo por línea de código cambia mes a mes, sabés si estás mejorando o empeorando.

### Tu dashboard personal de Copilot Chat

Casi idéntico, con otros nombres. En VSCode, abrís `settings.json` y agregás:

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.exporterType": "file",
  "github.copilot.chat.otel.outfile": "/tmp/copilot-otel.jsonl"
}
```

Esto te exporta a un archivo local (lo más simple, sin cloud). Si querés enviarlo a un backend, cambiás `exporterType` a `otlp` y agregás `otlpEndpoint`. Copilot Chat exporta traces (`invoke_agent`, `chat`, `execute_tool`, `execute_hook`), métricas de tokens y costos, y eventos de cada llamada LLM. La misma idea: lo conectás a Grafana Cloud o Aspire Dashboard y tenés tu dashboard.

Nota: el OTel monitoring viene **off por default** en Copilot Chat. Hay que activarlo. Está bien que sea así porque te da control de qué se exporta y a dónde.

### Cuando no es para vos, es para tu app

Toda la sección de arriba es para vos usando IA como dev. **Cuando una BU construye una feature de producto que llama LLMs por debajo**, ahí ya no hay vendor que te exporte: tenés que instrumentar el código tuyo. Es ahí donde la API directa con OTel propio es el único camino.

La buena noticia es que Anthropic, OpenAI y los providers grandes ya publican specs de OTel siguiendo las **GenAI Semantic Conventions**, así que no tenés que inventar nombres: usás los mismos atributos que ves en los dashboards de Claude Code o Copilot Chat. Una feature con cinco endpoints LLM, bien instrumentada, te da: consumo por endpoint, por modelo, por usuario final, por feature de producto. Eso es lo que después convierte una conversación de "subió la factura" en "subió la feature X porque el endpoint Z no tiene caching".

### Por qué importa que cada uno se mida

Esto no es un ejercicio de FinOps. Es algo más directo: **si vos no podés ver tu propio consumo, no podés mejorarlo.** Es la misma lógica de un atleta con su tracker, o de un dev con su profiler. No medirse no es neutral, es ceguera.

Y para Visma, la gobernanza colectiva no se construye desde arriba pidiendo reportes a las BUs. Se construye desde abajo, con cada persona que entiende su propio uso y ajusta. El tech lead que tiene un dashboard del equipo se basa en datos que existen porque la gente del equipo se midió primero. La BU que reporta consumo razonable a finance lo hace porque los devs midieron antes. Sin el primer paso, lo demás es teatro de presupuesto.

Diez minutos de setup. Cero euros de costo. Las preguntas de gobernanza personal contestadas para siempre. Si lo único que sacás de este artículo es armarte tu propio dashboard, ya valió la pena.

---

## El paralelismo que ayuda a pensarlo

En los 2010s, cuando AWS recién maduraba, hubo un patrón que se repitió en muchas empresas: alguien levantaba una instancia EC2 para probar algo, se olvidaba de bajarla, y a fin de mes alguien preguntaba "¿qué es esto de $4.000 en una región que ni usamos?". Las primeras facturas de cloud eran un shock no porque el cloud fuera caro, sino porque la unidad de costo había cambiado y nadie había actualizado los hábitos.

Compute pasó de "compré un servidor, lo amortizo en 3 años" a "pago por hora-CPU mientras esté prendida". El mismo equipo, con los mismos workloads, podía gastar 5x más o 5x menos según si alguien apagaba la instancia el viernes.

Tokens están en esa fase ahora. Pagábamos planos por seat —al menos en el vendor más usado—, ahora pagamos por uso. La factura va a tener varianzas grandes según hábito, no según workload. Y como en el caso de AWS, la diferencia entre los que ven el cambio venir y los que lo ven después en el dashboard no es 10%, es múltiplos.

La parte buena: ya sabemos cómo termina esta película. Las empresas que sobrevivieron a la transición de cloud no fueron las que volvieron a on-prem. Fueron las que aprendieron a tagear instancias, a apagar lo que no usaban, a elegir el tier correcto para cada workload, a instrumentar todo con métricas, a poner budgets y alertas. **Gastar bien, no gastar menos.**

Las empresas de Visma ya hicieron esa transición en cloud. No hay que reinventar. Las prácticas son las mismas, el activo a vigilar es nuevo.

---

## Qué se puede hacer ya, por nivel de esfuerzo

Tres niveles. Cualquier BU de Visma puede arrancar por el primero esta misma semana.

**Cero esfuerzo (el lunes a la mañana).** Activar Auto Mode en Copilot. Activar `chat.tools.compressOutput.enabled` en VSCode. Activar tool search en VSCode. En Cursor, asegurarse de estar en Auto. En Claude Code, revisar que el `CLAUDE.md` no tenga 500 líneas. Revisar y configurar spending limits donde aplique en tu caso. Esto es el piso. Ahorro estimado combinado: 15-25% sin tocar workflows.

**Esfuerzo medio (un sprint).** Cada dev se arma su dashboard personal de OpenTelemetry (Claude Code o Copilot Chat, según qué use más). Si una BU tiene una app o feature que llama a una API de modelo, activar prompt caching (automatic o explicit según el caso). Migrar workflows asíncronos (reportes, clasificaciones nocturnas, embeddings, evaluación de prompts en CI) a Batch API: **50% de descuento sin diferencia de calidad**, combinable con caching para **hasta 95% de ahorro**. Instrumentar las llamadas con OpenTelemetry siguiendo GenAI Semantic Conventions. Ahorro estimado: 60-90% en el costo de esa app específica.

**Esfuerzo grande (un trimestre, decisión de arquitectura).** Routing multi-modelo: clasificación a Haiku, código y razonamiento estándar a Sonnet, decisiones complejas a Opus, batch y experimentos a DeepSeek directo (donde no haya tema de datos sensibles) o vía Azure (donde sí). RouteLLM o LiteLLM o Portkey como gateway. **Ahorro reportado: 20-80% del OpEx**, dependiendo del mix actual. Esto no es para todas las BUs, pero para las que ya tienen carga seria de IA en producción es la palanca grande.

---

## Lo que no te puede arreglar ningún proveedor

Estas tres cosas dependen de cada empresa de Visma, y de cada persona dentro de ella. Las pongo en orden de impacto.

**Decidir qué tarea va a qué modelo.** Si todo va al modelo más caro, todo cuesta como el más caro. La factura no la fija el proveedor, la fija el routing. Es decisión de arquitectura, no de herramienta.

**Cortar contexto.** El 62% de basura en la factura no se va con un setting. Se va decidiendo qué información mandar y qué no, qué historia comprimir, cuándo abrir una sesión nueva en vez de seguir la del lunes. Context engineering reduce costos **entre 60% y 80%**. Es una habilidad, no un click.

**Mirar la factura — empezando por la propia.** Suena trivial y es el más importante. Mirar la propia, mirar la del equipo, mirar la de la BU, mirar la trayectoria mensual. **IDC proyecta que las Global 1000 están subestimando sus costos de AI en un 30% hacia 2027.** Eso es exactamente lo que pasa cuando nadie mira. Si no hay alguien con la pregunta "¿qué la está moviendo este mes?", la respuesta siempre va a ser "más uso de IA", y eso no es accionable.

---

## No es una receta. Es un momento.

Quiero ser claro en algo antes de los pasos accionables: nada de lo que escribí acá es "la forma" de trabajar con IA. No estoy diciendo que todos tengan que tener un dashboard de Grafana, ni que todos tengan que migrar features a API directa, ni que todos tengan que hacer routing multi-modelo el viernes. Lo que digo es otra cosa.

**Esta es la forma de entender. Y entender, hoy, es lo que hay que hacer.**

Tuvimos un par de años muy generosos. Aparecieron herramientas nuevas todos los meses, los modelos eran cada vez mejores, los precios bajaban, y los planos flat nos dejaban experimentar sin pensar dos veces. Si activaste extended thinking "por las dudas", si abriste 40 sesiones esta semana, si dejaste un agente corriendo todo el fin de semana —el costo de aprender de eso fue bajo. Llegaba la factura plana, todos seguíamos.

Ese período se está terminando. No de golpe, no para todos, no el 1 de junio. Pero se está terminando. Y la ventana para aprender barato —para experimentar, medir, equivocarse, ajustar, sin que duela— **es exactamente ahora**.

Los que aprovechen esta ventana van a salir del otro lado entendiendo cómo se gobierna el consumo de IA. Van a saber qué setting mueve qué número. Van a tener intuición. Van a ser los que en seis meses, cuando esto sea política de empresa en todas las BUs de Visma, ya tengan la respuesta. **Los que la dejen pasar van a tener que aprender lo mismo, pero con la factura corriendo y menos margen para experimentar.** Es la misma curva, distinto contexto emocional.

Por eso este artículo. No para decirte qué hacer, sino para mostrarte por dónde se empieza a aprender, y para que sepas que el momento de hacerlo es este. Convertirse en buenos en esto —los mejores, incluso— no es magia, es atravesar una curva de aprendizaje. La curva está. El timing también. Lo único que falta es decidir entrar.

---

## Qué hacer esta semana, por rol

**Si sos developer.** Lunes: Auto Mode, compressOutput, tool search. Martes a la tarde: armate tu dashboard personal de OpenTelemetry. Diez minutos, cero euros, te va a cambiar la vista. Miércoles: con el dashboard ya andando, mirá una sesión larga tuya y observá cuánto contexto está arrastrando, qué modelos usaste, cuántos tokens fueron output, qué hit rate de caching tenés. Viernes: si tu equipo tiene una feature con API propia, abrí el ticket para activar caching y para instrumentar las llamadas con OTel.

**Si sos tech lead.** Lunes: revisá qué herramientas usa tu BU y dónde aplican spending limits. Configurá lo que corresponda en tu caso. Martes: contale al equipo cómo armarse el dashboard personal. La gobernanza colectiva empieza por la individual. Miércoles: hacé una hora con el equipo para decidir en qué punto del espectro (caja negra opaca, caja gris, API) están parados los distintos casos de uso. Viernes: si tenés algo en API sin OpenTelemetry, ponelo en el roadmap del próximo sprint.

**Si tomás decisiones de presupuesto en una BU.** Lunes: pedí el dashboard de uso de los últimos 90 días por herramienta. Miércoles: identificá quién es el responsable nombrado de mirar la trayectoria mensual. Si nadie, asignalo. Viernes: agendá la conversación con finance para mostrarle el cambio de junio antes de que ellos te lo pregunten en julio.

**Si estás mirando Visma desde arriba.** Esto le toca a alguien. La factura agregada de las empresas de Visma en IA va a cambiar de forma este año. Las BUs que vean el cambio venir van a salir bien paradas. Las que no, van a explicar en julio por qué la línea de AI tools subió 3x.

**Gastar bien, no gastar menos.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: usuarios que entienden lo que consumen, equipos que monitorean lo que pagan, BUs que eligen entre caja gris y API con criterio. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La traen las empresas de Visma. Y empieza, literal, en cada dev que se arma su dashboard un martes a la tarde. No para hacer todo distinto. Para entender lo que ya está haciendo. Esa es la curva que vale la pena atravesar ahora, mientras todavía sale barata.

---

## Para profundizar

Los datos, fuentes y multiplicadores citados en este artículo están en `data/verified-metrics.md` del repo, con URL a cada fuente original. La documentación técnica (`PROD/technical-docs-draft-v2.md`) tiene el detalle por capa, con ejemplos de código, casos de aplicación y de no-aplicación. El documento sobre el descuento del hyperscaler y el caso GDPR/DeepSeek está en `PRODV2/hyperscaler-discount-vs-gdpr-ES.md`.

Para los detalles de OpenTelemetry en Claude Code, la documentación oficial está en `code.claude.com/docs/en/agent-sdk/observability`. Para Copilot Chat, en `code.visualstudio.com/docs/copilot/guides/monitoring-agents`. Para prompt caching de Anthropic, en `platform.claude.com/docs/en/build-with-claude/prompt-caching`. Para las GenAI Semantic Conventions estándar (usadas tanto por Anthropic como por OpenAI), en `opentelemetry.io/docs/specs/semconv/gen-ai/`.
