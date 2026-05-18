# Token Costs & Optimization

Repositorio de investigación para el artículo técnico sobre consumo, optimización, costos y gobernanza de tokens en IA.

**Audiencia objetivo:** Equipos técnicos enterprise (contexto Visma)
**Objetivo:** Generar conciencia colectiva y proveer guías accionables
**Formato final:** Artículo con storytelling + documentación técnica detallada linkada
**Idioma del artículo:** Español

---

## De qué trata este proyecto

El costo de usar IA en producción está creciendo de forma silenciosa y acelerada. En junio 2026, GitHub Copilot pasó de flat fee a usage-based billing — cada token enviado y recibido tiene ahora un precio directo. Esto convierte la optimización de tokens de "buena práctica" a "necesidad operacional".

Este repositorio reúne toda la investigación, datos verificados con fuentes, y la estructura del artículo que explica:
- Por qué el 62% de la factura de AI es contexto que ya pagaste antes
- Cómo el costo de una sesión agéntica crece de 5K a 200K tokens sin que nadie lo note
- Qué herramientas y estrategias concretas reducen ese costo — desde activar un setting en VSCode hasta implementar model routing enterprise

---

## Estado del proyecto

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional | ⏳ Pendiente |
| 4 | Redacción del artículo | ⏳ Pendiente |
| 5 | Redacción de documentación técnica | ⏳ Pendiente |

---

## Estructura del repositorio

```
TokenCostsAndOptimization/
│
├── README.md                              # Este archivo — overview completo del proyecto
├── PROJECT_INSTRUCTIONS.md               # Instrucciones, reglas editoriales y contexto completo
│
├── research/                             # Investigación por tema (fuente de verdad)
│   ├── 01-context-and-token-waste.md     # El mayor problema: contexto desperdiciado (62% de la factura)
│   ├── 02-model-selection.md             # Selección de modelo: el multiplicador que nadie mide
│   ├── 03-prompt-caching.md              # Prompt caching: 90% descuento que nadie usa
│   ├── 04-model-routing.md               # Routing automático e inteligente (3 niveles)
│   ├── 05-copilot-billing.md             # Nuevo billing Copilot desde junio 2026
│   ├── 06-vscode-tools.md                # Herramientas VSCode para reducir tokens
│   ├── 07-azure-deepseek.md              # Azure EA discounts + DeepSeek via Azure (compliance)
│   ├── 08-batch-and-compaction.md        # Batch API (50% off) + Compaction API Anthropic
│   └── 09-extended-thinking-costs.md     # El costo oculto del extended thinking
│
├── data/
│   └── verified-metrics.md               # TODOS los datos numéricos con URL de fuente verificada
│
└── article/
    └── outline.md                        # Estructura del artículo — borrador v1
```

---

## Reglas del proyecto

1. **Ningún dato numérico sin fuente verificada.** Si un número no tiene URL de origen, se marca `[VERIFICAR]` y se busca antes de usarlo en el artículo.
2. **Foco en costo y gobernanza.** Cada tema debe responder: ¿esto mueve la factura? ¿es accionable para el equipo?
3. **No interpolar ni estimar métricas.** Solo datos extraídos de artículos o estudios reales.
4. **El artículo no explica qué es un token.** La audiencia ya lo sabe.
5. **Tono de colega, no de manual corporativo.** Arrancar con storytelling, terminar con acciones concretas.

---

## Modelo elegido para redacción

**Claude Sonnet 4** (`claude-sonnet-4-20250514`)
Razón: mejor relación calidad/costo para escritura técnica con storytelling. Consistente con el mensaje central del artículo — no usar el modelo más caro para tareas que no lo justifican.

---

## Cómo continuar el proyecto

1. Leer `PROJECT_INSTRUCTIONS.md` para entender el contexto completo
2. Revisar `article/outline.md` para ver la estructura propuesta
3. Consultar `data/verified-metrics.md` antes de usar cualquier número
4. Los archivos en `research/` son la fuente de verdad para cada tema

---

## Fuentes clave

- https://redis.io/blog/llm-token-optimization-speed-up-apps/
- https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/
- https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/
- https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing
- https://www.lmsys.org/blog/2024-07-01-routellm/
- https://mem0.ai/blog/context-engineering-ai-agents-guide
- https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/
- https://platform.claude.com/docs/en/build-with-claude/compaction
- https://code.visualstudio.com/updates/v1_120
- https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx
- https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing
- https://deploybase.ai/articles/deepseek-v3-pricing
