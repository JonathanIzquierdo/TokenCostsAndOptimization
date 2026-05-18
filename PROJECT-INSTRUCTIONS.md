# PROJECT INSTRUCTIONS — Token Costs & Optimization

Este archivo es la fuente de verdad para retomar el proyecto desde cualquier cuenta, herramienta o sesión de AI. Contiene todo el contexto necesario para continuar sin depender de conversaciones previas.

---

## QUÉ ES ESTE PROYECTO

### El objetivo
Escribir DOS entregables:

1. **Un artículo** — con storytelling, no tan largo, que atrape al lector y lo lleve por distintos niveles de conciencia sobre el consumo de tokens en IA. Tono: un colega que descubrió algo importante y lo comparte. Incluye link a la documentación para profundizar.

2. **Documentación técnica detallada** — bien completa, con distintos niveles de profundidad, donde cada persona del equipo pueda ir aplicando las diferentes capas de optimización según su rol y contexto.

### La audiencia
Equipo técnico enterprise de Visma — desarrolladores y tech leads. No necesita explicar qué es un LLM ni qué es un token. Ya lo saben.

### El idioma
Español.

### Por qué ahora
Dos eventos concretos dispararon el proyecto:

**Evento 1 — Copilot usage-based billing:**
GitHub Copilot migra a usage-based billing el 1 de junio 2026. Cada seat incluye tokens equivalentes al precio del seat ($19 Business, $39 Enterprise) + bonus 3 meses. Después se paga por token consumido. Los seat tokens son pooled a nivel enterprise Visma. La estimación interna es que la factura enterprise sube ~3x, pero puede variar radicalmente según cuánto usen los desarrolladores.

**Evento 2 — VSCode feature:**
En el último VSCode se puede habilitar `chat.tools.compressOutput.enabled` para post-procesar el output de terminal antes de enviarlo al modelo — ahorrando tokens de contexto automáticamente. Esto motivó explorar qué más existe.

---

## ESTADO ACTUAL DEL PROYECTO

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional | ⏳ Pendiente |
| 4 | Redacción del artículo | ⏳ Pendiente |
| 5 | Redacción de documentación técnica | ⏳ Pendiente |

**Próximo paso al retomar:** Revisar el outline en `article/outline.md` y decidir si hace falta buscar más data antes de redactar. Si el outline está aprobado, ir directo a la redacción.

---

## TEMAS CONFIRMADOS (12 temas + 3 adicionales)

### Temas principales
1. **Selección de modelo según tarea y costo** → `research/02-model-selection.md`
2. **MCPs activos innecesariamente** → `research/02-model-selection.md`
3. **Ingeniería de contexto para agentes** → `research/01-context-and-token-waste.md`
4. **Prompt caching y memory compression** → `research/03-prompt-caching.md`
5. **Azure EA discount (rango negociado 15-28%, NO número fijo)** → `research/07-azure-deepseek.md`
6. **DeepSeek via Azure (compliance europeo, no precio)** → `research/07-azure-deepseek.md`
7. **Herramientas de medición VSCode + oTel** → `research/06-vscode-tools.md`
8. **`chat.tools.compressOutput.enabled` en VSCode** → `research/06-vscode-tools.md`
9. **Copilot usage-based billing desde junio 2026** → `research/05-copilot-billing.md`
10. **Output tokens 4-6x más caros que input** → `research/01-context-and-token-waste.md`
11. **Auto routing Copilot/OpenRouter (Nivel 1)** → `research/04-model-routing.md`
12. **RouteLLM + Gateways LiteLLM/Portkey (Niveles 2 y 3)** → `research/04-model-routing.md`

### Temas adicionales verificados
- **Compaction API Anthropic** (beta enero 2026) → `research/08-batch-and-compaction.md`
- **Batch API Anthropic** (50% descuento) → `research/08-batch-and-compaction.md`
- **Extended thinking tokens** (costo oculto) → `research/09-extended-thinking-costs.md`

---

## TEMAS DESCARTADOS Y POR QUÉ

### "Lost in the middle" (contexto largo degrada accuracy)
**Descartado.** El artículo es sobre costos y gobernanza de tokens, no sobre calidad de respuestas. El argumento de que el contexto largo puede degradar la performance es un tema de prompt engineering, no de economía de tokens. El vínculo con el costo es demasiado indirecto. Si aparece naturalmente al hablar de context engineering, se menciona de paso — no como punto principal.

### Criterio general para descartar temas
Cada tema nuevo debe pasar este filtro antes de incluirse:
- ¿Esto mueve la factura directamente?
- ¿Es accionable para el equipo esta semana?
- ¿El vínculo con el costo es directo e inevitable, o hay que construir un puente largo?

Si la respuesta a las primeras dos es "no" o "quizás", se descarta o se menciona de paso.

---

## REGLAS CRÍTICAS

### Regla 1: Ningún número sin fuente verificada
Ningún dato numérico o porcentaje puede usarse en el artículo sin una URL de artículo o estudio real. Si un número no tiene fuente, se marca `[VERIFICAR]` y se busca antes de escribir. No interpolar ni estimar métricas.

Todos los datos verificados están en `data/verified-metrics.md` con sus fuentes.

### Regla 2: Foco en costo y gobernanza
Temas de accuracy o calidad de respuesta solo se incluyen si el vínculo con el costo es directo e inevitable. Evitar desvíos hacia prompt engineering o performance de modelo por sí solos.

### Regla 3: No asumir que el lector no sabe nada
La audiencia son desarrolladores y tech leads de Visma. No explicar qué es un token, qué es un LLM, ni conceptos básicos. Ir directo al punto.

### Regla 4: Tono de colega, no de manual corporativo
El artículo debe sentirse como alguien del equipo que descubrió algo importante y lo comparte — no como documentación oficial ni como post de marketing.

---

## DECISIONES TOMADAS

### Modelo para redacción
**Claude Sonnet 4** (`claude-sonnet-4-20250514`)

Por qué Sonnet 4 y no Opus 4: Opus 4 es más poderoso pero más caro. Este proyecto no requiere razonamiento complejo — es escritura técnica con estructura clara y datos ya recolectados. Usar Opus 4 sería exactamente el anti-patrón que el artículo critica. Sonnet 4 da la mejor relación calidad/costo para esta tarea, y además es coherente con el mensaje del artículo.

### Azure "24% de descuento"
El número 24% que circula internamente en Visma es un acuerdo negociado específico, no un precio de lista publicado. El rango verificado es 15-28% dependiendo del leverage comercial (EA solo vs EA + MACC). En el artículo debe presentarse como rango negociable, no como número fijo.

### DeepSeek via Azure
El ángulo correcto NO es ahorro de costo (Azure cobra 20-35% más que DeepSeek directo). El ángulo es compliance: accedés a un modelo muy barato sin enviar datos a China, manteniendo data residency europeo. Incluso con el markup de Azure, DeepSeek via Azure es ~15x más barato que Claude Sonnet para tareas de alta volumetría.

---

## DATOS VERIFICADOS — RESUMEN EJECUTIVO

Los datos completos están en `data/verified-metrics.md`. Aquí los más importantes:

| Dato | Valor | Fuente |
|------|-------|--------|
| Contexto re-enviado = % de la factura | 62% | LeanOps 2026 (30 equipos) |
| Tokens en conversación de 20 turnos | 5K-10K innecesarios | Redis 2026 |
| Tokens turno 1 vs turno 50 (agentes) | 5K → 200K | Redblink |
| Multiplicador output vs input | 4–6x | Redis 2026 |
| Descuento prompt caching (Anthropic) | 90% en cache reads | Anthropic oficial |
| RouteLLM: calidad GPT-4 con % llamadas | 95% calidad con 26% llamadas | LMSYS ICLR 2025 |
| RouteLLM con augmentation: ahorro | 75% reducción | LMSYS ICLR 2025 |
| Copilot top usuarios = % del spend | 10-15% usuarios = 60-70% | Synapx |
| Multiplicador workflows agénticos | ~3.5x flat fee anterior | Synapx |
| Copilot Auto Mode descuento | 10% en multiplicador | GitHub Changelog |
| Batch API descuento | 50% en input y output | Anthropic oficial |
| Batch + caching combinados | hasta 95% de ahorro | PECollective |
| Extended thinking: display omitted ahorra | $0 (se cobra igual) | CheckThat.ai |
| DeepSeek via Azure markup | +20-35% sobre directo | DeployBase |
| Azure EA discount | 15-28% (negociado) | Microsoft Negotiations |

---

## FUENTES CLAVE

- https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/
- https://redis.io/blog/llm-token-optimization-speed-up-apps/
- https://redblink.com/ai-token-cost-optimization/
- https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies
- https://mem0.ai/blog/context-engineering-ai-agents-guide
- https://ai.koombea.com/blog/llm-cost-optimization
- https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/
- https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises
- https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/
- https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/
- https://www.openempower.com/blog/github-copilot-token-based-billing-enterprise-ai-cost-governance
- https://www.lmsys.org/blog/2024-07-01-routellm/
- https://github.com/lm-sys/RouteLLM
- https://openrouter.ai/docs/guides/routing/routers/auto-router
- https://openrouter.ai/works-with-openrouter/github-copilot
- https://www.edenai.co/post/best-llm-routers
- https://techsy.io/en/blog/best-llm-gateway-tools
- https://www.finout.io/blog/anthropic-api-pricing
- https://pecollective.com/tools/anthropic-api-pricing/
- https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration
- https://platform.claude.com/docs/en/build-with-claude/compaction
- https://claudelab.net/en/articles/api-sdk/compaction-api-context-management
- https://checkthat.ai/brands/anthropic/pricing
- https://platform.claude.com/docs/en/build-with-claude/extended-thinking
- https://code.visualstudio.com/updates/v1_120
- https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx
- https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing
- https://atonementlicensing.com/blog/openai-enterprise-pricing/
- https://tokenmix.ai/blog/azure-openai-cost
- https://deploybase.ai/articles/deepseek-v3-pricing
- https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/
- https://medium.com/@arvisionlab/prompt-caching-for-ai-agents-how-to-cut-cost-and-latency-without-breaking-context-245dc2502b4b
- https://www.parloa.com/knowledge-hub/prompt-caching/
- https://blogs.idc.com/2025/11/17/the-future-of-ai-is-model-routing/
- https://www.swfte.com/blog/intelligent-llm-routing-multi-model-ai
- https://www.augmentcode.com/guides/ai-model-routing-guide
- https://www.mindstudio.ai/blog/what-is-ai-model-router-optimize-cost-llm-providers
- https://inworld.ai/resources/best-llm-router-ai-gateway
- https://www.pkgpulse.com/guides/portkey-vs-litellm-vs-openrouter-llm-gateway-2026
- https://www.cloudzero.com/blog/claude-api-pricing/
- https://deepwiki.com/anthropics/claude-cookbooks/6.3-context-management-and-compaction
- https://blog.logrocket.com/llm-context-problem-strategies-2026/
- https://www.getmaxim.ai/articles/context-engineering-for-ai-agents-production-optimization-strategies/
- https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing-signaling-new-cost-model-for-enterprise-ai-tools.html

---

## CÓMO RETOMAR EL PROYECTO

### Para retomar con cualquier AI (Claude, ChatGPT, etc.)

Compartir este mensaje al inicio de la conversación:

> "Estoy trabajando en un proyecto de artículo técnico sobre optimización de tokens en IA para un equipo enterprise. El proyecto está documentado en GitHub en JonathanIzquierdo/TokenCostsAndOptimization. Lee el archivo PROJECT-INSTRUCTIONS.md para entender el contexto completo, luego lee article/outline.md para ver la estructura, y data/verified-metrics.md para los datos disponibles. El estado actual es: Pasos 1 y 2 completados, próximo paso es revisar el outline y arrancar la redacción del artículo."

### Archivos clave por orden de lectura
1. `PROJECT-INSTRUCTIONS.md` ← este archivo
2. `article/outline.md` ← estructura del artículo
3. `data/verified-metrics.md` ← todos los datos con fuentes
4. `research/0X-*.md` ← profundidad por tema específico

### Flujo recomendado para continuar
1. Revisar `article/outline.md` — ¿la estructura tiene sentido? ¿falta algo?
2. Si hace falta más data: buscar en web, agregar a `data/verified-metrics.md` y al archivo de research correspondiente
3. Redactar el artículo siguiendo el outline, usando los datos de `verified-metrics.md`
4. Redactar la documentación técnica (los archivos de `research/` son la base)
5. Review final: verificar que todos los números tienen fuente, el tono es correcto, y los links a la documentación funcionan
