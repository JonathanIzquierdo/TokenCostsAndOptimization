# Hyperscaler discount vs overhead — y dónde sí gana Azure: el caso GDPR + DeepSeek

**Audiencia:** Tech leads, architects, decisores de procurement y CFO de Visma
**Tiempo de lectura:** ~20–25 min
**Principio rector:** "Gastar bien, no gastar menos."

---

## Resumen ejecutivo

La idea de comprar Claude o GPT a través de Azure/AWS para "aprovechar el descuento del 25% del Enterprise Agreement" parece evidente sobre el papel y casi siempre falla en la realidad. La razón es simple: el descuento del EA aplica sobre el precio nominal del token, y el precio nominal del token es **idéntico** en el vendor directo y en el hyperscaler para los mismos modelos. El hyperscaler agrega un **15–40% de overhead** (support plans, egress, hosting de modelos fine-tuned, VNet, monitoring), que típicamente neutraliza o supera el descuento negociado. La factura neta después de aplicar descuento al precio inflado queda en una franja de **-14% a +12% vs vendor directo**, no en el -25% que sugiere el marketing.

A esto se suma un cambio relevante de noviembre 2025: Microsoft **eliminó los niveles automáticos de descuento por volumen B/C/D** para Online Services en el EA. Todos los clientes EA pagan ahora el equivalente al Level A (precio de lista). Cualquier descuento adicional es 100% negociado, no programático. Para organizaciones que antes estaban en Level C o D, esto representa un aumento estimado del 6–12% en la factura base — antes de cualquier negociación.

La excepción donde el hyperscaler sí justifica su markup es el caso DeepSeek. El acceso directo a DeepSeek rutea todo el tráfico por servidores chinos, y eso es inviable bajo GDPR: el Garante italiano ya prohibió DeepSeek en enero de 2025, y autoridades de Bélgica, Holanda, Irlanda y Alemania siguieron con investigaciones. Desde febrero de 2026, DeepSeek está disponible en Azure AI Foundry, AWS Bedrock y Google Vertex con EU Data Zones — es decir, con compliance europeo garantizado. El markup sobre el precio directo de DeepSeek es del 20–35%, pero el modelo sigue siendo aproximadamente **15× más barato que Claude Sonnet** para tareas masivas. Acá el hyperscaler no agrega overhead general: agrega **prima de compliance**, y vale la pena.

La regla operativa que se desprende: si el modelo está disponible directamente del vendor con compliance, el hyperscaler suma costo sin agregar valor. Si el modelo en sí no es accesible con compliance, el hyperscaler vale el markup. Es una decisión por modelo, no una decisión global de procurement.

---

## 1. Lo que dice el marketing

La narrativa estándar es conocida. Una conversación enterprise típica con un commercial team de Microsoft Azure suena así: "si pasás tu consumo de OpenAI a Azure dentro del Enterprise Agreement, te aplicamos hasta 25% de descuento sobre el listado." Para una empresa que gasta cientos de miles de dólares al año en LLMs, ese número convierte la conversación en una decisión obvia.

Los números publicados por consultoras especializadas en negociación de licenciamiento Microsoft son consistentes en el rango:

- **Azure EA solo**: 15–25% de descuento negociado sobre Online Services en general — [Microsoft Negotiations 2026](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing).
- **Azure EA + MACC** (Microsoft Azure Consumption Commitment, compromiso plurianual de consumo): 23–28% de descuento — misma fuente.
- **Azure EA + Copilot bundle**: variable, depende del tamaño total del deal.

Esos números son verdaderos como punto de partida de la negociación. El problema no está en los números: está en **sobre qué base** se aplica el descuento. Y la base no es la misma cuando comparás "precio Azure con descuento" contra "precio del vendor directo".

---

## 2. La realidad nominal: el precio por token es el mismo

El primer hecho que rompe la narrativa simple del descuento es que los hyperscalers **no tienen un precio por token más barato que el vendor directo**. Tienen exactamente el mismo precio nominal.

### 2.1 Claude en todos los canales

Anthropic publica oficialmente que mantiene **pricing parity per-token** entre su API directa y los tres hyperscalers principales:

| Modelo | Anthropic directo | AWS Bedrock | Google Vertex | Microsoft Foundry |
|--------|-------------------|-------------|---------------|-------------------|
| Claude Haiku 4.5 | $1.00 / $5.00 | $1.00 / $5.00 | $1.00 / $5.00 | $1.00 / $5.00 |
| Claude Sonnet 4.6 | $3.00 / $15.00 | $3.00 / $15.00 | $3.00 / $15.00 | $3.00 / $15.00 |
| Claude Opus 4.7 | $5.00 / $25.00 | $5.00 / $25.00 | $5.00 / $25.00 | $5.00 / $25.00 |

*Precios por millón de tokens (USD), formato Input/Output. Fuente: [Anthropic Pricing Docs](https://platform.claude.com/docs/en/about-claude/pricing) y [Anthropic — Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry).*

Los únicos modificadores publicados oficialmente son:
- **Bedrock regional endpoints**: +10% sobre el precio global ([Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing)).
- **Vertex regional / multi-region**: +10% sobre global.
- **API directa con `inference_geo: "us"`**: +10% (1.1× multiplier).

Cualquier otro markup que termine apareciendo en la factura corresponde a overhead del proveedor, no al precio del modelo.

### 2.2 OpenAI vs Azure OpenAI

El mismo patrón se repite con OpenAI. Las publicaciones de 2026 son taxativas:

- *"Per-token pricing is roughly identical. Azure adds ~5-10% infrastructure overhead (Private Link, Key Vault, monitoring)."* — [EPC Group, abril 2026](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026).
- *"Token pricing is identical to the direct OpenAI API."* — [CloudZero, mayo 2026](https://www.cloudzero.com/blog/azure-openai-pricing/).
- *"GPT-5.2 leads at $1.75/$14, GPT-4o sits mid-tier at $2.50/$10 — all identical to OpenAI's direct API."* — [TokenMix, mayo 2026](https://tokenmix.ai/blog/azure-openai-cost).

La implicación es directa: **el descuento del EA no se aplica sobre un precio nominal preferencial.** Se aplica sobre el mismo precio que cobra el vendor directo. Si el descuento fuera la única variable en juego, Azure sería más barato. No lo es.

---

## 3. El overhead del hyperscaler: las cinco categorías

Es acá donde la cuenta se da vuelta. El "costo de comprar Claude o GPT a través de un hyperscaler" no es solo el precio del token: es el precio del token **más** un conjunto de costos de infraestructura y servicios que no aparecen en la pricing page del modelo.

Las tres fuentes independientes más recientes sobre Azure OpenAI cuantifican el overhead en el mismo rango:

| Fuente | Rango documentado | Cita textual |
|--------|-------------------|--------------|
| [TokenMix mayo 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15–40%, avg 22% | "Token pricing identical. Total cost runs 15–40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [Inference.net enero 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15–40% | "Enterprise deployments consistently run 15–40% above advertised token costs." |
| [CloudZero mayo 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20–40% | "Production deployments typically add 20–40% above listed token rates." |

El ejemplo concreto que da Inference.net es esclarecedor: un equipo procesando 50M tokens/mes con GPT-4o **esperaba pagar $312,50 y terminó pagando $485,30 — 55% de overhead efectivo** ([Inference.net](https://inference.net/content/azure-openai-pricing-explained/)). El caso está en el extremo alto del rango, pero ilustra cómo se acumula. Las cinco categorías:

### 3.1 Support plans — el más subestimado

Azure ofrece soporte gratuito (Basic) pero solo cubre documentación self-service. Para cualquier despliegue en producción enterprise, **Standard support es de facto obligatorio** y arranca en **$100/mes, escalando hasta $1.000+/mes** según el plan elegido ([CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/), [Inference.net](https://inference.net/content/azure-openai-pricing-explained/)).

Es un costo fijo: no escala con el consumo, pero tampoco desaparece si el consumo baja. Para equipos chicos, $100/mes sobre $500/mes de tokens es un 20% de overhead instantáneo.

### 3.2 Data egress — gratis hasta los 100GB, después se paga

Microsoft no cobra el tráfico inbound (lo que vos mandás a Azure), pero sí cobra el outbound (lo que Azure devuelve) más allá de los primeros 100GB mensuales. La tarifa estándar es **$0,087 por GB** ([Inference.net](https://inference.net/content/azure-openai-pricing-explained/)).

En aplicaciones con outputs largos (generación de código, redacción extensa, RAG con citas), esto se acumula rápido. Para un servicio que produce, digamos, 500GB de output por mes, son ~$35 directos solo en egress.

### 3.3 Fine-tuned model hosting — el más caro de los "costos invisibles"

Este es el ítem que más sorprende cuando aparece por primera vez en la factura. Si entrenás un fine-tuned model, Azure cobra el training por separado (entre $1,50 y $25 por millón de tokens), **pero además te cobra el hosting del modelo entrenado al ritmo de $1,70–$3,00 por hora, esté siendo usado o no** ([Inference.net](https://inference.net/content/azure-openai-pricing-explained/)).

La cuenta neta: un fine-tuned model deployment cuesta **entre $1.224 y $2.160 al mes solo por existir**, sin haber procesado un solo request. Si tu equipo deployó tres fine-tuned models para distintos casos, son $3.600–$6.500/mes en overhead fijo.

Inference.net lo llama explícitamente "la trampa de fine-tuning" — *"the most expensive mistake"*. Y es razonable: muchos equipos arrancan fine-tunings exploratorios, deployan los modelos, los olvidan ahí, y descubren el costo dos meses después.

### 3.4 VNet integration, Private Link y content filtering

Las funcionalidades que justifican comercialmente la elección de Azure (red privada, integración con Entra ID, filtrado de contenido enterprise) tienen su propio costo de infraestructura: **$200–$2.000/mes** según configuración. Son costos legítimos cuando los necesitás por compliance, pero conviene incluirlos en la comparación porque la API directa no los tiene.

### 3.5 Monitoring infrastructure — Log Analytics y CloudWatch equivalente

La observabilidad sobre Azure OpenAI requiere infrastructure de monitoring (Log Analytics, Application Insights) que se factura por GB ingerido y por días de retención. Variable, pero en producción enterprise rara vez baja de $50–200/mes.

En AWS Bedrock el equivalente es CloudWatch + CloudTrail, que para Claude tienen una particularidad adicional: Bedrock factura **todos los errores HTTP 500** ([TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)), mientras que la API directa de Anthropic tiene un *forgiveness buffer* del 3% en errores del servidor que no se cobran. Para servicios con alto throughput, esa diferencia se nota.

### 3.6 La trampa Foundry específica para Claude

Un caso adicional documentado, exclusivo de Microsoft Foundry: Claude se factura en Foundry como **Marketplace de terceros**, no como recurso nativo de Azure. La consecuencia técnica está documentada por [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) y por el análisis de [AZ365.ai marzo 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> *"Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits. This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."*

Traducción operativa: **el consumo de Claude en Foundry no descuenta del MACC ni de los Azure credits que hayas negociado**. Va directo a la tarjeta de crédito de la cuenta. Los casos reales documentados incluyen:

- Founder japonés (Leach): ¥237.081 (~$1.600 USD) cargados directo a tarjeta.
- Founder alemán (Bogdan Sevriukov): €999,60 cargados directo.
- Otro caso documentado: ¥2.000.000 (~$13.000 USD) en un solo mes.

Esta es la "trampa Foundry". Si tu plan de procurement asumía que el MACC absorbía el gasto en Claude vía Azure, la respuesta es que no — y conviene pedir aclaración escrita explícita al commercial team de Microsoft antes de firmar.

---

## 4. La cuenta neta: descuento vs overhead

Una vez que tenés el overhead cuantificado y el descuento cuantificado, la comparación deja de ser "25% off" y pasa a ser una cuenta concreta.

Supongamos un workload base de 100 unidades de costo cuando se consume desde el vendor directo (sea OpenAI, Anthropic, etc.). La progresión típica que muestran las fuentes es:

| Escenario | Costo directo | Costo Azure (precio listado × overhead × descuento) | Neto vs directo |
|-----------|---------------|------------------------------------------------------|------------------|
| Sin compliance, sin descuento EA | 100 | 100 × 1,15–1,40 = **115–140** | **+15% a +40%** |
| Con EA solo (-20% sobre precio Azure) | 100 | 115–140 × 0,80 = **92–112** | **-8% a +12%** |
| Con EA + MACC (-25% sobre precio Azure) | 100 | 115–140 × 0,75 = **86–105** | **-14% a +5%** |
| Claude en Foundry (sin descuento aplicable) | 100 | 120–135 | **+20% a +35%** |

Esto es lo central. Aun con el descuento más agresivo que da Microsoft (EA + MACC, 25% de descuento headline), la factura final queda entre **-14% y +5% respecto del vendor directo**. En el mejor caso ahorrás 14 puntos; en el peor pagás 5 más. Y eso suponiendo que conseguís el descuento máximo y que tu overhead está en el extremo bajo del rango (15%).

Es notablemente distinto del "25% de ahorro" que sugiere el headline number del EA. **El descuento del 25% no se traduce en 25% de ahorro real porque no se está descontando sobre el precio del vendor directo: se está descontando sobre un precio inflado.**

### 4.1 El cambio de noviembre 2025: ya no hay descuento por volumen automático

Una información crítica que muchas presentaciones internas todavía no incorporaron: **desde el 1 de noviembre de 2025, Microsoft eliminó los niveles automáticos de descuento por volumen (A, B, C, D) para Online Services en el Enterprise Agreement**. Todos los clientes EA pagan ahora el equivalente al Level A — el precio de lista publicado en Microsoft.com.

La fuente es Microsoft mismo, recogida por múltiples consultoras especializadas:

- *"Starting November 1, 2025, volume-based price levels for Online Services (Levels B, C, and D) are being eliminated. In practical terms, every EA customer will pay Level A pricing for Microsoft's cloud services, regardless of purchase volume."* — [Redress Compliance](https://redresscompliance.com/microsoft-ea-pricing-changes-2025-all-customers-move-to-level-a-pricing/).
- *"From Level B up, customers could expect 6–12% off the list price. As of November 1, 2025, those volume discounts go away for Online Services."* — [Syskit, septiembre 2025](https://www.syskit.com/blog/microsoft-ending-volume-licensing/).
- *"This move raises costs for many enterprise customers by between 6% and 12%, depending on their current tier, and also inflates Unified Support bills, which are calculated as a percentage of total Microsoft spend."* — [US Cloud, marzo 2026](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/).
- *"This change creates an opportunity to reassess your entire Microsoft licensing strategy."* — [Chess ICT 2025](https://chessict.co.uk/blog/microsoft-enterprise-agreement-changes-what-you-need-to-know-before-november-1st/).

Qué significa esto para una organización mid-market (500–5.000 seats) que antes estaba en Level B, C o D:

1. **El descuento programático ya no existe.** Cualquier descuento debe ser negociado explícitamente cada renovación, sin garantía de obtenerlo.
2. **El "-25%" del marketing es un techo aspiracional, no un piso.** Para muchas organizaciones medianas, el rango realista de descuento negociado quedó más cerca de 10–18%, no de 23–28%.
3. **Para clientes que antes estaban en Level B, C, o D, la factura base aumenta entre 6% y 12% antes de cualquier negociación**, simplemente por la migración forzada a Level A.
4. **Unified Support se calcula como % del spend total**, por lo cual ese aumento del 6–12% se traslada también al soporte — un multiplicador permanente.

La conclusión operativa: cualquier proyección de ahorro basada en descuento histórico del EA conviene revisarla. El mundo donde Microsoft regalaba descuento por volumen se acabó.

### 4.2 Cuándo el hyperscaler sí tiene sentido (a pesar del overhead)

Nada de lo anterior dice que el hyperscaler nunca convenga. Dice que **no conviene por el descuento**. Cuando conviene, conviene por otras razones — y son razones legítimas:

- **Procurement unificado.** Si toda la organización compra cloud a Microsoft, agregar LLMs a la misma factura simplifica la operación contable y reduce la carga administrativa. El overhead del 15–40% es el precio de esa simplicidad.
- **Compliance de datos y red.** VNet integration, Private Link, Entra ID, audit logs centralizados. Si los necesitás por regulación, la API directa no los tiene de fábrica. La diferencia es real.
- **Burn de MACC comprometido.** Si firmaste un MACC plurianual de varios millones y no estás consumiendo lo suficiente para cumplirlo, mover workloads de LLM a Azure te ayuda a burn-down ese compromiso — siempre y cuando el modelo elegido no caiga en la trampa Foundry (Claude no aplica para esto).
- **Modelos no disponibles directamente con compliance europeo.** Y acá entra el caso DeepSeek, que se trata aparte.

Es decir: el hyperscaler es una decisión de **procurement y compliance**, no de precio. El error que el documento entero está marcando es presentar la elección como si fuera de precio.

---

## 5. La excepción que sí funciona: DeepSeek + GDPR via Azure AI Foundry

Hay un caso donde la elección de Azure se justifica por sí sola, sin necesidad de invocar otras razones de procurement. Es el caso DeepSeek — y es importante porque ilustra exactamente cuándo el markup del hyperscaler **no es overhead general, sino prima de compliance**, una distinción crítica para tomar la decisión correcta.

### 5.1 El problema: DeepSeek directo no es GDPR-viable

DeepSeek es, en pricing nominal, uno de los modelos más baratos del mercado. La versión V4-Flash cuesta aproximadamente **$0,14/MTok input y $0,28/MTok output** — entre 15 y 30 veces más barato que Claude Sonnet o GPT-5 para tareas comparables.

El problema es de jurisdicción y data residency. DeepSeek está operado por dos empresas chinas — Hangzhou DeepSeek Artificial Intelligence y Beijing DeepSeek Artificial Intelligence — y todos los requests a la API directa (`api.deepseek.com`) ratean a servidores en China. La política de privacidad oficial reconoce explícitamente almacenamiento de datos en China ([MIAI ai-regulation, febrero 2026](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/)).

Para una empresa europea que procesa datos de clientes, código propietario o información sensible, eso es directamente incompatible con GDPR. No es un tema teórico: hay un track record regulatorio concreto y reciente.

### 5.2 La cronología regulatoria 2025–2026

Los hechos relevantes están bien documentados:

- **28 enero 2025**: la organización italiana de consumidores Altroconsumo (parte de Euroconsumers) presenta una denuncia formal ante el Garante italiano por violaciones de GDPR de DeepSeek. El mismo día, Euroconsumers presenta denuncias paralelas en Bélgica y otros países ([Euroconsumers](https://www.euroconsumers.org/the-full-story-of-deepseek-how-euroconsumers-is-driving-action-for-consumers/)).
- **29 enero 2025**: la DPC irlandesa abre una investigación pidiendo información sobre el procesamiento de datos de residentes irlandeses ([MIAI](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/)).
- **30 enero 2025**: el **Garante italiano impone una limitación definitiva al procesamiento de datos personales de usuarios italianos por parte de DeepSeek**, con efecto inmediato. Las empresas chinas habían respondido el 29 de enero alegando que GDPR no les aplicaba porque no operaban en Italia; el Garante rechazó ese argumento aplicando el Artículo 3(2)(a) de GDPR — extraterritorialidad — porque la web seguía accesible y los usuarios italianos previamente registrados podían seguir usando el servicio ([Bird & Bird](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data), [D'Andrea & Partners](https://www.dandreapartners.com/deepseek-blocked-in-italy-what-happened-legal-consequence%EF%BC%9F/)).
- **30 enero 2025**: la autoridad belga de protección de datos lanza su propia investigación.
- **Febrero 2025**: investigaciones formales adicionales en Holanda y Alemania. Berlín exige a Apple y Google retirar DeepSeek de sus app stores en territorio alemán.
- **14 febrero 2025**: DeepSeek publica una política de privacidad actualizada con un "Supplemental Clause, Jurisdiction-Specific" para EEA/Suiza/UK — más de dos semanas después del ban italiano. El parche reconoce explícitamente almacenamiento en China, pero llega tarde para revertir el bloqueo ([MIAI](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/)).
- **A lo largo de 2025**: investigaciones también en Australia, Corea del Sur, Taiwán. Varios gobiernos prohíben el uso en dispositivos gubernamentales.

La pauta es clara: cualquier empresa europea que use DeepSeek directo está expuesta a doble riesgo. Por un lado, riesgo regulatorio activo, porque las autoridades están investigando y los precedentes ya están sentados; bajo GDPR, controller y processor pueden ser responsabilizados solidariamente, lo cual significa que aunque DeepSeek sea el procesador, la empresa contratante puede ser sancionada como controller ([ComplyDog](https://complydog.com/blog/is-deepseek-gdpr-compliant)). Por el otro, riesgo de continuidad de servicio, ya que si más autoridades EU siguen el camino italiano, el acceso al servicio puede cortarse sin previo aviso, lo cual rompe cualquier aplicación de producción que dependa del modelo.

### 5.3 La solución: DeepSeek via Azure AI Foundry con EU Data Zones

Desde febrero de 2026, los tres hyperscalers principales ofrecen DeepSeek con compliance europeo:

- *"Update February 2026: With availability on AWS Bedrock, Azure AI, and Google Vertex AI in EU regions, enterprises can now use DeepSeek GDPR-compliant in the cloud. Cloud Hosting (EU): Data remains in EU regions with AWS/Azure/Google. Direct API: DeepSeek servers in China (caution with sensitive data)."* — [innFactory AI Consulting, febrero 2026](https://innfactory.ai/en/ai-models/deepseek/).
- *"Azure AI Foundry offers DeepSeek as a managed deployment. The advantage for organisations with existing Microsoft environments: integration with Azure Entra ID, established network and security configurations, and region selection for EU data residency."* — [Gosign GmbH, febrero 2026](https://www.gosign.de/en/magazine/deepseek-enterprise-hosting/).

Qué significa esto técnicamente: en Azure AI Foundry, DeepSeek se sirve desde infraestructura Microsoft. Si elegís una región europea (West Europe, North Europe, Switzerland North, Germany West Central, etc.) o configurás una EU Data Zone, el procesamiento ocurre dentro de la jurisdicción de la UE, bajo el régimen GDPR estándar de Azure. Microsoft actúa como data processor; tu empresa retiene el rol de data controller; el DPA (Data Protection Addendum) de Microsoft cubre las cláusulas estándar.

La diferencia operativa es categórica: **los datos no salen de la UE en ningún momento**, no hay transferencia internacional a China, y el modelo de gobernanza es exactamente el mismo que el que aplicarías a Azure OpenAI o a cualquier otro servicio Azure. *"Azure Document Intelligence and AI Foundry services run in the Azure region you select, so deploying these services in an EU region guarantees that data processing occurs within EU boundaries."* — [Microsoft Q&A oficial](https://learn.microsoft.com/en-us/answers/questions/2338327/how-can-we-manage-documents-in-privacy-sensitive-d).

### 5.4 El cálculo del costo: prima de compliance, no overhead general

Azure agrega un markup del **20–35% sobre el precio directo de DeepSeek** ([DeployBase 2026](https://deploybase.ai/articles/deepseek-v3-pricing)). Es el rango típico para modelos open-source servidos managed.

Supongamos $0,14/MTok input directo. Con un markup del 30%:

- DeepSeek V4-Flash directo: $0,14 / $0,28
- DeepSeek V4-Flash vía Azure (EU): ~$0,18 / $0,36

Comparado con Claude Sonnet ($3,00 / $15,00), eso significa:

- Sonnet input: $3,00/MTok
- DeepSeek Azure input: $0,18/MTok
- Ratio: **~16,5× más barato en input**

Para procesamiento masivo offline — clasificación de tickets, extracción de entidades, batch de embeddings, evaluación de respuestas, traducción automática de baja sensibilidad — donde la calidad de DeepSeek es suficiente, el ahorro es de un orden de magnitud completo. Y el compliance europeo está cubierto.

Es importante separar dos conceptos que hasta acá veníamos mezclando bajo "overhead":

1. **Overhead de plataforma** (Sección 3): support plans, egress, hosting de fine-tuned models, VNet, monitoring. Costos que existen porque consumís un modelo a través de Azure en lugar de directo. En el caso Claude/GPT, son evitables (la API directa los elimina), por lo cual aparecen como pura sobrecarga.
2. **Prima de compliance**: el markup específico que cobra Azure (20–35%) por hostear un modelo que de otra forma no podrías usar bajo GDPR. En el caso DeepSeek, no es evitable — la API directa no es viable. La prima compra acceso al modelo con compliance, no infraestructura adicional.

Esta distinción es crítica para tomar la decisión correcta. **Si el modelo está disponible directamente del vendor con compliance europeo (Claude, GPT, Mistral), el hyperscaler suma overhead sin agregar valor distintivo.** **Si el modelo en sí no es accesible directamente con compliance (DeepSeek, Qwen, modelos chinos en general), el hyperscaler vale la prima** — porque no hay alternativa viable.

Puesto al revés: el caso DeepSeek + Azure no contradice la regla general de que el hyperscaler no suele convenir por precio. La confirma desde otro ángulo: cuando la decisión es por compliance del modelo y no por precio del token, ahí sí el hyperscaler es la elección correcta.

### 5.5 Las opciones alternativas (y por qué Azure suele ganar)

No es la única forma de tener DeepSeek con compliance europeo. Las alternativas viables:

- **AWS Bedrock con EU regions** (Frankfurt, Irlanda, Estocolmo, París, Milán). Equivalente a Azure en términos de compliance. La elección entre Azure y Bedrock suele determinarla qué cloud ya usás para el resto de la stack.
- **Google Vertex AI con regiones europeas** (europe-west*). Similar a las anteriores. Como diferencia operativa, en general tiene mejor integración con stack data/ML de Google (BigQuery, Vertex Pipelines).
- **Self-hosting on-premises o en cloud propio**. DeepSeek es open source (licencia MIT). Podés correrlo en tus propias GPUs (H100, A100) en cualquier región europea. Da máximo control y minimiza dependencia de tercero, pero requiere expertise operacional significativa: capacity planning, scaling, monitoring, on-call ([Gosign 2026](https://www.gosign.de/en/magazine/deepseek-enterprise-hosting/)). Sólo se justifica económicamente a volúmenes muy altos (5K-10K USD/mes mínimo de consumo sostenido, según fuentes citadas en el documento técnico v2).

Entre los tres hyperscalers, **Azure AI Foundry tiende a ser la elección natural para una empresa que ya tiene Enterprise Agreement con Microsoft**: integración con Entra ID existente, billing consolidado, contrato único. AWS Bedrock es preferible si tu stack es mayoritariamente AWS. Vertex si es Google.

La elección entre los tres no es por precio (los tres están en rango similar de markup), es por **alineación con el resto de tu cloud strategy**.

---

## 6. Matriz de decisión: cuándo cada camino

Poniendo todo junto, esta es la matriz que conviene tener en mente cuando se evalúa un proveedor:

| Situación | Camino recomendado | Por qué |
|-----------|-------------------|---------|
| Claude / GPT / Mistral con uso general, sin mandato Azure-only | **Vendor directo** | Pricing parity nominal, sin overhead, mejor soporte de fábrica |
| Modelo geopolíticamente sensible (DeepSeek, Qwen, modelos chinos) y empresa europea | **Hyperscaler (Azure preferido)** | La prima compra compliance, no hay alternativa viable |
| Mandato corporativo "todo en Azure/AWS" | **Hyperscaler** | Decisión de gobernanza, asumir 15–40% como costo |
| Necesidad de MACC burn-down forzado | **Hyperscaler, excepto Claude en Foundry** | Decisión contable; Claude no descuenta del MACC |
| Compliance GDPR / data residency obligatorio y modelo no europeo | **Hyperscaler con región EU**, o **Mistral directo** | Mistral es la única alternativa major europea sin pasar por hyperscaler |
| Equipo en exploración, POCs, prototipos | **Vendor directo** | Onboarding rápido, sin negociación pesada |
| Workload de alto volumen y baja complejidad con datos EU | **DeepSeek vía Azure EU** | 15× más barato que Sonnet con compliance asegurado |

La regla general que se desprende: **el hyperscaler agrega valor cuando agrega algo que no podés obtener directamente (compliance específico del modelo, integración Entra ID, MACC burn). No agrega valor cuando todo lo que ofrece es "el mismo modelo más caro con un descuento que apenas compensa".**

---

## 7. Inputs concretos para una negociación con Microsoft

Si tu equipo está por sentarse a renovar el EA o evaluar mover workloads a Azure, estos son los datos verificados que conviene tener encima de la mesa:

| Dato | Valor | Fuente | Uso en la conversación |
|------|-------|--------|------------------------|
| Azure overhead vs OpenAI directo | 15–40% (avg 22%) | [TokenMix](https://tokenmix.ai/blog/azure-openai-cost), [Inference.net](https://inference.net/content/azure-openai-pricing-explained/), [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/) | "Necesitamos descuento que cubra al menos 25% para break-even contra OpenAI directo." |
| Bedrock overhead vs Anthropic directo | 20–35% | [TokenMix Bedrock](https://tokenmix.ai/blog/aws-bedrock-pricing) | "Mismo problema con Bedrock. El descuento debe cubrir overhead." |
| Pricing parity nominal en hyperscalers | Confirmado | [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing) | "El hyperscaler no nos da mejor precio por token — eso lo verificamos." |
| Eliminación Level B/C/D | Confirmada desde nov 2025 | [Redress Compliance](https://redresscompliance.com/microsoft-ea-pricing-changes-2025-all-customers-move-to-level-a-pricing/), [SAM Expert](https://samexpert.com/microsoft-ea-discount-removal-2025/), [ITAA](https://itaa.com/insights/microsoft-to-end-cloud-volume-discounts-on-november-1-2025/) | "Sabemos que el descuento programático ya no existe. ¿Qué descuento negociado nos ofrecen específicamente?" |
| Aumento por migración a Level A | 6–12% antes de negociar | [Syskit](https://www.syskit.com/blog/microsoft-ending-volume-licensing/), [US Cloud](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/) | "Cualquier oferta debe compensar primero ese aumento antes de poder hablar de ahorro real." |
| Foundry Claude credits exclusion | Documentada | [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352), [AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/) | "Solicitamos aclaración escrita sobre qué credits aplican a Claude en Foundry." |
| DeepSeek Azure EU markup | 20–35% | [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing) | "DeepSeek Azure es la única razón fuerte para mover este workload a Azure." |
| Casos reales Foundry trap | $1.600–$13.000/mes a tarjeta | [AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/) | "Tenemos casos documentados. Necesitamos cláusula explícita." |

---

## 8. Conclusión

El descuento del Enterprise Agreement no es falso, pero es engañoso si se interpreta como ahorro neto sobre el vendor directo. La realidad documentada por múltiples fuentes independientes de 2026 muestra tres cosas:

Primero, **el precio nominal por token es idéntico entre el vendor directo y los hyperscalers** para los mismos modelos (Claude, GPT, etc.). El descuento del EA se aplica sobre el precio inflado por overhead, no sobre un precio nominal preferencial.

Segundo, **el overhead del hyperscaler (15–40% para Azure OpenAI, 20–35% para Bedrock) típicamente neutraliza o supera el descuento negociado**. La factura neta queda en una franja de -14% a +12% vs vendor directo, no en el -25% que sugiere el marketing. Con la eliminación de los niveles B/C/D de descuento por volumen en noviembre 2025, ese balance se inclinó aún más hacia el lado del overhead para clientes mid-market.

Tercero, **el caso DeepSeek + Azure es la excepción clara donde el hyperscaler vale la prima**. No por procurement, no por descuento — por compliance. El acceso directo a DeepSeek es regulatoriamente inviable en la UE post-Garante; el acceso vía Azure EU es la única forma de capturar la economía de un modelo 15× más barato sin exposición legal. La prima de compliance del 20–35% es real, pero está comprando algo que sin el hyperscaler no existe.

La síntesis operativa: **"gastar bien" en este caso significa no firmar acuerdos con hyperscalers asumiendo ahorros que no se materializan, y simultáneamente reconocer los casos puntuales donde el hyperscaler resuelve un problema regulatorio real**. Es una decisión por modelo y por workload, no una decisión global de procurement.

---

## Anexo — Métricas verificadas con fuente

| Métrica | Valor | Fuente |
|---------|-------|--------|
| Azure overhead vs OpenAI directo | 15–40%, avg 22% | [TokenMix](https://tokenmix.ai/blog/azure-openai-cost) |
| Azure overhead vs OpenAI directo | 15–40% | [Inference.net](https://inference.net/content/azure-openai-pricing-explained/) |
| Azure overhead vs OpenAI directo | 20–40% | [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/) |
| Caso real overhead efectivo | 55% sobre $312,50 esperados | [Inference.net](https://inference.net/content/azure-openai-pricing-explained/) |
| Bedrock overhead vs Anthropic directo | 20–35% | [TokenMix Bedrock](https://tokenmix.ai/blog/aws-bedrock-pricing) |
| Bedrock regional endpoints | +10% sobre global | [Anthropic docs](https://platform.claude.com/docs/en/about-claude/pricing) |
| Bedrock forgiveness buffer 3% | Solo aplica en API directa | [TokenMix Bedrock](https://tokenmix.ai/blog/aws-bedrock-pricing) |
| Pricing parity Claude todos los canales | Sonnet $3/$15 idéntico | [Anthropic Claude in Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry) |
| Support plan Azure mensual | $100–$1.000+ | [CloudZero](https://www.cloudzero.com/blog/azure-openai-pricing/) |
| Data egress Azure | $0,087/GB después de 100GB | [Inference.net](https://inference.net/content/azure-openai-pricing-explained/) |
| Fine-tuned model hosting | $1,70–$3,00/hora ($1.224–$2.160/mes) | [Inference.net](https://inference.net/content/azure-openai-pricing-explained/) |
| VNet/Private Link mensual | $200–$2.000 | [Inference.net](https://inference.net/content/azure-openai-pricing-explained/) |
| Azure EA solo descuento | 15–25% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Azure EA + MACC descuento | 23–28% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Eliminación Level B/C/D | Vigente desde 1 nov 2025 | [Redress Compliance](https://redresscompliance.com/microsoft-ea-pricing-changes-2025-all-customers-move-to-level-a-pricing/) |
| Aumento por Level A flat | 6–12% sin negociar | [Syskit](https://www.syskit.com/blog/microsoft-ending-volume-licensing/) |
| Aumento Level C/D anterior | Hasta 12% perdido | [SAM Expert](https://samexpert.com/microsoft-ea-discount-removal-2025/) |
| Unified Support cálculo | % del spend total | [US Cloud](https://www.uscloud.com/blog/microsoft-enterprise-agreement-pricing-reset-ea-tier-elimination-unified-support-cost/) |
| Foundry Claude credits exclusion | No aplican Azure credits/MACC | [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) |
| Caso real Foundry trap | $13.000 USD/mes startup | [AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/) |
| Caso Leach | ¥237.081 cargados a tarjeta | [AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/) |
| Caso Sevriukov | €999,60 cargados a tarjeta | [AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/) |
| Garante ban DeepSeek | 30 enero 2025 | [Bird & Bird](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data) |
| GDPR Art. 3(2)(a) aplicado | Extraterritorialidad confirmada | [D'Andrea & Partners](https://www.dandreapartners.com/deepseek-blocked-in-italy-what-happened-legal-consequence%EF%BC%9F/) |
| Países con investigación activa | IT, BE, IE, NL, DE, AU, KR, TW | [Euroconsumers](https://www.euroconsumers.org/the-full-story-of-deepseek-how-euroconsumers-is-driving-action-for-consumers/), [MIAI](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/) |
| Almacenamiento datos DeepSeek | En China (admitido feb 2025) | [MIAI](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/) |
| DeepSeek disponible Azure EU | Desde feb 2026 | [innFactory](https://innfactory.ai/en/ai-models/deepseek/) |
| DeepSeek en AWS Bedrock + Google Vertex EU | Desde feb 2026 | [innFactory](https://innfactory.ai/en/ai-models/deepseek/) |
| DeepSeek Azure markup | 20–35% sobre directo | [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing) |
| DeepSeek V4-Flash directo | $0,14 / $0,28 /MTok | [TechJack](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/) |
| DeepSeek V4-Flash via Azure | ~$0,18 / $0,36 /MTok | Cálculo aplicando markup 30% |
| DeepSeek Azure vs Sonnet | ~16,5× más barato en input | $3,00 vs $0,18 |
| Microsoft data processor role | DPA cubre cláusulas GDPR | [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/2338327/how-can-we-manage-documents-in-privacy-sensitive-d) |
