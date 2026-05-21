# El descuento del 25% que no es 25%
## Por qué los hyperscalers no son más baratos, y dónde sí valen la prima

**Audiencia:** Tech leads, architects, decisores de procurement y CFOs/CTOs que estén evaluando o renegociando contratos enterprise para consumo de LLMs.
**Tema:** Cómo se forma el costo real cuando comprás un LLM a través de un hyperscaler (Azure, AWS Bedrock, Google Vertex) versus directo del vendor (Anthropic, OpenAI). Y dónde, a pesar de la cuenta neta, el hyperscaler sí es la única opción correcta.

---

## Resumen para los que tienen 90 segundos

El argumento comercial que escuchás en propuestas de Microsoft, AWS o Google suena así: "comprá tus LLMs a través de nosotros y obtené hasta 25% de descuento vía Enterprise Agreement". El número es real, pero la lectura es incompleta. Cuando se mide el TCO efectivo:

- **El precio nominal por token es idéntico** entre vendor directo y hyperscaler. Anthropic, OpenAI y otros mantienen pricing parity per-token explícita y públicamente documentada.
- **El hyperscaler agrega entre 15% y 40% de overhead** (support plans, data egress, hosting de modelos fine-tuned, monitoring infra, networking, compliance overhead).
- **El descuento del 15–28% del EA o MACC se compensa con ese overhead**, no se suma.
- **Resultado neto:** la factura final queda en una banda entre **-14% y +12%** comparada con vendor directo. En el mejor caso, ahorrás un par de puntos; en el peor, pagás un 12% más. Lejos del 25% prometido.

La excepción donde el hyperscaler sí justifica claramente la prima: **modelos que no son accesibles directamente con compliance europeo**. El caso más nítido es DeepSeek, baneado en la UE en su acceso directo por incumplir GDPR, pero accesible con compliance europeo dentro de Azure AI Foundry pagando un markup. Para ese caso, la prima compra compliance, no infraestructura.

El resto del documento desarrolla esta cuenta con fuentes verificadas, y explica por qué la estructura del modelo de reventa hace que la promesa del descuento sea inherentemente engañosa.

---

## 1. Lo que dice el marketing

Las propuestas comerciales de los hyperscalers para consumo de LLMs suelen construirse sobre dos pilares:

**Pilar 1: el descuento del Enterprise Agreement.** Microsoft, AWS y Google ofrecen descuentos negociables vía EA (Enterprise Agreement) o equivalentes. Para Azure, los rangos documentados son:

- **EA solo: 15–25%** de descuento negociable sobre el precio de lista.
- **EA + MACC (Microsoft Azure Consumption Commitment): 23–28%.** Cuanto más grande el commitment plurianual, más descuento.

Fuente: [Microsoft Negotiations — GitHub Copilot Enterprise Licensing](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing)

**Pilar 2: el bundling.** "Si ya estás en Azure, sumá tus LLMs acá y simplificás procurement, facturación, soporte y compliance. Es más fácil y más barato."

La conclusión que el comercial quiere que saques: comprando los LLMs a través del hyperscaler, vas a pagar al menos un 20–25% menos que comprándolos directo al vendor.

El problema con esta lectura es que **se compara el precio nominal con descuento contra el precio nominal sin descuento**, y omite por completo la estructura de overhead que el hyperscaler le agrega al consumo. Esa omisión es lo que da vuelta la cuenta.

---

## 2. La realidad del precio: pricing parity nominal

Lo primero que conviene tener claro antes de cualquier negociación: **el precio nominal por token es idéntico entre vendor directo y hyperscaler para los mismos modelos.**

### 2.1 Claude en hyperscalers

Anthropic mantiene **pricing parity per-token explícita** entre su API directa y todos los hyperscalers principales. Es decir, el mismo modelo cuesta lo mismo por millón de tokens en todos los canales:

| Modelo Claude | Anthropic directo | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|----------------|---------------------|----------------------|--------------------|---------------------|
| Sonnet 4.6 | $3 / $15 | $3 / $15 | $3 / $15 | $3 / $15 |
| Opus 4.7 | $5 / $25 | $5 / $25 | $5 / $25 | $5 / $25 |
| Haiku 4.5 | $1 / $5 | $1 / $5 | $1 / $5 | $1 / $5 |

Fuente: [Anthropic Pricing Docs oficial](https://platform.claude.com/docs/en/about-claude/pricing) y [Anthropic — Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry).

Los únicos modificadores nominales publicados por Anthropic son:
- Bedrock regional endpoints (vs global): **+10%**.
- Vertex regional/multi-region (vs global): **+10%**.
- Claude API directa con `inference_geo: "us"`: **+10%** (1.1x multiplier).

Cualquier otro markup que aparezca en la factura corresponde a overhead del proveedor — no al precio del modelo.

### 2.2 OpenAI en Azure OpenAI

El caso de OpenAI es análogo. Tres fuentes independientes de 2026 lo verifican:

> "Pricing parity is roughly 1:1 for the same models. The cost differentiation comes from Azure infrastructure overhead (Private Link, Key Vault, monitoring) — adds ~5-10% to total cost."

Fuente: [EPC Group — Azure OpenAI vs OpenAI API Enterprise Comparison 2026](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026)

> "Per-token pricing is identical for the same models. But Azure's total cost of ownership runs higher due to mandatory support plans for production use, data egress fees, and infrastructure overhead."

Fuente: [CloudZero — Azure OpenAI Pricing 2026](https://www.cloudzero.com/blog/azure-openai-pricing/)

> "Azure OpenAI cost is straightforward at the token level — prices match OpenAI's direct API exactly. The complexity and the budget surprises come from everything around the tokens."

Fuente: [TokenMix — Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost)

### 2.3 La consecuencia inmediata

Si el precio nominal por token es idéntico, entonces el "descuento" que ofrece el hyperscaler **no aplica sobre el precio del modelo**: aplica sobre el conjunto de servicios Azure/AWS/GCP que la empresa ya consume. El LLM viaja dentro de ese paquete, no es el origen del descuento.

Esto importa porque cuando alguien dice "Azure nos da 25% off en GPT-5", el enunciado es estrictamente incorrecto. Lo correcto sería: "Azure nos da 25% off sobre nuestro Azure spend total, y ahí adentro está el consumo de GPT-5". La diferencia parece sutil pero es decisiva, porque el descuento aplica sobre un denominador que incluye mucho más que el LLM.

---

## 3. El overhead: por qué la cuenta se da vuelta

Si el precio nominal es el mismo, ¿de dónde sale entonces la diferencia que termina apareciendo en la factura mensual? De cinco categorías de overhead que el hyperscaler factura **adicionalmente al precio del token**, y que la pricing page del modelo no muestra.

### 3.1 Las cinco categorías de overhead documentadas

**1. Support plans obligatorios.** Para producción enterprise en Azure, el Standard support (o superior) es de facto necesario. Costo: $100 a más de $1.000 por mes, dependiendo del nivel.

Fuente: [CloudZero — Azure OpenAI Pricing 2026](https://www.cloudzero.com/blog/azure-openai-pricing/)

**2. Data egress (transferencia saliente).** Los primeros 100GB de tráfico outbound son gratuitos, pero a partir de ahí se paga $0.087/GB. En aplicaciones con tráfico moderado o alto (por ejemplo, una app que devuelve respuestas largas a muchos usuarios), esto se acumula con velocidad sorprendente.

Fuente: [CloudZero — Azure OpenAI Pricing 2026](https://www.cloudzero.com/blog/azure-openai-pricing/)

**3. Hosting de modelos fine-tuned.** Si fine-tuneás un modelo, el costo del fine-tuning en sí es $1.50 a $25 por millón de tokens — manejable. **Pero el modelo fine-tuneado, una vez desplegado, cobra entre $1.70 y $3 por hora aun sin uso.** Eso son **entre $1.224 y $2.160 por mes por modelo, simplemente por tenerlo desplegado, aunque conteste cero queries.**

Fuente: [CloudZero — Azure OpenAI Pricing 2026](https://www.cloudzero.com/blog/azure-openai-pricing/)

**4. VNet, Private Link, content filtering.** Costos de networking aislado y compliance: $200 a $2.000/mes según configuración. Para muchas empresas estos son requeridos por compliance, pero no aparecen en la calculadora del modelo.

Fuente: [EPC Group — Azure OpenAI vs OpenAI API Enterprise Comparison 2026](https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026)

**5. Monitoring y log infrastructure.** Log Analytics, Application Insights, audit retention. Costo variable según volumen, pero típicamente añade cientos de dólares por mes en producción.

### 3.2 El número agregado: 15–40% de overhead

Cuatro fuentes independientes coinciden en el rango del overhead total para Azure OpenAI:

| Fuente | Rango documentado | Promedio típico |
|--------|---------------------|------------------|
| [TokenMix — Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15–40% | 22% |
| [Inference.net — Azure OpenAI Pricing Explained 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15–40% | — |
| [CloudZero — Azure OpenAI Pricing 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20–40% | — |
| [TokenMix — Azure OpenAI Alternative 2026](https://tokenmix.ai/blog/azure-openai-alternative) | 15–40% | — |

Cita textual de TokenMix, que mide producción real:

> "Token pricing is identical across all three; the spread is overhead — Azure adds 15-40%, OpenAI Direct adds 5-10%, TokenMix.ai's unified API adds 0-5%."

Fuente: [TokenMix — Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost)

Casos medidos por TokenMix sobre deployments reales:
- **Startup (10M tokens/mes):** overhead negligible, el token cost domina.
- **Mid-size enterprise:** $485/mes de factura real vs $313 que predice la calculadora de Azure. **55% por encima** del estimado oficial.
- **Enterprise grande:** $7.452/mes reales vs ~$2.200 estimados. **3.4x el cálculo oficial.** Casi la mitad del exceso es por fine-tuned model hosting.

Fuente: [TokenMix — Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost)

El patrón es consistente: **a mayor escala, más se separa la factura real de la calculadora**, porque más servicios del paquete enterprise se activan.

### 3.3 Caso paralelo en AWS Bedrock

El patrón se repite en Bedrock. Pricing nominal idéntico a Anthropic directo, pero overhead efectivo de **20–35%**.

Fuente: [TokenMix — AWS Bedrock Pricing 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)

Los componentes específicos:
- Regional endpoints: +10% sobre global (este es publicado por Anthropic).
- Cross-region inference: +10% adicional cuando se rutea entre regiones.
- Data transfer (egress) cobrado por Bedrock; la API directa no lo cobra.
- Bedrock factura **todos los errores HTTP 500**. La API directa de Anthropic tiene un buffer del 3% en errores del servidor que no se cobran.
- CloudWatch + CloudTrail logging mandatorio en deployments enterprise.

> "TokenMix.ai cost tracking shows enterprises running Claude on Bedrock pay an average of 20-35% more than those using Anthropic's direct API — and most do not realize it."

Fuente: [TokenMix — AWS Bedrock Pricing 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)

### 3.4 Caso especial: Claude en Microsoft Foundry

Este es el caso más sutil y donde más empresas se llevan la sorpresa. Claude se factura en Azure AI Foundry como **Marketplace de terceros**, no como recurso nativo de Azure. La implicación crítica:

> "Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."

Fuente: [AZ365.ai — Claude on Azure: The Marketplace Billing Trap (marzo 2026)](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/), confirmado por [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352).

En castellano: **el consumo de Claude en Foundry no descuenta de tus Azure credits ni de tu MACC.** Va directo a la tarjeta de crédito. Si tu plan de procurement asumía que el MACC absorbía el gasto en Claude, te vas a llevar una sorpresa cuando lleguen las primeras facturas.

Casos reales documentados:
- Founder japonés (Leach): ~¥237.081 (~$1.600 USD) cargados directo a tarjeta, con credits sin aplicar.
- Founder alemán (Bogdan Sevriukov): €999,60 cargados directo.
- Otro founder japonés: ¥2.000.000 (~$13.000 USD) en un solo mes.

Fuente: [AZ365.ai — Claude on Azure: The Marketplace Billing Trap](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/)

**Acción concreta cuando estés en una negociación con Microsoft:** pedir aclaración escrita explícita sobre qué tipos de credits aplican a Claude vía Foundry, qué porcentaje del consumo se imputa al MACC y bajo qué condiciones. Si la respuesta es ambigua, asumir que no aplican.

---

## 4. La cuenta neta: descuento vs overhead

Llegamos al núcleo. La pregunta operativa es: ¿qué pasa cuando aplicamos al mismo workload los dos efectos juntos — descuento del EA por un lado, overhead del hyperscaler por el otro?

### 4.1 La matriz neta

Tomando como base 100 unidades = costo del workload consumido directo al vendor, esta es la cuenta sobre los rangos documentados:

| Escenario | Costo directo | Costo Azure (con overhead 15–40%) | Neto vs directo |
|-----------|----------------|--------------------------------------|-------------------|
| GPT-5.4, sin descuento, sin compliance | 100 | 115–140 | **+15% a +40%** |
| GPT-5.4, EA solo (-20%) sobre 115–140 | 100 | 92–112 | **-8% a +12%** |
| GPT-5.4, EA + MACC (-25%) sobre 115–140 | 100 | 86–105 | **-14% a +5%** |
| Claude Sonnet en Foundry, sin descuento aplicable a Claude | 100 | 115–140 | **+15% a +40%** (Foundry no aplica MACC) |
| Claude Sonnet en Bedrock, sin compliance | 100 | 120–135 | **+20% a +35%** |

### 4.2 La lectura

- **El descuento del EA (15–25%) y el overhead del hyperscaler (15–40%) son del mismo orden de magnitud.** Por construcción matemática, se neutralizan entre sí en buena parte de los casos.
- **El mejor escenario para el hyperscaler** (EA + MACC sobre overhead bajo) ahorra alrededor de 14% vs vendor directo. **No 25%.**
- **El peor escenario para el hyperscaler** (descuento menor, overhead alto) termina pagando hasta 12% más que vendor directo. La promesa del descuento se da vuelta completa.
- **Para Claude en Foundry, el descuento del MACC no aplica al consumo del modelo** (Sección 3.4). Eso significa que pagás el overhead sin la compensación del descuento — el peor escenario es el escenario base.

### 4.3 Por qué pasa esto: la lógica del modelo de reventa

La razón estructural por la que la cuenta da así no es accidental. Es consecuencia directa del modelo de negocio del hyperscaler cuando revende capacidad de un tercero.

El hyperscaler **no produce el modelo**: lo licencia o lo distribuye desde OpenAI, Anthropic, DeepSeek, etc. Por contrato comercial, está obligado a mantener pricing parity con el vendor directo en el precio nominal del token — si vendiera más barato, canibaliza el canal directo del vendor y rompe el acuerdo.

Lo que el hyperscaler **sí controla** y donde sí puede generar margen es todo lo que rodea al modelo: networking, identity, monitoring, support, compliance, integración con su ecosistema, billing unificado. Ese es el espacio donde el hyperscaler agrega valor, y donde naturalmente agrega costo. **El descuento del EA es entonces, en buena medida, una devolución parcial del overhead que el mismo hyperscaler agregó.** No es un descuento sobre el precio "puro" del modelo, porque ese precio puro lo está respetando por contrato con el vendor.

Vista así, la cuenta cierra: el "25% de descuento" del hyperscaler **compensa, no supera**, los costos adicionales que el mismo hyperscaler introduce al canalizar el consumo a través de su infraestructura. La promesa de ahorro neto no es engañosa por mala fe del comercial — es matemáticamente improbable por construcción del modelo.

### 4.4 Una nota importante: los descuentos por niveles ya no son automáticos

Un cambio reciente que conviene tener en el radar: **Microsoft eliminó los niveles automáticos de descuento (Level A-D) para servicios online en noviembre de 2025**. Los descuentos por niveles ahora solo aplican a licencias on-premises.

> "Microsoft ended volume discounts (Level A-D pricing) for online services effective November 2025. Level discounts are available only for on-premises licences."

Fuente: [SAM Expert — Microsoft Enterprise Agreement Guide 2026](https://samexpert.com/what-is-a-microsoft-enterprise-agreement/)

> "The focus shifts from automatic volume discounts to negotiated value, usage optimization, and selecting the right licensing vehicle."

Fuente: [Sourcepass — Microsoft Ended Enterprise Agreement Discount Levels](https://sourcepassmcoe.com/articles/microsoft-ended-enterprise-agreement-discount-levels-sourcepass-mcoe)

Estimación industrial del impacto: organizaciones que estaban en Level C o D pueden esperar **incrementos del 6–12%** en su factura comparado con su estructura anterior. Cualquier descuento de servicios online en 2026 es íntegramente negociado, ya no surge automáticamente del tamaño del contrato.

Esto refuerza el argumento del documento: la promesa del "descuento automático grande" está más lejos de la realidad en 2026 que hace dos años. El descuento existe, pero hay que pelearlo, y aun así no supera el overhead salvo en casos muy específicos.

---

## 5. Dónde el hyperscaler sí vale la prima: el caso DeepSeek + GDPR

Hasta acá el argumento podría leerse como "el hyperscaler nunca conviene". Eso sería incorrecto. Hay un caso específico — y crítico para empresas europeas — donde el hyperscaler **es la única opción viable**, y donde la prima que cobra compra algo real que no se puede conseguir de otra forma: **compliance europeo para modelos que el vendor directo no puede proveer dentro de la UE.**

El caso paradigmático es DeepSeek.

### 5.1 El problema: DeepSeek directo no es viable en la UE

DeepSeek es un modelo desarrollado por una empresa china. Sus términos de servicio especifican que está regido por las leyes de la República Popular China, y su política de privacidad declara que los datos de los usuarios se almacenan en servidores en China.

> "DeepSeek's privacy policy says the company stores user data on servers located in China."

Fuente: [The Record — Italy blocks Chinese AI tool DeepSeek over privacy concerns (enero 2025)](https://therecord.media/italy-blocks-chinese-ai-tool-deepseek-over-privacy-concerns)

Esto choca de frente con el GDPR. El reglamento europeo prohíbe transferir datos personales de ciudadanos europeos a países que no ofrezcan garantías de protección adecuadas, y China no está en la lista de países adecuados.

> "A key concern is the cross-border transfer of personal data. GDPR prohibits the transfer of EU citizens' data to countries that lack adequate privacy protections, and China is not considered a country with sufficient safeguards."

Fuente: [HIMSS — DeepSeek Blocked in Italy Due to Privacy Risks](https://www.himss.org/news-center/deepseek-blocked-italy-due-privacy-risks-setting-significant-precedent/)

### 5.2 El precedente regulatorio: Italia, Alemania, y la ola que vino después

En **enero de 2025**, la autoridad italiana de protección de datos (Garante) impuso una **limitación definitiva sobre el procesamiento de datos** de usuarios italianos por parte de DeepSeek, con efecto inmediato. Es decir: ban operacional.

> "On 30 January 2025, the Italian Data Protection Authority (Garante or Authority) imposed, as a matter of urgency and with immediate effect, a definitive limitation on the processing of Italian users' personal data on Hangzhou DeepSeek Artificial Intelligence and Beijing DeepSeek Artificial Intelligence."

Fuente: [Bird & Bird — The Garante imposes a definitive limitation on processing of Italian users' personal data](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data)

DeepSeek argumentó que las leyes europeas no le aplicaban por estar basada en China. El Garante rechazó el argumento — y este es el precedente regulatorio importante:

> "The Garante therefore concluded that the Companies unquestionably offered DeepSeek's services to Italian data subjects and, as a result, the provisions of the GDPR are applicable, including the obligation to appoint a representative in the EU under Article 27 GDPR."

Fuente: [Bird & Bird — The Garante imposes a definitive limitation](https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data)

La ola se expandió rápido:

| País | Acción | Fecha |
|------|--------|-------|
| Italia | Ban operacional definitivo | enero 2025 |
| Alemania (Berlin DPA) | Notificación DSA Article 16 a Apple y Google para delistar la app | junio 2025 |
| Holanda | Warning oficial sobre uploads de información personal | 2025 |
| Irlanda | Investigación formal abierta | 2025 |
| Bélgica | Investigación formal abierta + denuncia de grupo consumidor | 2025 |
| Australia | Ban en dispositivos de gobierno | 2025 |
| Corea del Sur | Ban tras encontrar transferencia de datos sin consentimiento | 2025 |
| Taiwán | Bloqueo por seguridad nacional | 2025 |

Fuente: [DataNorth AI — DeepSeek Data Privacy and Security](https://datanorth.ai/blog/deepseek-data-privacy-and-security-key-insights-on-protection-and-risks) y [MIAI — DeepSeek One Year Later: Regulatory Storm](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/)

Al cierre del primer trimestre de 2026, el ban italiano sigue vigente, la investigación alemana sigue abierta, y el cumplimiento sustantivo de DeepSeek con los requisitos europeos de protección de datos sigue siendo, en palabras de los reguladores, "subject of ongoing investigation and considerable doubt".

Fuente: [MIAI — DeepSeek One Year Later](https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/)

**Conclusión operativa:** para una empresa europea, consumir DeepSeek vía `api.deepseek.com` no es una opción razonable. No es solo un riesgo teórico de GDPR — es un riesgo regulatorio real, ya materializado en al menos un país de la UE y bajo investigación activa en otros.

### 5.3 La solución viable: DeepSeek vía Azure AI Foundry

Microsoft anunció a inicios de 2025 la disponibilidad de DeepSeek R1 (y posteriormente otros modelos de DeepSeek) en Azure AI Foundry. La incorporación incluyó red-teaming y safety evaluations propios de Azure, y se sumó al catálogo de más de 1.800 modelos disponibles en la plataforma.

> "DeepSeek R1 is now available in the model catalog on Azure AI Foundry and GitHub, joining a diverse portfolio of over 1,800 models. As part of Azure AI Foundry, DeepSeek R1 is accessible on a trusted, scalable, and enterprise-ready platform, enabling businesses to seamlessly integrate advanced AI while meeting SLAs, security, and responsible AI commitments — all backed by Microsoft's reliability and innovation."

Fuente: [Microsoft Azure Blog — DeepSeek R1 is now available on Azure AI Foundry and GitHub](https://azure.microsoft.com/en-us/blog/deepseek-r1-is-now-available-on-azure-ai-foundry-and-github/)

Lo crítico para el caso europeo: Azure permite desplegar el modelo en regiones específicas, incluyendo **EU Data Zones**, que garantizan que los datos no abandonen el perímetro europeo.

> "DeepSeek R1 can be deployed in specific Azure regions or even in special environments like Azure 'Data Zone' regions that ensure data residency (for example, to keep data within the EU or within specific sovereign clouds)."

Fuente: [Deep AI Chat — Getting Started with DeepSeek R1 on Azure AI Foundry](https://chat-deep.ai/guide/deepseek-r1-on-azure-ai-foundry/)

Confirmado también por consultorías regulatorias europeas a febrero de 2026:

> "Update February 2026: DeepSeek is now available on AWS Bedrock, Azure AI, and Google Vertex AI in EU regions! [...] Cloud Hosting (EU): Data remains in EU regions with AWS/Azure/Google."

Fuente: [innFactory AI Consulting — DeepSeek GDPR-compliant deployment options](https://innfactory.ai/en/ai-models/deepseek/)

En este setup, los datos no se envían a servidores chinos. El contrato de la empresa europea es con Microsoft (DPA, BAA donde aplique, residencia EU), no con DeepSeek. **El argumento de GDPR — el que el Garante usó para banear el acceso directo — desaparece**, porque la cadena de custodia de los datos queda íntegramente bajo Azure y dentro de la UE.

### 5.4 La prima: cuánto cuesta el compliance

Microsoft no provee este servicio gratis. El acceso a DeepSeek vía Azure agrega un markup sobre el precio directo de DeepSeek.

> "Microsoft integra modelos DeepSeek con un markup de 20–35% sobre las tarifas oficiales de DeepSeek."

Fuente: [DeployBase — DeepSeek V3 Pricing 2026](https://deploybase.ai/articles/deepseek-v3-pricing)

A primera vista esto se podría leer como "más overhead injustificado", pero conviene contextualizarlo contra el precio base del modelo. DeepSeek es **uno de los modelos más baratos del mercado**:

- DeepSeek V4-Flash directo: **$0.14 / $0.28** por millón de tokens (input / output).
- DeepSeek V4-Flash vía Azure (con markup 35%): **~$0.19 / $0.38** por millón.

Fuente: [TechJack Solutions — DeepSeek Pricing 2026](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/) y [DeployBase — DeepSeek V3 Pricing 2026](https://deploybase.ai/articles/deepseek-v3-pricing)

Comparemos contra modelos enterprise occidentales:
- Claude Sonnet 4.6: $3 / $15 por millón.
- GPT-5.5 (flagship): $5 / $30 por millón.

**Conclusión cuantitativa:** incluso con el markup del 35% aplicado, DeepSeek vía Azure cuesta aproximadamente **1/15 lo que Sonnet** y **1/25 lo que GPT-5.5** en input. Para tareas de alto volumen y complejidad baja-media (clasificación masiva, extracción estructurada, batch processing offline, embeddings, moderación), sigue siendo la mejor relación precio/compliance del mercado europeo.

### 5.5 Por qué este caso es estructuralmente distinto al de Claude/GPT en Azure

Y acá es donde el argumento del documento cierra. En el caso de Claude o GPT consumidos vía hyperscaler:

- El vendor (Anthropic, OpenAI) **sí ofrece acceso directo con compliance europeo** vía sus propios canales enterprise y data residency opcional.
- El hyperscaler agrega overhead que no compra capacidad nueva — compra integración con su ecosistema.
- El descuento del EA neutraliza parcialmente ese overhead, pero el resultado neto está cerca del costo del vendor directo.
- **Conclusión:** el hyperscaler es opcional. Puede convenir por razones de procurement (facturación unificada, MACC burn, alineación con stack existente), pero el precio del modelo no es la razón.

En el caso de DeepSeek consumido vía Azure:

- El vendor (DeepSeek) **no ofrece acceso directo con compliance europeo**. Su acceso directo está baneado en al menos un país de la UE y bajo investigación en otros.
- El hyperscaler agrega un markup, pero el markup compra **acceso al modelo bajo un régimen de compliance que no existe de otra forma**.
- No hay descuento que neutralizar contra qué — directo no es una alternativa.
- **Conclusión:** el hyperscaler es la única opción viable. El markup es prima de compliance, no overhead de plataforma. Y aun pagándolo, el modelo sigue siendo varias veces más barato que las alternativas occidentales.

La distinción entre "overhead de plataforma" (que se compensa, parcialmente, con descuento del EA) y "prima de compliance" (que compra capacidad inalcanzable de otra forma) es clave. Las dos pueden parecer "el hyperscaler cobra extra", pero económica y operativamente son cosas distintas. Una se compara contra una alternativa equivalente; la otra no tiene alternativa equivalente.

---

## 6. Matriz de decisión

Resumen operativo para llevar a una reunión de procurement.

| Criterio dominante | Camino recomendado | Razón |
|----------------------|-----------------------|---------|
| Anthropic o OpenAI con compliance europeo disponible directo | **Vendor directo** | Pricing parity nominal, sin overhead, features primero, soporte nativo |
| Mandato corporativo "todo en Azure/AWS" | **Hyperscaler** | Asumir 15–40% overhead como costo de gobernanza interna, esperar -14% a +12% neto post-descuento |
| MACC burn / facturación unificada Microsoft | **Hyperscaler para todo excepto Claude en Foundry** | Decisión contable, no de costo; **excluir Claude de la cuenta del MACC** (Sección 3.4) |
| Modelo geopolíticamente sensible (DeepSeek, Qwen) | **Hyperscaler (Azure preferido)** | La prima compra compliance, no plataforma; sin alternativa directa viable en la UE |
| Equipo en exploración, prototipos, POCs | **Vendor directo** | Onboarding rápido, menos burocracia, sin compromiso de commitment |
| GDPR / data residency obligatorio para LLM occidental | **Vendor directo con data zone EU** (Anthropic, OpenAI Enterprise) o **Mistral** (único major europeo) | El vendor directo ya ofrece compliance EU; no necesitás hyperscaler para esto |
| GDPR / data residency obligatorio para modelo chino o no-EU | **Hyperscaler con región EU** | Única vía viable |

---

## 7. Inputs concretos para negociación

Cuando te sentás con un comercial del hyperscaler, estos son los datos verificados que conviene tener encima de la mesa. Sirven para detectar promesas exageradas y para pedir términos comparables a los del vendor directo.

| Dato | Valor verificado | Uso en negociación |
|------|---------------------|---------------------|
| **Pricing nominal idéntico entre canales** | Confirmado oficialmente por Anthropic; análogo para OpenAI | "El hyperscaler no nos da mejor precio por token — solo procurement diferente" |
| **Overhead Azure OpenAI vs OpenAI directo** | 15–40% (promedio 22%) | "Necesitamos descuento que cubra al menos 25% para break-even contra directo" |
| **Overhead Bedrock vs Anthropic directo** | 20–35% efectivo | "Foundry o directo nos da el mismo precio sin ese markup adicional" |
| **Foundry Claude credits exclusion** | Documentado, no aplica MACC | "Aclaración escrita explícita sobre qué credits aplican a Claude" |
| **EA + MACC descuento** | 23–28% negociable, requiere commitment | "Sin commitment grande, el descuento es menor y no compensa el overhead" |
| **Volume discounts automáticos** | Eliminados nov 2025 para online services | "Cualquier descuento es 100% negociado, no automático" |
| **DeepSeek vía Azure markup** | +20–35% sobre directo | "Aceptable porque DeepSeek directo no es viable por GDPR" |
| **Anthropic Enterprise volume discount** | Disponible, negociable | "Solicitar términos comparables al hyperscaler con MACC" |
| **OpenAI Enterprise custom pricing** | Disponible | "Solicitar mismo régimen que Azure pero sin overhead" |

---

## 8. Conclusión

El descuento del 25% que ofrecen los hyperscalers para el consumo de LLMs es real como número nominal sobre el Azure spend total, pero **engañoso como indicador de ahorro neto sobre el costo del modelo**. La estructura del modelo de reventa hace que el descuento del EA esté del mismo orden de magnitud que el overhead que el mismo hyperscaler introduce: support plans, egress, fine-tuned model hosting, networking, monitoring. Resultado neto típico: factura final entre **-14% y +12%** vs vendor directo. En el mejor caso, ahorrás un par de puntos; en el peor, pagás más. Lejos del 25%.

La excepción es estructural, no comercial: **cuando el modelo en sí no es accesible directamente con compliance europeo, el hyperscaler vale la prima.** DeepSeek es el caso paradigmático — baneado en Italia, bajo investigación en otros países EU, sin acceso directo viable para empresas europeas, y al mismo tiempo, uno de los modelos más baratos del mercado. Consumirlo vía Azure AI Foundry con región EU paga un markup del 20–35%, pero compra algo concreto: cadena de custodia de datos dentro de la UE, contrato con Microsoft (no con DeepSeek), compliance europeo asegurado por DPA. Y aun con ese markup, el modelo sigue costando aproximadamente 1/15 lo que Sonnet, lo cual hace que la cuenta cierre incluso pagando la prima.

La regla operativa que conviene internalizar es esta: **antes de elegir hyperscaler por precio, validá si el modelo está disponible directamente del vendor con compliance europeo.** Si lo está (Claude, GPT, Mistral), el vendor directo es la opción de menor TCO. Si no lo está (DeepSeek, Qwen, modelos chinos en general), el hyperscaler es la única vía viable y la prima es justa porque no compra plataforma, compra acceso bajo régimen regulatorio europeo.

> **El descuento del 25% del hyperscaler compensa el overhead; no lo supera. Y la única razón sólida para pagar la prima es cuando el modelo, por sí mismo, no se puede consumir bajo compliance europeo sin pasar por la infraestructura del hyperscaler.**

---

## Anexo — Referencia de métricas verificadas

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Azure EA descuento solo | 15–25% negociable | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC descuento | 23–28% negociable | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Microsoft EA Level A-D discounts eliminados | Noviembre 2025 (online services) | SAM Expert | https://samexpert.com/what-is-a-microsoft-enterprise-agreement/ |
| Microsoft EA Level A-D impacto en factura | +6–12% para Levels C-D | Sourcepass | https://sourcepassmcoe.com/articles/microsoft-ended-enterprise-agreement-discount-levels-sourcepass-mcoe |
| Azure OpenAI overhead vs OpenAI directo | 15–40% (avg 22%) | TokenMix | https://tokenmix.ai/blog/azure-openai-cost |
| Azure OpenAI overhead vs OpenAI directo | 15–40% | Inference.net | https://inference.net/content/azure-openai-pricing-explained/ |
| Azure OpenAI overhead vs OpenAI directo | 20–40% | CloudZero | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| Azure OpenAI Markup alternative | 15–40% | TokenMix Alternative | https://tokenmix.ai/blog/azure-openai-alternative |
| Azure overhead enterprise real vs calculadora | 3.4x el estimado | TokenMix | https://tokenmix.ai/blog/azure-openai-cost |
| Bedrock Claude overhead vs Anthropic directo | 20–35% efectivo | TokenMix Bedrock | https://tokenmix.ai/blog/aws-bedrock-pricing |
| Bedrock regional endpoints premium | +10% sobre global | Anthropic oficial | https://platform.claude.com/docs/en/about-claude/pricing |
| Pricing parity Claude en Bedrock/Vertex/Foundry | Idéntico al directo | Anthropic oficial | https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry |
| Pricing parity GPT en Azure OpenAI | Idéntico al directo | EPC Group | https://www.epcgroup.net/blog/azure-openai-vs-openai-api-enterprise-comparison-2026 |
| Foundry Claude credits exclusion | No aplican Azure credits ni MACC | Microsoft Q&A | https://learn.microsoft.com/en-us/answers/questions/5851352 |
| Caso documentado Foundry billing trap | ~$13K USD/mes startup japonesa | AZ365.ai | https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/ |
| Data egress Azure (después de 100GB gratis) | $0.087/GB | CloudZero | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| Fine-tuned model hosting Azure | $1.70–$3.00/hora aun sin uso | CloudZero | https://www.cloudzero.com/blog/azure-openai-pricing/ |
| DeepSeek ban Italia (Garante) | Limitación definitiva, 30 enero 2025 | Bird & Bird | https://www.twobirds.com/en/insights/2025/the-garante-imposes-a-definitive-limitation-on-the-processing-of-italian-users%E2%80%99-personal-data |
| DeepSeek datos almacenados en China | Confirmado en su privacy policy | The Record | https://therecord.media/italy-blocks-chinese-ai-tool-deepseek-over-privacy-concerns |
| GDPR + China cross-border transfer | Prohibido sin safeguards adecuados | HIMSS | https://www.himss.org/news-center/deepseek-blocked-italy-due-privacy-risks-setting-significant-precedent/ |
| DeepSeek investigaciones / acciones por país | IT, DE, NL, IE, BE, AU, KR, TW | DataNorth AI / MIAI | https://datanorth.ai/blog/deepseek-data-privacy-and-security-key-insights-on-protection-and-risks |
| DeepSeek estado regulatorio 2026 | Ban Italia vigente, investigaciones activas | MIAI | https://ai-regulation.com/deepseek-one-year-later-regulatory-storm-global-surge/ |
| DeepSeek en Azure AI Foundry con EU Data Zone | Disponible, data residency EU | Microsoft Azure Blog | https://azure.microsoft.com/en-us/blog/deepseek-r1-is-now-available-on-azure-ai-foundry-and-github/ |
| DeepSeek EU regions disponibles | AWS Bedrock, Azure AI, Google Vertex (feb 2026) | innFactory AI | https://innfactory.ai/en/ai-models/deepseek/ |
| Azure Data Zone EU para data residency | Configuración soportada en DeepSeek | Deep AI Chat | https://chat-deep.ai/guide/deepseek-r1-on-azure-ai-foundry/ |
| DeepSeek vía Azure markup | +20–35% sobre directo | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash directo | $0.14 / $0.28 por MTok | TechJack Solutions | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| Claude Sonnet 4.6 oficial | $3 / $15 por MTok | Anthropic oficial | https://platform.claude.com/docs/en/about-claude/pricing |
| GPT-5.5 oficial | $5 / $30 por MTok | OpenAI | https://openai.com/api/pricing/ |
