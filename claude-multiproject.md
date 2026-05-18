# CLAUDE MULTIPROJECT — Instrucciones para Agente AI

Este archivo es el PRIMERO que debe leer cualquier agente AI antes de trabajar en este repositorio. Contiene todo lo necesario para retomar el proyecto sin contexto previo, desde cualquier cuenta.

---

## QUIÉN SOS Y QUÉ ESTÁS HACIENDO

Sos un agente AI trabajando en un proyecto de escritura técnica para el equipo de Visma. El proyecto está 100% documentado en este repositorio. No dependés de ninguna conversación previa.

El dueño del proyecto es Jonathan Izquierdo. Al arrancar una sesión nueva, confirmá que leíste este archivo y los archivos clave antes de hacer cualquier cosa.

---

## QUÉ ES EL PROYECTO

### El objetivo
Dos entregables:

**Entregable 1 — Artículo:** storytelling técnico, tono de colega que comparte algo importante. Con links a la documentación para profundizar. Legible de corrido, no un manual.

**Entregable 2 — Documentación técnica:** detallada, sin resumir, con toda la profundidad necesaria. Métricas verificadas, ejemplos de código, guías accionables. Jonathan fue explícito: "Si hay mucha data no me molesta, no trates de resumir."

### La audiencia
Equipo técnico de Visma — desarrolladores, tech leads y architects. Audiencia mixta: desde los que solo usan Copilot hasta los que construyen agentes. El artículo va para TODOS. La documentación para los perfiles más técnicos.

**No explicar:** qué es un LLM, un token, un modelo. La audiencia ya lo sabe.

### El idioma
Español.

### Por qué existe este proyecto
1. GitHub Copilot cambia a usage-based billing el 1 de junio 2026. Cada token tiene precio. Estimación interna: factura enterprise sube ~3x.
2. VSCode lanzó `chat.tools.compressOutput.enabled` — esto motivó explorar qué más existe.

---

## ESTADO ACTUAL

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional | 🔄 Opcional |
| 4 | Redacción del artículo | ✅ Borrador v1 en PROD/ |
| 5 | Redacción de documentación técnica | ✅ Borrador v1 en PROD/ |
| 6 | Review y refinamiento | ⏳ PRÓXIMO PASO |

**Próxima acción:** revisar los borradores en `PROD/` con Jonathan, incorporar feedback, y producir la versión final.

---

## ARCHIVOS DEL REPOSITORIO

```
README.md                          — Overview general
PROJECT-INSTRUCTIONS.md            — Contexto completo, decisiones, reglas
claude-multiproject.md             — Este archivo (leer primero)
research/
  01-context-and-token-waste.md    — El 62% de la factura
  02-model-selection.md            — Selección de modelo + MCPs
  03-prompt-caching.md             — 90% descuento + memory compression
  04-model-routing.md              — 3 niveles de routing
  05-copilot-billing.md            — Billing junio 2026
  06-vscode-tools.md               — Herramientas VSCode
  07-azure-deepseek.md             — Azure EA + DeepSeek compliance
  08-batch-and-compaction.md       — Batch API + Compaction API
  09-extended-thinking-costs.md    — Costo oculto del thinking
data/
  verified-metrics.md              — TODOS los números con fuente y URL
article/
  outline.md                       — Estructura del artículo (aprobada)
PROD/
  article-draft-v1.md              — Borrador del artículo (REVISAR)
  technical-docs-draft-v1.md       — Borrador documentación técnica (REVISAR)
```

---

## REGLAS QUE NO SE PUEDEN ROMPER

1. **Ningún número sin fuente.** Si no está en `data/verified-metrics.md` con URL real, se marca `[VERIFICAR]` y se busca antes de usar.
2. **No resumir.** La documentación debe ser detallada y completa.
3. **Foco en costo y gobernanza.** ¿Esto mueve la factura? ¿Es accionable para el equipo?
4. **Tono correcto.** Colega que comparte algo importante — no manual corporativo.
5. **No explicar lo básico.** La audiencia sabe qué es un token.
6. **Guardar el progreso en el repo.** No dejar trabajo solo en el chat.

---

## DECISIONES YA TOMADAS — NO REABRIR

- **Modelo para redacción:** Claude Sonnet 4 (`claude-sonnet-4-20250514`)
- **Azure descuento:** rango negociado 15-28%, no número fijo publicado
- **DeepSeek via Azure:** el ángulo es compliance, no precio. Azure cobra 20-35% más que directo
- **Tema descartado:** "Lost in the middle" — vínculo con el costo demasiado indirecto

---

## DATOS MÁS IMPORTANTES

| Dato | Valor | Fuente |
|------|-------|--------|
| Contexto re-enviado = % factura | 62% | LeanOps 2026 |
| Tokens turno 1 vs turno 50 | 5K → 200K | Redblink |
| Multiplicador output vs input | 4–6x | Redis 2026 |
| Descuento prompt caching | 90% | Anthropic oficial |
| RouteLLM: calidad con % llamadas | 95% calidad / 26% llamadas | LMSYS ICLR 2025 |
| RouteLLM ahorro | 75% con augmentation | LMSYS ICLR 2025 |
| Copilot top usuarios = spend | 10-15% = 60-70% | Synapx |
| Multiplicador agéntico | ~3.5x flat fee | Synapx |
| Auto Mode descuento | 10% | GitHub Changelog |
| Batch API | 50% descuento | Anthropic oficial |
| Batch + caching | hasta 95% ahorro | PECollective |
| Extended thinking omitted | No ahorra nada | CheckThat.ai |
| DeepSeek vs Sonnet | ~15x más barato | Precios 2026 |
| Azure EA discount | 15-28% negociado | Microsoft Negotiations |

---

## CÓMO ARRANCAR UNA SESIÓN NUEVA

### Mensaje para pegar al inicio

> "Trabajá en el repositorio GitHub JonathanIzquierdo/TokenCostsAndOptimization. Leé primero `claude-multiproject.md`, luego los borradores en `PROD/`. El estado actual: los borradores v1 están listos para review. El próximo paso es revisar con Jonathan e incorporar feedback para producir la versión final. Idioma: español. Nada de datos sin fuente verificada en `data/verified-metrics.md`."

### Flujo de trabajo
1. Leer este archivo
2. Leer `PROD/article-draft-v1.md` y `PROD/technical-docs-draft-v1.md`
3. Revisar con Jonathan
4. Incorporar feedback y subir versión revisada
5. Versión final: `PROD/article-final.md` y `PROD/technical-docs-final.md`

---

## CONTEXTO ADICIONAL

**Visma es empresa europea** — GDPR y data residency importan. DeepSeek via Azure (no directo) y Azure compliance son relevantes por esto.

**El mensaje de fondo del proyecto:** no es reducción de costos solamente. Es cultura de ingeniería. Gastar bien, no gastar menos. Visibilidad, gobernanza, conciencia colectiva.

**Tono correcto:**
- ✅ "El 62% de lo que pagás en AI no es por el trabajo que hace el modelo."
- ✅ "La sesión que empieza en 5.000 tokens llega a 200.000 en el turno 50."
- ❌ "En este artículo exploraremos las mejores prácticas..."
- ❌ "Es importante destacar que los output tokens tienen un costo significativamente mayor."
