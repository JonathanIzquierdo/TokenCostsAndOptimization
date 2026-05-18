# 02 — Selección de Modelo: El Multiplicador que Nadie Mide

Elegir el modelo equivocado no es solo un problema de calidad. Es el multiplicador de costo más silencioso que tiene un equipo de ingeniería.

---

## La brecha de precios en 2026

| Modelo | Input (por 1M tokens) | Output (por 1M tokens) |
|--------|----------------------|------------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 |
| Claude Sonnet 4.6 | $3.00 | $15.00 |
| Claude Opus 4.7 | $5.00 | $25.00 |

Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

> Haiku 4.5 es 5x más barato que Sonnet en input, y 25x más barato que Opus. La mayoría de los equipos usa Sonnet o Opus para tareas que Haiku resuelve perfectamente.

Fuente: [CloudZero — Claude API Pricing 2026](https://www.cloudzero.com/blog/claude-api-pricing/)

---

## El problema de "un modelo para todo"

> La mayoría de los frameworks agénticos usan un único modelo para todo. Esto es tremendamente ineficiente.

Fuente: [LeanOps — Agentic AI Cost Runaway 2026](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/)

### La lógica de asignación correcta

| Tipo de tarea | Modelo apropiado | Razón |
|---------------|-----------------|-------|
| Clasificación, extracción, formateo | Haiku | Pattern matching, no razonamiento |
| Chat, summarización, generación general | Sonnet | Punto óptimo calidad/costo |
| Razonamiento complejo, arquitectura | Opus | Aquí sí justifica el precio |
| Navegación de archivos, resolución de símbolos | Haiku | Operaciones simples de recuperación |

Fuente: [Augment Code — AI Model Routing Guide 2026](https://www.augmentcode.com/guides/ai-model-routing-guide)

---

## El impacto financiero

> Una reducción del 30% en uso de modelos premium en enterprises con $250K+ de gasto anual representa $75.000 de ahorro por año.

Fuente: [MindStudio — What Is an AI Model Router 2026](https://www.mindstudio.ai/blog/what-is-ai-model-router-optimize-cost-llm-providers)

---

## La regla de los output tokens

> Los output tokens cuestan 5x los input tokens en todos los modelos actuales de Anthropic.

Fuente: [CloudZero — Claude API Pricing 2026](https://www.cloudzero.com/blog/claude-api-pricing/)

**Estrategia:** Setear `max_tokens` según el tipo de tarea. Clasificación: 50 tokens. Resumen ejecutivo: 500 tokens.

---

## MCPs activos innecesariamente: el costo invisible

Cada MCP activo agrega su schema completo al contexto en cada llamada — aunque nunca lo uses.

**Ejemplo:** 10 MCPs × 500 tokens/schema = 5.000 tokens de overhead por request.
A $3/MTok en Sonnet: **$1.500/mes en 100K requests** por herramientas que nadie pidió.

### La solución: tool search de VSCode

> VS Code 1.118 dividió el toolset en ~30 herramientas core (88% de los casos) y un set diferido que no carga hasta que se solicita explícitamente. Genera hasta un **20% de ahorro en tokens**.

Fuente: [Visual Studio Magazine — VS Code Curbs Token Use 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

---

## Métricas verificadas

| Escenario | Ahorro | Fuente |
|-----------|--------|--------|
| Haiku vs Sonnet | 5x más barato por request | Anthropic pricing |
| Haiku vs Opus | 25x más barato por request | Anthropic pricing |
| Reducción 30% uso premium ($250K spend) | $75K/año | MindStudio |
| Tool search VSCode | hasta 20% tokens | VS Magazine |

---

## Archivos relacionados
- `04-model-routing.md` — automatizar la selección con routing inteligente
- `06-vscode-tools.md` — herramientas VSCode para controlar el toolset
