# 02 — Estado Vivo

> **Este es el único archivo del repo que cambia entre sesiones.** Actualizalo al cerrar cualquier sesión donde hayas trabajado. La próxima sesión depende de esto.

---

## Última actualización

**Fecha:** 21 mayo 2026
**Por:** Opus 4.7 — creación de carpeta PRODV2 con copia del v2 + documento nuevo sobre hyperscaler discount vs overhead + caso GDPR/DeepSeek vía Azure
**Trigger:** Pedido del usuario de empezar a consolidar entregables finales en PRODV2 y producir un documento standalone sobre por qué el descuento del 25% del EA no se traduce en ahorro real, salvo en el caso compliance + DeepSeek

---

## Estado de los entregables

| Entregable | Idioma | Archivo | Estado |
|------------|--------|---------|--------|
| Artículo | ES | `PROD/article-draft-v1.md` | Borrador v1 — pendiente review |
| Artículo | EN | `PROD/article-draft-v1-EN.md` | Borrador v1 — pendiente review |
| Documentación técnica | ES | `PROD/technical-docs-draft-v1.md` | Borrador v1 — pendiente review |
| Documentación técnica | EN | `PROD/technical-docs-draft-v1-EN.md` | Borrador v1 — pendiente review |
| Documentación técnica v2 | ES | `PROD/technical-docs-draft-v2.md` | Versión expandida — replicada en PRODV2 |
| **Documentación técnica v2 (final)** | **ES** | **`PRODV2/technical-docs-draft-v2.md`** | **Copia 1:1 desde PROD/ — base de entregable final** |
| **Hyperscaler discount vs GDPR/DeepSeek** | **ES** | **`PRODV2/hyperscaler-discount-vs-gdpr-ES.md`** | **Nuevo, standalone, ~5.300 palabras, todas las cifras con URL a fuente** |
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
| `research/07-azure-deepseek.md` | ✅ Completo (ampliado con datos de noviembre 2025 sobre eliminación Level B/C/D en el nuevo doc PRODV2) |
| `research/08-batch-and-compaction.md` | ✅ Completo |
| `research/09-extended-thinking-costs.md` | ✅ Completo |
| `data/verified-metrics.md` | ⚠️ Pendiente: agregar las cifras nuevas verificadas en sesión 21 mayo (eliminación Level A-D, caso real overhead 55% Inference.net, casos Foundry trap específicos) |
| `article/outline.md` | ✅ Aprobado |

---

## PRÓXIMO PASO CONCRETO

**Acción inmediata recomendada (a elegir):**

**Opción A — Consolidar entregables en PRODV2.** Replicar también en PRODV2 los archivos finales que el usuario quiera mantener como "versión final": artículo (ES/EN), index ejecutivo JON.md, briefs de diseño. El criterio: PRODV2 es la carpeta de entregables aprobados; PROD se mantiene como working/draft.

**Opción B — Sincronizar `data/verified-metrics.md`.** Las cifras nuevas verificadas en esta sesión (eliminación Level A-D Microsoft nov 2025, caso 55% overhead Inference.net, caso $13K USD Foundry trap) están citadas en `PRODV2/hyperscaler-discount-vs-gdpr-ES.md` con URL pero todavía no están agregadas a la fuente de verdad central. Regla #1 del proyecto: ningún número sin fuente verificada en verified-metrics → conviene mover esas filas allí.

**Opción C — Producir versión EN del nuevo documento.** El doc nuevo está sólo en ES; si se quiere distribuir internacionalmente, generar `PRODV2/hyperscaler-discount-vs-gdpr-EN.md` siguiendo el mismo nivel de detalle y manteniendo los links a fuente.

**Sugerencia de orden:** primero B (mantener la regla #1 del proyecto), después A (consolidar PRODV2 completo), después C (versión EN cuando el contenido ES esté aprobado por Jonathan).

---

## Decisiones pendientes

- ¿Cuál es exactamente el criterio para "qué va a PRODV2"? La sesión actual asumió: copia del v2 (entregable final) + documento standalone nuevo. Conviene confirmar si:
  - el v1 también se considera entregable final (en cuyo caso también a PRODV2), o si v1 queda obsoleto.
  - el artículo storytelling (PROD/article-draft-v1.md) también es entregable final post-review, o si solo v2 lo es.

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
| 21 may 2026 | Opus 4.7 | Rediseño de ingeniería de contexto |
| 21 may 2026 | Opus 4.7 | Creación de PRODV2: copia v2 + documento standalone "hyperscaler-discount-vs-gdpr-ES.md" sobre por qué el 25% no se traduce en ahorro + caso DeepSeek/GDPR. Búsquedas web confirmaron eliminación Level A-D nov 2025, ban Garante DeepSeek enero 2025, disponibilidad DeepSeek en Azure AI Foundry EU desde feb 2026 |

---

## Cómo actualizar este archivo

Al cerrar una sesión donde hayas trabajado:

1. Cambiar la "Última actualización" arriba (fecha, modelo, qué se hizo)
2. Si cambió el estado de algún entregable, actualizar la tabla
3. Si hay un nuevo próximo paso, actualizar la sección "PRÓXIMO PASO CONCRETO"
4. Agregar una línea a "Sesiones anteriores"
5. Commit con mensaje claro: `state: [qué se hizo en una línea]`
