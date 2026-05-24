# Tokens, factura, y una conversación que se nos venía

El 1 de junio GitHub Copilot cambia su modelo de cobro. En Visma es uno de los vendors de IA más adoptados, así que para muchas BUs este es el primer mes en el que la pregunta "¿cuánto consumimos en tokens?" deja de ser teórica. Un artículo para entender el cambio, el impacto, y por dónde se empieza a optimizar.

---

## Por qué este artículo, y por qué ahora

El 1 de junio GitHub Copilot deja de cobrarse plano y pasa a usage-based billing. Cada seat sigue costando lo mismo, pero ahora trae AI Credits incluidos: mientras estás dentro de esos créditos no se cobra extra; cuando se acaban, se paga por consumo. Es un cambio que afecta directo a Visma porque Copilot es uno de los vendors de IA más adoptados internamente: una porción importante de nuestro consumo de IA hoy pasa por ahí.

Ese es el disparador. Pero conviene decirlo desde el principio: **el resto del stack ya funcionaba así**. Cursor cobra por planes con cuota y overage desde hace meses. Claude (Desktop, Code, Cowork) lo mismo. Las APIs directas de Anthropic, OpenAI, Azure OpenAI, Bedrock siempre fueron pay-per-token desde el día uno, nunca tuvieron flat fee. Lo único que cambia el 1 de junio es Copilot. Lo demás ya estaba.

Y ahí está la cosa incómoda. La pregunta "¿cuánto nos cuesta cada token que consumimos?" debería haber estado dando vueltas en nuestras cabezas desde hace bastante. Si no estaba, fue porque el flat fee de Copilot nos tapaba la vista en la parte del stack donde más consumimos, y porque en el resto (Claude, Cursor, APIs) el consumo era todavía chico o estaba sumido dentro de presupuestos más grandes que nadie auditaba.

Ese período se termina. Y se termina de la peor manera posible: con el cambio del vendor más usado, justo cuando nadie en Visma está particularmente entrenado en pensar consumo de tokens como una capacidad del usuario. Porque eso es lo que es. **Usar IA bien no es solo prompt engineering, es saber consumir menos tokens para obtener el mismo resultado.** Y esa capacidad, igual que el FinOps de cloud hace 10 años, se construye con hábito, no con una herramienta mágica.

Visto así, **este evento es la excusa correcta para una conversación que ya nos debíamos.** El 1 de junio no inventa el problema, lo hace visible. Y mientras el problema fue invisible, nadie tenía razones para construir la capacidad. Ahora sí.

Este artículo cuenta la historia: qué está pasando, por qué importa, qué optimizaciones existen, dónde están. Es la lectura "para entender". Cuando algún punto te enganche y quieras profundizar en el cómo (los snippets, los settings exactos, los comandos para configurar), todo eso vive en la **documentación técnica del proyecto** (Link al documento técnico). Este texto te dice qué buscar; el técnico te dice cómo hacerlo.

Hay una idea que va a aparecer varias veces: gastar bien, no gastar menos. No estamos buscando achicar la IA. Estamos buscando que cada euro que se pone en IA esté trabajando, no calentando aire.

---

## Lo que está pasando con el dinero (y nadie lo está mirando)

Antes de hablar de qué hacer, vale mostrar dónde está el problema.

En 30 equipos relevados entre marzo y mayo de 2026, **el 62% de la factura de IA no es por trabajo del modelo**. Es por mandar el mismo contexto, una y otra vez. De cada 100 euros que una empresa pone en IA, 62 son literalmente el mismo system prompt, la misma documentación de proyecto, el mismo historial, viajando ida y vuelta hasta el modelo. Es como si un courier te cobrara cada vez que mueve una caja y vos lo hicieras pasar 50 veces por la misma esquina porque te olvidaste de pedirle que la dejara en el destino.

Lo concreto: una sesión típica empieza en 5.000 tokens. Para el turno 50 está en 200.000. Nadie lo ve. No hay un cartel que diga "estás a 40x del costo de cuando empezaste". La factura llega a fin de mes y se atribuye a "más uso de IA".

Hasta ahora esto no nos pegaba directo porque en Copilot pagábamos plano (Copilot Business $19/seat, Enterprise $39/seat) y en el resto de las herramientas el consumo todavía no era el suficiente como para que alguien levantara la mano. Eso cambia el 1 de junio. Y la pregunta operativa, para cualquier dev, equipo o BU, pasa a ser una: ¿cuánto de tu factura es trabajo real, y cuánto es el courier dando vueltas?

---

## El espectro: productos cerrados, herramientas con configuración, API directa

Antes de seguir, una pausa conceptual que vale para gente de todos los niveles técnicos. Es la distinción más importante del artículo.

Hay una idea muy difundida que dice que el mundo de la IA se divide en dos: las herramientas (cajas negras) y las APIs (transparentes). Esa división era cierta hace año y medio. Hoy no. Lo que tenemos es un espectro de tres niveles, y entender dónde está parada cada cosa cambia bastante el análisis de costo y de control.

**Productos cerrados.** Aplicaciones tipo chat sin instrumentación pensada para el usuario: ChatGPT versión consumer, Gemini, Claude.ai en planes Free/Pro. Lo único que ves es el chat, la respuesta, alguna cuota mensual del plan. No tenés model picker, no podés modificar parámetros, no hay telemetría que se pueda exportar. Cualquier optimización seria depende de lo que el vendor decida exponer, que es típicamente nada.

**Herramientas con configuración.** Productos que sí permiten controlar bastante: Copilot dentro de VSCode (el caso más común en Visma), Claude Code, Cursor, Claude Cowork en planes Team o Enterprise. Acá podés elegir modelo, activar telemetría, configurar settings que mueven el costo (Auto Mode, compresión de output, control de MCPs), monitorear consumo desde el propio IDE. El caching lo hace la herramienta por vos. No tenés acceso a las opciones más profundas pero sí a una buena cantidad de optimización accesible.

**API directa.** Vos escribís el código que llama al modelo: Anthropic API directa, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Control total: prompt exacto, contexto, modelo, parámetros como temperature/top_p/top_k, caching explícito, métricas por llamada, costos por token. Es donde viven los descuentos más grandes (caching al 90%, batch al 50%) pero requiere construir.

Una BU puede estar en los tres niveles simultáneamente: herramientas con configuración para que los devs usen Copilot, API para una feature de producto que llama LLM por debajo, y quizás algún equipo informal usando ChatGPT consumer. Eso no es problema, es la realidad. Lo que importa es saber **dónde está cada caso** y no asumir que todo lo que no es API es un producto cerrado.

La regla simple para diseñar nuevas implementaciones: **herramientas con configuración para uso humano** (un dev escribiendo código, alguien explorando ideas), **API para uso programático** (una feature de producto que llama un modelo sin humano del otro lado). Los productos cerrados son un problema sobre todo cuando se usan para casos que ameritan otra cosa: si tu equipo está usando ChatGPT consumer para tareas que requieren observabilidad, gobernanza o repetibilidad, ahí hay algo para revisar.

---

## Las objeciones que ya escuché tres veces esta semana

Liderar esta conversación dentro de Visma me dejó una colección de respuestas. Vale la pena visitarlas, porque las preguntas son las mismas en todas las BUs.

### "Yo uso Cursor o Claude Code, esto no me toca"

Te toca igual, aunque por una razón distinta. El 1 de junio no cambia ni Cursor ni Claude, esos ya estaban en planes con cuota y overage desde antes. Lo que cambia es que **antes nadie miraba el consumo en ninguna parte del stack**, porque Copilot era flat y el resto era todavía chico. Ahora que la línea más gorda del presupuesto (Copilot) pasa a usage-based, las BUs van a empezar a mirar todo. Y cuando empiecen a mirar Cursor o Claude se van a encontrar exactamente los mismos vicios: sesiones eternas, contexto repetido, modelos caros para tareas chiquitas.

Lo concreto para vos: las técnicas que mueven la aguja son las mismas. Mantené sesiones cortas, abrí una nueva cuando la conversación cambió de tema, sacá lo que el modelo no necesita ver, elegí el modelo más chico que sirva para la tarea. En Cursor eso se traduce en usar Auto mode y no forzar Claude Opus para todo. En Claude Code, en cuidar el archivo de contexto del proyecto y no dejar que crezca sin freno.

### "¿Cómo funciona prompt caching y para qué sirve?"

Acá está una de las optimizaciones más grandes del sistema. La idea es simple: **el mismo contenido enviado dos veces no cuesta dos veces**. Anthropic cobra los tokens cacheados a un 10% del precio normal, descuento del 90% en los reads. El primer envío (cache write) cuesta un poquito más, 25% extra, pero con que el mismo contenido se lea del cache una sola vez, ya ahorrás. A partir del segundo hit, el ahorro se multiplica.

¿Qué significa esto en la práctica?

- Si usás **Claude Code o Claude Desktop**, no tenés que activar nada. La herramienta cachea por vos. Tu archivo de contexto del proyecto, el system prompt, los archivos que cargás, todo va al cache automáticamente. Lo que vos podés hacer es mantener estables tus prompts y la estructura de archivos abiertos para favorecer hits; cada vez que cambiás demasiado, invalidás cache.
- Si usás **Copilot en VSCode**, lo mismo. La herramienta tiene un cache reuse rate publicado del 93% dentro del editor. Está funcionando aunque no lo veas.
- Si **construís con API directa**, ahí lo activás vos. Hay un modo automático (un solo campo en tu request y listo) y un modo explícito con control fino sobre qué cachear y qué no.

Una app típica con system prompt estable y volumen alto, con caching bien activado, puede llegar al **88 a 95% de ahorro** respecto al precio sin cache. Es la diferencia entre tener una factura razonable y una factura que duele.

El detalle implementacional (cómo activar cada modo, cómo verificás que el cache está pegando, qué pasa en Bedrock y Vertex) está en la documentación técnica del proyecto, sección 5.1 (Link al documento técnico).

### "Mi sesión no es tan larga"

Eso es lo que pensaba yo también. Después miré.

Una conversación normal de 20 turnos arrastra entre **5.000 y 10.000 tokens innecesarios** que se podrían haber dejado afuera. Eso es por conversación. Multiplicalo por la cantidad de conversaciones por día. Multiplicalo por el equipo. Multiplicalo por las empresas de Visma.

Y un detalle técnico que se subestima mucho: **el output cuesta 4 a 6x más que el input**. No es un detalle menor. Cuando le pedís a un modelo que te resuma todo lo que hablamos hasta acá, el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

### "Tengo extended thinking ocultado en el display. No se ve, no cuenta"

Cuenta. Cuenta como output token. Que no se muestre no significa que no se facture. Ocultar el razonamiento te ahorra latencia perceptual, no dinero.

Una llamada con 2.000 tokens de thinking y 500 de output cuesta **5x más** que una sin thinking. Si activaste extended thinking porque "siempre da mejores respuestas" y nunca lo apagaste, esto te toca. Pensar es caro. Decidir cuándo vale la pena es trabajo del que prompt-ea, no del modelo.

### "Tenemos Microsoft Enterprise Agreement, ya tenemos descuento"

Lo tenemos. El EA (Enterprise Agreement, el contrato marco corporativo con Microsoft) negocia un descuento del **15 a 25%**, y con MACC (Microsoft Azure Consumption Commitment, el compromiso plurianual de gasto en Azure que habilita descuentos adicionales) sube a **23 a 28%**. Pero ese descuento aplica sobre el consumo Azure, no sobre la API directa. Y Azure sobre OpenAI directo tiene un overhead de **15 a 40% en TCO** (Total Cost of Ownership, el costo real total sumando support, egress, monitoring y todo lo que no figura en la pricing page).

Hagamos la cuenta. Descuento del 25% sobre un servicio que tiene 30% de overhead sobre la alternativa directa. El "descuento" desaparece. No es que el EA no sirva, sirve, pero no como ahorro automático. Sirve como compliance, data residency y consolidación de factura. Y eso, para empresas europeas con GDPR encima como las de Visma, **es exactamente lo que estamos comprando.** Pagamos hyperscaler por seguridad jurídica, no por precio.

El caso extremo es DeepSeek vía Azure: **+35% de markup** sobre el precio directo, pero el precio directo no es opción porque los servidores están en China. Acá el descuento del EA compra compliance, no ahorro. Está bien que así sea, **pero hay que llamarlo por su nombre.**

### "Esto se va a resolver solo con modelos más baratos"

En parte, sí. Pero ya hay una técnica que da el **95% de la calidad de GPT-4 con el 26% de las llamadas al modelo fuerte**. Se llama RouteLLM. Con data augmentation, baja al **14% de llamadas al modelo fuerte y 75% de ahorro**.

El **37% de las enterprises ya corre 5+ modelos en producción**. IDC proyecta **70% para 2028**. El ahorro promedio por model routing inteligente es **20 a 80% del OpEx**.

Esto ya está. No es futuro. La pregunta no es "¿llegarán modelos más baratos?", es "¿estamos usando los baratos que ya existen para las tareas que no necesitan los caros?".

---

## "Pero antes de cobrarme, ¿no me avisa? ¿No se bloquea solo?"

Esta es la pregunta más importante del artículo, y la que más gente me hizo. La respuesta corta, después de revisarla con cuidado, es: **por default no te explotan la factura. Pero la diferencia entre "no te explotan" y "te alcanza el presupuesto" depende de cómo configures cada herramienta.**

Acá hay un mito que conviene desarmar primero. La imagen mental de "te cobran sin límite hasta que te llega un susto a fin de mes" es exagerada. La realidad es más matizada. Casi todas las herramientas tienen alguno de estos tres mecanismos:

1. **Rate limit del proveedor.** Un techo duro de requests o tokens por minuto que el vendor te aplica por default. No lo configurás vos. Te frena antes de cualquier explosión, devolviéndote un error tipo "rate limit exceeded".
2. **Cuota del plan.** Lo que pagaste te da X consumo. Cuando se acaba, te devuelve "limit reached" y tenés que esperar al reset (mensual típicamente) o pagar más.
3. **Spending limit configurable.** Vos definís cuánto estás dispuesto a gastar arriba del plan. Si lo dejás en cero (el default razonable), no hay overage. Si lo subís, ahí empieza a haber riesgo controlado por vos.

**Cómo se comporta cada herramienta por default:**

- **Claude Pro / Team / Enterprise (Desktop, Code, Cowork):** te corta limpio cuando se acaba la cuota del plan. No hay overage. Esperás al reset o subís de plan.
- **Cursor:** mismo modelo. Tope por plan. Si querés overage tenés que activarlo explícitamente desde settings de la org.
- **GitHub Copilot bajo el nuevo billing:** trae AI Credits incluidos en el seat. Cuando se acaban, lo que pase depende del **spending limit** de la organización. Por default suele venir en cero (corta y listo). Si el admin lo subió, consume hasta ese tope y después corta.
- **Anthropic API directa:** tu workspace tiene un **monthly spend limit** configurable desde la consola. Mientras no lo toques, hay un valor por default (no es infinito). Cuando lo alcanzás, te devuelve error. Aparte de eso, los rate limits por minuto te paran cualquier corrida descontrolada antes de que la factura crezca.
- **OpenAI API directa:** mismo modelo. Dashboard de Settings → Limits trae un **monthly budget** y alertas configurables.
- **Azure OpenAI / AWS Bedrock:** acá es donde hay que poner más atención. La factura va al cloud account general. No es "canilla libre" (los rate limits del modelo y las cuotas del subscription te frenan antes), pero sí podés terminar pagando más de lo previsto si no configuraste **budget alerts** en Azure Cost Management o en AWS Budgets. La buena práctica es siempre configurarlos.

**La verdad operativa entonces es esta:** ningún proveedor serio te factura silenciosamente sin avisar. Lo que sí pasa es que **el tope por default puede no coincidir con tu presupuesto real**. Por eso el ejercicio es revisar, no entrar en pánico.

Lo que hay que hacer **no es una receta uniforme**. Cada BU de Visma tiene su mix de proveedores, sus planes, sus convenios con corporate. Es un ejercicio de revisar y configurar:

- **Revisá tu situación específica.** ¿Qué herramientas usás? ¿Bajo qué plan? ¿El billing está centralizado en una organización o cada equipo paga aparte? Esa información determina dónde tenés que ir a ver, y a veces revela que la conversación es con IT o con el admin de la organización, no algo que vos toques directo.
- **Confirmá el comportamiento en tu caso.** En cada consola, mirá: ¿el plan corta solo cuando se acaba la cuota o permite overage? ¿Hay spending limit configurado? ¿Hay alertas de budget? El default razonable es: cuota dura por plan, sin overage. Si querés overage para no quedarte sin servicio a mitad de mes, activalo con un tope explícito que tenga sentido para tu presupuesto.
- **Revisá de nuevo en un mes.** Las herramientas cambian settings, los planes se actualizan, los contratos se renegocian.

Este artículo no te viene a decir "te van a hacer mierda la factura". Te viene a decir lo contrario: **usá bien tus tokens así te alcanza con lo que ya pagás**. Y si en algún caso decidís abrir overage porque no querés quedarte sin servicio en una semana crítica, que sea una decisión consciente, con un tope, y sabiendo cuánto estás dispuesto a poner. La canilla no está abierta sola, pero podés abrirla vos sin querer si no mirás.

Dónde encontrar cada setting en cada plataforma (rutas de la consola, formato de los budget alerts, valores recomendados) está en la documentación técnica del proyecto, sección 8.3 (Link al documento técnico).

---

## Gobernanza personal: cómo te medís vos mismo

Acá llega la parte concreta del artículo. Porque toda esta conversación sobre factura, BUs y empresas arranca en algo mucho más chico: **cada persona que usa IA todos los días puede, y debería, ver su propio consumo.** No el del equipo. No el de la BU. El propio.

No te lo propongo como obligación ni como única forma. Te lo propongo como **la forma más rápida de empezar a entender lo que consumís**.

Si vos no sabés cuántas sesiones abriste esta semana, cuánto contexto repetido mandaste, qué modelos disparaste, cuántos tokens de input contra tokens de output (que cuestan 4 a 6x más), entonces no estás haciendo gobernanza, estás adivinando. Y la buena noticia es que **medirse a uno mismo en 2026 lleva diez minutos de setup y cero euros**, si usás las herramientas correctas.

### Las herramientas con configuración ya no son "ciegas"

Antes de entrar en cómo, una corrección importante a algo que se dice mucho. Las herramientas IDE no son productos cerrados. Son herramientas con configuración real: pagás por plan, no por token, pero **exponen OpenTelemetry de fábrica**. Lo que cambia es quién instrumenta: vos en lugar del vendor.

Herramientas que sí exponen OpenTelemetry hoy:

- **Claude Code:** soporte oficial. Cinco variables de entorno en tu shell y empieza a exportar métricas y eventos a cualquier backend OTLP, incluyendo un archivo local en tu propia máquina.
- **Claude Cowork (Claude Desktop) en planes Team y Enterprise** desde la versión 1.1.4173 en adelante.
- **GitHub Copilot Chat:** settings nativos en VSCode. Exporta traces, métricas y eventos siguiendo GenAI Semantic Conventions (un estándar abierto del propio OpenTelemetry). Soporta también exportar a archivo local.
- **Codex CLI (OpenAI):** exporta logs estructurados y métricas OTel.

La que todavía no lo expone de forma nativa es **Cursor**. Tiene Admin Dashboard con API propia y exports CSV en Enterprise, pero no OTel out-of-the-box. Si te urge, hay hooks de comunidad (cursor-otel-hook, CursorLens como proxy) que cierran el gap.

La regla actualizada: antes de adoptar o profundizar en una herramienta de IA, preguntá si expone OTel. Si lo hace, podés tener observabilidad real sin necesidad de migrar nada a API directa. Si no lo hace, tu visibilidad va a depender del dashboard del vendor.

### Tu dashboard personal en diez minutos, todo en tu máquina

Acá viene la parte concreta. Cualquier dev que use Claude Code o Copilot puede armarse su propio dashboard de consumo en diez minutos, **sin pedirle nada a IT, sin pagar nada extra, sin subir métricas a ningún servicio externo, sin abrir un ticket**.

La idea es esta. Tanto Claude Code como Copilot Chat pueden exportar su telemetría a un archivo local en formato JSONL. Cada línea del archivo es un evento (una sesión, un request, tokens consumidos, modelo usado, costo estimado). Con eso ya tenés todo lo que necesitás para responder las preguntas de gobernanza personal: cuántas sesiones por día, cuánto contexto repetido, qué modelos, cuántos tokens fueron input vs output, cuánto te cuesta.

Lo único que falta es algo que lea ese archivo y te lo muestre lindo. Esto es **un script chico en Python que lee el JSONL, calcula las agregaciones, y genera un HTML estático con gráficos**. Abrís el HTML en el browser, mirás. Cero docker, cero cuenta cloud, cero servidor corriendo, cero compartir métricas con nadie.

El flujo, conceptualmente:

1. Setear las variables de entorno (Claude Code) o el setting JSON (Copilot Chat) para que exporten a un archivo local.
2. Usar las herramientas normalmente durante una semana. El archivo se llena solo, sin que tengas que hacer nada.
3. Correr el script Python (un comando: `python dashboard.py`) que lee el archivo, hace las cuentas, y escribe un `dashboard.html` en la misma carpeta.
4. Abrir el HTML en el browser. Ver el consumo del último período.
5. Repetir cuando quieras una foto actualizada.

Lo más interesante de esto: las métricas no son aproximaciones, son datos exactos sacados del propio CLI. Y nunca salen de tu máquina. El archivo JSONL vive en tu disco, el script lo lee localmente, el HTML se genera localmente, lo mirás localmente.

Si esto te enganchó, el cómo exacto está en la documentación técnica del proyecto, sección 9.6 (Link al documento técnico): las variables de entorno textuales para Claude Code, el JSON de settings para Copilot Chat, el script Python completo, y la plantilla HTML con los gráficos para responder cada una de las preguntas de gobernanza personal.

### Cuando no es para vos, es para tu app

Toda la sección de arriba es para vos usando IA como dev. **Cuando una BU construye una feature de producto que llama LLMs por debajo**, ahí ya no hay vendor que te exporte: tenés que instrumentar el código tuyo. Es ahí donde la API directa con OTel propio es el único camino.

La buena noticia es que Anthropic, OpenAI y los providers grandes ya publican specs de OTel siguiendo las GenAI Semantic Conventions, así que no tenés que inventar nombres. Una feature con cinco endpoints LLM, bien instrumentada, te da: consumo por endpoint, por modelo, por usuario final, por feature de producto. Eso es lo que después convierte una conversación de "subió la factura" en "subió la feature X porque el endpoint Z no tiene caching".

### Por qué importa que cada uno se mida

Esto no es un ejercicio de FinOps. Es algo más directo: **si vos no podés ver tu propio consumo, no podés mejorarlo.** Es la misma lógica de un atleta con su tracker, o de un dev con su profiler. No medirse no es neutral, es ceguera.

Y para Visma, la gobernanza colectiva no se construye desde arriba pidiendo reportes a las BUs. Se construye desde abajo, con cada persona que entiende su propio uso y ajusta. El tech lead que tiene un dashboard del equipo se basa en datos que existen porque la gente del equipo se midió primero. La BU que reporta consumo razonable a finance lo hace porque los devs midieron antes. Sin el primer paso, lo demás es teatro de presupuesto.

Diez minutos de setup. Cero euros de costo. Las preguntas de gobernanza personal contestadas para siempre. Si lo único que sacás de este artículo es armarte tu propio dashboard, ya valió la pena.

---

## El paralelismo que ayuda a pensarlo

En los 2010s, cuando AWS recién maduraba, hubo un patrón que se repitió en muchas empresas: alguien levantaba una instancia EC2 para probar algo, se olvidaba de bajarla, y a fin de mes alguien preguntaba "¿qué es esto de $4.000 en una región que ni usamos?". Las primeras facturas de cloud eran un shock no porque el cloud fuera caro, sino porque la unidad de costo había cambiado y nadie había actualizado los hábitos.

Compute pasó de "compré un servidor, lo amortizo en 3 años" a "pago por hora-CPU mientras esté prendida". El mismo equipo, con los mismos workloads, podía gastar 5x más o 5x menos según si alguien apagaba la instancia el viernes.

Tokens están en esa fase ahora. Pagábamos planos por seat (al menos en el vendor más usado), ahora pagamos por uso. La factura va a tener varianzas grandes según hábito, no según workload. Y como en el caso de AWS, la diferencia entre los que ven el cambio venir y los que lo ven después en el dashboard no es 10%, es múltiplos.

La parte buena: ya sabemos cómo termina esta película. Las empresas que sobrevivieron a la transición de cloud no fueron las que volvieron a on-prem. Fueron las que aprendieron a tagear instancias, a apagar lo que no usaban, a elegir el tier correcto para cada workload, a instrumentar todo con métricas, a poner budgets y alertas. **Gastar bien, no gastar menos.**

Las empresas de Visma ya hicieron esa transición en cloud. No hay que reinventar. Las prácticas son las mismas, el activo a vigilar es nuevo.

---

## Qué se puede hacer ya, por nivel de esfuerzo

Tres niveles. Cualquier BU de Visma puede arrancar por el primero esta misma semana. Cada nivel está cubierto en detalle en la documentación técnica del proyecto.

**Cero esfuerzo (el lunes a la mañana).** Activar Auto Mode en Copilot. Activar la compresión de output del terminal en VSCode. Activar tool search para los MCPs. En Cursor, asegurarse de estar en Auto. En Claude Code, revisar que el archivo de contexto del proyecto no tenga 500 líneas. Revisar y configurar spending limits donde aplique en tu caso. Esto es el piso. Ahorro estimado combinado: **15 a 25%** sin tocar workflows. Todos los settings exactos están en la sección 4 del doc técnico (Link al documento técnico).

**Esfuerzo medio (un sprint).** Cada dev se arma su dashboard personal con el script Python local (Claude Code o Copilot Chat, según qué use más). Si una BU tiene una app o feature que llama a una API de modelo, activar prompt caching. Migrar workflows asíncronos (reportes, clasificaciones nocturnas, embeddings, evaluación de prompts en CI) a Batch API: **50% de descuento sin diferencia de calidad**, combinable con caching para **hasta 95% de ahorro**. Instrumentar las llamadas con OpenTelemetry siguiendo GenAI Semantic Conventions. Ahorro estimado: **60 a 90%** en el costo de esa app específica. Cómo activar cada cosa: secciones 5 y 9 del doc técnico (Link al documento técnico).

**Esfuerzo grande (un trimestre, decisión de arquitectura).** Routing multi-modelo: clasificación a Haiku, código y razonamiento estándar a Sonnet, decisiones complejas a Opus, batch y experimentos a DeepSeek directo (donde no haya tema de datos sensibles) o vía Azure (donde sí). RouteLLM o LiteLLM o Portkey como gateway. **Ahorro reportado: 20 a 80% del OpEx**, dependiendo del mix actual. Esto no es para todas las BUs, pero para las que ya tienen carga seria de IA en producción es el ajuste grande. Secciones 6, 7 y 8 del doc técnico cubren los detalles (Link al documento técnico).

---

## Lo que no te puede arreglar ningún proveedor

Estas tres cosas dependen de cada empresa de Visma, y de cada persona dentro de ella. Las pongo en orden de impacto.

**Decidir qué tarea va a qué modelo.** Si todo va al modelo más caro, todo cuesta como el más caro. La factura no la fija el proveedor, la fija el routing. Es decisión de arquitectura, no de herramienta.

**Cortar contexto.** El 62% de basura en la factura no se va con un setting. Se va decidiendo qué información mandar y qué no, qué historia comprimir, cuándo abrir una sesión nueva en vez de seguir la del lunes. Context engineering reduce costos **entre 60% y 80%**. Es una habilidad, no un click.

**Mirar la factura, empezando por la propia.** Suena trivial y es el más importante. Mirar la propia, mirar la del equipo, mirar la de la BU, mirar la trayectoria mensual. **IDC proyecta que las Global 1000 están subestimando sus costos de AI en un 30% hacia 2027.** Eso es exactamente lo que pasa cuando nadie mira. Si no hay alguien con la pregunta "¿qué la está moviendo este mes?", la respuesta siempre va a ser "más uso de IA", y eso no es accionable.

---

## No es una receta. Es un momento.

Quiero ser claro en algo antes de los pasos accionables: nada de lo que escribí acá es "la forma" de trabajar con IA. No estoy diciendo que todos tengan que tener un dashboard, ni que todos tengan que migrar features a API directa, ni que todos tengan que hacer routing multi-modelo el viernes. Lo que digo es otra cosa.

**Esta es la forma de entender. Y entender, hoy, es lo que hay que hacer.**

Tuvimos un par de años muy generosos. Aparecieron herramientas nuevas todos los meses, los modelos eran cada vez mejores, los precios bajaban, y los planos flat nos dejaban experimentar sin pensar dos veces. Si activaste extended thinking "por las dudas", si abriste 40 sesiones esta semana, si dejaste un agente corriendo todo el fin de semana, el costo de aprender de eso fue bajo. Llegaba la factura plana, todos seguíamos.

Ese período se está terminando. No de golpe, no para todos, no el 1 de junio. Pero se está terminando. Y la ventana para aprender barato, para experimentar, medir, equivocarse, ajustar, sin que duela, **es exactamente ahora**.

Los que aprovechen esta ventana van a salir del otro lado entendiendo cómo se gobierna el consumo de IA. Van a saber qué setting mueve qué número. Van a tener intuición. Van a ser los que en seis meses, cuando esto sea política de empresa en todas las BUs de Visma, ya tengan la respuesta. **Los que la dejen pasar van a tener que aprender lo mismo, pero con la factura corriendo y menos margen para experimentar.** Es la misma curva, distinto contexto emocional.

Por eso este artículo. No para decirte qué hacer, sino para mostrarte por dónde se empieza a aprender, y para que sepas que el momento de hacerlo es este. Convertirse en buenos en esto, los mejores, incluso, no es magia, es atravesar una curva de aprendizaje. La curva está. El timing también. Lo único que falta es decidir entrar.

---

## Qué hacer esta semana, por rol

**Si sos developer.** Lunes: activar Auto Mode, compresión de output y tool search en VSCode. Martes a la tarde: armate tu dashboard personal con el script local. Diez minutos, cero euros, te va a cambiar la vista. Miércoles: con el dashboard ya andando, mirá una sesión larga tuya y observá cuánto contexto está arrastrando, qué modelos usaste, cuántos tokens fueron output, qué hit rate de caching tenés. Viernes: si tu equipo tiene una feature con API propia, abrí el ticket para activar caching y para instrumentar las llamadas con OTel.

**Si sos tech lead.** Lunes: revisá qué herramientas usa tu BU y dónde aplican spending limits. Confirmá que ninguna está operando con default = "sin tope" sin saberlo. Martes: contale al equipo cómo armarse el dashboard personal. La gobernanza colectiva empieza por la individual. Miércoles: hacé una hora con el equipo para decidir en qué punto del espectro (productos cerrados, herramientas con configuración, API directa) están parados los distintos casos de uso. Viernes: si tenés algo en API sin OpenTelemetry, ponelo en el roadmap del próximo sprint.

**Si tomás decisiones de presupuesto en una BU.** Lunes: pedí el dashboard de uso de los últimos 90 días por herramienta. Miércoles: identificá quién es el responsable nombrado de mirar la trayectoria mensual. Si nadie, asignalo. Viernes: agendá la conversación con finance para mostrarle el cambio de junio antes de que ellos te lo pregunten en julio.

**Si estás mirando Visma desde arriba.** Esto le toca a alguien. Las facturas de IA en las empresas de Visma van a cambiar de forma este año. Las BUs que vean el cambio venir van a salir bien paradas. Las que no, van a explicar en julio por qué la línea de AI tools subió 3x.

**Gastar bien, no gastar menos.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: usuarios que entienden lo que consumen, equipos que monitorean lo que pagan, BUs que eligen entre herramientas con configuración y API directa con criterio. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La traen las personas que se animan a entender lo que están consumiendo. Y empieza, literal, en cada dev que se arma su dashboard un martes a la tarde. No para hacer todo distinto. Para entender lo que ya está haciendo. Esa es la curva que vale la pena atravesar ahora, mientras todavía sale barata.

---

## Para profundizar

Cada uno de los puntos técnicos mencionados en este artículo (prompt caching, OpenTelemetry, Batch API, routing multi-modelo, spending limits por vendor, dashboards personales con script local, observabilidad enterprise) está cubierto en detalle en la **documentación técnica del proyecto** (Link al documento técnico). Ahí vas a encontrar los snippets exactos, las variables de entorno textuales, los settings de IDE, el script Python completo del dashboard personal, las matrices de decisión de procurement, y la referencia completa de métricas verificadas con sus fuentes originales.

La idea es que este artículo te ayude a entender **qué pasa, por qué, y dónde están los puntos de optimización**. La documentación técnica te ayuda a **implementarlas en tu caso específico**. Las dos lecturas son complementarias: la una sin la otra queda incompleta.
