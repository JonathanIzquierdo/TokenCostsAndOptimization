# 05 — Guía de modelos

Qué modelo usar para qué tarea. Esto evita el problema clásico de "usé Haiku para escribir el artículo y salió mal" o "usé Opus para listar archivos y gasté 10x más de lo necesario".

---

## Modelo por tipo de tarea

| Tarea | Modelo recomendado | Por qué |
|-------|-------------------|---------|
| Redacción del artículo | Claude Sonnet 4 | Calidad narrativa + costo razonable. Opus es caro y no aporta diferencial; Haiku no tiene la fluidez narrativa. |
| Redacción de documentación técnica | Claude Sonnet 4 | Mismo razonamiento. |
| Investigación / web search / fact-checking | Claude Sonnet 4 con web search | Sonnet maneja bien chains de búsqueda y síntesis. Haiku se pierde en cadenas largas. |
| Refinar/pulir un párrafo específico | Claude Sonnet 4 o Haiku | Tareas chicas con contexto bien acotado. |
| Listar archivos, mover archivos, commits | Haiku o Sonnet | Operaciones mecánicas. |
| Buscar en archivos del repo | Haiku | Lectura + extracción simple. |
| Resumir un research file para uso interno | Sonnet | Síntesis con criterio. |
| Generar visuales / diseño / diagramas | Claude Design (Opus) | Razonamiento espacial + estética. |
| Revisar coherencia editorial entre EN y ES | Sonnet | Necesita matiz cultural. |
| Tomar decisiones editoriales nuevas | Opus o Sonnet | Cualquier modelo con razonamiento robusto. |

---

## Capacidades por modelo (referencia)

### Claude Opus 4.7 (`claude-opus-4-7`)
**Fortalezas:** razonamiento profundo, planeamiento, escritura compleja, código difícil.  
**Costo:** alto.  
**Cuándo NO usarlo:** tareas mecánicas, lectura simple, operaciones de archivo. Es overkill.  
**En este proyecto:** usado para diseño visual y decisiones editoriales mayores.

### Claude Sonnet 4 (`claude-sonnet-4-20250514`)
**Fortalezas:** balance calidad/costo, escritura técnica, web search, multi-turn coherente.  
**Costo:** medio.  
**Cuándo SÍ usarlo:** la mayoría de tareas del proyecto. **Es el modelo default para redacción.**  
**Cuándo NO usarlo:** tareas triviales (listar, mover archivos) — Haiku es más barato y suficiente.

### Claude Haiku 4.5 (`claude-haiku-4-5-20251001`)
**Fortalezas:** velocidad, costo bajo, instrucciones mecánicas.  
**Costo:** muy bajo.  
**Cuándo SÍ usarlo:** lectura, extracción, listing, parsing, operaciones mecánicas con instrucciones claras.  
**Cuándo NO usarlo:**
- Redacción del artículo (le falta fluidez narrativa)
- Decisiones editoriales (no tiene contexto interpretativo)
- Investigación con múltiples saltos lógicos
- Síntesis de información cruzada

**Si estás usando Haiku ahora y querés trabajar en el artículo:** avisale al usuario que para esa tarea conviene cambiar a Sonnet 4. Mejor avisar que entregar algo mediocre.

---

## Reglas específicas si sos un modelo chico (Haiku, GPT-mini, Flash)

### Sí podés hacer
- Leer archivos del repo y reportar contenido
- Buscar archivos por nombre o keyword
- Listar el estado del proyecto leyendo `02-state.md`
- Actualizar `02-state.md` con instrucciones claras del usuario
- Hacer commits con mensajes que el usuario te dicte
- Verificar que un número esté en `verified-metrics.md`
- Reportar discrepancias entre archivos

### Pedile a un modelo más grande para
- Escribir o reescribir secciones del artículo
- Redactar la documentación técnica
- Tomar decisiones de estructura
- Sintetizar 5 research files en 1 sección de artículo
- Hacer investigación nueva con múltiples web searches encadenadas
- Decidir si un dato "mueve la factura o no"

### Cómo redirigir al usuario

Ejemplo de respuesta correcta cuando Haiku enfrenta una tarea fuera de su zona:

> "Para esta tarea (escribir una nueva sección del artículo con storytelling) te recomiendo cambiar a Claude Sonnet 4. El proyecto define Sonnet 4 como el modelo de redacción porque la calidad narrativa lo justifica. Yo (Haiku) puedo ayudarte con lectura, búsqueda en el repo, o actualizar el state — ¿querés que haga algo de eso mientras tanto?"

---

## Cómo elegir el modelo correcto en cada momento

1. **¿La tarea requiere razonamiento profundo o creatividad?** → Opus o Sonnet
2. **¿La tarea es escritura técnica con storytelling?** → Sonnet (default del proyecto)
3. **¿La tarea es mecánica con instrucciones claras?** → Haiku
4. **¿La tarea es síntesis cruzada de varios archivos?** → Sonnet
5. **¿La tarea es búsqueda web con múltiples queries?** → Sonnet
6. **¿La tarea es navegación de archivos / commits / state updates?** → Haiku

**Default seguro:** Sonnet 4. Cuesta más que Haiku pero menos que Opus, y es el mejor balance para 80% de las tareas del proyecto.
