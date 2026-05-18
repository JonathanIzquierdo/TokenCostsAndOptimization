# 03 — Prompt Caching y Memory Compression

El 90% de descuento que casi nadie usa. El prompt caching es probablemente la palanca de costo más potente disponible hoy.

---

## Qué es prompt caching

Almacena prefijos de prompts pre-computados (instrucciones de sistema, tool definitions, documentos de referencia) para que el modelo no los reprocese en cada request.

> Cada vez que un agente re-lee desde cero las mismas instrucciones de sistema, reglas de compliance y definiciones de herramientas — multiplicado por millones de interacciones — el desperdicio se acumula silenciosamente.

Fuente: [Parloa — Prompt Caching for Lower AI Cost and Latency 2026](https://www.parloa.com/knowledge-hub/prompt-caching/)

---

## El descuento

> Las cache reads de Anthropic cuestan 0.1x el precio base de input — **90% de descuento**. GPT-5.4 ahora también ofrece 90% de descuento en input cacheado.

Fuente: [TokenOptimize.dev — LLM Token Optimization Strategies 2026](https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies)

### Mecánica de pricing

| Operación | Costo relativo |
|-----------|---------------|
| Cache write TTL 5 min | 1.25x precio base |
| Cache write TTL 1 hora | 2.0x precio base |
| Cache read | 0.10x precio base (90% off) |

Break-even: **1 cache hit** con TTL de 5 minutos.

Fuente: [MetaCTO — Claude API Pricing 2026](https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration)

### Ejemplo concreto

App RAG con 50.000 tokens en system prompt, 1.000 queries/día:
- Sin caching: full input price en cada request
- Con caching: **88–95% de ahorro** en el costo del system prompt

Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

---

## Errores más comunes

> La mayoría de los equipos no usa prompt caching, o lo usa incorrectamente.

Fuente: [TokenOptimize.dev](https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies)

1. **Cachear demasiado poco** — solo el system prompt, ignorando tool definitions y documentos
2. **Cachear demasiado** — servir instrucciones desactualizadas donde la frescura importa
3. **No separar estático del dinámico** — mezclarlos invalida el cache en cada turno

---

## Qué cachear vs qué no

**Cachear:** system prompt, tool definitions, knowledge base, few-shot examples, políticas de compliance.

**NO cachear:** estado de conversación, input del usuario, resultados de tools del turno actual, timestamps.

---

## Stack de ahorro combinado

> Un cached batch request puede costar tan solo el **5% de un request estándar no cacheado**.

Fuente: [PECollective — Anthropic API Pricing 2026](https://pecollective.com/tools/anthropic-api-pricing/)

- Base: 100% → Batch API: 50% → Con caching: ~5%

---

## Memory compression

> La compresión inteligente de memoria puede reducir el uso de tokens en hasta un **80%** preservando fidelidad de la información.

Fuente: [Mem0 — Context Engineering AI Agents Guide](https://mem0.ai/blog/context-engineering-ai-agents-guide)

**Cómo funciona:** analiza historial → extrae hechos relevantes → descarta ruido → almacena representación comprimida → inyecta solo lo relevante en el próximo turno.

---

## Semantic caching

> El semantic caching puede reducir los costos de API hasta un **73%**, eliminando la llamada de inferencia en cache hits.

Fuente: [Redis — LLM Token Optimization 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)

Diferencia clave:
- **Prompt caching**: cachea el procesamiento de tokens de input estáticos
- **Semantic caching**: cachea respuestas completas para queries semánticamente similares

---

## Métricas verificadas

| Estrategia | Ahorro | Fuente |
|------------|--------|--------|
| Prompt caching (cache reads) | 90% en tokens cacheados | Anthropic docs |
| Memory compression | hasta 80% | Mem0 |
| Semantic caching | hasta 73% costos API | Redis |
| Batch + caching combinados | hasta 95% vs estándar | PECollective |

---

## Archivos relacionados
- `08-batch-and-compaction.md` — Batch API y Compaction API
