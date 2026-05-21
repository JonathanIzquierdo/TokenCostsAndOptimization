# Token Costs & Optimization

Investigación y redacción de un artículo técnico + documentación detallada sobre **costo y gobernanza de tokens en IA**, para el equipo de ingeniería de Visma.

**Idioma primario:** Español (versiones EN paralelas).
**Audiencia:** Developers, tech leads y architects de Visma.
**Entregables:** Artículo (storytelling técnico) + Documentación técnica detallada por capas.

---

## 🚨 ¿Sos un agente AI? Empezá acá

**Leé primero `CLAUDE.md` en la raíz del repo.** Es el bootstrap obligatorio del proyecto.

No saltees ese paso aunque te apuren. Está diseñado para que cualquier modelo (Haiku, Sonnet, Opus, ChatGPT, Gemini, Copilot) pueda retomar el proyecto sin perder contexto.

---

## De qué trata el proyecto

El costo de usar IA en producción está creciendo de forma silenciosa. En junio 2026, GitHub Copilot pasó de flat fee a usage-based billing — cada token tiene ahora precio directo. Esto convierte la optimización de tokens de "buena práctica" a "necesidad operacional".

El proyecto reúne investigación verificada y produce dos entregables:

1. **Artículo** — storytelling técnico que explica por qué el 62% de la factura de AI es contexto que ya pagaste, cómo una sesión agéntica crece de 5K a 200K tokens sin que nadie lo note, y qué herramientas concretas reducen ese costo.

2. **Documentación técnica** — exhaustiva, organizada en 5 capas según rol y alcance (Capa 0: settings individuales → Capa 4: gobernanza enterprise).

---

## Estructura del repositorio

```
TokenCostsAndOptimization/
│
├── CLAUDE.md                     ← Bootstrap obligatorio para agentes AI
├── README.md                     ← Este archivo
│
├── .context/                     ← Ingeniería de contexto (leer en orden)
│   ├── 00-START-HERE.md
│   ├── 01-project.md
│   ├── 02-state.md               ← Estado vivo del proyecto
│   ├── 03-rules.md
│   ├── 04-glossary.md
│   ├── 05-model-guide.md
│   ├── 06-files-map.md
│   ├── 07-decisions.md
│   └── 08-prompts.md
│
├── research/                     ← Investigación por tema (9 archivos)
│   ├── 01-context-and-token-waste.md
│   ├── 02-model-selection.md
│   ├── 03-prompt-caching.md
│   ├── 04-model-routing.md
│   ├── 05-copilot-billing.md
│   ├── 06-vscode-tools.md
│   ├── 07-azure-deepseek.md
│   ├── 08-batch-and-compaction.md
│   └── 09-extended-thinking-costs.md
│
├── data/
│   └── verified-metrics.md       ← TODOS los números con URL de fuente
│
├── article/
│   └── outline.md                ← Estructura aprobada del artículo
│
└── PROD/                         ← Entregables en producción
    ├── article-draft-v1.md       (ES)
    ├── article-draft-v1-EN.md
    ├── technical-docs-draft-v1.md (ES)
    ├── technical-docs-draft-v1-EN.md
    ├── JON.md / JON-EN.md        ← Índice ejecutivo para el dueño del proyecto
    ├── claude-design-brief-ES.md
    └── claude-design-brief-EN.md
```

---

## Estado actual

Borradores v1 del artículo y documentación técnica completos (ES + EN), en `PROD/`. Próximo paso: review con Jonathan e incorporación de feedback para versión final.

**Detalle actualizado:** `.context/02-state.md`

---

## Las 4 reglas innegociables

1. **Ningún número sin fuente verificada** en `data/verified-metrics.md` con URL real.
2. **No resumir documentación técnica** — es exhaustiva por diseño.
3. **Foco en costo y gobernanza** — cada tema debe mover la factura o ser accionable.
4. **No explicar lo básico** — la audiencia sabe qué es un token.

Detalle completo: `.context/03-rules.md`.

---

## Modelo recomendado para redacción

**Claude Sonnet 4** (`claude-sonnet-4-20250514`).

Razón: mejor balance calidad/costo. Opus es overkill para escritura técnica con estructura clara; Haiku no tiene la fluidez narrativa requerida. Esta elección es consistente con el mensaje del propio artículo — no usar el modelo más caro para tareas que no lo justifican.

Guía completa por tipo de tarea: `.context/05-model-guide.md`.

---

## Cómo continuar el proyecto desde cualquier herramienta

Si estás retomando desde una cuenta nueva, otra herramienta de AI, o después de tiempo:

1. Leé `CLAUDE.md` (raíz)
2. Leé `.context/02-state.md` (estado vivo)
3. Confirmá al usuario qué entendiste antes de tocar nada

Prompts listos para copiar: `.context/08-prompts.md`.

---

## Principio rector

> *"Gastar bien, no gastar menos."*

Es el mensaje del artículo y la regla operativa del proyecto.
