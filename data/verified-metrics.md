# Métricas Verificadas con Fuentes

Todos los datos numéricos del proyecto con su fuente. Ningún número debe usarse en el artículo sin estar listado aquí con URL verificada.

**Regla:** Si un número no está en esta lista, se marca `[VERIFICAR]` y se busca antes de escribir.

---

## Desperdicio de contexto

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Contexto re-enviado % de la factura | 62% | LeanOps, 30 equipos mar–may 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens en conversación 20 turnos | 5.000–10.000 innecesarios | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Tokens turno 1 vs turno 50 (agentes) | 5K → 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Multiplicador output vs input | 4–6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Reducción tokens con compresión de memoria | hasta 80% | Mem0 | https://mem0.ai/blog/context-engineering-ai-agents-guide |
| Reducción tokens con RAG | hasta 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| Reducción costos con context engineering | 60–80% | Maxim AI | https://www.getmaxim.ai/articles/context-engineering-for-ai-agents-production-optimization-strategies/ |
| AI % del gasto total IT (algunas firmas) | hasta 50% | Deloitte 2026 via Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Subestimación costos AI Global 1000 | 30% hacia 2027 | IDC via InfoWorld | https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing-signaling-new-cost-model-for-enterprise-ai-tools.html |

---

## Prompt Caching

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Descuento cache reads Anthropic | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Cache write TTL 5 min | 1.25x precio base | MetaCTO | https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration |
| Cache write TTL 1 hora | 2.0x precio base | MetaCTO | https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration |
| Break-even caching | 1 cache hit | MetaCTO | https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration |
| Ahorro app RAG 50K tokens × 1000 queries/día | 88–95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| Semantic caching reducción costos API | hasta 73% | Redis | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Prompt caching reuse rate en VSCode | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |

---

## Model Routing

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| RouteLLM calidad con % llamadas modelo fuerte | 95% calidad con 26% llamadas | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM ahorro vs baseline | 48% más barato | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con data augmentation (llamadas) | 14% al modelo fuerte | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con data augmentation (ahorro) | 75% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Ahorro model routing inteligente | 20–80% OpEx | IDC FutureScape 2026 | https://www.einpresswire.com/article/903464074/2026-agentic-ai-era-why-multi-model-routing-has-become-a-must-have-not-a-nice-to-have |
| Descuento Copilot Auto Mode | 10% multiplicador | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Enterprises 5+ modelos en producción (2026) | 37% | Swfte AI | https://www.swfte.com/blog/intelligent-llm-routing-multi-model-ai |
| Enterprises multi-model routing para 2028 | 70% proyección | IDC FutureScape 2026 | https://blogs.idc.com/2025/11/17/the-future-of-ai-is-model-routing/ |
| Tool search VSCode ahorro | hasta 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |

---

## Copilot Billing

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Copilot Business precio | $19/usuario/mes | GitHub Docs | https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises |
| AI Credits incluidos Business | $19/usuario/mes | GitHub Docs | https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises |
| 1 AI Credit en USD | $0.01 | GitHub Docs | https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing |
| % usuarios que concentran el spend | top 10–15% = 60–70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Costo workflows agénticos intensivos | ~£52/dev/mes ≈ 3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |

---

## Batch API y Compaction

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Descuento Batch API | 50% input y output | Anthropic oficial | https://www.finout.io/blog/anthropic-api-pricing |
| Diferencia de calidad Batch vs síncrono | Ninguna | Anthropic oficial | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching combinados | hasta 95% vs estándar | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Output máximo en Batch (Opus/Sonnet 4.6) | 300K por request (beta) | Anthropic docs | https://www.finout.io/blog/anthropic-api-pricing |
| Compaction API disponibilidad | Beta desde ene 2026 | Claude Lab | https://claudelab.net/en/articles/api-sdk/compaction-api-context-management |
| Compaction elegibilidad ZDR | Sí | Claude Lab | https://claudelab.net/en/articles/api-sdk/compaction-api-context-management |

---

## Extended Thinking

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Tipo de tokens (extended thinking) | Output tokens | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Ahorro de display "omitted" en costo | Ninguno (solo latencia) | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Costo 2000 thinking + 500 output vs sin thinking | 5x más caro | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |

---

## Azure y DeepSeek

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Descuento EA solo | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Descuento EA + MACC | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure overhead vs OpenAI directo TCO | +15–40% | TokenMix | https://tokenmix.ai/blog/azure-openai-cost |
| DeepSeek via Azure markup | +20–35% sobre directo | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Claude Sonnet input | ~15x más barato | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| DeepSeek servidores en China | Sí — todos los requests | TechJack | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |

---

## Precios de referencia 2026 (verificados)

| Modelo | Input /MTok | Output /MTok | Fuente |
|--------|------------|-------------|--------|
| Claude Haiku 4.5 | $1.00 | $5.00 | Anthropic oficial |
| Claude Sonnet 4.6 | $3.00 | $15.00 | Anthropic oficial |
| Claude Opus 4.7 | $5.00 | $25.00 | Anthropic oficial |
| DeepSeek V4-Flash | $0.14 | $0.28 | DeepSeek oficial |
| DeepSeek V4-Flash via Azure (estimado) | ~$0.19 | ~$0.38 | DeployBase markup 35% |

Fuente precios Anthropic: https://www.finout.io/blog/anthropic-api-pricing (verificado abril 2026)
Fuente precios DeepSeek: https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ (verificado mayo 2026)
