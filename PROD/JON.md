# JON — Índice Ejecutivo: Optimización de Tokens en IA

Resumen de todos los temas cubiertos en la documentación técnica completa (`technical-docs-draft-v1.md`). Sin código, sin lujo de detalles. El objetivo es entender qué hay, por qué importa, y cuánto mueve la factura.

---

## El problema base

El **62% de la factura de AI** no es por el trabajo que hace el modelo. Es contexto repetido que le estamos enviando de nuevo en cada request. (LeanOps, auditoría 30 equipos, 2026)

Los **output tokens cuestan 4–6x más** que los input tokens. Nadie limita los outputs.

Una sesión agéntica que empieza en 5.000 tokens puede llegar a **200.000 tokens por llamada en el turno 50**. Sin que nadie lo vea, sin que nadie lo pare.

---

## Nivel 0 — Configuraciones inmediatas (5 minutos, sin código)

**Auto Mode en Copilot** — Cambiar el selector de modelo a "Auto" en VSCode. Copilot elige el modelo más eficiente por tarea. 10% de descuento automático en el multiplicador del modelo. Cero configuración adicional.

**chat.tools.compressOutput.enabled** — Una línea en `settings.json`. VSCode comprime el output de terminal antes de enviarlo al modelo: colapsa diffs sin cambios, descarta lockfiles, elimina barras de progreso. Reduce significativamente los tokens de contexto en sesiones con operaciones de terminal frecuentes.

**Tool search (MCPs diferidos)** — Activo por defecto en Anthropic. Divide las herramientas en core (~30, siempre activas) y diferidas (se cargan solo cuando se necesitan). Hasta 20% de ahorro en tokens por request. 10 MCPs activos que no usás = potencialmente $1.500/mes de overhead puro en 100K requests.

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

El mapa no es rígido pero la pregunta siempre es la misma: ¿esta tarea realmente necesita razonamiento profundo, o es pattern matching?

**Extended thinking** — Los tokens de razonamiento interno se facturan como output. Usar `display: omitted` NO reduce el costo — solo oculta el razonamiento al usuario. 500 tokens de respuesta visible + 2.000 de thinking = **5x más caro** que sin thinking. Reservar para tareas que genuinamente lo requieren.

---

## Nivel 3 — Model routing automático

En lugar de decidir manualmente qué modelo usar, un sistema decide por cada request.

**Auto Mode Copilot + OpenRouter** — El nivel más simple. Zero setup. Auto Mode da 10% de descuento. OpenRouter enruta entre 300+ modelos powered by NotDiamond, sin markup.

**RouteLLM** (open source, UC Berkeley/LMSYS, ICLR 2025) — Usa ML para predecir qué modelo performará mejor por query. El router de matrix factorization logra el **95% de la calidad de GPT-4 usando solo el 26% de las llamadas a GPT-4** — 48% más barato que baseline. Con data augmentation: **75% de reducción de costo**. Drop-in replacement del cliente de OpenAI — no requiere reescribir la aplicación.

**AI Gateways enterprise** (LiteLLM, Portkey, OpenRouter) — Capas entre la aplicación y los providers. El argumento más importante no es el routing — es la protección: sin un gateway, un solo agente descontrolado puede quemar el presupuesto mensual en una noche.

- LiteLLM: open source, self-hosted, budget controls por equipo o API key
- Portkey: managed, 1.600+ modelos, guardrails, PII filtering, SOC2
- OpenRouter: zero setup, 300+ modelos, 5.5% markup

---

## Nivel 4 — Palancas de infraestructura

**Batch API** — Para workloads sin necesidad de respuesta en tiempo real. **50% de descuento** en input y output, calidad idéntica. Combinado con prompt caching: hasta **95% de ahorro** vs un request estándar. Ideal para: clasificación masiva, generación de tests, análisis nocturnos, evaluaciones de modelos.

**DeepSeek via Azure** — DeepSeek V4-Flash es ~$0.19/MTok input via Azure — **~15x más barato que Sonnet** para tareas de alta volumetría. La API directa de DeepSeek no es viable para Visma (datos rutean por servidores chinos). Azure resuelve el compliance europeo con un markup de 20–35% sobre el precio directo — que igual sigue siendo muchísimo más barato que los modelos frontier.

**Negociación EA Microsoft** — El descuento en Copilot via Enterprise Agreement no es fijo — es negociado. Rango documentado: 15–25% con EA solo, **23–28% con EA + Azure MACC**. Requiere estructurar la conversación en el contexto del renewal, no negociar Copilot en aislamiento.

---

## Nivel 5 — Observabilidad con OpenTelemetry

Sin instrumentación, volás a ciegas. El billing te muestra el total del mes — no quién lo genera, qué workflow lo causa, ni cómo evoluciona en el tiempo.

**Claude Code tiene OTel nativo** — Desactivado por defecto. Una variable de entorno lo enciende: `CLAUDE_CODE_ENABLE_TELEMETRY=1`. A partir de ahí exporta tokens (input, output, cache read, cache write), latencia, y cada tool call, a cualquier backend OTLP.

**Copilot Chat también tiene OTel** — Activo desde VSCode settings. Produce una jerarquía completa de spans: request al LLM, tool calls, subagentes, duración, tokens.

**Stack local en 30 segundos** — El Aspire Dashboard de Microsoft corre en Docker y da un trace viewer completo sin necesidad de cuenta en la nube.

**Stack de producción** — OTel Collector + Prometheus + Grafana via docker-compose. Los tokens de Claude Code y Copilot llegan al mismo pipeline que el resto de la infra.

**Configuración centralizada** — Via managed settings distribuible por MDM, cada desarrollador del equipo queda instrumentado automáticamente sin configuración individual.

**Atributos de filtrado disponibles:** modelo usado, provider, herramienta ejecutada, equipo, departamento, usuario. Permiten responder: ¿quién consume más? ¿qué workflow es el más caro? ¿cuál es el cache hit rate real?

Backends compatibles (todos hablan OTLP, sin lock-in): Grafana, Jaeger, Datadog, Honeycomb, Langfuse, SigNoz, Azure Monitor.

---

## Nivel 6 — Proveedores alternativos de menor costo

El mercado de inference APIs en 2026 tiene 11+ providers en producción. Los precios varían **625x** entre el más barato y el más caro. La mayoría de los equipos paga entre 4 y 30x más de lo necesario para muchas tareas.

**Groq** — Silicon propio (LPU) diseñado para inferencia de transformers. 500–800 tokens/segundo. Llama 4 a ~$0.11/MTok input — 4–10x más barato que GPT-4o. Ideal para tareas latency-sensitive simples. No para razonamiento complejo.

**Together AI** — Más de 100 modelos open-source, fine-tuning disponible. Precios desde $0.14/MTok. Ideal para batch de alta volumetría y workloads donde querés fine-tuning en datos propios.

**Fireworks AI** — Production-grade con SLAs formales, function calling optimizado. Para producción que necesita garantía formal de disponibilidad.

**Mistral** — El único provider major con sede en Europa. GDPR nativo, precios por debajo de OpenAI, modelos open-weight disponibles para self-hosting. Para Visma, la opción más directa cuando se necesita data residency EU sin pasar por Azure.

**Google Gemini Flash** — $0.075/MTok input — el modelo de alta calidad más barato de un provider major en 2026. Contexto de 1M tokens. Evaluar para workloads con contextos muy largos.

**Self-hosting con vLLM** — Elimina el costo por token. Solo tiene sentido a partir de $5K–10K/mes en API spend y con 70%+ de utilización de GPU sostenida. Debajo de esos umbrales, los API providers ganan en costo.

**La estrategia no es migrar todo** — es routing por tipo de tarea. Razonamiento complejo: Claude/GPT-5. Clasificación masiva: Groq. Batch offline: Together AI. Data residency EU: Mistral. Compliance europeo + costo mínimo: DeepSeek via Azure.

---

## Resumen de métricas clave

| Palanca | Ahorro / Impacto | Complejidad |
|---------|-----------------|-------------|
| Auto Mode Copilot | 10% descuento | ⭐ Inmediata |
| compressOutput + tool search | hasta 20% tokens | ⭐ Inmediata |
| Prompt caching | 90% en tokens cacheados | ⭐⭐ Baja |
| RAG vs context stuffing | hasta 70% reducción | ⭐⭐ Baja |
| Selección correcta de modelo | hasta 25x diferencia | ⭐⭐ Baja |
| Batch API | 50% descuento | ⭐⭐ Baja |
| Batch + caching combinados | hasta 95% ahorro | ⭐⭐ Baja |
| RouteLLM | 75% reducción llamadas modelo fuerte | ⭐⭐⭐ Media |
| AI Gateway (LiteLLM/Portkey) | Budget controls + protección | ⭐⭐⭐ Media |
| OTel observabilidad | Visibilidad completa | ⭐⭐⭐ Media |
| DeepSeek via Azure | ~15x más barato que Sonnet | ⭐⭐⭐ Media |
| Groq/Together/Mistral | 4–30x más barato que frontier | ⭐⭐⭐⭐ Alta |
| Self-hosting vLLM | 100% ahorro en tokens (infra propia) | ⭐⭐⭐⭐⭐ Alta |

---

*Documentación completa con código, ejemplos y detalles de implementación: `PROD/technical-docs-draft-v1.md`*
