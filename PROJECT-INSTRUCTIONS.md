# Instrucciones del Proyecto — Token Costs & Optimization

Este archivo permite retomar el proyecto desde cualquier cuenta o herramienta de AI, sin depender de memoria de conversaciones anteriores.

---

## De qué se trata el proyecto

Escribir DOS cosas:
1. **Un artículo** — no tan largo, con storytelling, que atrape al lector y lo lleve por distintos niveles de conciencia sobre el consumo de tokens en IA. Con link a la documentación para profundizar.
2. **Documentación técnica detallada** — bien completa, con distintos niveles de profundidad, donde cada persona pueda ir aplicando las diferentes capas de optimización.

**El objetivo:** que a partir de este contenido, todos en el equipo técnico de Visma tengan conciencia y empiecen a aplicar optimización de tokens en su trabajo diario.

---

## Contexto que motivó el proyecto

El equipo recibió dos mensajes importantes:

**Mensaje 1 — Copilot billing:**
GitHub Copilot migra a usage-based billing el 1 de junio 2026. Cada seat incluye tokens equivalentes al precio del seat + bonus 3 meses. Después se paga por token. Los "seat tokens" son pooled a nivel enterprise Visma. La estimación es que la factura enterprise sube ~3x, pero puede variar radicalmente según cuánto usen los desarrolladores.

**Mensaje 2 — VSCode feature:**
En el último VSCode se puede habilitar `chat.tools.compressOutput.enabled` para post-procesar el output de terminal antes de enviarlo al modelo — ahorrando tokens de contexto automáticamente.

---

## Temas confirmados del artículo (12 temas)

1. Selección de modelo según tarea y costo
2. MCPs activos innecesariamente
3. Ingeniería de contexto para agentes
4. Prompt caching y memory compression
5. Azure EA discount (15-28% negociado, no número fijo)
6. DeepSeek via Azure (compliance europeo, no precio)
7. Herramientas de medición VSCode + oTel
8. `chat.tools.compressOutput.enabled` en VSCode
9. Copilot usage-based billing desde junio 2026
10. Output tokens 4-6x más caros que input
11. Auto routing Copilot/OpenRouter (Nivel 1)
12. RouteLLM + Gateways LiteLLM/Portkey (Niveles 2 y 3)

**Temas adicionales verificados:**
- Compaction API Anthropic (beta enero 2026)
- Batch API Anthropic (50% descuento)
- Extended thinking tokens (costo oculto)

---

## Pasos del proyecto

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional | ⏳ Pendiente |
| 4 | Redacción del artículo | ⏳ Pendiente |
| 5 | Redacción de documentación técnica | ⏳ Pendiente |

---

## Reglas críticas

1. **Ningún dato numérico sin fuente verificada.** Si un número no tiene URL real, se marca `[VERIFICAR]` y se busca antes de escribir.
2. **No interpolar ni estimar métricas.** Solo datos de artículos o estudios reales.
3. **Foco en costo y gobernanza.** Cada tema debe responder: ¿esto mueve la factura? ¿es accionable?
4. **Criterio editorial:** temas de accuracy o calidad solo se incluyen si el vínculo con el costo es directo e inevitable.

---

## Modelo elegido para redacción

**Claude Sonnet 4** (`claude-sonnet-4-20250514`)

Razón: mejor relación calidad/costo para escritura técnica con storytelling. Consistente con el mensaje del artículo — no usar el modelo más caro para tareas que no lo justifican. Opus 4 descartado por costo.

---

## Fuentes clave recolectadas

- https://redis.io/blog/llm-token-optimization-speed-up-apps/
- https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/
- https://medium.com/@arvisionlab/prompt-caching-for-ai-agents-how-to-cut-cost-and-latency-without-breaking-context-245dc2502b4b
- https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/
- https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- https://ai.koombea.com/blog/llm-cost-optimization
- https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies
- https://mem0.ai/blog/context-engineering-ai-agents-guide
- https://www.openempower.com/blog/github-copilot-token-based-billing-enterprise-ai-cost-governance
- https://www.lmsys.org/blog/2024-07-01-routellm/
- https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/
- https://platform.claude.com/docs/en/build-with-claude/compaction
- https://code.visualstudio.com/updates/v1_120
- https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing
- https://deploybase.ai/articles/deepseek-v3-pricing
- https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/
- https://tokenmix.ai/blog/azure-openai-cost
- https://pecollective.com/tools/anthropic-api-pricing/
- https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx
- https://claudelab.net/en/articles/api-sdk/compaction-api-context-management

---

## Cómo retomar el proyecto

1. Leer este archivo completo
2. Leer `article/outline.md` para entender la estructura del artículo
3. Leer `data/verified-metrics.md` para conocer todos los datos disponibles
4. Leer los archivos de `research/` según el tema que se vaya a trabajar
5. Continuar desde el Paso 3: buscar data adicional si hace falta, luego redactar
