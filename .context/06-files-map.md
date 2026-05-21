# 06 — Mapa de archivos por tarea

Qué archivo leer (y en qué orden) según lo que el usuario te pida. Este archivo te ahorra contexto: no leas todo el repo, leé solo lo necesario.

---

## "Continuá donde quedamos" / "Seguí trabajando"

1. `.context/02-state.md` — qué se hizo, qué falta, próximo paso
2. El archivo mencionado como próximo paso
3. **Confirmar al usuario antes de tocar nada**

---

## "Revisemos el artículo" / "Editar el artículo"

1. `.context/02-state.md` — ver qué versión está actual
2. `PROD/article-draft-v1.md` (ES) o `PROD/article-draft-v1-EN.md` (EN)
3. `data/verified-metrics.md` — si vas a tocar números
4. `.context/01-project.md` — recordar tono y audiencia

---

## "Editar la documentación técnica"

1. `.context/02-state.md`
2. `PROD/technical-docs-draft-v1.md` (ES) o `PROD/technical-docs-draft-v1-EN.md` (EN)
3. `research/0X-tema.md` — el research file del tema que estás tocando
4. `data/verified-metrics.md`
5. `.context/03-rules.md` — recordar la regla "no resumir"

---

## "Agregar un dato/número/métrica"

1. `data/verified-metrics.md` — verificar si ya existe
2. Si no existe: web search para encontrar la fuente
3. Si encontrás fuente: agregar primero a `verified-metrics.md`, después al texto
4. Si no encontrás: marcar `[VERIFICAR]` en el texto y avisar al usuario

---

## "Investigar tema nuevo" / "Agregar tema X"

1. `.context/03-rules.md` — confirmar que el tema pasa el filtro "mueve la factura / accionable"
2. `.context/07-decisions.md` — verificar que no esté descartado ya
3. Web search con múltiples queries
4. Crear nuevo `research/XX-tema.md`
5. Agregar números a `verified-metrics.md`
6. Actualizar `02-state.md`

---

## "Hacer visuales / diagramas"

1. `PROD/claude-design-brief-ES.md` o `PROD/claude-design-brief-EN.md`
2. `data/verified-metrics.md` — todo número visualizado debe estar acá
3. Recordar regla: cada visual lleva caption con fuente

---

## "Resumime el proyecto" / "De qué se trata esto"

1. `CLAUDE.md` raíz — overview rápido
2. `.context/01-project.md` — detalle
3. `.context/02-state.md` — estado actual

**No leas todos los research files para esto.** Es desperdicio de contexto.

---

## "Decidiste algo, no me acuerdo qué"

1. `.context/07-decisions.md` — historial de decisiones cerradas

---

## "Hay un término que no entiendo"

1. `.context/04-glossary.md`

---

## "Qué modelo uso para X"

1. `.context/05-model-guide.md`

---

## "Querés cambiar la estructura del repo"

1. `CLAUDE.md` — leer mapa actual
2. `.context/06-files-map.md` — este archivo
3. **Confirmar con el usuario antes de hacer cambios estructurales**
4. Actualizar este archivo, `CLAUDE.md` y `02-state.md` después del cambio

---

## Tabla rápida — archivo según pregunta del usuario

| Pregunta del usuario | Archivo primario |
|----------------------|------------------|
| "¿En qué estamos?" | `02-state.md` |
| "¿De qué se trata?" | `01-project.md` |
| "¿Cuál es la regla con los números?" | `03-rules.md` |
| "¿Qué significa Capa 2?" | `04-glossary.md` |
| "¿Con qué modelo escribo esto?" | `05-model-guide.md` |
| "¿Decidimos algo sobre Azure?" | `07-decisions.md` |
| "¿Cómo arranco una sesión nueva?" | `08-prompts.md` |
| "¿Cuánto cuesta Sonnet?" | `verified-metrics.md` |
| "¿Qué dijimos de prompt caching?" | `research/03-prompt-caching.md` |
