# El descuento del 25% que no es del 25%
## Por qué los hyperscalers no abaratan los LLMs, pero sí compran compliance: el caso GDPR + DeepSeek vía Azure

**Audiencia:** Tech leads, architects y decisores de presupuesto (CTOs, líderes de ingeniería, finance partners) que están evaluando o renegociando un acuerdo con Microsoft, AWS o Google para consumir LLMs en producción.
**Prerequisito:** Familiaridad con pricing de LLMs y con la estructura básica de un Enterprise Agreement.
**Ubicación:** Documento complementario al `technical-docs-draft-v2.md`, profundiza Sección 3 (procurement) y Sección 8.2 (DeepSeek vía Azure) de esa guía.

---

## Resumen ejecutivo (90 segundos de lectura)

En la mesa de negociación de un Enterprise Agreement con Microsoft (y, con matices, también con AWS y Google) suele aparecer una promesa: *"contraten Azure y obtienen entre 15% y 28% de descuento sobre el consumo de IA"*. El número suena bien y por sí solo decide muchas migraciones. **El problema es que ese descuento se calcula sobre el precio nominal por token, y el precio efectivo de consumir un LLM por hyperscaler trae un overhead estructural que se come la mayor parte —o la totalidad— del descuento.**

Los datos coincidentes de cinco fuentes independientes en 2026 ([TokenMix](https://tokenmix.ai/blog/azure-openai-cost), [Inference.net](https://inference.net/content/azure-openai-pricing-explained/), [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/), [EPC Group](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026), [MarginDash](https://margindash.com/azure-openai-pricing-calculator)) muestran que:

1. **El precio nominal por token es idéntico** entre Azure OpenAI y OpenAI directo, entre Bedrock y Anthropic directo, entre Vertex y los respectivos vendors. No hay descuento por canal a nivel token.
2. **El TCO efectivo en Azure es 15–40% más caro** que ir directo al vendor, con un promedio reportado de 22%. La diferencia viene de cinco rubros estructurales: support plans, data egress, hosting de fine-tuned models, networking/VNet, y observabilidad obligatoria.
3. **El descuento EA típico de 15–28%** termina compensando ese overhead, pero rara vez lo supera. La factura final queda en una franja entre **-14% y +12%** comparado con vendor directo, una vez aplicado todo.
4. **Desde noviembre de 2025, Microsoft eliminó los niveles de descuento por volumen B/C/D** del Enterprise Agreement para Online Services. Hoy todos los clientes empiezan en "Level A" (precio de lista). Cualquier descuento es ahora 100% negociado, y la línea base es 6–12% más cara que antes para muchas empresas medianas y grandes ([USCloud, 2026](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/), [SAM Expert, 2026](https://samexpert.com/microsoft-ea-discount-removal-2025/)).

Dicho de otra forma: **"vamos a Azure porque nos hacen 25%" es una conversación de pricing nominal que no se traduce a la factura real**. El hyperscaler puede ser la decisión correcta —pero por otras razones: compliance, procurement unificado, MACC burn, mandato corporativo, o acceso compliance-ready a modelos que de otra forma no serían viables.

Y ahí entra la **excepción que confirma la regla**: el caso DeepSeek + GDPR. DeepSeek V4 es ~15 veces más barato que Claude Sonnet, pero acceder al modelo directamente (api.deepseek.com) implica enviar datos a servidores en China —algo prohibido por GDPR para datos de clientes europeos, como dejó claro el Garante italiano cuando bloqueó DeepSeek el 30 de enero de 2025 ([Bird & Bird](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data)). Azure AI Foundry hostea el mismo modelo dentro del Azure boundary con data residency europea ([Microsoft Azure Blog](https://azure.microsoft.com/en-us/blog/deepseek-r1-is-now-available-on-azure-ai-foundry-and-github/)), agrega un markup de 20–35%, y aun así el modelo sigue costando **1/15 lo que cuesta Sonnet**. Acá el markup del hyperscaler no es overhead inevitable: **es prima de compliance**, y compra acceso a un modelo que sin Azure no es legal usar en producción para datos europeos.

La conclusión operativa, en una línea: **el hyperscaler no abarata el token; compra gobernanza y, en algunos casos puntuales, compra acceso a modelos que de otra forma serían inviables.**

---

## 1. Lo que dice el marketing

En cualquier reunión de negociación con un vendor de cloud, en algún momento aparece la diapositiva del descuento. La estructura es siempre parecida: "si firmás Enterprise Agreement, te corresponde X% de descuento sobre todo el consumo de IA dentro de la plataforma".

Los números públicos de referencia para Microsoft Azure, según [Microsoft Negotiations (2026)](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) y [Atonment Licensing (2026)](https://atonementlicensing.com/blog/openai-enterprise-pricing/), son los siguientes:

| Configuración de acuerdo | Descuento típico negociado |
|---|---|
| Azure EA solo | 15–25% |
| Azure EA + renovación M365 | 18–22% |
| Azure EA + Azure MACC commitment | 23–28% |

A esto se le suelen sumar argumentos colaterales: "facturación unificada", "los créditos de Azure absorben el consumo", "un solo proveedor para todo el cloud, simplifica procurement". Todos los argumentos son razonables —y algunos son ciertos— pero el más fuerte en términos de venta, y el que más decisiones empuja, es **el descuento sobre el token**.

Este documento se ocupa de auditar precisamente ese argumento. Los otros (procurement, facturación unificada, compliance) los discutimos al final, pero conviene atacar primero el más fuerte y el más fácil de medir: ¿el descuento se traduce en factura más baja?

---

## 2. El precio nominal por token es idéntico

La primera cosa importante de internalizar antes de entrar al overhead: **el hyperscaler no te da un mejor precio por token**. El modelo cuesta exactamente lo mismo por token independientemente del canal por el que lo consumás. Esto está documentado oficialmente y se puede verificar trivialmente.

Para Claude (Anthropic), los precios per-token son idénticos en todos los canales, según la [documentación oficial de Anthropic](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry):

| Modelo Claude | Anthropic directo | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|---|---|---|---|---|
| Sonnet 4.6 | $3.00 / $15.00 | $3.00 / $15.00 | $3.00 / $15.00 | $3.00 / $15.00 |
| Opus 4.7 | $5.00 / $25.00 | $5.00 / $25.00 | $5.00 / $25.00 | $5.00 / $25.00 |
| Haiku 4.5 | $1.00 / $5.00 | $1.00 / $5.00 | $1.00 / $5.00 | $1.00 / $5.00 |

Para OpenAI vs Azure OpenAI, el dato lo confirman varias fuentes en 2026:

- **EPC Group ([Azure OpenAI vs OpenAI direct API 2026](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026)):** *"Same models (GPT-4o, GPT-4 Turbo, o1), different infrastructure and governance. (...) Per-token pricing is roughly identical."*
- **CloudZero ([OpenAI API Cost in 2026](https://www.cloudzero.com/blog/openai-pricing/)):** *"Pay-as-you-go rates on Azure are generally comparable to OpenAI's direct API."*
- **TokenMix ([Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost)):** *"Token pricing is identical to OpenAI's direct API. The difference is in everything else."*
- **MarginDash ([Azure OpenAI Pricing Calculator 2026](https://margindash.com/azure-openai-pricing-calculator)):** *"Azure OpenAI Service and the direct OpenAI API use the same underlying models with the same per-token pricing. GPT-4o, GPT-4.1, o3, o4-mini, and every other model cost the same whether you call them through Azure or through api.openai.com."*

Esto importa porque **descarta el argumento de pricing nominal**. Nadie está ofreciendo un Claude más barato vía Bedrock que vía Anthropic directo. Nadie está ofreciendo un GPT más barato vía Azure que vía OpenAI directo. Lo que el hyperscaler ofrece es **otra cosa**: descuento sobre tu consumo agregado en su plataforma (vía EA/MACC), pero no sobre el modelo en sí.

La pregunta que sigue, entonces, es: si el precio nominal es el mismo en ambos canales y el hyperscaler te aplica un descuento sobre el total, ¿no debería terminar saliendo más barato? La respuesta es no, y la razón está en el overhead que no aparece en la pricing page del modelo.

---

## 3. Los cinco overheads del hyperscaler

El precio del token es solo uno de los componentes de lo que terminás pagando por consumir un LLM en producción a través de un hyperscaler. Hay otros cinco rubros que se acumulan y que están bien documentados por múltiples fuentes independientes. Los listo en orden aproximado de impacto:

### 3.1 Support plans

El plan básico de Azure (gratuito) **no es elegible para producción enterprise** —solo da acceso a documentación y self-service. Para tener SLAs reales, escalation paths y respuesta humana ante incidentes, hace falta contratar Standard support como mínimo.

Según [TokenMix (2026)](https://tokenmix.ai/blog/azure-openai-cost) e [Inference.net (2026)](https://inference.net/content/azure-openai-pricing-explained/), los rangos son:

- Standard support: **$100/mes** mínimo.
- Professional Direct: **$1.000+/mes**.

Más allá del costo absoluto, el problema es que **Unified Support se cobra como porcentaje del gasto total con Microsoft**. Cualquier aumento futuro de precios en Microsoft 365, Azure o Copilot multiplica automáticamente el costo de support. Es un costo que escala con la factura, no que se mantiene constante ([USCloud, 2026](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/)).

### 3.2 Data egress (transferencia de datos saliente)

Los primeros 100 GB de outbound al mes son gratuitos. A partir de ahí, el cobro es de **$0.087 por GB** según [Inference.net (2026)](https://inference.net/content/azure-openai-pricing-explained/).

En aplicaciones de chat o asistente de bajo volumen este costo es despreciable. Pero en cualquier caso donde el LLM genere outputs largos (generación de código, redacción de documentos, RAG con respuestas extensas) y se multiplique por miles de usuarios o de requests diarios, llega rápido al umbral. Una aplicación con 100.000 outputs largos al día de 500 palabras (~5 KB cada uno) ya genera ~15 GB/día = ~450 GB/mes; los primeros 100 GB son gratis, los 350 GB restantes son $30/mes. Es chico pero acumula —y en aplicaciones más grandes la curva se vuelve no despreciable.

El punto importante no es el monto en sí: es que **en la API directa de OpenAI o Anthropic, este costo no existe**. La transferencia de datos está incluida en el precio del token. Es un cargo que aparece exclusivamente cuando consumís a través del hyperscaler.

### 3.3 Fine-tuned model hosting

Este es probablemente el rubro con peor relación costo/visibilidad. Si entrenás un modelo custom (fine-tune) sobre Azure OpenAI, **te cobran por hora de deployment, independientemente de si lo usás o no**.

Las tarifas, según [Inference.net (2026)](https://inference.net/content/azure-openai-pricing-explained/):

- $1.70/hora hasta $3.00/hora dependiendo del modelo base.
- Eso equivale a entre **$1.224 y $2.160 al mes por modelo** desplegado, sin que nadie lo invoque.

En la práctica eso significa que un fine-tune que se usa 100 veces al día tiene el mismo costo de hosting que uno que se usa 100.000 veces. La consecuencia operativa típica en empresas: equipos que olvidan apagar deployments de experimentos, modelos huérfanos pagados durante meses, y un cargo recurrente difícil de atribuir.

### 3.4 Networking enterprise: VNet, Private Link, Content Filtering

Para casos enterprise donde no se puede exponer el endpoint del LLM a internet pública —es decir, casi cualquier caso de producción serio—, Azure cobra por:

- **VNet integration** y **Private Link**: networking privado entre tu VNet y el endpoint de Azure OpenAI.
- **Managed identity** + **Key Vault** para autenticación.
- **Content filtering** y **Azure AI Content Safety**.
- **Log Analytics** para retención de logs (con costo de ingesta + storage).

Las estimaciones agregadas, según [Inference.net (2026)](https://inference.net/content/azure-openai-pricing-explained/) y [CloudZero (2026)](https://www.cloudzero.com/blog/azure-openai-pricing/): **$200 a $2.000 al mes** según configuración. EPC Group lo estima en ~5–10% adicional sobre el TCO base ([EPC Group, 2026](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026)).

Muchas empresas necesitan estos componentes por compliance —no es "gold plating" ni elección estética. El punto es que estos costos son **parte estructural del TCO en Azure** y rara vez se ven reflejados en las propuestas comerciales iniciales.

### 3.5 Trampa específica: Claude en Foundry no aplica MACC ni credits

Este merece sección aparte porque es uno de los riesgos menos comunicados y más caros del paquete Azure. Cuando se contrata Foundry para usar Claude (o cualquier modelo de terceros) **el consumo se factura como Marketplace de terceros, no como recurso nativo de Azure**.

La implicancia, según [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) y la cobertura de [AZ365.ai (marzo 2026)](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> *"Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."*

Dicho en castellano: **el consumo de Claude en Foundry no descuenta de tus Azure credits ni de tu MACC (Microsoft Azure Consumption Commitment) salvo casos muy específicos**. Se va directo a la tarjeta de crédito asociada a la cuenta. Si tu plan de procurement asumía que el MACC absorbía el gasto en Claude, te vas a llevar una sorpresa cuando llegue la factura.

Casos reales documentados:

- Founder japonés (Leach): **¥237.081** (~$1.600 USD) cargados directo a tarjeta, con credits no aplicados.
- Founder alemán (Bogdan Sevriukov): **€999.60** cargados directo.
- Otro founder japonés: **¥2.000.000** (~$13.000 USD) en un mes.

**Acción concreta para una negociación en curso:** si tu empresa está armando un acuerdo con Microsoft asumiendo que el MACC absorbe el consumo de Claude o de cualquier modelo Marketplace, **pedir aclaración escrita explícita** sobre qué tipos de credits aplican a qué modelos y bajo qué condiciones. Si la respuesta es ambigua o pospuesta, **asumir que no aplican** y modelar el budget en consecuencia.

---

## 4. La matriz neta: descuento menos overhead

Con los componentes claros, el cálculo es directo. Supongamos un workload base de 100 unidades de costo nominal cuando se consume directo del vendor (Anthropic, OpenAI). Los datos coinciden en estos rangos:

| Escenario | Costo directo | Costo via Azure | Neto vs directo |
|---|---|---|---|
| GPT-5.4 sin descuento (Standard support, sin egress alto) | 100 | 115–140 | **+15% a +40%** |
| GPT-5.4 con EA solo (-20% típico) sobre 115–140 | 100 | 92–112 | **-8% a +12%** |
| GPT-5.4 con EA + MACC (-25% típico) sobre 115–140 | 100 | 86–105 | **-14% a +5%** |
| Claude Sonnet via Foundry (sin descuento por credits exclusion) | 100 | 120–135 | **+20% a +35%** |
| Claude Sonnet via Bedrock | 100 | 120–135 | **+20% a +35%** |

**Promedios reportados**, según las tres fuentes principales que midieron el TCO efectivo en 2026:

- **TokenMix:** *"Azure OpenAI = 15-40% premium over OpenAI direct (avg 22%)."* ([TokenMix Azure OpenAI Alternative 2026](https://tokenmix.ai/blog/azure-openai-alternative))
- **Inference.net:** *"Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure."*
- **CloudZero:** *"Production deployments typically add 20-40% above listed token rates."*

Después de aplicar el descuento EA/MACC, la factura queda entre **-14% y +12%** comparada con consumir directo del vendor. **La intuición del documento que originó este pedido es exactamente correcta**: el 25% nominal del hyperscaler se diluye en el overhead y termina saliendo casi lo mismo. En el mejor caso ahorrás 14%; en el peor pagás 12% más.

### 4.1 Excepción: PTU (Provisioned Throughput Units) bien dimensionadas

Un caso donde Azure sí puede salir más barato es cuando se compran **PTUs (Provisioned Throughput Units)** y se las utiliza por encima del 85% de capacidad. Las PTUs son capacidad reservada con tarifa fija mensual (a partir de ~$2.448/mes por unidad, según [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/)) y permiten descuentos de hasta 70% sobre pay-as-you-go cuando la utilización es alta.

El problema, según las mediciones de [TokenMix (2026)](https://tokenmix.ai/blog/azure-openai-alternative): *"TokenMix.ai's analysis shows that most teams overcommit on PTUs by 30-50%, making pay-as-you-go cheaper."*

O sea: la mayoría de los equipos compran más PTUs de las que efectivamente usan, y terminan pagando más que en pay-as-you-go. **PTUs solo justifican el cambio si tenés volumen muy constante y predecible** (típicamente más de 300M tokens/mes según [Inference.net](https://inference.net/content/azure-openai-pricing-explained/)) y podés sostener utilización >85%. Para todo lo demás, pay-as-you-go es más barato.

### 4.2 El cambio de noviembre 2025: ya no hay descuento por volumen automático

Un dato fresco que conviene incorporar al análisis: **a partir del 1 de noviembre de 2025, Microsoft eliminó los niveles de descuento por volumen (Levels A, B, C, D) para Online Services en el Enterprise Agreement, MPSA y OSPA**.

De acuerdo con [SAM Expert (2026)](https://samexpert.com/microsoft-ea-discount-removal-2025/), [Sourcepass (2026)](https://sourcepassmcoe.com/articles/microsoft-ended-enterprise-agreement-discount-levels-sourcepass-mcoe), [USU (2025)](https://www.usu.com/en/blog/microsoft-no-discounts-for-enterprise-agreements) e [ITAA (2025)](https://itaa.com/insights/microsoft-to-end-cloud-volume-discounts-on-november-1-2025/):

- Hasta octubre 2025, organizaciones con más usuarios obtenían descuentos automáticos (Level B/C/D) sobre las listas de precios.
- Desde noviembre 2025, **todos los clientes EA empiezan en Level A**, el precio público de Microsoft.com, sin importar el tamaño.
- Para clientes que antes estaban en Level C o D, el impacto es **+6% a +12% en la factura**, dependiendo de su tier anterior ([USCloud, 2026](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/)).
- Cualquier descuento sigue siendo posible —pero **ahora es 100% negociado, sin descuento automático garantizado**.

La frase de SAM Expert resume bien la situación: *"What had been a guaranteed discount would now have to be negotiated. You would now have to ask for it. If Microsoft gives it to you upfront as a gesture of goodwill, that's what it will be: a gesture of goodwill that can be taken away."*

Esto cambia el cálculo de la negociación: el "15–25% EA solo" que se cita como benchmark histórico hoy es **el techo realista, no la línea base**. Empezás de cero y tenés que justificar cada punto. Para equipos chicos o medianos sin leverage, el descuento real conseguible puede ser bastante menor que ese rango.

---

## 5. La excepción: GDPR + DeepSeek vía Azure

Hasta acá el documento demuestra que el hyperscaler no abarata el consumo de LLMs estándar. Ahora viene la excepción importante, que es justo el caso que motivó este documento: **DeepSeek vía Azure**. Acá el cálculo se da vuelta —no porque el overhead desaparezca, sino porque sin Azure el modelo simplemente no es legal usar para datos europeos.

### 5.1 El modelo barato del mundo y su problema

DeepSeek V4-Flash cuesta aproximadamente **$0.14 por millón de tokens de input** y **$0.28 por millón de output** ([TechJack Solutions, 2026](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/)), lo cual lo convierte en uno de los modelos más baratos del mercado. La comparación con Claude Sonnet es contundente: Sonnet cuesta $3.00 input / $15.00 output, es decir **DeepSeek directo es ~20× más barato en input**.

Pero hay un problema operativo serio. Según la [investigación de Recorded Future News (enero 2025)](https://therecord.media/italy-blocks-chinese-ai-tool-deepseek-over-privacy-concerns) y de [Bird & Bird (2025)](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data):

> *"DeepSeek's privacy policy says the company stores user data on servers located in China."*

Todos los requests al endpoint oficial de DeepSeek se rutean a través de servidores chinos. Para una empresa europea como Visma, enviar datos de clientes, código propietario o información confidencial a servidores en China no es viable: el GDPR exige una base legal explícita para transferencias internacionales de datos personales fuera del EEE, y China no figura entre los países con "adequacy decision" de la Comisión Europea.

### 5.2 El caso Garante: cómo se materializó el problema

El 30 de enero de 2025, **el Garante (autoridad italiana de protección de datos) bloqueó DeepSeek en Italia** después de que la empresa china respondiera de forma "totalmente insuficiente" a las preguntas regulatorias.

La cronología, según [Euronews](https://www.euronews.com/next/2025/01/31/deepseek-ai-blocked-by-italian-authorities-as-others-member-states-open-probes) y [Bird & Bird (2025)](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data):

1. **28 de enero 2025:** Altroconsumo (organización italiana de consumidores) presenta queja formal por presuntas violaciones graves del GDPR. El Garante lanza investigación.
2. **29 de enero 2025:** DeepSeek responde alegando que no opera en Italia, que retiró la app de las stores, y que no está sujeta al GDPR.
3. **30 de enero 2025:** El Garante rechaza la posición de DeepSeek, concluye que ofrece servicios a sujetos italianos y por lo tanto aplica el GDPR, e **impone limitación definitiva al procesamiento de datos personales con efecto inmediato**.

La cita clave del Garante, según el reporte de [MIAI (2026)](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/):

> *"It was confirmed that the Companies unquestionably offer the DeepSeek service to data subjects located in the European Union, more specifically in Italy, and therefore process the personal data of such data subjects."*

Y la pregunta crítica que el Garante hizo y que DeepSeek no respondió adecuadamente: si los datos personales se almacenaban en servidores ubicados en China. Las respuestas de DeepSeek fueron descritas como insuficientes, motivando el bloqueo inmediato.

### 5.3 La ola regulatoria post-Italia

Lo importante no es solo el bloqueo italiano —es que **detonó una cascada regulatoria en múltiples jurisdicciones**. Según [ComplianceHub (febrero 2025)](https://www.compliancehub.wiki/global-ai-regulation-wave-how-italys-deepseek-ban-triggered-a-worldwide-scrutiny-of-chinese-ai-models-germany-netherlands-taiwan/) y [MIAI (2026)](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/):

- **Italia (30 enero 2025):** ban total.
- **Países Bajos (3 febrero 2025):** investigación formal del Autoriteit Persoonsgegevens sobre transferencias a China.
- **Irlanda (29 enero 2025):** la Data Protection Commission solicita información formal.
- **Bélgica:** investigación abierta por queja de consumer group.
- **Luxemburgo (3 febrero 2025):** CNPD emite recomendaciones contra uso por preocupación por transferencias.
- **Francia, España, Portugal:** consultas regulatorias formales.
- **Alemania (Berlin DPA):** investigación.
- **Australia, Corea del Sur, Taiwán:** restricciones para sector público.

Para una empresa que opera en múltiples países europeos —como Visma— el patrón es claro: **acceder a DeepSeek directamente desde producción no es defendible legalmente para cualquier dato sujeto a GDPR**, incluso si tu jurisdicción específica no lo bloqueó aún. El riesgo regulatorio es alto y crece.

### 5.4 La solución: DeepSeek vía Azure AI Foundry

Acá entra el rol específico del hyperscaler. Microsoft anunció oficialmente la disponibilidad de **DeepSeek R1 en Azure AI Foundry el 30 de enero de 2025** ([Microsoft Azure Blog](https://azure.microsoft.com/en-us/blog/deepseek-r1-is-now-available-on-azure-ai-foundry-and-github/)) —exactamente el mismo día del bloqueo italiano, una coincidencia que vale la pena notar.

La cita clave del [Microsoft Security Blog (febrero 2025)](https://www.microsoft.com/en-us/security/blog/2025/02/13/securing-deepseek-and-other-ai-systems-with-microsoft-security/):

> *"Microsoft's hosting safeguards for AI models are designed to keep customer data within Azure's secure boundaries."*

Y más específicamente, en términos de data residency, según [Deep AI Chat (2025)](https://chat-deep.ai/guide/deepseek-r1-on-azure-ai-foundry/):

> *"DeepSeek R1 can be deployed in specific Azure regions or even in special environments like Azure 'Data Zone' regions that ensure data residency (for example, to keep data within the EU or within specific sovereign clouds). (...) Choose a region that meets your compliance requirements – e.g., EU customers might deploy in EU regions for GDPR compliance."*

Lo que cambia con esta configuración:

1. **Los datos quedan en la región Azure elegida.** Si elegís una EU Data Zone (por ejemplo West Europe o North Europe), los inputs y outputs se procesan dentro de la UE.
2. **El contrato es con Microsoft, no con DeepSeek.** Microsoft, como controller/processor según el caso, asume los compromisos de procesamiento bajo el marco de Microsoft Products and Services Data Protection Addendum (DPA), que sí está alineado con GDPR.
3. **El modelo es el mismo.** No es una versión distinta, no es un fork: es DeepSeek R1 (o R1-0528 según el catálogo actual) servido sobre infraestructura Microsoft.

### 5.5 La economía de la prima de compliance

El costo de esta configuración es **20–35% por encima del precio nominal de DeepSeek**, según [DeployBase (2026)](https://deploybase.ai/articles/deepseek-v3-pricing). Aplicado a los precios oficiales:

- DeepSeek directo: $0.14 input / $0.28 output por millón de tokens.
- DeepSeek vía Azure (markup +30% promedio): ~$0.18 input / ~$0.36 output por millón de tokens.

Esto **sigue siendo radicalmente más barato que Claude Sonnet ($3 / $15)**: aproximadamente **15× más barato en input y 40× más barato en output**, según los rangos verificados.

### 5.6 Por qué este caso es distinto al resto del documento

Esta es la parte que conviene internalizar bien, porque parece contradictoria con todo lo anterior: las primeras 4 secciones argumentan que el hyperscaler suma overhead que se come el descuento, y ahora estamos diciendo que vale la pena pagar overhead para usar DeepSeek vía Azure. ¿Cuál es la diferencia?

La diferencia está en **qué compra el markup**:

- **En Azure OpenAI o Bedrock Claude** (Sección 3), el overhead compra cosas que ya tenés disponibles si vas directo al vendor: support, infrastructure, observabilidad. El vendor directo te las ofrece mejor y más barato. El markup es **overhead inevitable de la plataforma**, no agrega capacidad nueva.

- **En DeepSeek vía Azure** (esta sección), el overhead compra algo que **directamente no podés conseguir de otra forma**: data residency europea y un contrato con un controller/processor sujeto a GDPR. DeepSeek directo no te lo va a dar nunca: su privacy policy explícitamente dice que los datos van a China. El markup es **prima de compliance**, y compra acceso legal a un modelo que sin Azure no podrías usar en producción para datos europeos.

Dicho de otra forma: **el markup de Azure OpenAI es overhead; el markup de Azure DeepSeek es prima de compliance**. Son cosas distintas que se ven parecidas en la factura, pero conviene distinguirlas en la conversación de procurement.

### 5.7 Caveat operativo importante

Una nota técnica que conviene tener en cuenta antes de cerrar un acuerdo: **"hosted on Azure" no es automáticamente equivalente a "GDPR compliant"**. Hay configuración específica que validar con legal/compliance antes de mover datos productivos:

1. **Confirmar la región de deployment**: el endpoint debe estar en una región EU (West Europe, North Europe, Sweden Central, etc.), no en una región global o US.
2. **Habilitar Data Zones si están disponibles**: las EU Data Zones de Azure dan garantías reforzadas de procesamiento dentro de la UE.
3. **Revisar y firmar el DPA con Microsoft** específico para el servicio.
4. **Verificar el flujo de logs**: los logs operativos pueden ir a otras regiones por defecto; configurar Log Analytics workspace en EU.
5. **Validar con tu DPO** (Data Protection Officer) o legal antes de production: "hosted on Azure EU" es una afirmación que tu equipo de compliance tiene que poder defender con documentación.

La lección operativa: el setup técnico es relativamente simple, pero la validación legal es no negociable. **Hostear DeepSeek en Azure sin la documentación correcta de compliance es exactamente igual a usar DeepSeek directo, desde la óptica de un regulador.**

---

## 6. Matriz de decisión actualizada

Con todo el análisis, esta es la matriz que conviene tener encima de la mesa cuando se decide canal de consumo. No hay respuesta universal: depende del criterio dominante.

| Criterio dominante | Camino recomendado | Razón |
|---|---|---|
| Modelo del vendor enterprise-maduro (Claude, GPT), sin mandato cloud-only | **Vendor directo** | Features primero, mejor precio neto, soporte de fábrica |
| Modelo del propio vendor + mandato corporativo "todo en Azure" | **Hyperscaler** | Procurement unificado, pero asumir 15–40% overhead como costo de gobernanza |
| Necesidad de MACC burn / facturación unificada Microsoft | **Hyperscaler** | Decisión contable, **excluir Claude en Foundry del MACC** |
| Modelo geopolíticamente sensible (DeepSeek, Qwen) + GDPR obligatorio | **Hyperscaler (Azure EU)** | Prima de compliance vale la pena, modelo ~15× más barato que Sonnet incluso con markup |
| Modelo open-source self-hosted (Llama, Mistral) | **Bedrock / Vertex managed**, o GPUs propias si >$5K/mes y >70% util | Modelo no disponible directo del vendor original con SLA |
| Equipo en exploración / prototipos / POCs | **Vendor directo** | Onboarding rápido, menos burocracia |
| Compliance GDPR / data residency obligatorio, sin restricción de modelo | **Vendor directo si tiene EU regions** (Anthropic, OpenAI ofrecen EU), o **Mistral** (sede Francia, único major europeo) | Evitá hyperscaler si no necesitás procurement unificado |

**Patrón general que emerge:**

- **Hyperscaler como decisión de procurement** (facturación unificada, MACC, mandato corporativo) → válido, pero asumir que el costo neto va a ser **igual o levemente mayor** que vendor directo. No es un ahorro.
- **Hyperscaler como acceso a modelos no-compliance-ready directamente** (DeepSeek y similares) → válido y económico, porque la prima de compliance compra algo real.
- **Hyperscaler buscando "mejor precio"** sobre modelos disponibles directamente → conversación equivocada. El descuento no compensa el overhead.

---

## 7. Inputs concretos para una negociación

Si vas a sentarte con Microsoft, AWS o Google a discutir un acuerdo de consumo de LLMs en 2026, estos son los datos que conviene tener encima de la mesa. Sirven tanto para pedir mejores términos como para detectar promesas exageradas en una propuesta comercial:

| Dato verificado | Valor | Uso en negociación |
|---|---|---|
| Azure overhead vs OpenAI directo | 15–40% (avg 22%) | "Necesitamos descuento que cubra al menos 25% para break-even contra directo" |
| Bedrock Claude overhead vs Anthropic directo | 20–35% efectivo | "Foundry o directo nos da el mismo precio sin ese markup" |
| Pricing parity Claude en hyperscalers | Idéntico | "El hyperscaler no nos da mejor precio nominal — solo procurement" |
| Pricing parity GPT en Azure | Idéntico | "El descuento se aplica sobre el mismo precio nominal que OpenAI directo" |
| Foundry Claude credits exclusion | Documentado | "Aclaración escrita explícita sobre qué credits aplican a qué modelos" |
| Cambio noviembre 2025: fin descuentos por volumen | Confirmado | "El Level C/D que teníamos antes ya no aplica — necesitamos negociar todo desde Level A" |
| Anthropic Enterprise volume discount | Disponible | "Solicitar términos comparables a hyperscaler con MACC" |
| OpenAI Enterprise custom pricing | Disponible | "Solicitar mismo régimen que Azure pero sin overhead" |
| DeepSeek Azure EU Data Zones | Confirmado | "Necesitamos confirmación escrita de región EU y DPA firmado" |
| PTU break-even | >85% util, >300M tokens/mes | "PTUs solo si demostramos volumen sostenido — pay-as-you-go por default" |

Una observación general que aplica a cualquier ronda de negociación: **los vendors no ofrecen lo que no se pide explícitamente**. Si no llevás los datos a la mesa, te van a presentar el descuento nominal y te van a omitir el overhead. El trabajo del comprador informado es traer el dato y forzar la conversación al precio efectivo, no al precio de lista.

---

## 8. Conclusión

La frase de Microsoft Negotiations que abre cualquier conversación sobre Azure y LLMs —*"15 a 28% de descuento"*— es **un descuento nominal sobre un precio que viene con 15–40% de overhead inevitable**. La factura neta queda casi igual, en una franja entre -14% y +12% comparada con ir directo al vendor.

No es que el hyperscaler sea una mala decisión. Es que **es una mala decisión por la razón del precio**. Cuando se elige hyperscaler tiene que ser por razones reales, y esas son:

1. **Procurement unificado**: una sola factura, un solo contrato, un solo proveedor.
2. **MACC burn**: ya tenés compromiso de gasto con Microsoft y necesitás consumirlo.
3. **Compliance específica**: HIPAA BAA, FedRAMP High, FedRAMP Moderate, requisitos sectoriales.
4. **Mandato corporativo "cloud-only on X"**: política corporativa que excluye otros proveedores.
5. **Acceso compliance-ready a modelos que no tienen alternativa directa con GDPR**: y acá entra el caso DeepSeek + Azure, que es la excepción importante.

En el caso específico de **DeepSeek + Azure**, la lógica cambia. El bloqueo italiano del 30 de enero de 2025 y la cascada regulatoria que detonó en al menos 8 jurisdicciones europeas hacen que acceder a DeepSeek directamente para datos GDPR sea inviable. Azure AI Foundry resuelve ese problema agregando un markup de 20–35% sobre el precio directo —que en este caso **no es overhead inevitable de plataforma, sino prima de compliance**: compra algo que sin Azure no tenés (data residency europea + contrato con un controller/processor sujeto a GDPR). Incluso con ese markup, DeepSeek vía Azure sigue siendo aproximadamente **15× más barato que Claude Sonnet** para tareas masivas y mecánicas, y por eso es una pieza interesante del stack de optimización para una empresa europea.

**La frase que resume el documento:**

> El hyperscaler no abarata el token. Compra gobernanza, y en algunos casos puntuales —como DeepSeek + GDPR— compra acceso legal a modelos que de otra forma serían inviables. Para todo lo demás, vendor directo gana en factura efectiva.

---

## Anexo — Métricas verificadas del documento

Todas las cifras citadas en este documento, con su fuente y URL para verificación independiente. Para datos económicos, esta tabla es la referencia primaria; cualquier número que aparezca en negociaciones derivadas debería poder rastrearse hasta acá.

| Métrica | Valor | Fuente | URL |
|---|---|---|---|
| Descuento Azure EA solo (histórico, pre-nov 2025) | 15–25% | Microsoft Negotiations 2026 | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Descuento Azure EA + renovación M365 | 18–22% | Microsoft Negotiations 2026 | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Descuento Azure EA + MACC | 23–28% | Microsoft Negotiations 2026 | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Fin de Levels B/C/D EA Online Services | 1 noviembre 2025 | USU, USCloud, SAM Expert | https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/ |
| Impacto del cambio: +6–12% factura | 6–12% para ex-Level C/D | USCloud, ChessICT | https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/ |
| Ejemplo concreto post-cambio | $2M extra/año org 24K usuarios M365 E5 | SAM Expert 2026 | https://samexpert.com/microsoft-ea-discount-removal-2025/ |
| Azure OpenAI overhead vs OpenAI directo | 15–40% (avg 22%) | TokenMix 2026 | https://tokenmix.ai/blog/azure-openai-cost |
| Azure OpenAI overhead vs OpenAI directo | 15–40% | Inference.net 2026 | https://inference.net/content/azure-openai-pricing-explained/ |
| Azure OpenAI overhead vs OpenAI directo | 20–40% | CloudZero 2026 | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| Azure OpenAI overhead vs OpenAI directo | 5–10% (escenario conservador) | EPC Group 2026 | https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026 |
| Pricing parity Claude Bedrock/Vertex/Foundry vs Anthropic directo | Idéntico nominal | Anthropic docs oficial | https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry |
| Pricing parity GPT Azure vs OpenAI directo | Idéntico nominal | TokenMix, EPC Group, MarginDash, CloudZero | https://margindash.com/azure-openai-pricing-calculator |
| Bedrock Claude overhead efectivo | 20–35% | TokenMix Bedrock 2026 | https://tokenmix.ai/blog/aws-bedrock-pricing |
| Bedrock regional endpoints | +10% sobre global | Anthropic docs | https://platform.claude.com/docs/en/about-claude/pricing |
| Support plan Standard mínimo | $100/mes | TokenMix, Inference.net 2026 | https://tokenmix.ai/blog/azure-openai-cost |
| Support plan Professional Direct | $1.000+/mes | TokenMix 2026 | https://tokenmix.ai/blog/azure-openai-cost |
| Data egress después de 100GB free | $0.087/GB | Inference.net 2026 | https://inference.net/content/azure-openai-pricing-explained/ |
| Fine-tuned model hosting | $1.70–$3.00/hora | Inference.net 2026 | https://inference.net/content/azure-openai-pricing-explained/ |
| Fine-tuned hosting mensual | $1.224–$2.160/mes por modelo | Inference.net 2026 | https://inference.net/content/azure-openai-pricing-explained/ |
| Networking enterprise (VNet, Private Link, etc.) | $200–$2.000/mes | Inference.net, CloudZero 2026 | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| PTU costo base por unidad | ~$2.448/mes | CloudZero 2026 | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| PTU descuento sobre PAYG | hasta 70% si util >85% | CloudZero 2026 | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| PTU overcommit típico equipos | 30–50% | TokenMix 2026 | https://tokenmix.ai/blog/azure-openai-alternative |
| PTU break-even volumen | 300M–500M tokens/mes | Inference.net 2026 | https://inference.net/content/azure-openai-pricing-explained/ |
| Foundry Claude credits exclusion | No aplican Azure credits ni MACC típicamente | Microsoft Q&A | https://learn.microsoft.com/en-us/answers/questions/5851352 |
| Caso documentado Foundry trap (1) | ¥237.081 (~$1.600 USD) tarjeta directa | AZ365.ai marzo 2026 | https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/ |
| Caso documentado Foundry trap (2) | €999.60 tarjeta directa | AZ365.ai marzo 2026 | https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/ |
| Caso documentado Foundry trap (3) | ¥2.000.000 (~$13.000 USD)/mes | AZ365.ai marzo 2026 | https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/ |
| DeepSeek V4-Flash precio oficial | $0.14 input / $0.28 output por MTok | TechJack Solutions 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| DeepSeek vía Azure markup | +20–35% sobre directo | DeployBase 2026 | https://deploybase.ai/articles/deepseek-v3-pricing |
| Garante DeepSeek ban Italia | 30 enero 2025 | Bird & Bird 2025 | https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data |
| DeepSeek privacy: datos en China | Confirmado oficialmente | Recorded Future News | https://therecord.media/italy-blocks-chinese-ai-tool-deepseek-over-privacy-concerns |
| Cascada regulatoria post-Italia | 8+ jurisdicciones EU + global | ComplianceHub feb 2025 | https://www.compliancehub.wiki/global-ai-regulation-wave-how-italys-deepseek-ban-triggered-a-worldwide-scrutiny-of-chinese-ai-models-germany-netherlands-taiwan/ |
| Países con investigación abierta | IT, NL, IE, BE, LU, FR, ES, PT, DE | MIAI 2026, ComplianceHub | https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/ |
| DeepSeek R1 en Azure AI Foundry oficial | Anuncio 30 enero 2025 | Microsoft Azure Blog | https://azure.microsoft.com/en-us/blog/deepseek-r1-is-now-available-on-azure-ai-foundry-and-github/ |
| Microsoft hosting safeguards Azure boundary | Confirmado oficialmente | Microsoft Security Blog feb 2025 | https://www.microsoft.com/en-us/security/blog/2025/02/13/securing-deepseek-and-other-ai-systems-with-microsoft-security/ |
| Azure Data Zones EU para DeepSeek | Disponibles | Deep AI Chat / Microsoft Foundry | https://chat-deep.ai/guide/deepseek-r1-on-azure-ai-foundry/ |
| DeepSeek-R1-0528 actualización | Disponible Azure AI Foundry | Microsoft Tech Community jun 2025 | https://techcommunity.microsoft.com/blog/azure-ai-foundry-blog/deepseek-r1-0528-is-now-available-on-azure-ai-foundry/4420730 |
