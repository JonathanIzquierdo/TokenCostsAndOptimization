# JON — Índice Ejecutivo: Optimización de Tokens en IA

Resumen de todos los temas cubiertos en la documentación técnica completa (`technical-docs-draft-v1.md`). Sin código, sin lujo de detalles. El objetivo es entender qué hay, por qué importa, y cuánto mueve la factura.

---

## El problema base

**El 62% de la factura de AI** no es por el trabajo que hace el modelo. Es contexto repetido que le estamos enviando de nuevo en cada request. (LeanOps, auditoría 30 equipos, 2026)

Los **output tokens cuestan 4–6x más** que los input tokens. Nadie limita los outputs.

Una sesión agéntica que empieza en 5.000 tokens puede llegar a **200.000 tokens por llamada en el turno 50**. Sin que nadie lo vea, sin que nadie lo pare.

---

## El cambio de paradigma: vendor cerrado vs construir con API

Antes de entrar a las palancas de optimización hay una decisión fundacional que determina qué palancas existen para vos: **en qué relación estamos con el modelo.**

### Producto cerrado vs API key

**Como usuarios de producto cerrado** (Claude.ai, ChatGPT, Copilot, Cursor): el proveedor controla todos los parámetros. Pagás suscripción, consumís. Las palancas de optimización disponibles son limitadas pero existen:
- Elegir el modelo del dropdown cuando el producto lo permite
- Activar Auto Mode en Copilot (10% descuento)
- Configurar `compressOutput` y tool search en VSCode (hasta 20% ahorro)
- Auditar qué MCPs tenés activos
- Inspeccionar consumo con Agent Debug Log y Chat Debug View

Sirven, pero el techo es bajo. No podés tocar thinking budgets, no podés activar prompt caching, no podés elegir batch processing, no podés rutear a Haiku desde Sonnet, no podés setear `max_tokens`.

**Como constructores con API key** (Anthropic, OpenAI, DeepSeek, etc., consumidos desde código propio): todas las palancas son tuyas. Los ahorros del 50%, 90% y 95% que aparecen en este documento viven acá:
- Prompt caching: hasta **90% de descuento** sobre lo cacheado
- Batch API: **50% de descuento** sobre input y output, stacks con caching → hasta **95%**
- Output budgets explícitos por tarea (5–25x menos tokens en outputs caros)
- Extended thinking controlado o desactivado (evita el 5x multiplier)
- Routing entre modelos (hasta 25x diferencia entre Haiku y Opus)
- Gateways con budget controls por equipo (LiteLLM, Portkey, OpenRouter)

**Este documento asume que estás construyendo, o vas hacia ahí.** Las capas que siguen son palancas accionables desde código de aplicación, settings de IDE, o capas de routing — no desde la app de escritorio.

---

## Vendor directo vs hyperscaler: la decisión de procurement

Cuando se decide construir con API, la siguiente pregunta es **dónde comprarla**: vendor directo (Anthropic, OpenAI, DeepSeek) o vía hyperscaler (Azure OpenAI / Microsoft Foundry, AWS Bedrock, Google Vertex AI).

La intuición común — "el hyperscaler me sale más barato por el descuento EA" — es **frecuentemente falsa** o se compensa apenas. Los datos verificados, todos de fuentes 2026:

### Precios nominales por modelo (mayo 2026)

| Modelo | Vendor directo | Vía hyperscaler | Diferencia nominal |
|--------|----------------|------------------|---------------------|
| Claude Sonnet 4.6 | $3 / $15 per MTok ([Anthropic](https://platform.claude.com/docs/en/about-claude/pricing)) | Mismo precio en Bedrock global, Vertex global, Foundry ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)) | 0% nominal |
| Claude Sonnet 4.6 (regional EU) | $3 / $15 + 10% data residency | $3 / $15 + 10% regional endpoint | Mismo overhead |
| Claude Opus 4.7 | $5 / $25 per MTok | Mismo en hyperscalers | 0% nominal |
| Claude Haiku 4.5 | $1 / $5 per MTok | Mismo en hyperscalers | 0% nominal |
| GPT-5.4 | $2.50 / $15 ([OpenAI](https://openai.com/api/pricing/)) | Azure OpenAI mismo per-token | 0% nominal |
| GPT-5.5 (flagship) | $5 / $30 ([Finout](https://www.finout.io/blog/openai-pricing-in-2026)) | Azure OpenAI mismo per-token | 0% nominal |
| DeepSeek V4-Flash | $0.14 / $0.28 ([TLDL](https://www.tldl.io/resources/anthropic-api-pricing)) | Vía Azure: ~$0.19 / $0.38 | +20–35% ([DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing)) |

**Lectura:** para Claude y GPT, el precio por token nominal es **idéntico** en vendor directo y hyperscaler. Para DeepSeek hay markup directo del 20–35% vía Azure (Azure cobra premium por hostear un modelo no propio de Microsoft).

### TCO efectivo: la trampa del overhead

El problema no es el precio nominal — es lo que se acumula alrededor:

**Azure OpenAI:** TCO efectivo **15–40% por encima de OpenAI directo**. Promedio del cliente típico: **+22%**. Fuentes:
- [TokenMix, mayo 2026](https://tokenmix.ai/blog/azure-openai-cost): "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure."
- [Inference.net, enero 2026](https://inference.net/content/azure-openai-pricing-explained/): mismo dato confirmado con desglose por componente.
- [CloudZero, mayo 2026](https://www.cloudzero.com/blog/azure-openai-pricing/): "production deployments typically add 20–40% above listed token rates"

¿De dónde viene ese overhead?
- Support plans: $100–$1.000+/mes (mandatorios para producción)
- Data egress: $0.087/GB después de los primeros 100GB
- Fine-tuned model hosting: $1.70–$3/hora **aun sin uso** = $1.224–$2.160/mes
- VNet integration, Private Link, content filtering: $200–$2.000/mes
- Log Analytics y monitoring infra

**AWS Bedrock para Claude:** precio per-token idéntico al directo, **pero TCO efectivo 20–35% más alto en promedio**. ([TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)) Causas:
- Regional/multi-region endpoints: +10% sobre global (oficial de Anthropic)
- Cross-region inference: +10% adicional
- Data transfer fees (Bedrock cuenta egress)
- Bedrock cobra todos los errores HTTP 500 (la API directa tiene 3% forgiveness buffer)
- Mandatory CloudWatch / CloudTrail logging

**Microsoft Foundry para Claude:** precio per-token idéntico al directo. **Trampa documentada**: los créditos Azure / Founders Hub / MACC **NO se aplican** a Claude en Foundry — se factura como Marketplace de terceros directo a tarjeta de crédito. ([AZ365.ai marzo 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/), [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352)). Caso documentado: una startup acumuló ~$13.000 USD en un mes pensando que sus credits de Foundry cubrían Claude. No lo hicieron.

### Descuentos EA / MACC

- **Azure EA solo:** 15–25% negociado. ([Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing))
- **Azure EA + MACC:** 23–28%. (misma fuente)
- **Anthropic Enterprise / volume discounts:** disponibles, negociados directamente.
- **OpenAI Enterprise tier:** custom pricing, negociado.

### El cálculo neto para una decisión informada

| Escenario | Costo directo | Costo Azure | Costo Bedrock | Neto vs directo |
|-----------|---------------|--------------|----------------|------------------|
| GPT-5.4, sin compliance, sin descuento | 100 | 115–140 | n/a | **+15% a +40%** |
| GPT-5.4, EA solo (-20%) | 100 | 92–112 | n/a | **-8% a +12%** |
| GPT-5.4, EA + MACC (-25%) | 100 | 86–105 | n/a | **-14% a +5%** |
| Claude Sonnet, sin compliance | 100 | n/a (Foundry sin desc.) | 120–135 | **+20% a +35% Bedrock** |
| DeepSeek vs Azure (sin compliance) | 100 | 120–135 | n/a | **+20% a +35%** |
| DeepSeek vs Azure (con GDPR EU) | inviable | 120–135 | n/a | **prima de compliance** |

**Conclusión accionable**: el descuento EA/MACC típicamente **compensa pero no supera** el overhead Azure. La factura final queda entre -14% y +12% vs el vendor directo. La decisión real **no se gana con el descuento, se gana o se pierde por compliance y mandato corporativo**.

### Cuándo el hyperscaler sí vale la pena

1. **GDPR / data residency obligatorio**: la prima compra compliance que el directo no ofrece. Esencial con datos de clientes EU.
2. **Mandato corporativo "todo en Azure/AWS"**: el overhead se asume como costo de gobernanza unificada.
3. **Modelo geopolíticamente sensible**: el caso DeepSeek vía Azure. Acceso directo a DeepSeek no es viable cuando hay GDPR (datos rutean por servidores chinos). Azure resuelve compliance europeo aún con +20–35% markup — y aun así DeepSeek vía Azure sigue siendo ~15x más barato que Sonnet para tareas masivas. ([DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing))
4. **MACC burn**: si la organización ya comprometió gasto anual con Microsoft, consumir Foundry/Azure OpenAI lo aplica contra ese compromiso (excepto Claude en Foundry — ver trampa arriba).

**Si ninguno de esos cuatro aplica**, vendor directo gana en precio neto y en velocidad de features — las betas y nuevas capacidades salen primero en la API del fabricante.

### Input para negociación con vendors

Datos verificados que conviene tener encima de la mesa para acuerdos en curso:

| Dato | Valor | Fuente | Uso en negociación |
|------|-------|--------|---------------------|
| Azure overhead vs OpenAI directo | 15–40% (avg 22%) | TokenMix, Inference.net, CloudZero | "Necesitamos descuento que cubra al menos 25% para break-even" |
| Bedrock overhead vs Anthropic directo | 20–35% efectivo | TokenMix Bedrock | "Foundry o directo nos da el mismo precio sin ese markup" |
| Foundry Claude credits exclusion | Documentado | Microsoft Q&A, AZ365.ai | "Aclaración explícita en el contrato sobre qué credits aplican" |
| Anthropic Enterprise volume discount | Disponible | CloudZero, Anthropic sales | "Solicitar términos comparables a los del hyperscaler con MACC" |
| OpenAI Enterprise custom pricing | Disponible | Finout, OpenAI | "Solicitar mismo régimen que Azure pero sin overhead" |

---

## Nivel 0 — Configuraciones inmediatas (5 minutos, sin código)

**Auto Mode en Copilot** — Cambiar el selector de modelo a "Auto" en VSCode. Copilot elige el modelo más eficiente por tarea. 10% de descuento automático en el multiplicador del modelo.

**chat.tools.compressOutput.enabled** — Una línea en `settings.json`. VSCode comprime el output de terminal antes de enviarlo al modelo: colapsa diffs sin cambios, descarta lockfiles, elimina barras de progreso. Reduce significativamente los tokens de contexto en sesiones con operaciones de terminal frecuentes.

**Tool search (MCPs diferidos)** — Activo por defecto en Anthropic. Divide las herramientas en core (~30, siempre activas) y diferidas (se cargan solo cuando se necesitan). Hasta 20% de ahorro en tokens por request. 10 MCPs activos que no usás = potencialmente $1.500/mes de overhead puro en 100K requests.

**Agent Debug Log panel** — La forma más rápida de ver qué está pasando sin configurar nada externo. Muestra el trace completo de la sesión: tokens totales, tool calls, model turns, errores y un flow chart visual del agente. Se abre desde el menú `(...)` del panel de Copilot Chat → "Show Agent Debug Logs". Requiere habilitar `github.copilot.chat.agentDebugLog.enabled`. Para persistir sesiones pasadas en disco: `github.copilot.chat.agentDebugLog.fileLogging.enabled`.

**Chat Debug View** — Para inspeccionar request a request: muestra el system prompt completo, el contexto enviado, y la respuesta. Se abre desde el mismo menú `(...)` → "Show Chat Debug View".

---

## Nivel 1 — Arquitectura de contexto

**Prompt caching** — El contenido estático del sistema (instrucciones, tool definitions, knowledge base) se cachea. Las lecturas del cache cuestan el 10% del precio normal — **90% de descuento**. Una app RAG con 50.000 tokens en el system prompt, consultada 1.000 veces al día, ahorra hasta el 84% del costo de esa porción simplemente activando caching. Break-even: un solo cache hit.

**Gestión de historial** — Tres enfoques de menor a mayor sofisticación:
- Ventana deslizante: solo los últimos N turnos
- Resumen progresivo: comprimir historia vieja (usando Haiku, el modelo más barato)
- Compaction API de Anthropic (beta enero 2026): resume automáticamente al acercarse al límite, con comprensión semántica completa, elegible para Zero Data Retention

**RAG en lugar de context stuffing** — Recuperar solo los fragmentos relevantes para la query, no documentos completos. Puede reducir tokens de contexto hasta un **70%**.

**Output budgets por tarea** — Setear `max_tokens` explícitamente según el tipo de tarea. Clasificación: 50 tokens. Resumen: 500. Generación de código: 2.000. Sin esto, el modelo genera lo que quiere y los outputs (5x más caros) se van de las manos.

---

## Nivel 2 — Selección correcta de modelo

La diferencia entre Haiku y Opus es **25x en precio**. La mayoría de los workflows usa Sonnet u Opus para tareas que Haiku resuelve igual.

| Modelo | Para qué | Precio input |
|--------|----------|--------------|
| Haiku 4.5 | Clasificación, extracción, formateo, navegación de archivos | $1/MTok |
| Sonnet 4.6 | Código general, chat, summarización, debugging | $3/MTok |
| Opus 4.7 | Razonamiento complejo, arquitectura, coordinación de agentes | $5/MTok |

**Nota importante:** Opus 4.7 incluye un nuevo tokenizer que puede generar hasta un 35% más tokens para el mismo texto de input vs Opus 4.6. El precio por token no cambió, pero el costo efectivo por request puede ser hasta 35% mayor. Benchmarkear workloads antes de migrar de Opus 4.6.

**Extended thinking** — Los tokens de razonamiento interno se facturan como output. Usar `display: omitted` NO reduce el costo — solo oculta el razonamiento al usuario. 500 tokens de respuesta visible + 2.000 de thinking = **5x más caro** que sin thinking. Reservar para tareas que genuinamente lo requieren.

---

## Nivel 3 — Model routing automático

**Auto Mode Copilot + OpenRouter** — El nivel más simple. Zero setup. Auto Mode da 10% de descuento. OpenRouter enruta entre 300+ modelos powered by NotDiamond, sin markup.

**RouteLLM** (open source, UC Berkeley/LMSYS, ICLR 2025) — 95% de la calidad de GPT-4 usando solo el 26% de las llamadas — 48% más barato que baseline. Con data augmentation: **75% de reducción de costo**. Drop-in replacement del cliente de OpenAI.

**AI Gateways enterprise** (LiteLLM, Portkey, OpenRouter) — El argumento más importante no es el routing — es la protección: sin un gateway, un solo agente descontrolado puede quemar el presupuesto mensual en una noche.

---

## Nivel 4 — Palancas de infraestructura

**Batch API** — 50% de descuento en input y output, calidad idéntica. Combinado con prompt caching: hasta **95% de ahorro**. Ideal para clasificación masiva, generación de tests, análisis nocturnos.

**DeepSeek via Azure** — ~$0.19/MTok input via Azure — **~15x más barato que Sonnet**. La API directa de DeepSeek no es viable con datos EU (rutean por servidores chinos). Azure resuelve el compliance europeo con un markup de 20–35% sobre directo — pagás compliance, no descuento, y aun así es 15x más barato que Sonnet.

**Negociación EA Microsoft** — Rango documentado: 15–25% con EA solo, **23–28% con EA + Azure MACC**. No es un precio publicado — es negociado. Importante: como se ve en la sección de procurement, este descuento típicamente compensa pero no supera el overhead Azure. La decisión gana o pierde por compliance, no por descuento.

---

## Nivel 5 — Observabilidad con OpenTelemetry

Para gobernanza a nivel equipo: quién consume más, qué workflow es el más caro, cuál es el cache hit rate real.

**Claude Code tiene OTel nativo** — Una variable de entorno lo enciende: `CLAUDE_CODE_ENABLE_TELEMETRY=1`. Exporta tokens, latencia y tool calls a cualquier backend OTLP.

**Copilot Chat también tiene OTel** — Activo desde VSCode settings. Produce jerarquía completa de spans.

**Stack local en 30 segundos** — Aspire Dashboard de Microsoft en Docker. Sin cuenta en la nube.

**Configuración centralizada** — Via managed settings distribuible por MDM. Todos los developers instrumentados automáticamente.

Backends compatibles: Grafana, Jaeger, Datadog, Honeycomb, Langfuse, SigNoz, Azure Monitor.

---

## Nivel 6 — Proveedores alternativos de menor costo

El mercado de inference APIs en 2026 tiene 11+ providers. Los precios varían **625x** entre el más barato y el más caro.

**Groq** — Silicon propio (LPU). Llama 4 a ~$0.11/MTok input — 4–10x más barato que GPT-4o. Latencia sub-100ms.

**Together AI** — 100+ modelos open-source, fine-tuning disponible. Ideal para batch alta volumetría.

**Fireworks AI** — Production-grade con SLAs formales.

**Mistral** — El único provider major con sede en Europa. GDPR nativo. Opción más directa para data residency EU sin pasar por Azure.

**Google Gemini Flash** — $0.075/MTok input. El más barato del tier alto. Contexto de 1M tokens.

**Self-hosting con vLLM** — Solo tiene sentido a partir de $5K–10K/mes y con 70%+ de utilización de GPU.

---

## Resumen de métricas clave

| Palanca | Ahorro / Impacto | Complejidad |
|---------|------------------|-------------|
| Copilot Auto Mode | 10% descuento | ⭐ Inmediata |
| compressOutput + tool search | hasta 20% tokens | ⭐ Inmediata |
| Agent Debug Log + Chat Debug View | Visibilidad inmediata sin setup | ⭐ Inmediata |
| Prompt caching | 90% en tokens cacheados | ⭐⭐ Baja |
| RAG vs context stuffing | hasta 70% reducción | ⭐⭐ Baja |
| Selección correcta de modelo | hasta 25x diferencia | ⭐⭐ Baja |
| Batch API | 50% descuento | ⭐⭐ Baja |
| Batch + caching combinados | hasta 95% ahorro | ⭐⭐ Baja |
| RouteLLM | 75% reducción llamadas modelo fuerte | ⭐⭐⭐ Media |
| AI Gateway (LiteLLM/Portkey) | Budget controls + protección | ⭐⭐⭐ Media |
| OTel observabilidad | Visibilidad completa equipo | ⭐⭐⭐ Media |
| DeepSeek via Azure | ~15x más barato que Sonnet | ⭐⭐⭐ Media |
| Groq/Together/Mistral | 4–30x más barato que frontier | ⭐⭐⭐⭐ Alta |
| Self-hosting vLLM | 100% ahorro en tokens (infra propia) | ⭐⭐⭐⭐⭐ Alta |

---

*Documentación completa con código, ejemplos y detalles de implementación: `PROD/technical-docs-draft-v1.md`*
