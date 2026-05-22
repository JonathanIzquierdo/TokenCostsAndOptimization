# Tokens, factura, y una conversación que se nos venía

El 1 de junio GitHub Copilot cambia su modelo de cobro. En Visma es el vendor de IA más adoptado, así que para muchas BUs del grupo este es el primer mes en el que la pregunta "¿cuánto consumimos en tokens?" deja de ser teórica. Un artículo para entender el cambio, el impacto, las estrategias para optimizar consumo, y cómo monitorearlo en serio —empezando por vos mismo.

---

## Por qué este artículo, y por qué ahora

El 1 de junio GitHub Copilot deja de cobrarse plano y pasa a usage-based billing. Cada seat sigue costando lo mismo, pero ahora trae AI Credits incluidos: mientras estás dentro de esos créditos no se cobra extra; cuando se acaban, se paga por consumo. Es un cambio que afecta directo a Visma porque Copilot es probablemente el vendor de IA más adoptado del grupo: una porción grande de nuestro consumo de IA hoy pasa por ahí.

Ese es el disparador. Pero conviene decirlo desde el principio: **el resto del stack ya funcionaba así**. Cursor cobra por planes con cuota y overage desde hace meses. Claude (Desktop, Code, Cowork) lo mismo. Las APIs directas de Anthropic, OpenAI, Azure OpenAI, Bedrock —siempre fueron pay-per-token desde el día uno, nunca tuvieron flat fee. Lo único que cambia el 1 de junio es Copilot. Lo demás ya estaba.

Y ahí está la cosa incómoda. La pregunta "¿cuánto nos cuesta cada token que consumimos?" debería haber estado dando vueltas en nuestras cabezas desde hace bastante. Si no estaba, fue porque el flat fee de Copilot nos tapaba la vista en la parte del stack donde más consumimos, y porque en el resto —Claude, Cursor, APIs— el consumo era todavía chico o estaba sumido dentro de presupuestos más grandes que nadie auditaba.

Ese período se termina. Y se termina de la peor manera posible: con el cambio del vendor más usado, justo cuando nadie del grupo está particularmente entrenado en pensar consumo de tokens como una capacidad del usuario. Porque eso es lo que es. **Usar IA bien no es solo prompt engineering, es saber consumir menos tokens para obtener el mismo resultado.** Y esa capacidad, igual que el FinOps de cloud hace 10 años, se construye con hábito, no con una herramienta mágica.

Visto así, **este evento es la excusa correcta para una conversación que ya nos debíamos.** El 1 de junio no inventa el problema, lo hace visible. Y mientras el problema fue invisible, nadie tenía razones para construir la capacidad. Ahora sí.

Este artículo es para empezar a construir ese hábito en el grupo. Va a hablar del impacto concreto que tiene el cambio de Copilot. Va a traer tips y estrategias prácticas para reducir consumo y costo, aplicables tanto si usás Copilot como si usás Cursor o Claude. Va a poner sobre la mesa una distinción conceptual que cambia cómo se piensa todo lo demás: **caja negra vs API Key.** Y, sobre todo, va a mostrar **cómo cada uno se puede medir a sí mismo** — porque la gobernanza individual es la base de toda gobernanza colectiva.

Hay una idea que va a aparecer varias veces: gastar bien, no gastar menos. No estamos buscando achicar la IA. Estamos buscando que cada euro que el grupo pone en IA esté trabajando, no calentando aire.

---

## Lo que está pasando con el dinero (y nadie lo está mirando)

Antes de hablar de qué hacer, vale mostrar dónde está el problema.

En 30 equipos relevados entre marzo y mayo de 2026, **el 62% de la factura de IA no es por trabajo del modelo**. Es por mandar el mismo contexto, una y otra vez. De cada 100 euros que una empresa pone en IA, 62 son literalmente el mismo system prompt, la misma documentación de proyecto, el mismo historial, viajando ida y vuelta hasta el modelo. Es como si un courier te cobrara cada vez que mueve una caja y vos lo hicieras pasar 50 veces por la misma esquina porque te olvidaste de pedirle que la dejara en el destino.

Lo concreto: una sesión típica empieza en 5.000 tokens. Para el turno 50 está en 200.000. Nadie lo ve. No hay un cartel que diga "estás a 40x del costo de cuando empezaste". La factura llega a fin de mes y se atribuye a "más uso de IA".

Hasta ahora esto no nos pegaba directo porque en Copilot pagábamos plano —Copilot Business $19/seat, Enterprise $39/seat— y en el resto de las herramientas el consumo todavía no era el suficiente como para que alguien levantara la mano. Eso cambia. Los workflows agénticos intensivos, que es donde la industria va, se están midiendo en torno a **3.5x el flat fee anterior**. Y no se distribuye parejo: el **top 10-15% de usuarios concentra el 60-70% del gasto**. Si tu BU tiene devs que usan IA todo el día —y son los que la usan bien y producen más—, son ellos los que mueven la aguja para todos los demás.

Esto importa para cada empresa del grupo, no para "el equipo que más usa IA". La factura es de Visma como grupo, y los presupuestos individuales de cada BU son el lugar donde el cambio se va a sentir. Si una BU resuelve esto bien y otra no, el grupo paga el promedio.

---

## El concepto que cambia todo: caja negra vs API Key

Antes de seguir con tips concretos, una pausa conceptual que vale para gente de todos los niveles técnicos del grupo. Es la distinción más importante del artículo.

Cuando una BU "usa IA", está eligiendo entre dos formas muy distintas de consumirla. Importa para la factura, para la gobernanza y para qué optimizaciones están disponibles. Lo más importante: **mucha gente no sabe en cuál de las dos está parada.**

**Caja negra:** una herramienta empaquetada que por adentro llama a un modelo. Copilot, Cursor, Claude Desktop, Claude Code, ChatGPT Enterprise. Vos no ves el prompt que se manda. No ves cuánto contexto carga la herramienta antes de tu pregunta. No ves qué modelo elige para qué tarea (salvo que te lo deje configurar). Pagás un plan, consumís, te llega la factura. La herramienta hace optimizaciones por vos (Copilot ya cachea, Cursor también, Claude Code también) pero las decisiones grandes están afuera de tu alcance.

**API Key:** consumir el modelo directo del proveedor (Anthropic, OpenAI, DeepSeek, Google) o de un hyperscaler (Azure, AWS Bedrock, Google Vertex), con una clave y código propio. Vos armás el prompt, vos decidís qué contexto va, vos elegís el modelo, vos activás caching, vos medís. El costo es por token consumido, sin plan flat.

| Aspecto | Caja negra | API Key |
|---|---|---|
| **Modelo de cobro** | Plan flat o créditos del plan + overage | Por token, sin plan |
| **Quién arma el prompt** | La herramienta (no lo ves) | Tu código |
| **Quién elige el modelo** | La herramienta (con o sin tu input) | Tu código |
| **Prompt caching** | Lo aplica la herramienta si quiere | Lo activás vos |
| **Batch API** | No disponible | Disponible, 50% off |
| **Observabilidad por persona** | Depende — Claude Code, Claude Cowork (Team/Ent) y Copilot Chat ya exponen OpenTelemetry; Cursor todavía no | OpenTelemetry propio, control total |
| **Stopper de costo** | Depende del plan y de los settings del admin | Usage limits en la consola del proveedor |
| **Time-to-value** | Bajo (descargás y andás) | Alto (hay que construir) |
| **Donde se controla el costo** | Eligiendo plan y settings | En cada llamada |

No hay opción "mejor". Para un dev individual escribiendo código, la caja negra es la respuesta. Para una feature de producto con miles de llamadas diarias y patrón estable, la API es la respuesta porque te abre 60-95% de descuentos que la caja negra no puede darte.

Lo importante para cada empresa del grupo es esto: **decidir caja negra vs API por defecto, sin pensarlo, es la decisión más cara que se está tomando hoy.** Si una BU está corriendo en producción una feature que llama a un LLM 10.000 veces por día a través de una caja negra cuando podría hacerlo con API + caching, está pagando 5-10x sin razón. Y al revés, si una BU está consumiendo IA conversacional para tareas de dev exploratorias vía API artesanal cuando podría usar Copilot o Cursor, está pagando con tiempo del equipo lo que el flat fee cubre.

La regla simple: **caja negra para uso humano, API para uso programático.** Cualquier cosa que se ejecute sin un humano esperando del otro lado debería ir por API.

---

## Las objeciones que ya escuché tres veces esta semana

Liderar esta conversación dentro de Visma me dejó una colección de respuestas. Vale la pena visitarlas, porque las preguntas son las mismas en todas las BUs.

### "Yo uso Cursor o Claude Code, esto no me toca"

Te toca igual, aunque por una razón distinta. El 1 de junio no cambia ni Cursor ni Claude —esos ya estaban en planes con cuota y overage desde antes. Lo que cambia es que **antes nadie miraba el consumo en ninguna parte del stack**, porque Copilot era flat y el resto era todavía chico. Ahora que la línea más gorda del presupuesto (Copilot) pasa a usage-based, las BUs van a empezar a mirar todo. Y cuando empiecen a mirar Cursor o Claude se van a encontrar exactamente los mismos vicios: sesiones eternas, contexto repetido, modelos caros para tareas chiquitas.

Lo concreto para vos: las técnicas que mueven la aguja son las mismas. Mantené sesiones cortas, abrí una nueva cuando la conversación cambió de tema, sacá lo que el modelo no necesita ver, elegí el modelo más chico que sirva para la tarea. En Cursor eso se traduce en usar Auto mode y no forzar Claude Opus para todo. En Claude Code, en cuidar el `CLAUDE.md` y no dejar que el contexto crezca sin freno.

### "Prompt caching suena a optimización prematura"

Knuth dijo que la optimización prematura es la raíz de todos los males. Tenía razón sobre CPU en los 70s. No tiene razón sobre tokens en 2026, porque acá la "optimización" no es reescribir un loop —es activar un descuento del **90%** que Anthropic puso ahí para que lo uses.

El break-even del cache es **un solo hit**. Cualquier app con system prompt estable o RAG está dejando dinero en la mesa si no lo activa. Una app típica con 50K tokens de contexto y 1000 queries diarias ahorra **entre 88% y 95%** con caching.

VSCode ya implementó esto silenciosamente: el prompt caching reuse rate dentro del editor es del **93%**. No es teoría, es lo que ya está pasando en tu IDE sin que lo veas.

### "Mi sesión no es tan larga"

Eso es lo que pensaba yo también. Después miré.

Una conversación normal de 20 turnos arrastra entre **5.000 y 10.000 tokens innecesarios** que se podrían haber dejado afuera. Eso es por conversación. Multiplicalo por la cantidad de conversaciones por día. Multiplicalo por el equipo. Multiplicalo por las empresas del grupo.

Y un detalle técnico que se subestima mucho: **el output cuesta 4-6x más que el input**. No es un detalle menor. Cuando le pedís a un modelo que te resuma todo lo que hablamos hasta acá, el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

### "Tengo extended thinking en `display: omitted`. No se ve, no cuenta"

Cuenta. Cuenta como output token. Que no se muestre no significa que no se facture. La opción `omitted` te ahorra latencia perceptual, no dinero.

Una llamada con 2.000 tokens de thinking y 500 de output cuesta **5x más** que una sin thinking. Si activaste extended thinking porque "siempre da mejores respuestas" y nunca lo apagaste, esto te toca. Pensar es caro. Decidir cuándo vale la pena es trabajo del que prompt-ea, no del modelo.

### "Tenemos Microsoft Enterprise Agreement, ya tenemos descuento"

Lo tenemos. Es del **15-25%**, y con MACC sube a **23-28%**. Pero ese descuento aplica sobre el consumo Azure, no sobre la API directa. Y Azure sobre OpenAI directo tiene un **overhead de 15-40% en TCO**.

Hagamos la cuenta. Descuento del 25% sobre un servicio que tiene 30% de overhead sobre la alternativa directa. El "descuento" desaparece. No es que el EA no sirva —sirve, pero no como ahorro automático. Sirve como compliance, data residency y consolidación de factura. Y eso, para una empresa europea con GDPR encima como cualquiera del grupo Visma, **es exactamente lo que estamos comprando.** Pagamos hyperscaler por seguridad jurídica, no por precio.

El caso extremo es DeepSeek vía Azure: **+35% de markup** sobre el precio directo, pero el precio directo no es opción porque los servidores están en China. Acá el descuento del EA compra compliance, no ahorro. Está bien que así sea, **pero hay que llamarlo por su nombre.**

### "Esto se va a resolver solo con modelos más baratos"

En parte, sí. Pero ya hay una técnica que da el **95% de la calidad de GPT-4 con el 26% de las llamadas al modelo fuerte**. Se llama RouteLLM. Con data augmentation, baja al **14% de llamadas al modelo fuerte y 75% de ahorro**.

El **37% de las enterprises ya corre 5+ modelos en producción**. IDC proyecta **70% para 2028**. El ahorro promedio por model routing inteligente es **20-80% del OpEx**.

Esto ya está. No es futuro. La pregunta no es "¿llegarán modelos más baratos?", es "¿estamos usando los baratos que ya existen para las tareas que no necesitan los caros?".

---

## "Pero antes de cobrarme, ¿no me avisa? ¿No se bloquea solo?"

Esta es la pregunta más importante del artículo, y la que más gente del grupo me hizo. La respuesta corta es: **no, no se bloquea solo. Por default no hay stopper.**

Vale la pena explicarlo herramienta por herramienta, porque cada una funciona distinto.

| Herramienta | Hay stopper por default | Cómo se configura el tope |
|---|---|---|
| **GitHub Copilot (nuevo billing)** | No. Por encima de los AI Credits del seat, el overage se factura sin límite | Admin organización: setear "spending limit" en cero o en un monto X en billing settings |
| **Cursor** | Hay tope por plan, pero permite "additional usage" si lo activás | Settings de la org: desactivar overage |
| **Claude Desktop / Code (planes Pro/Team/Enterprise)** | Sí, te corta cuando se acaba la cuota del plan | Subir de plan o esperar reset mensual |
| **Anthropic API directa** | No. Cobra hasta que llegue el límite de tu tarjeta o tu workspace limit | Consola Anthropic → Limits → Workspace spend limit |
| **OpenAI API directa** | No. Mismo modelo que Anthropic | Dashboard OpenAI → Settings → Limits → Monthly budget |
| **Azure OpenAI / Bedrock** | No. Va a la factura cloud sin tope salvo que se ponga budget alert | Azure Cost Management / AWS Budgets, alertas en X% del presupuesto |

Hay dos formas de leer esto, y las dos importan.

La primera, defensiva: **alguien en cada BU del grupo tiene que configurar el spending limit antes del 1 de junio.** No es una recomendación, es una decisión que ya está tomada por default si no se hace nada —y la decisión default es "no hay tope". Para Copilot eso significa entrar a la configuración de billing de la organización en GitHub y poner un número. Para Anthropic API significa entrar a la consola y poner un workspace spend limit. Para Azure significa configurar un budget alert. No lleva más de media hora por proveedor, y es la diferencia entre "supimos a tiempo" y "nos enteramos en julio".

La segunda, de gobernanza: **el stopper no es una solución, es una alarma de incendio.** Si tu factura llega al tope, algo se rompió antes. La pregunta no es "¿cuándo nos cortan?", es "¿quién está mirando la trayectoria y avisando antes de que el tope se acerque?". Si nadie tiene ese rol explícito en una BU del grupo, ese es el primer puesto que hay que crear o asignar. No es un rol full-time, son 2 horas al mes mirando un dashboard. Pero tiene que ser de alguien.

---

## Gobernanza personal: cómo te medís vos mismo

Acá llega la parte concreta del artículo. Porque toda esta conversación sobre factura, BUs y grupos arranca en algo mucho más chico: **cada persona que usa IA todos los días puede —y debería— ver su propio consumo.** No el del equipo. No el de la BU. El propio.

No te estoy proponiendo esto como obligación ni como única forma. Te lo propongo como **la forma más rápida de empezar a entender lo que consumís**.

Si vos no sabés cuántas sesiones abriste esta semana, cuánto contexto repetido mandaste, qué modelos disparaste, cuántos tokens de input contra tokens de output —que cuestan 4-6x más—, entonces no estás haciendo gobernanza, estás adivinando. Y la buena noticia es que **medirse a uno mismo en 2026 lleva diez minutos de setup y cero euros**, si usás las herramientas correctas.

Vamos por partes.

### Lo que tenés que actualizar primero: caja negra ya no significa "ciego"

Antes de la sección técnica, una corrección importante a algo que se dice mucho. Caja negra **no significa más** que no se puede instrumentar. Eso era cierto hace 12-18 meses. Hoy, varias de las herramientas que técnicamente son cajas negras —porque pagás por plan, no por token— exponen OpenTelemetry de fábrica. Lo que cambia es **quién instrumenta**: vos en lugar del vendor.

Herramientas que **sí** exponen OpenTelemetry hoy:

- **Claude Code:** soporte oficial. Activás una variable de entorno y empieza a exportar métricas y eventos a cualquier backend OTLP.
- **Claude Cowork (Claude Desktop)** en planes Team y Enterprise, desde la versión 1.1.4173 en adelante. Stream de eventos con prompts, invocaciones de tools y MCP, latencias.
- **GitHub Copilot Chat:** settings nativos en VSCode (`github.copilot.chat.otel.enabled`). Exporta traces, métricas y eventos de cada llamada LLM, tool execution y sesión de agente.
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

Toda la sección de arriba es para vos usando IA como dev. **Cuando una BU del grupo construye una feature de producto que llama LLMs por debajo**, ahí ya no hay vendor que te exporte: tenés que instrumentar el código tuyo. Es ahí donde la API directa con OTel propio es el único camino.

La buena noticia es que Anthropic, OpenAI y los providers grandes ya publican specs de OTel siguiendo las **GenAI Semantic Conventions**, así que no tenés que inventar nombres: usás los mismos atributos que ves en los dashboards de Claude Code o Copilot Chat. Una feature con cinco endpoints LLM, bien instrumentada, te da: consumo por endpoint, por modelo, por usuario final, por feature de producto. Eso es lo que después convierte una conversación de "subió la factura" en "subió la feature X porque el endpoint Z no tiene caching".

### Por qué importa que cada uno se mida

Esto no es un ejercicio de FinOps. Es algo más directo: **si vos no podés ver tu propio consumo, no podés mejorarlo.** Es la misma lógica de un atleta con su tracker, o de un dev con su profiler. No medirse no es neutral, es ceguera.

Y para el grupo Visma, la gobernanza colectiva no se construye desde arriba pidiendo reportes a las BUs. Se construye desde abajo, con cada persona que entiende su propio uso y ajusta. El tech lead que tiene un dashboard del equipo se basa en datos que existen porque la gente del equipo se midió primero. La BU que reporta consumo razonable a finance lo hace porque los devs midieron antes. Sin el primer paso, lo demás es teatro de presupuesto.

Diez minutos de setup. Cero euros de costo. Las preguntas de gobernanza personal contestadas para siempre. Si lo único que sacás de este artículo es armarte tu propio dashboard, ya valió la pena.

---

## El paralelismo que ayuda a pensarlo

En los 2010s, cuando AWS recién maduraba, hubo un patrón que se repitió en muchas empresas: alguien levantaba una instancia EC2 para probar algo, se olvidaba de bajarla, y a fin de mes alguien preguntaba "¿qué es esto de $4.000 en una región que ni usamos?". Las primeras facturas de cloud eran un shock no porque el cloud fuera caro, sino porque la unidad de costo había cambiado y nadie había actualizado los hábitos.

Compute pasó de "compré un servidor, lo amortizo en 3 años" a "pago por hora-CPU mientras esté prendida". El mismo equipo, con los mismos workloads, podía gastar 5x más o 5x menos según si alguien apagaba la instancia el viernes.

Tokens están en esa fase ahora. Pagábamos planos por seat —al menos en el vendor más usado—, ahora pagamos por uso. La factura va a tener varianzas grandes según hábito, no según workload. Y como en el caso de AWS, la diferencia entre los que ven el cambio venir y los que lo ven después en el dashboard no es 10%, es múltiplos.

La parte buena: ya sabemos cómo termina esta película. Las empresas que sobrevivieron a la transición de cloud no fueron las que volvieron a on-prem. Fueron las que aprendieron a tagear instancias, a apagar lo que no usaban, a elegir el tier correcto para cada workload, a instrumentar todo con métricas, a poner budgets y alertas. **Gastar bien, no gastar menos.**

El grupo Visma ya hizo esa transición en cloud. No hay que reinventar. Las prácticas son las mismas, el activo a vigilar es nuevo.

---

## Qué se puede hacer ya, por nivel de esfuerzo

Tres niveles. Cualquier BU del grupo puede arrancar por el primero esta misma semana.

**Cero esfuerzo (el lunes a la mañana).** Activar Auto Mode en Copilot. Activar `chat.tools.compressOutput.enabled` en VSCode. Activar tool search en VSCode. En Cursor, asegurarse de estar en Auto. En Claude Code, revisar que el `CLAUDE.md` no tenga 500 líneas. Configurar spending limits en cada herramienta donde se consume. Esto es el piso. Ahorro estimado combinado: 15-25% sin tocar workflows.

**Esfuerzo medio (un sprint).** Cada dev se arma su dashboard personal de OpenTelemetry (Claude Code o Copilot Chat, según qué use más). Si una BU tiene una app o feature que llama a una API de modelo, activar prompt caching. Migrar workflows asíncronos (reportes, clasificaciones nocturnas, embeddings, evaluación de prompts en CI) a Batch API: **50% de descuento sin diferencia de calidad**, combinable con caching para **hasta 95% de ahorro**. Instrumentar las llamadas con OpenTelemetry siguiendo GenAI Semantic Conventions. Ahorro estimado: 60-90% en el costo de esa app específica.

**Esfuerzo grande (un trimestre, decisión de arquitectura).** Routing multi-modelo: clasificación a Haiku, código y razonamiento estándar a Sonnet, decisiones complejas a Opus, batch y experimentos a DeepSeek directo (donde no haya tema de datos sensibles) o vía Azure (donde sí). RouteLLM o LiteLLM o Portkey como gateway. **Ahorro reportado: 20-80% del OpEx**, dependiendo del mix actual. Esto no es para todas las BUs, pero para las que ya tienen carga seria de IA en producción es la palanca grande.

---

## Lo que no te puede arreglar ningún proveedor

Estas tres cosas dependen de cada empresa del grupo, y de cada persona dentro de ella. Las pongo en orden de impacto.

**Decidir qué tarea va a qué modelo.** Si todo va al modelo más caro, todo cuesta como el más caro. La factura no la fija el proveedor, la fija el routing. Es decisión de arquitectura, no de herramienta.

**Cortar contexto.** El 62% de basura en la factura no se va con un setting. Se va decidiendo qué información mandar y qué no, qué historia comprimir, cuándo abrir una sesión nueva en vez de seguir la del lunes. Context engineering reduce costos **entre 60% y 80%**. Es una habilidad, no un click.

**Mirar la factura — empezando por la propia.** Suena trivial y es el más importante. Mirar la propia, mirar la del equipo, mirar la de la BU, mirar la del grupo, mirar la trayectoria mensual. **IDC proyecta que las Global 1000 están subestimando sus costos de AI en un 30% hacia 2027.** Eso es exactamente lo que pasa cuando nadie mira. Si no hay alguien con la pregunta "¿qué la está moviendo este mes?", la respuesta siempre va a ser "más uso de IA", y eso no es accionable.

---

## No es una receta. Es un momento.

Quiero ser claro en algo antes de los pasos accionables: nada de lo que escribí acá es "la forma" de trabajar con IA. No estoy diciendo que todos tengan que tener un dashboard de Grafana, ni que todos tengan que migrar features a API directa, ni que todos tengan que hacer routing multi-modelo el viernes. Lo que digo es otra cosa.

**Esta es la forma de entender. Y entender, hoy, es lo que hay que hacer.**

Tuvimos un par de años muy generosos. Aparecieron herramientas nuevas todos los meses, los modelos eran cada vez mejores, los precios bajaban, y los planos flat nos dejaban experimentar sin pensar dos veces. Si activaste extended thinking "por las dudas", si abriste 40 sesiones esta semana, si dejaste un agente corriendo todo el fin de semana —el costo de aprender de eso fue bajo. Llegaba la factura plana, todos seguíamos.

Ese período se está terminando. No de golpe, no para todos, no el 1 de junio. Pero se está terminando. Y la ventana para aprender barato —para experimentar, medir, equivocarse, ajustar, sin que duela— **es exactamente ahora**.

Los que aprovechen esta ventana van a salir del otro lado entendiendo cómo se gobierna el consumo de IA. Van a saber qué setting mueve qué número. Van a tener intuición. Van a ser los que en seis meses, cuando esto sea política de empresa en todas las BUs del grupo, ya tengan la respuesta. **Los que la dejen pasar van a tener que aprender lo mismo, pero con la factura corriendo y menos margen para experimentar.** Es la misma curva, distinto contexto emocional.

Por eso este artículo. No para decirte qué hacer, sino para mostrarte por dónde se empieza a aprender, y para que sepas que el momento de hacerlo es este. Convertirse en buenos en esto —los mejores, incluso— no es magia, es atravesar una curva de aprendizaje. La curva está. El timing también. Lo único que falta es decidir entrar.

---

## Qué hacer esta semana, por rol

**Si sos developer.** Lunes: Auto Mode, compressOutput, tool search. Martes a la tarde: armate tu dashboard personal de OpenTelemetry. Diez minutos, cero euros, te va a cambiar la vista. Miércoles: con el dashboard ya andando, mirá una sesión larga tuya y observá cuánto contexto está arrastrando, qué modelos usaste, cuántos tokens fueron output. Viernes: si tu equipo tiene una feature con API propia, abrí el ticket para activar caching y para instrumentar las llamadas con OTel.

**Si sos tech lead.** Lunes: chequeá que tu BU tiene spending limits configurados en todos los proveedores que usa. Martes: contale al equipo cómo armarse el dashboard personal. La gobernanza colectiva empieza por la individual. Miércoles: hacé una hora con el equipo para decidir cuáles casos siguen como caja negra y cuáles ameritan API. Viernes: si tenés algo en API sin OpenTelemetry, ponelo en el roadmap del próximo sprint.

**Si tomás decisiones de presupuesto en una BU.** Lunes: pedí el dashboard de uso de los últimos 90 días por herramienta. Miércoles: identificá quién es el responsable nombrado de mirar la trayectoria mensual. Si nadie, asignalo. Viernes: agendá la conversación con finance para mostrarle el cambio de junio antes de que ellos te lo pregunten en julio.

**Si estás mirando el grupo desde arriba.** Esto le toca a alguien. La factura consolidada del grupo en IA va a cambiar de forma este año. Las BUs que vean el cambio venir van a salir bien paradas. Las que no, van a explicar en julio por qué la línea de AI tools subió 3x.

**Gastar bien, no gastar menos.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: usuarios que entienden lo que consumen, equipos que monitorean lo que pagan, BUs que eligen entre caja negra y API con criterio. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La trae el grupo. Y empieza, literal, en cada dev que se arma su dashboard un martes a la tarde. No para hacer todo distinto. Para entender lo que ya está haciendo. Esa es la curva que vale la pena atravesar ahora, mientras todavía sale barata.

---

## Para profundizar

Los datos, fuentes y multiplicadores citados en este artículo están en `data/verified-metrics.md` del repo, con URL a cada fuente original. La documentación técnica (`PROD/technical-docs-draft-v2.md`) tiene el detalle por capa, con ejemplos de código, casos de aplicación y de no-aplicación. El documento sobre el descuento del hyperscaler y el caso GDPR/DeepSeek está en `PRODV2/hyperscaler-discount-vs-gdpr-ES.md`.

Para los detalles de OpenTelemetry en Claude Code, la documentación oficial está en `code.claude.com/docs/en/agent-sdk/observability`. Para Copilot Chat, en `code.visualstudio.com/docs/copilot/guides/monitoring-agents`. Para las GenAI Semantic Conventions estándar (usadas tanto por Anthropic como por OpenAI), en `opentelemetry.io/docs/specs/semconv/gen-ai/`.
