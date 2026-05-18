# 08 — Batch API y Compaction API: Las Palancas que se Acumulan

Dos features de Anthropic que la mayoría de los equipos no tiene en su radar. Combinadas con prompt caching, pueden reducir costos hasta el 95%.

---

## Batch API: 50% de descuento sin cambio en calidad

> El Message Batches API procesa requests asincrónicamente en 24 horas al **50% del precio estándar**. No hay diferencia de calidad entre batch y tiempo real — solo en timing.

Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### Precios Batch vs estándar

| Modelo | Input estándar | Output estándar | Input Batch | Output Batch |
|--------|---------------|----------------|-------------|--------------|
| Claude Opus 4.7 | $5.00/MTok | $25.00/MTok | $2.50/MTok | $12.50/MTok |
| Claude Sonnet 4.6 | $3.00/MTok | $15.00/MTok | $1.50/MTok | $7.50/MTok |
| Claude Haiku 4.5 | $1.00/MTok | $5.00/MTok | $0.50/MTok | $2.50/MTok |

Fuente: [PECollective — Anthropic API Pricing 2026](https://pecollective.com/tools/anthropic-api-pricing/)

### Stack de ahorro combinado

> Un cached batch request puede costar tan solo el **5% de un request estándar no cacheado**.

Fuente: [PECollective](https://pecollective.com/tools/anthropic-api-pricing/)

Desglose: Base 100% → Batch API 50% → Con caching en porción estática → **~5%**

### Capacidad adicional en Batch

> Opus 4.7, Opus 4.6 y Sonnet 4.6 soportan hasta **300K output tokens por request** en Batch API usando el beta header `output-300k-2026-03-24`. Para generación long-form a escala, Batch no solo es más barato — es más capaz.

Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### Workloads ideales
- Pipelines de procesamiento de documentos
- Análisis de datos nocturnos
- Sweeps de evaluación de modelos
- Clasificación y extracción offline
- Cualquier tarea donde latencia de horas sea aceptable

---

## Compaction API: conversaciones sin costo lineal

**El problema:** en el turno 50, una sesión que empezó en 5K tokens puede estar costando 200K tokens por llamada.

> Lanzada en beta enero 2026, la Compaction API resume automáticamente el historial cuando se acerca al límite configurado. Claude genera el resumen con comprensión semántica completa — significativamente mejor que truncamiento naïve.

Fuente: [Claude Lab — Context Compaction API](https://claudelab.net/en/articles/api-sdk/compaction-api-context-management)

### Elegibilidad Zero Data Retention (ZDR)

> La Compaction API es elegible para arreglos de Zero Data Retention — viable para salud, finanzas y servicios legales regulados.

Fuente: [Claude Lab](https://claudelab.net/en/articles/api-sdk/compaction-api-context-management)

Relevante para Visma: compresión sin comprometer requerimientos de compliance europeos.

### Implementación básica

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages,
    context_management={
        "edits": [{"type": "compact_20260112"}]
    }
)
```

Fuente: [Anthropic Docs — Compaction](https://platform.claude.com/docs/en/build-with-claude/compaction)

### Combinación con prompt caching

> Solo el resumen de compaction necesita escribirse como nuevo cache entry — el system prompt permanece cacheado a través de múltiples compactions.

Fuente: [DeepWiki — Context Management and Compaction](https://deepwiki.com/anthropics/claude-cookbooks/6.3-context-management-and-compaction)

---

## Cuándo usar cada uno

| Herramienta | Usar cuando... | No usar cuando... |
|-------------|---------------|-------------------|
| Batch API | Latencia horas aceptable, offline processing | Usuario espera respuesta en tiempo real |
| Compaction API | Conversaciones largas, agentes multi-turno | Sesiones cortas |
| Ambos combinados | Pipelines agénticos en batch | Interacciones directas usuario-modelo |

---

## Métricas verificadas

| Feature | Ahorro | Fuente |
|---------|--------|--------|
| Batch API | 50% en input y output | Anthropic oficial |
| Batch + caching combinados | hasta 95% vs estándar | PECollective |
| Compaction API | Evita crecimiento 5K → 200K tokens | Redblink |
