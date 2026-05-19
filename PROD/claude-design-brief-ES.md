# Brief para Claude Design — article-draft-v1.md

Este archivo contiene todas las instrucciones, datos y fuentes verificadas para que Claude Design agregue gráficos y métricas visuales al artículo `PROD/article-draft-v1.md`.

---

## REGLA CRÍTICA — Ningún número sin fuente verificada

**Cada gráfico, diagrama o métrica visual debe tener su fuente linkeada directamente debajo o en el caption.** Ningún número puede aparecer en un visual sin una URL real que lo respalde. Si un número no está en `data/verified-metrics.md` con URL verificada, no puede usarse en un visual.

Esto es no negociable. Aplica a cada gráfico, cada porcentaje, cada cifra en dólares.

---

## Contexto del artículo

**Archivo a mejorar:** `PROD/article-draft-v1.md`
**Audiencia:** Equipos de ingeniería enterprise (contexto Visma) — desarrolladores y tech leads
**Tono:** Técnico pero accesible. Un colega compartiendo algo importante — no un manual corporativo.
**Objetivo de los visuales:** Hacer que los datos golpeen más fuerte. Los números son sólidos — los visuales deben hacerlos imposibles de ignorar.

---

## Visualizaciones propuestas con datos verificados

### Visual 1 — A dónde va el dinero realmente (Apertura)

**Tipo:** Pie chart o donut chart
**Mensaje:** La mayor parte de la factura es ruido repetido, no trabajo real.

| Segmento | Valor | Fuente |
|----------|-------|--------|
| Contexto re-enviado (desperdicio) | 62% | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Trabajo nuevo real | 38% | Derivado del anterior (100% - 62%) |

**Caption:** *"El 62% de tu factura de AI es contexto que el modelo ya procesó. Fuente: LeanOps, auditoría de 30 equipos en producción, mayo 2026."*

---

### Visual 2 — La explosión del contexto en una sesión (Sección 1)

**Tipo:** Gráfico de línea
**Mensaje:** El costo por llamada crece exponencialmente en sesiones agénticas largas.

| Turno | Tokens por llamada |
|-------|--------------------|
| 1 | 5.000 |
| 10 | ~15.000 |
| 20 | ~40.000 |
| 30 | ~80.000 |
| 40 | ~140.000 |
| 50 | 200.000 |

**Fuente:** https://redblink.com/ai-token-cost-optimization/ y https://redis.io/blog/llm-token-optimization-speed-up-apps/
**Caption:** *"Una sesión que empieza en 5.000 tokens puede llegar a 200.000 tokens por llamada en el turno 50, sin que nadie lo vea. Fuente: Redblink AI Token Cost Optimization 2026."*

---

### Visual 3 — El multiplicador de output tokens (Sección 1)

**Tipo:** Bar chart comparando precio input vs output por modelo
**Mensaje:** Los output tokens cuestan 4–6x más. Nadie los limita.

| Modelo | Input /MTok | Output /MTok | Multiplicador |
|--------|------------|-------------|---------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | 5x |
| Claude Sonnet 4.6 | $3.00 | $15.00 | 5x |
| Claude Opus 4.7 | $5.00 | $25.00 | 5x |

**Fuente:** https://www.finout.io/blog/anthropic-api-pricing (verificado abril 2026)
**Caption:** *"Los output tokens cuestan 5x más que los input en todos los modelos actuales de Anthropic. Fuente: Finout, Anthropic API Pricing 2026."*

---

### Visual 4 — La distribución del gasto en Copilot (Sección 2)

**Tipo:** Bar chart o gráfico de Pareto
**Mensaje:** Un grupo pequeño de usuarios genera la mayor parte del costo.

| Grupo | % de usuarios | % del gasto |
|-------|--------------|-------------|
| Usuarios intensivos | 10–15% | 60–70% |
| Resto del equipo | 85–90% | 30–40% |

**Fuente:** https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/
**Caption:** *"El top 10–15% de usuarios concentra el 60–70% del gasto de AI. Fuente: Synapx, GitHub Copilot Usage-Based Billing Executive Guide 2026."*

---

### Visual 5 — La brecha de precios entre modelos (Sección 3, Capa 2)

**Tipo:** Horizontal bar chart
**Mensaje:** La diferencia de precio entre modelos es enorme — y la mayoría de los equipos la ignora.

| Modelo | Input /MTok | Caso de uso |
|--------|------------|-------------|
| Claude Haiku 4.5 | $1.00 | Clasificación, extracción, formateo |
| Claude Sonnet 4.6 | $3.00 | Código general, chat, summarización |
| Claude Opus 4.7 | $5.00 | Razonamiento complejo, arquitectura |

**Fuente:** https://www.finout.io/blog/anthropic-api-pricing
**Caption:** *"Haiku es 5x más barato que Sonnet y Opus en input. La mayoría de los workflows usa Sonnet u Opus para tareas que Haiku resuelve igual. Fuente: Precios oficiales Anthropic, abril 2026."*

---

### Visual 6 — El stack de ahorro acumulado (Sección 3, Capa 4)

**Tipo:** Waterfall chart o stacked bar
**Mensaje:** Las optimizaciones se acumulan — podés llegar al 95% de ahorro.

| Escenario | Costo relativo al baseline |
|-----------|---------------------------|
| Request estándar (baseline) | 100% |
| Con Batch API | 50% |
| Con prompt caching (porción cacheada) | ~10% |
| Batch + caching combinados | ~5% |

**Fuentes:**
- Batch API 50%: https://www.finout.io/blog/anthropic-api-pricing
- Batch + caching = 5%: https://pecollective.com/tools/anthropic-api-pricing/
- Descuento caching 90%: https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies

**Caption:** *"Batch API solo reduce el costo a la mitad. Combinado con prompt caching, un request puede costar tan solo el 5% de la tarifa estándar. Fuentes: Anthropic oficial, PECollective 2026."*

---

### Visual 7 — Las capas de optimización (Sección 3, vista general)

**Tipo:** Pirámide o diagrama por niveles
**Mensaje:** Hay capas de optimización — podés empezar en cualquier punto y obtener valor inmediato.

```
Capa 4 — Infraestructura (Batch, DeepSeek, negociación EA)
Capa 3 — Routing inteligente (RouteLLM, LiteLLM, Portkey)
Capa 2 — Selección de modelo (el correcto para cada tarea)
Capa 1 — Arquitectura de contexto (caching, RAG, historial)
Capa 0 — Configuraciones (Auto Mode, compressOutput, tool search) ← Empezar acá
```

No necesita datos numéricos — es estructural.
**Nota:** *"Cada capa es independiente — la Capa 0 toma 5 minutos y cero cambios de código."*

---

### Visual 8 — Resultados de RouteLLM (Sección 3, Capa 3)

**Tipo:** Comparación de barras o callout boxes
**Mensaje:** El routing basado en ML logra calidad casi igual a GPT-4 con una fracción de las llamadas.

| Enfoque | % llamadas a GPT-4 | Calidad vs GPT-4 | Ahorro |
|---------|-------------------|------------------|--------|
| Siempre GPT-4 | 100% | 100% | 0% |
| RouteLLM (matrix factorization) | 26% | 95% | ~48% |
| RouteLLM con data augmentation | 14% | 95% | ~75% |

**Fuente:** https://www.lmsys.org/blog/2024-07-01-routellm/ (LMSYS / UC Berkeley, ICLR 2025)
**Caption:** *"RouteLLM logra el 95% de la calidad de GPT-4 usando solo el 26% de las llamadas. Con data augmentation: misma calidad, 14% de las llamadas. Fuente: LMSYS, RouteLLM ICLR 2025."*

---

## Guía de diseño

**Paleta de colores:** 2–3 colores consistentes. Sugerencia: neutro oscuro para baselines, acento fuerte (rojo o naranja) para el problema, acento positivo (verde o azul) para ahorros/soluciones.

**Tipografía:** Limpia, técnica. Sin fuentes decorativas.

**Cada visual debe tener:**
1. Un título corto (qué muestra)
2. Un caption con la URL de la fuente
3. Sin números que no estén en este brief

**Formato:** El artículo es markdown. Los visuales pueden ser:
- Diagramas Mermaid (se renderizan nativamente en GitHub)
- SVG embebido en markdown
- PNG/SVG adjunto referenciado con `![alt](path)`

**Ubicación:** Cada visual va inmediatamente después del párrafo que introduce su dato. No agrupar visuales — distribuirlos a lo largo del artículo.

---

## Referencia de fuentes — todas las URLs verificadas

| Dato | URL |
|------|-----|
| 62% contexto re-enviado | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| 5K → 200K tokens turno 50 | https://redblink.com/ai-token-cost-optimization/ |
| Output 4–6x más caro que input | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Precios modelos Anthropic 2026 | https://www.finout.io/blog/anthropic-api-pricing |
| Top 10–15% usuarios = 60–70% spend | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Workflows agénticos ~3.5x flat fee | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Prompt caching 90% descuento | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Batch API 50% descuento | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching = 5% del baseline | https://pecollective.com/tools/anthropic-api-pricing/ |
| RouteLLM 95% calidad / 26% llamadas | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con augmentation 75% | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Auto Mode 10% descuento | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| VSCode tool search 20% ahorro | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| DeepSeek via Azure ~15x más barato | https://deploybase.ai/articles/deepseek-v3-pricing |
| Azure EA 15–28% negociado | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Opus 4.7 tokenizer +35% tokens | https://www.finout.io/blog/anthropic-api-pricing |
