# 02 — Estado Vivo

> **Este es el único archivo del repo que cambia entre sesiones.** Actualizalo al cerrar cualquier sesión donde hayas trabajado. La próxima sesión depende de esto.

---

## Última actualización

**Fecha:** 21 mayo 2026  
**Por:** Rediseño completo de ingeniería de contexto (sesión con Opus 4.7)  
**Trigger:** El usuario cambió a Haiku en una sesión anterior y el modelo no logró retomar el proyecto. Hace falta robustecer las instrucciones para que cualquier modelo (incluso chico) pueda continuar.

---

## Estado de los entregables

| Entregable | Idioma | Archivo | Estado |
|------------|--------|---------|--------|
| Artículo | ES | `PROD/article-draft-v1.md` | Borrador v1 — pendiente review |
| Artículo | EN | `PROD/article-draft-v1-EN.md` | Borrador v1 — pendiente review |
| Documentación técnica | ES | `PROD/technical-docs-draft-v1.md` | Borrador v1 — pendiente review |
| Documentación técnica | EN | `PROD/technical-docs-draft-v1-EN.md` | Borrador v1 — pendiente review |
| Índice ejecutivo (para Jonathan) | ES | `PROD/JON.md` | Listo |
| Índice ejecutivo (para Jonathan) | EN | `PROD/JON-EN.md` | Listo |
| Brief para Claude Design | ES | `PROD/claude-design-brief-ES.md` | Listo |
| Brief para Claude Design | EN | `PROD/claude-design-brief-EN.md` | Listo |

---

## Estado de la investigación

| Archivo | Estado |
|---------|--------|
| `research/01-context-and-token-waste.md` | ✅ Completo |
| `research/02-model-selection.md` | ✅ Completo |
| `research/03-prompt-caching.md` | ✅ Completo |
| `research/04-model-routing.md` | ✅ Completo |
| `research/05-copilot-billing.md` | ✅ Completo |
| `research/06-vscode-tools.md` | ✅ Completo |
| `research/07-azure-deepseek.md` | ✅ Completo |
| `research/08-batch-and-compaction.md` | ✅ Completo |
| `research/09-extended-thinking-costs.md` | ✅ Completo |
| `data/verified-metrics.md` | ✅ Completo, fuente de verdad |
| `article/outline.md` | ✅ Aprobado |

---

## PRÓXIMO PASO CONCRETO

**Acción inmediata:** Review de los borradores v1 con Jonathan e incorporación de feedback para producir versión final.

**Sugerencia de flujo:**
1. Jonathan lee `PROD/JON.md` (índice ejecutivo) primero
2. Decide qué entregable revisar primero (ES o EN, artículo o doc)
3. Marca feedback en el archivo o lo lista en chat
4. Agente AI (Sonnet 4) incorpora feedback en versión `-v2`
5. Iterar hasta `-final`

**Cuando aprobado:** generar visuales con Claude Design usando los briefs.

---

## Decisiones pendientes

Ninguna bloqueante al día de hoy. Todas las decisiones cerradas están en `07-decisions.md`.

---

## Bloqueos

Ninguno.

---

## Sesiones anteriores (log resumido)

| Fecha | Modelo | Qué se hizo |
|-------|--------|-------------|
| 19 may 2026 | Opus 4.6 | Investigación completa (9 temas), `verified-metrics.md`, outline aprobado, primer push al repo, primeras instrucciones |
| 19 may 2026 | Opus 4.6 | Borradores v1 de artículo + documentación técnica (ES + EN), briefs de diseño, índices JON.md |
| (sesión Haiku) | Haiku | ⚠️ El modelo no logró retomar contexto. Trigger del rediseño actual. |
| 21 may 2026 | Opus 4.7 | Rediseño de ingeniería de contexto — este archivo y los demás de `.context/` |

---

## Cómo actualizar este archivo

Al cerrar una sesión donde hayas trabajado:

1. Cambiar la "Última actualización" arriba (fecha, modelo, qué se hizo)
2. Si cambió el estado de algún entregable, actualizar la tabla
3. Si hay un nuevo próximo paso, actualizar la sección "PRÓXIMO PASO CONCRETO"
4. Agregar una línea a "Sesiones anteriores"
5. Commit con mensaje claro: `state: [qué se hizo en una línea]`
