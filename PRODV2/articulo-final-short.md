# Tokens, factura, y una conversación que se nos venía (versión corta)

El 1 de junio GitHub Copilot cambia su modelo de cobro. En Visma es uno de los vendors de IA más adoptados, así que para muchas BUs este es el primer mes en el que la pregunta "¿cuánto consumimos en tokens?" deja de ser teórica. Este artículo cuenta qué cambia, qué no, y por dónde se empieza a optimizar. La versión larga, con todas las objeciones y el detalle implementacional, está en `articulo-final.md` (Link al artículo extendido) y la documentación técnica en `doc-tecnico-final.md` (Link al documento técnico).

---

## Por qué este artículo, y por qué ahora

El 1 de junio Copilot deja de cobrarse plano y pasa a usage-based billing. Cada seat sigue costando lo mismo, pero ahora trae AI Credits incluidos: mientras estás dentro de esos créditos no se cobra extra; cuando se acaban, se paga por consumo.

Pero conviene decirlo desde el principio: **el resto del stack ya funcionaba así**. Cursor cobra por planes con cuota y overage desde hace meses. Claude (Desktop, Code, Cowork) lo mismo. Las APIs directas de Anthropic, OpenAI, Azure OpenAI, Bedrock siempre fueron pay-per-token desde el día uno. Lo único que cambia el 1 de junio es Copilot.

La pregunta "¿cuánto nos cuesta cada token que consumimos?" debería haber estado dando vueltas en nuestras cabezas desde hace bastante. Si no estaba, fue porque el flat fee de Copilot nos tapaba la vista en la parte del stack donde más consumimos, y porque en el resto el consumo era todavía chico o estaba sumido dentro de presupuestos más grandes que nadie auditaba. Ese período se termina.

**Usar IA bien no es solo prompt engineering, es saber consumir menos tokens para obtener el mismo resultado.** Esa capacidad, igual que el FinOps de cloud hace 10 años, se construye con hábito, no con una herramienta mágica. El 1 de junio no inventa el problema, lo hace visible.

Idea que va a aparecer varias veces: **gastar bien, no gastar menos**. No estamos buscando achicar la IA. Estamos buscando que cada euro que se pone en IA esté trabajando, no calentando aire.

---

## Lo que está pasando con el dinero (y nadie lo está mirando)

En 30 equipos relevados entre marzo y mayo de 2026, **el 62% de la factura de IA no es por trabajo del modelo**. Es por mandar el mismo contexto, una y otra vez. De cada 100 euros que una empresa pone en IA, 62 son literalmente el mismo system prompt, la misma documentación de proyecto, el mismo historial, viajando ida y vuelta hasta el modelo. Es como si un courier te cobrara cada vez que mueve una caja y vos lo hicieras pasar 50 veces por la misma esquina porque te olvidaste de pedirle que la dejara en el destino.

Lo concreto: una sesión típica empieza en 5.000 tokens. Para el turno 50 está en 200.000. Nadie lo ve. No hay un cartel que diga "estás a 40x del costo de cuando empezaste". La factura llega a fin de mes y se atribuye a "más uso de IA".

Hasta ahora esto no nos pegaba directo porque en Copilot pagábamos plano y en el resto el consumo todavía no era el suficiente como para que alguien levantara la mano. Eso cambia el 1 de junio. Y la pregunta operativa, para cualquier dev, equipo o BU, pasa a ser una: ¿cuánto de tu factura es trabajo real, y cuánto es el courier dando vueltas?

---

## El espectro: productos cerrados, herramientas con configuración, API directa

Es la distinción más importante del artículo. Una idea muy difundida dice que el mundo de la IA se divide en dos: las herramientas (cajas negras) y las APIs (transparentes). Esa división era cierta hace año y medio. Hoy no. Tenemos un espectro de tres niveles.

**Productos cerrados.** Aplicaciones tipo chat sin instrumentación pensada para el usuario: ChatGPT versión consumer, Gemini, Claude.ai en planes Free/Pro. Lo único que ves es el chat, la respuesta, alguna cuota mensual. No tenés model picker, no podés modificar parámetros, no hay telemetría que se pueda exportar.

**Herramientas con configuración.** Productos que sí permiten controlar bastante: Copilot dentro de VSCode (el caso más común en Visma), Claude Code, Cursor, Claude Cowork en planes Team o Enterprise. Acá podés elegir modelo, activar telemetría, configurar settings que mueven el costo (Auto Mode, compresión de output, control de MCPs), monitorear consumo desde el propio IDE. El caching lo hace la herramienta por vos.

**API directa.** Vos escribís el código que llama al modelo: Anthropic API directa, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Control total: prompt exacto, contexto, modelo, parámetros, caching explícito, métricas por llamada. Es donde viven los descuentos más grandes (caching al 90%, batch al 50%) pero requiere construir.

Una BU puede estar en los tres niveles simultáneamente. Eso no es problema, es la realidad. La regla simple para diseñar nuevas implementaciones: **herramientas con configuración para uso humano** (un dev escribiendo código), **API para uso programático** (una feature de producto que llama un modelo sin humano del otro lado).

---

## Tres cosas que conviene saber antes de tocar nada

Son las objeciones más frecuentes que escuché esta semana liderando esta conversación. Si entendés estas tres, ya tenés el 80% del valor de este artículo.

### Prompt caching: la palanca más grande del sistema

El mismo contenido enviado dos veces no cuesta dos veces. Anthropic cobra los tokens cacheados a un 10% del precio normal, **descuento del 90% en los reads**. El primer envío cuesta 25% extra, pero con que el mismo contenido se lea del cache una sola vez, ya ahorrás.

¿Cómo aplica en cada caso?
- **Claude Code o Claude Desktop:** la herramienta cachea por vos, no tenés que activar nada.
- **Copilot en VSCode:** lo mismo, con cache reuse rate publicado del 93%.
- **API directa:** lo activás vos, hay modo automático y modo explícito con control fino.

Una app típica con system prompt estable y volumen alto, con caching bien activado, puede llegar al **88 a 95% de ahorro** respecto al precio sin cache. Es la diferencia entre tener una factura razonable y una factura que duele.

### El output cuesta 4 a 6x más que el input

Detalle técnico que se subestima mucho. Cuando le pedís a un modelo que te resuma todo lo que hablamos hasta acá, el resumen sale carísimo. Cuando le pedís que reescriba un archivo entero en vez de hacer un diff, también. La asimetría está en cada decisión.

Y un caso particular: si tenés extended thinking activado, **esos tokens se facturan como output aunque no los veas**. Ocultar el razonamiento en el display te ahorra latencia perceptual, no dinero. Una llamada con 2.000 tokens de thinking y 500 de output cuesta 5x más que una sin thinking. Si lo activaste "por las dudas" y nunca lo apagaste, esto te toca.

### El descuento del EA no es ahorro automático

El EA (Enterprise Agreement con Microsoft) negocia un descuento del **15 a 25%**, y con MACC (Microsoft Azure Consumption Commitment) sube a **23 a 28%**. Pero ese descuento aplica sobre el consumo Azure, no sobre la API directa. Y Azure sobre OpenAI directo tiene un overhead de **15 a 40% en TCO** (Total Cost of Ownership).

Hagamos la cuenta. Descuento del 25% sobre un servicio que tiene 30% de overhead. El "descuento" desaparece. El EA sirve, pero no como ahorro automático: sirve como compliance, data residency y consolidación de factura. Y eso, para empresas europeas con GDPR encima, es exactamente lo que estamos comprando. **Pagamos hyperscaler por seguridad jurídica, no por precio**. Está bien que así sea, pero hay que llamarlo por su nombre.

---

## "Pero antes de cobrarme, ¿no me avisa?"

La respuesta corta: **por default no te explotan la factura**. Pero la diferencia entre "no te explotan" y "te alcanza el presupuesto" depende de cómo configures cada herramienta.

La imagen mental de "te cobran sin límite hasta que te llega un susto a fin de mes" es exagerada. Casi todas las herramientas tienen rate limits del proveedor, cuota del plan o spending limit configurable. El comportamiento varía:

- **Claude Pro / Team / Enterprise:** corta limpio cuando se acaba la cuota. No hay overage.
- **Cursor:** tope por plan. Si querés overage tenés que activarlo explícitamente.
- **Copilot bajo el nuevo billing:** AI Credits incluidos, después depende del spending limit que la organización haya configurado.
- **Anthropic API / OpenAI API:** workspace tiene monthly spend limit configurable, con valor por default. Cuando lo alcanzás, devuelve error.
- **Azure OpenAI / AWS Bedrock:** acá hay que poner más atención. La factura va al cloud account general. No es canilla libre (los rate limits frenan antes), pero conviene siempre configurar budget alerts.

**La verdad operativa:** ningún proveedor serio te factura silenciosamente. Lo que sí pasa es que el tope por default puede no coincidir con tu presupuesto real. Por eso el ejercicio es revisar, no entrar en pánico. Este artículo no viene a decirte "te van a hacer mierda la factura". Viene a decirte lo contrario: **usá bien tus tokens así te alcanza con lo que ya pagás**. Y si decidís abrir overage, que sea decisión consciente, con un tope.

Dónde encontrar cada setting en cada plataforma está en la documentación técnica del proyecto, sección 8.3 (Link al documento técnico).

---

## Gobernanza personal: cómo te medís vos mismo

Toda esta conversación arranca en algo mucho más chico: **cada persona que usa IA todos los días puede, y debería, ver su propio consumo**. No el del equipo. No el de la BU. El propio. Si vos no sabés cuántas sesiones abriste esta semana, cuánto contexto repetido mandaste, qué modelos disparaste, no estás haciendo gobernanza, estás adivinando.

La buena noticia: medirse a uno mismo lleva diez minutos de setup y cero euros. Tanto Claude Code como Copilot Chat pueden exportar su telemetría a un archivo local. Con un script chico de Python que lee ese archivo y genera un HTML estático con gráficos, tenés tu dashboard personal funcionando, **todo en tu máquina, sin subir métricas a ningún servicio externo**.

El cómo exacto (variables de entorno, settings, script completo, plantilla HTML) está en la documentación técnica del proyecto, sección 9.6 (Link al documento técnico).

Para Visma, la gobernanza colectiva no se construye desde arriba pidiendo reportes a las BUs. Se construye desde abajo, con cada persona que entiende su propio uso y ajusta. El tech lead que tiene un dashboard del equipo se basa en datos que existen porque la gente del equipo se midió primero. La BU que reporta consumo razonable a finance lo hace porque los devs midieron antes. **Sin el primer paso, lo demás es teatro de presupuesto**.

---

## No es una receta. Es un momento.

Nada de lo que escribí acá es "la forma" de trabajar con IA. No estoy diciendo que todos tengan que tener un dashboard, ni que todos tengan que migrar features a API directa, ni que todos tengan que hacer routing multi-modelo el viernes.

**Esta es la forma de entender. Y entender, hoy, es lo que hay que hacer.**

Tuvimos un par de años muy generosos. Aparecieron herramientas nuevas todos los meses, los modelos eran cada vez mejores, los precios bajaban, y los planos flat nos dejaban experimentar sin pensar dos veces. Si activaste extended thinking "por las dudas", si abriste 40 sesiones esta semana, si dejaste un agente corriendo todo el fin de semana, el costo de aprender de eso fue bajo. Llegaba la factura plana, todos seguíamos.

Ese período se está terminando. No de golpe, no para todos, no el 1 de junio. Pero se está terminando. Y la ventana para aprender barato, para experimentar, medir, equivocarse, ajustar, sin que duela, **es exactamente ahora**.

Los que aprovechen esta ventana van a salir del otro lado entendiendo cómo se gobierna el consumo de IA. Van a saber qué setting mueve qué número. Van a tener intuición. Van a ser los que en seis meses, cuando esto sea política de empresa, ya tengan la respuesta. **Los que la dejen pasar van a tener que aprender lo mismo, pero con la factura corriendo y menos margen para experimentar.** Es la misma curva, distinto contexto emocional.

Por eso este artículo. No para decirte qué hacer, sino para mostrarte por dónde se empieza a aprender, y para que sepas que el momento es este. Convertirse en buenos en esto no es magia, es atravesar una curva de aprendizaje. La curva está. El timing también. Lo único que falta es decidir entrar.

---

## Por dónde empezar esta semana

Tres pasos por nivel de esfuerzo. Cualquiera puede arrancar por el primero hoy mismo.

**Cero esfuerzo, hoy.** Activar Auto Mode en Copilot (10% de descuento automático). Activar la compresión de output del terminal en VSCode. Activar tool search para los MCPs. En Cursor, asegurarse de estar en Auto. Revisá y configurá spending limits donde aplique en tu caso. Ahorro estimado combinado: **15 a 25%** sin tocar workflows.

**Un sprint.** Armate tu dashboard personal con el script local. Si tu equipo tiene una app que llama a una API de modelo, activá prompt caching. Migrá workflows asíncronos a Batch API: 50% de descuento, combinable con caching para hasta 95% de ahorro. Ahorro estimado: **60 a 90%** en el costo de esa app.

**Un trimestre.** Routing multi-modelo: clasificación a Haiku, código a Sonnet, decisiones complejas a Opus, batch barato a DeepSeek vía Azure. RouteLLM o LiteLLM como gateway. **Ahorro reportado: 20 a 80% del OpEx**.

El detalle de cada paso, con los settings exactos, está en la documentación técnica del proyecto (Link al documento técnico).

---

**Gastar bien, no gastar menos.** El cambio del 1 de junio es solo el evento que nos forzó a mirar. Lo que importa es la capacidad que se construye después: usuarios que entienden lo que consumen, equipos que monitorean lo que pagan, BUs que eligen entre herramientas con configuración y API directa con criterio. Esa capacidad no la trae Copilot, ni Cursor, ni Anthropic. La traen las personas que se animan a entender lo que están consumiendo. Y empieza, literal, en cada dev que se arma su dashboard un martes a la tarde.
