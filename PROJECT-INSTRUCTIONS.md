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
Español (versiones EN también disponibles en PROD/).

### Por qué ahora
Dos eventos concretos dispararon el proyecto:

**Evento 1 — Copilot usage-based billing:**
GitHub Copilot migra a usage-based billing el 1 de junio 2026. Cada seat incluye tokens equivalentes al precio del seat ($19 Business, $39 Enterprise) + bonus 3 meses. Después se paga por token consumido. Los seat tokens son pooled a nivel enterprise Visma.

**Evento 2 — VSCode feature:**
En el último VSCode se puede habilitar `chat.tools.compressOutput.enabled` para post-procesar el output de terminal antes de enviarlo al modelo.

---

## ESTADO ACTUAL DEL PROYECTO

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Redacción del artículo | ✅ Borrador v1 en PROD/ |
| 4 | Redacción de documentación técnica | ✅ Borrador v1 en PROD/ |
| 5 | Visualizaciones y diseño | 🔄 En curso — briefs en PROD/ |
| 6 | Review y refinamiento final | ⏳ Pendiente |

---

## HERRAMIENTAS DEL PROYECTO

Este proyecto usa dos herramientas de AI con roles distintos:

### Claude Code
- Mejora el contenido: agrega datos, verifica fuentes, actualiza archivos, busca información nueva
- Mantiene el repo organizado
- **Regla crítica:** nunca agregar números, porcentajes o métricas sin fuente verificada en `data/verified-metrics.md`
- Si un número no tiene URL real como fuente, marcarlo `[VERIFICAR]` y buscarlo antes de publicarlo

### Claude Design
- Toma los briefs en `PROD/claude-design-brief-*.md` y genera los visuales
- Formatea los artículos para facilidad de lectura
- **Regla crítica:** cada visual debe tener caption con URL de fuente
- Solo usar datos que estén en el brief correspondiente — no inventar ni estimar métricas
- Archivos de brief: `PROD/claude-design-brief-EN.md` y `PROD/claude-design-brief-ES.md`

### Workflow de mejora
1. **Claude Code** agrega contenido, verifica datos, actualiza `data/verified-metrics.md`
2. **Claude Design** toma los artículos actualizados y los briefs para agregar visuales y mejorar formato
3. Los resultados se guardan en `PROD/` con sufijo `-final` cuando están aprobados

---

## REGLAS CRÍTICAS

### Regla 1: Ningún número sin fuente verificada
Ningún dato numérico o porcentaje puede usarse en el artículo, documentación, o visuales sin una URL de artículo o estudio real. Si un número no tiene fuente, se marca `[VERIFICAR]` y se busca antes de escribir. No interpolar ni estimar métricas.

Todos los datos verificados están en `data/verified-metrics.md` con sus fuentes.

### Regla 1b: Ningún visual sin fuente verificada
Esta regla aplica también a gráficos, diagramas y tablas visuales. Cada visual debe tener:
- Un caption que mencione la fuente
- La URL directa al artículo o estudio que respalda el número
- Solo datos que estén en `data/verified-metrics.md` o en el brief correspondiente

### Regla 2: Foco en costo y gobernanza
Temas de accuracy o calidad de respuesta solo se incluyen si el vínculo con el costo es directo e inevitable.

### Regla 3: No asumir que el lector no sabe nada
La audiencia son desarrolladores y tech leads. No explicar conceptos básicos.

### Regla 4: Tono de colega, no de manual corporativo
El artículo debe sentirse como alguien del equipo que descubrió algo importante y lo comparte.

---

## ARCHIVOS DEL REPOSITORIO

```
README.md                              — Overview general
PROJECT-INSTRUCTIONS.md               — Este archivo
claude-multiproject.md                 — Onboarding rápido para cualquier agente AI
research/
  01-context-and-token-waste.md
  02-model-selection.md
  03-prompt-caching.md
  04-model-routing.md
  05-copilot-billing.md
  06-vscode-tools.md
  07-azure-deepseek.md
  08-batch-and-compaction.md
  09-extended-thinking-costs.md
data/
  verified-metrics.md                 — TODOS los números con fuente y URL verificada
article/
  outline.md
PROD/
  article-draft-v1.md                 — Artículo en español (borrador v1)
  article-draft-v1-EN.md              — Artículo en inglés (borrador v1)
  technical-docs-draft-v1.md          — Documentación técnica en español
  technical-docs-draft-v1-EN.md       — Documentación técnica en inglés
  JON.md                              — Índice ejecutivo en español
  JON-EN.md                           — Índice ejecutivo en inglés
  claude-design-brief-ES.md           — Brief para Claude Design (español)
  claude-design-brief-EN.md           — Brief para Claude Design (inglés)
```

---

## DECISIONES TOMADAS

### Modelo para redacción
**Claude Sonnet 4** (`claude-sonnet-4-20250514`) — mejor relación calidad/costo para esta tarea.

### Azure "24% de descuento"
Rango verificado: 15–28% negociado. Nunca presentar como número fijo.

### DeepSeek via Azure
El ángulo es compliance, no precio. Azure cobra 20–35% más que directo pero resuelve data residency europeo.

### Tema descartado: "Lost in the middle"
Vínculo con el costo demasiado indirecto. Descartado como punto standalone.

---

## DATOS VERIFICADOS — RESUMEN EJECUTIVO

| Dato | Valor | Fuente |
|------|-------|--------|
| Contexto re-enviado = % de la factura | 62% | LeanOps 2026 |
| Tokens turno 1 vs turno 50 | 5K → 200K | Redblink |
| Multiplicador output vs input | 4–6x | Redis 2026 |
| Descuento prompt caching | 90% en cache reads | Anthropic oficial |
| RouteLLM calidad/llamadas | 95% calidad con 26% llamadas | LMSYS ICLR 2025 |
| RouteLLM con augmentation | 75% reducción | LMSYS ICLR 2025 |
| Copilot top usuarios = spend | 10–15% = 60–70% | Synapx |
| Multiplicador agéntico | ~3.5x flat fee | Synapx |
| Auto Mode descuento | 10% en multiplicador | GitHub Changelog |
| Batch API | 50% descuento | Anthropic oficial |
| Batch + caching | hasta 95% de ahorro | PECollective |
| Extended thinking omitted | No ahorra nada | CheckThat.ai |
| Opus 4.7 tokenizer overhead | hasta 35% más tokens | Finout |
| DeepSeek via Azure markup | +20–35% sobre directo | DeployBase |
| Azure EA discount | 15–28% negociado | Microsoft Negotiations |

Datos completos con URLs en `data/verified-metrics.md`.

---

## CÓMO RETOMAR EL PROYECTO

### Para Claude Code
Leer este archivo + `claude-multiproject.md` + `data/verified-metrics.md`. Luego trabajar en los archivos de `PROD/` según lo que se necesite mejorar.

### Para Claude Design
Leer `PROD/claude-design-brief-EN.md` o `PROD/claude-design-brief-ES.md` según el idioma. El brief contiene todos los datos, fuentes, y especificaciones de los visuales a generar.

### Mensaje de onboarding universal
> "Trabajá en el repositorio JonathanIzquierdo/TokenCostsAndOptimization. Leé primero `claude-multiproject.md` y `PROJECT-INSTRUCTIONS.md`. Para agregar visuales al artículo en inglés, leé `PROD/claude-design-brief-EN.md`. Para contenido en español, `PROD/claude-design-brief-ES.md`. Regla crítica: ningún número sin URL verificada en `data/verified-metrics.md`."
