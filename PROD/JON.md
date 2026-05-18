# JON — Índice Ejecutivo: Optimización de Tokens en IA

Resumen de todos los temas cubiertos en la documentación técnica completa (`technical-docs-draft-v1.md`). Sin código, sin lujo de detalles. El objetivo es entender qué hay, por qué importa, y cuánto mueve la factura.

---

## El problema base

**El 62% de la factura de AI** no es por el trabajo que hace el modelo. Es contexto repetido que le estamos enviando de nuevo en cada request. (LeanOps, auditoría 30 equipos, 2026)

Los **output tokens cuestan 4–6x más** que los input tokens. Nadie limita los outputs.

Una sesión agéntica que empieza en 5.000 tokens puede llegar a **200.000 tokens por llamada en el turno 50**. Sin que nadie lo vea, sin que nadie lo pare.

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

**DeepSeek via Azure** — ~$0.19/MTok input via Azure — **~15x más barato que Sonnet**. La API directa de DeepSeek no es viable para Visma (datos rutean por servidores chinos). Azure resuelve el compliance europeo con un markup de 20–35%.

**Negociación EA Microsoft** — Rango documentado: 15–25% con EA solo, **23–28% con EA + Azure MACC**. No es un precio publicado — es negociado.

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
|---------|-----------------|-------------|
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
