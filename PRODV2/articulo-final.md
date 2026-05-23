# Tokens, factura, y una conversación que se nos venía

El 1 de junio GitHub Copilot cambia su modelo de cobro. En Visma es uno de los vendors de IA más adoptados, así que para muchas BUs este es el primer mes en el que la pregunta "¿cuánto consumimos en tokens?" deja de ser teórica. Un artículo para entender el cambio, el impacto, y por dónde se empieza a optimizar.

---

## Por qué este artículo, y por qué ahora

El 1 de junio GitHub Copilot deja de cobrarse plano y pasa a usage-based billing. Cada seat sigue costando lo mismo, pero ahora trae AI Credits incluidos: mientras estás dentro de esos créditos no se cobra extra; cuando se acaban, se paga por consumo. Es un cambio que afecta directo a Visma porque Copilot es uno de los vendors de IA más adoptados internamente: una porción importante de nuestro consumo de IA hoy pasa por ahí.

Ese es el disparador. Pero conviene decirlo desde el principio: **el resto del stack ya funcionaba así**. Cursor cobra por planes con cuota y overage desde hace meses. Claude (Desktop, Code, Cowork) lo mismo. Las APIs directas de Anthropic, OpenAI, Azure OpenAI, Bedrock —siempre fueron pay-per-token desde el día uno, nunca tuvieron flat fee. Lo único que cambia el 1 de junio es Copilot. Lo demás ya estaba.

Y ahí está la cosa incómoda. La pregunta "¿cuánto nos cuesta cada token que consumimos?" debería haber estado dando vueltas en nuestras cabezas desde hace bastante. Si no estaba, fue porque el flat fee de Copilot nos tapaba la vista en la parte del stack donde más consumimos, y porque en el resto —Claude, Cursor, APIs— el consumo era todavía chico o estaba sumido dentro de presupuestos más grandes que nadie auditaba.

Ese período se termina. Y se termina de la peor manera posible: con el cambio del vendor más usado, justo cuando nadie en Visma está particularmente entrenado en pensar consumo de tokens como una capacidad del usuario. Porque eso es lo que es. **Usar IA bien no es solo prompt engineering, es saber consumir menos tokens para obtener el mismo resultado.** Y esa capacidad, igual que el FinOps de cloud hace 10 años, se construye con hábito, no con una herramienta mágica.

Visto así, **este evento es la excusa correcta para una conversación que ya nos debíamos.** El 1 de junio no inventa el problema, lo hace visible. Y mientras el problema fue invisible, nadie tenía razones para construir la capacidad. Ahora sí.

Este artículo cuenta la historia: qué está pasando, por qué importa, qué palancas existen, dónde están. Es la lectura "para entender". Cuando algún punto te enganche y quieras profundizar en el cómo —los snippets, los settings exactos, los comandos para configurar—, todo eso vive en la **documentación técnica del proyecto** (`PRODV2/doc-tecnico-final.md`). Este texto te dice qué buscar; el técnico te dice cómo hacerlo.

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

Antes de seguir, una pausa conceptual que vale para gente de todos los niveles técnicos. Es la distinción más importante del artículo.

Hay una idea muy difundida que dice que el mundo de la IA se divide en dos: las herramientas (caja negra) y las APIs (transparentes). Esa división era cierta hace año y medio. Hoy no. Lo que tenemos es un espectro de tres niveles, y entender dónde está parada cada cosa cambia bastante el análisis de costo y de control.

**Caja negra opaca.** Productos consumer sin instrumentación pensada para el usuario: ChatGPT versión consumer, Gemini consumer, Claude.ai en planes Free/Pro. Lo único que ves es el chat, la respuesta, alguna cuota mensual del plan. No tenés model picker, no hay telemetría exportable, no podés modificar parámetros.

**Caja gris configurable.** Herramientas enterprise o IDEs con visibilidad, model picker, telemetría OpenTelemetry y settings ricos. Acá entran Copilot en VSCode/Visual Studio, Claude Code, Claude Cowork en planes Team/Enterprise. Tenés bastante control: podés elegir modelo, activar telemetría, configurar settings que mueven el costo. El caching lo hace la herramienta por vos. Cursor está en este grupo pero todavía no expone OTel nativo —tiene Admin Dashboard y exports CSV en Enterprise, y hay hooks de comunidad que cierran el gap.

**API Key.** Vos escribís el código que llama al modelo: Anthropic API directa, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Control total: prompt exacto, contexto, modelo, parámetros, caching, métricas por llamada, costos por token. Es donde viven los descuentos más grandes (caching al 90%, batch al 50%) pero requiere construir.

Una BU puede estar en los tres niveles simultáneamente: caja gris para que los devs usen Copilot, API para una feature de producto que llama LLM por debajo, y quizás algún equipo informal usando ChatGPT consumer. Eso no es problema —es la realidad. Lo que importa es saber **dónde está cada caso** y no asumir que todo lo que no es API es caja negra opaca.

La regla simple para diseñar nuevas implementaciones: **caja gris para uso humano** (un dev escribiendo código, alguien explorando ideas), **API para uso programático** (una feature de producto que llama un modelo sin humano del otro lado). Las cajas negras opacas son un problema sobre todo cuando se usan para casos que ameritan otra cosa: si tu equipo está usando ChatGPT consumer para tareas que requieren observabilidad, gobernanza o repetibilidad, ahí hay algo para revisar.

---

## Las objeciones que ya escuché tres veces esta semana

Liderar esta conversación dentro de Visma me dejó una colección de respuestas. Vale la pena visitarlas, porque las preguntas son las mismas en todas las BUs.

### "Yo uso Cursor o Claude Code, esto no me toca"

Te toca igual, aunque por una razón distinta. El 1 de junio no cambia ni Cursor ni Claude —esos ya estaban en planes con cuota y overage desde antes. Lo que cambia es que **antes nadie miraba el consumo en ninguna parte del stack**, porque Copilot era flat y el resto era todavía chico. Ahora que la línea más gorda del presupuesto (Copilot) pasa a usage-based, las BUs van a empezar a mirar todo. Y cuando empiecen a mirar Cursor o Claude se van a encontrar exactamente los mismos vicios: sesiones eternas, contexto repetido, modelos caros para tareas chiquitas.

Lo concreto para vos: las técnicas que mueven la aguja son las mismas. Mantené sesiones cortas, abrí una nueva cuando la conversación cambió de tema, sacá lo que el modelo no necesita ver, elegí el modelo más chico que sirva para la tarea. En Cursor eso se traduce en usar Auto mode y no forzar Claude Opus para todo. En Claude Code, en cuidar el archivo de contexto del proyecto y no dejar que crezca sin freno.

### "¿Cómo funciona prompt caching y para qué sirve?"

Acá está una de las palancas más grandes del sistema. La idea es simple: **el mismo contenido enviado dos veces no cuesta dos veces**. Anthropic cobra los tokens cacheados a un 10% del precio normal —descuento del 90% en los reads. El primer envío (cache write) cuesta un poquito más, 25% extra, pero con que el mismo contenido se lea del cache una sola vez, ya ahorrás. A partir del segundo hit, el ahorro se multiplica.

¿Qué significa esto en la práctica?

- Si usás **Claude Code o Claude Desktop**, no tenés que activar nada. La herramienta cachea por vos. Tu archivo de contexto del proyecto, el system prompt, los archivos que cargás —todo va al cache automáticamente. Lo que vos podés hacer es mantener estables tus prompts y la estructura de archivos abiertos para favorecer hits; cada vez que cambiás demasiado, invalidás cache.
- Si usás **Copilot en VSCode**, lo mismo. La herramienta tiene un cache reuse rate publicado del 93% dentro del editor. Está funcionando aunque no lo veas.
- Si **construís con API directa**, ahí lo activás vos. Hay un modo automático (un solo campo en tu request y listo) y un modo explícito con control fino sobre qué cachear y qué no.

Una app típica con system prompt estable y volumen alto, con caching bien activado, puede llegar al **88-95% de ahorro** respecto al precio sin cache. Es la diferencia entre tener una factura razonable y una factura que duele.

El detalle implementacional —cómo activar cada modo, cómo verificás que el cache está pegando, qué pasa en Bedrock y Vertex— está en la documentación técnica del proyecto, sección 5.1.

### "Mi sesión no es tan larga"

Eso es lo que pensaba yo también. Después miré.

Una conversación normal de 20 turnos arrastra entre **5.000 y 10.000 tokens innecesarios** que se podrían haber dejado afuera. Eso es por conversación. Multiplicalo por la cantidad de conversaciones por día. Multiplicalo por el equipo. Multiplicalo por las empresas de Visma.

Y un detalle técnico que se subestima mucho: **el output cuesta 4-6x más que el input**. No es un detalle menor. Cuando le pedís a un modelo que te resuma todo lo que hablamos hasta acá, el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

### "Tengo extended thinking ocultado en el display. No se ve, no cuenta"

Cuenta. Cuenta como output token. Que no se muestre no significa que no se facture. Ocultar el razonamiento te ahorra latencia perceptual, no dinero.

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

Cada herramienta funciona distinto. GitHub Copilot bajo el nuevo billing no tiene stopper por default —el overage se factura sin límite a menos que el admin de la organización configure un spending limit. Anthropic API directa lo mismo: cobra hasta que llegue el límite de la tarjeta o el workspace limit que vos hayas configurado. OpenAI API igual. Azure OpenAI y Bedrock van directo a la factura cloud, sin tope, salvo que armes budget alerts. Cursor permite "additional usage" si lo activás. Solo los planes Claude Desktop/Code te cortan cuando se acaba la cuota mensual.

Lo que hay que hacer con esto **no es una receta uniforme**. Cada BU de Visma tiene su mix de proveedores, sus planes, sus convenios con corporate. Es un ejercicio de **revisar y configurar**:

- **Revisá tu situación específica.** ¿Qué herramientas usás? ¿Bajo qué plan? ¿El convenio con Visma corporate cubre todo o algunos seats están fuera? ¿El billing está centralizado o cada equipo paga aparte? Esa información determina dónde tenés que ir a poner un límite, y a veces revela que no tenés permisos para hacerlo vos —que la conversación es con IT o con el admin de la organización.
- **Configurá lo que corresponda a tu situación.** Si la herramienta soporta spending limit y aplica a tu caso, ponelo. Si soporta alertas de budget, configuralas. Si no soporta ninguna de las dos, asegurate de que alguien esté mirando la trayectoria del consumo manualmente. Lo importante no es que todos hagan exactamente lo mismo —es que **nadie esté operando con default = "sin tope"** sin saberlo.
- **Revisá de nuevo en un mes.** Las herramientas cambian settings, los planes se actualizan, los contratos se renegocian. El stopper no es un evento, es una práctica.

Una nota sobre cómo leer esto: **el stopper no es una solución, es una alarma de incendio.** Si tu factura llega al tope, algo se rompió antes. La pregunta no es "¿cuándo nos cortan?", es "¿quién está mirando la trayectoria y avisando antes de que el tope se acerque?". Si nadie tiene ese rol explícito en tu BU, ese es el primer puesto que hay que crear o asignar. No es un rol full-time, son 2 horas al mes mirando un dashboard. Pero tiene que ser de alguien.

Dónde mirar exactamente en cada plataforma (rutas de la consola, settings, formato de los budget alerts) está en la documentación técnica del proyecto, sección 8.3.

---

## Gobernanza personal: cómo te medís vos mismo

Acá llega la parte concreta del artículo. Porque toda esta conversación sobre factura, BUs y empresas arranca en algo mucho más chico: **cada persona que usa IA todos los días puede —y debería— ver su propio consumo.** No el del equipo. No el de la BU. El propio.

No te lo propongo como obligación ni como única forma. Te lo propongo como **la forma más rápida de empezar a entender lo que consumís**.

Si vos no sabés cuántas sesiones abriste esta semana, cuánto contexto repetido mandaste, qué modelos disparaste, cuántos tokens de input contra tokens de output —que cuestan 4-6x más—, entonces no estás haciendo gobernanza, estás adivinando. Y la buena noticia es que **medirse a uno mismo en 2026 lleva diez minutos de setup y cero euros**, si usás las herramientas correctas.

### Caja gris ya no significa "ciego"

Antes de entrar en cómo, una corrección importante a algo que se dice mucho. Las herramientas IDE no son cajas negras opacas. Son cajas grises configurables: pagás por plan, no por token, pero **exponen OpenTelemetry de fábrica**. Lo que cambia es quién instrumenta: vos en lugar del vendor.

Herramientas que sí exponen OpenTelemetry hoy:

- **Claude Code:** soporte oficial. Cinco variables de entorno en tu shell y empieza a exportar métricas y eventos a cualquier backend OTLP.
- **Claude Cowork (Claude Desktop) en planes Team y Enterprise** desde la versión 1.1.4173 en adelante.
- **GitHub Copilot Chat:** settings nativos en VSCode. Exporta traces, métricas y eventos siguiendo GenAI Semantic Conventions (un estándar abierto del propio OpenTelemetry).
- **Codex CLI (OpenAI):** exporta logs estructurados y métricas OTel.

La que todavía no lo expone de forma nativa es **Cursor**. Tiene Admin Dashboard con API propia y exports CSV en Enterprise, pero no OTel out-of-the-box. Si te urge, hay hooks de comunidad (cursor-otel-hook, CursorLens como proxy) que cierran el gap.

La regla actualizada: antes de adoptar o profundizar en una herramienta de IA, preguntá si expone OTel. Si lo hace, podés tener observabilidad de grado enterprise sin necesidad de migrar nada a API directa. Si no lo hace, tu visibilidad va a depender del dashboard del vendor.

### Tu dashboard personal en diez minutos

Acá viene la parte concreta. Cualquier dev que use Claude Code o Copilot puede armarse su propio dashboard de consumo en **diez minutos**, sin pedirle nada a IT, sin pagar nada extra, sin abrir un ticket.

La idea es esta: las herramientas exportan telemetría OpenTelemetry. Vos necesitás un backend que la reciba y la grafique. Para uso personal hay dos caminos limpios:

- **Grafana Cloud** (free tier): te registrás gratis, te dan un endpoint OTLP, lo configurás en tu shell, y ya estás mandando métricas a un Grafana que vive en la nube. Mirás tu dashboard desde el browser. Cero infraestructura local.
- **Aspire Dashboard local** (un solo container Docker): si preferís que nada salga de tu máquina, levantás el container de Aspire Dashboard que Microsoft publica gratis. Te da un trace viewer con endpoint OTLP built-in. Apuntás Claude Code ahí y listo.

Una vez que el dashboard está vivo, las preguntas que podés contestar son exactamente las que importan para gobernanza personal: cuántas sesiones por día, cuánto contexto repetido estás reenviando, qué modelos disparaste y en qué proporción, cuántos tokens fueron input vs output, cuánto te cuesta cada día.

Lo más interesante de esto: no son aproximaciones, son datos exactos sacados del propio CLI. Y vienen del vendor —no es algo construido por arriba.

Si esto te enganchó, el cómo exacto está en la documentación técnica del proyecto, sección 9.6: las variables de entorno textuales para Claude Code, el JSON de settings para Copilot Chat, los pasos para levantar Grafana Cloud o Aspire Dashboard, y cómo armar las queries para responder cada una de las preguntas de arriba.

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

Tokens están en esa fase ahora. Pagábamos planos por seat —al menos en el vendor más usado—, ahora pagamos por uso. La factura va a tener varianzas grandes según hábito, no según workload. Y como en el caso de AWS, la diferencia entre los que ven el cambio venir y los que lo ven después en el dashboard no es 10%, es múltiplos.

La parte buena: ya sabemos cómo termina esta película. Las empresas que sobrevivieron a la transición de cloud no fueron las que volvieron a on-prem. Fueron las que aprendieron a tagear instancias, a apagar lo que no usaban, a elegir el tier correcto para cada workload, a instrumentar todo con métricas, a poner budgets y alertas. **Gastar bien, no gastar menos.**

Las empresas de Visma ya hicieron esa transición en cloud. No hay que reinventar. Las prácticas son las mismas, el activo a vigilar es nuevo.

---

## Qué se puede hacer ya, por nivel de esfuerzo

Tres niveles. Cualquier BU de Visma puede arrancar por el primero esta misma semana. Cada nivel está cubierto en detalle en la documentación técnica del proyecto.

**Cero esfuerzo (el lunes a la mañana).** Activar Auto Mode en Copilot. Activar la compresión de output del terminal en VSCode. Activar tool search para los MCPs. En Cursor, asegurarse de estar en Auto. En Claude Code, revisar que el archivo de contexto del proyecto no tenga 500 líneas. Revisar y configurar spending limits donde aplique en tu caso. Esto es el piso. Ahorro estimado combinado: **15-25%** sin tocar workflows. Todos los settings exactos están en la sección 4 del doc técnico.

**Esfuerzo medio (un sprint).** Cada dev se arma su dashboard personal de OpenTelemetry (Claude Code o Copilot Chat, según qué use más). Si una BU tiene una app o feature que llama a una API de modelo, activar prompt caching. Migrar workflows asíncronos (reportes, clasificaciones nocturnas, embeddings, evaluación de prompts en CI) a Batch API: **50% de descuento sin diferencia de calidad**, combinable con caching para **hasta 95% de ahorro**. Instrumentar las llamadas con OpenTelemetry siguiendo GenAI Semantic Conventions. Ahorro estimado: **60-90%** en el costo de esa app específica. Cómo activar cada cosa: secciones 5 y 9 del doc técnico.

**Esfuerzo grande (un trimestre, decisión de arquitectura).** Routing multi-modelo: clasificación a Haiku, código y razonamiento estándar a Sonnet, decisiones complejas a Opus, batch y experimentos a DeepSeek directo (donde no haya tema de datos sensibles) o vía Azure (donde sí). RouteLLM o LiteLLM o Portkey como gateway. **Ahorro reportado: 20-80% del OpEx**, dependiendo del mix actual. Esto no es para todas las BUs, pero para las que ya tienen carga seria de IA en producción es la palanca grande. Secciones 6, 7 y 8 del doc técnico cubren los detalles.

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

**Si sos developer.** Lunes: activar Auto Mode, compresión de output y tool search en VSCode. Martes a la tarde: armate tu dashboard personal de OpenTelemetry. Diez minutos, cero euros, te va a cambiar la vista. Miércoles: con el dashboard ya andando, mirá una sesión larga tuya y observá cuánto contexto está arrastrando, qué modelos usaste, cuántos tokens fueron output, qué hit rate de caching tenés. Viernes: si tu equipo tiene una feature con API propia, abrí el ticket para activar caching y para instrumentar las llamadas con OTel.

**Si sos tech lead.** Lunes: revisá qué herramientas usa tu BU y dónde aplican spending limits. Configurá lo que corresponda en tu caso. Martes: contale al equipo cómo armarse el dashboard personal. La gobernanza colectiva empieza por la individual. Miércoles: hacé una hora con el equipo para decidir en qué punto del espectro (caja negra opaca, caja gris, API) están parados los distintos casos de uso. Viernes: si tenés algo en API sin OpenTelemetry, ponelo en el roadmap del próximo sprint.

**Si tomás decisiones de presupuesto en una BU.** Lunes: pedí el dashboard de uso de los últimos 90 días por herramienta. Miércoles: identificá quién es el responsable nombrado de mirar la trayectoria mensual. Si nadie, asignalo. Viernes: agendá la conversación con finance para mostrarle el cambio de junio antes de que ellos te lo pregunten en julio.

**Si estás mirando Visma desde arriba.** Esto le toca a alguien. La factura agregada de las empresas de Visma en IA va a cambiar de forma este año. Las BUs que vean el cambio venir van a salir bien paradas. Las que no, van a explicar en julio por qué la línea de AI tools subió 3x.

**Gastar bien, no gastar menos.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: usuarios que entienden lo que consumen, equipos que monitorean lo que pagan, BUs que eligen entre caja gris y API con criterio. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La traen las empresas de Visma. Y empieza, literal, en cada dev que se arma su dashboard un martes a la tarde. No para hacer todo distinto. Para entender lo que ya está haciendo. Esa es la curva que vale la pena atravesar ahora, mientras todavía sale barata.

---

## Para profundizar

Cada uno de los puntos técnicos mencionados en este artículo (prompt caching, OpenTelemetry, Batch API, routing multi-modelo, spending limits por vendor, dashboards personales, observabilidad enterprise) está cubierto en detalle en la **documentación técnica del proyecto**: `PRODV2/doc-tecnico-final.md`. Ahí vas a encontrar los snippets exactos, las variables de entorno textuales, los settings de IDE, las queries de PromQL para los dashboards, las matrices de decisión de procurement, y la referencia completa de métricas verificadas con sus fuentes originales.

La idea es que este artículo te ayude a entender **qué pasa, por qué, y dónde están las palancas**. La documentación técnica te ayuda a **implementarlas en tu caso específico**. Las dos lecturas son complementarias: la una sin la otra queda incompleta.
