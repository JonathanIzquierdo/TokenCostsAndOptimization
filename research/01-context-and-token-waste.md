# 01 — Contexto y Desperdicio de Tokens

El mayor problema de costo en producción no es el modelo que elegiste. Es el contexto que estás enviando sin necesidad, una y otra vez.

---

## El número que lo explica todo

> **El contexto re-enviado representa el 62% de la factura.**

Fuente: [LeanOps — Agentic AI Cost Runaway 2026](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/)
Metodología: Auditoría de 30 equipos de ingeniería corriendo IA agéntica en producción, marzo–mayo 2026.

Más de la mitad de lo que pagás en AI no es por el trabajo nuevo que hace el modelo — es por contexto repetido que ya procesó antes.

---

## Cómo el contexto crece hasta devorarte el presupuesto

### El historial acumulado

> Una conversación de 20 turnos puede consumir entre 5.000 y 10.000 tokens cuando bastarían 500–1.000 tokens de contexto reciente.

Fuente: [Redis — LLM Token Optimization 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)

### El efecto compuesto en agentes

> Una sesión que comienza con 5.000 tokens por llamada puede llegar a 200.000 tokens por llamada en el turno 50 — pagás el mismo contexto repetidamente.

Fuente: [Redblink — AI Token Cost Optimization 2026](https://redblink.com/ai-token-cost-optimization/)

---

## Por qué los output tokens hacen el problema peor

> Los modelos flagship cobran $2–3 por millón de tokens de input y $10–15 por millón de output — un multiplicador de 4 a 6x.

Fuente: [Redis — LLM Token Optimization 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)

Cualquier estrategia que reduzca outputs impacta el costo más que reducir inputs.

---

## Context engineering: el nombre correcto del problema

> La optimización de tokens es un problema de ingeniería de contexto, no de acortamiento de prompts. Los verdaderos drivers de costo son el contexto bloqueado, los schemas de herramientas ociosas y el historial de conversación desactualizado.

Fuente: [TokenOptimize.dev — LLM Token Optimization Strategies 2026](https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies)

### Las 5 fuentes principales de desperdicio en producción

1. **Prompts verbosos e instrucciones repetidas**
2. **Historial de conversación ineficiente** — acumula miles de tokens innecesarios
3. **Tool calling no optimizado** — schemas verbosos en cada llamada
4. **Generación de output excesiva** — sin `max_tokens` el modelo genera de más (4–6x más caro)
5. **Contexto RAG sobredimensionado** — más contexto del necesario, sin mejor respuesta

Fuente: [Redis — LLM Token Optimization 2026](https://redis.io/blog/llm-token-optimization-speed-up-apps/)

---

## El impacto financiero a nivel empresa

> Deloitte 2026: enterprise AI es la categoría de costo IT de más rápido crecimiento. Algunas firmas reportan que AI consume hasta la mitad del gasto total de IT.

Fuente: [Redblink — AI Token Cost Optimization 2026](https://redblink.com/ai-token-cost-optimization/)

> IDC predice que las empresas Global 1000 subestimarán sus costos de infraestructura AI en un 30% hacia 2027.

Fuente: [InfoWorld — GitHub Copilot Usage-Based Billing 2026](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing-signaling-new-cost-model-for-enterprise-ai-tools.html)

---

## Estrategias de mitigación

### 1. Truncamiento inteligente del historial
Ventana deslizante de últimos N turnos + resumen de lo anterior.

### 2. Compresión de contexto
> Reducción de hasta 80% preservando fidelidad de la información.

Fuente: [Mem0 — Context Engineering AI Agents Guide](https://mem0.ai/blog/context-engineering-ai-agents-guide)

### 3. RAG en lugar de context stuffing
> RAG puede reducir el uso de tokens de contexto en hasta un 70%.

Fuente: [Koombea — LLM Cost Optimization](https://ai.koombea.com/blog/llm-cost-optimization)

### 4. Output budgets por tarea
Setear `max_tokens` explícitamente en cada workflow.

### 5. Context engineering como disciplina
> Los equipos exitosos filtran, rankean, podan, resumen y aíslan información deliberadamente. Tratan las ventanas de contexto como recursos escasos.

Fuente: [LogRocket — The LLM Context Problem 2026](https://blog.logrocket.com/llm-context-problem-strategies-2026/)

---

## Métricas verificadas

| Estrategia | Reducción | Fuente |
|------------|-----------|--------|
| Contexto re-enviado (problema base) | 62% de la factura | LeanOps 2026 |
| Compresión de memoria | hasta 80% | Mem0 |
| RAG vs context stuffing | hasta 70% | Koombea |
| Context engineering sistemático | 60–80% reducción operacional | Maxim AI |

---

## Archivos relacionados
- `03-prompt-caching.md` — cachear contexto estático para pagar 90% menos
- `08-batch-and-compaction.md` — Compaction API para conversaciones largas
