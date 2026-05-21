# 08 — Prompts para arrancar sesión

Prompts listos para copiar y pegar al iniciar una sesión nueva. Diseñados para que cualquier modelo (cualquier cuenta, cualquier herramienta) arranque alineado.

---

## Prompt universal — para cualquier modelo, cualquier sesión

```
Trabajamos sobre el repositorio github.com/JonathanIzquierdo/TokenCostsAndOptimization.

Protocolo de arranque obligatorio (en orden):
1. Leé CLAUDE.md de la raíz del repo.
2. Leé .context/02-state.md.
3. Leé .context/01-project.md.
4. Leé .context/03-rules.md.
5. Antes de tocar cualquier archivo, decime en una línea qué entendiste y qué vas a hacer. Esperá mi confirmación.

No improvises. Si algo no te queda claro, preguntá.
```

---

## Prompt corto — cuando ya tenés el contexto cargado

```
Seguimos en TokenCostsAndOptimization. Estado en .context/02-state.md.
Tarea: [describir tarea].
```

---

## Prompt para Claude Code (CLI o IDE)

```
Este repo tiene CLAUDE.md en la raíz. Leelo primero.
Después leé .context/02-state.md para ver el estado actual.
No modifiques archivos sin confirmar primero.
```

---

## Prompt para Claude Design (visuales)

```
Proyecto TokenCostsAndOptimization.
Leé primero CLAUDE.md raíz para contexto.
Después leé PROD/claude-design-brief-ES.md (o EN según idioma).
Regla crítica: cada visual debe llevar caption con URL de fuente.
Todos los números deben estar en data/verified-metrics.md — si no están ahí, no se usan.
```

---

## Prompt para retomar después de pausa larga

```
Vuelvo al proyecto TokenCostsAndOptimization después de un tiempo.

Leé en este orden:
1. CLAUDE.md raíz
2. .context/02-state.md (estado vivo)
3. .context/07-decisions.md (decisiones tomadas)

Después contame:
- Dónde quedó el proyecto
- Cuál es el próximo paso concreto
- Si hay alguna decisión pendiente

No arranques tareas hasta que confirme.
```

---

## Prompt para un modelo chico (Haiku, GPT-mini, Flash)

```
Proyecto TokenCostsAndOptimization. Sos un modelo de capacidad limitada, así que:

1. Leé CLAUDE.md raíz.
2. Leé .context/05-model-guide.md para saber qué tareas podés hacer.
3. Leé .context/02-state.md.

Reglas para vos:
- No tomes decisiones editoriales.
- No escribas secciones nuevas del artículo (esa tarea es para Sonnet 4).
- Si te pido algo fuera de tu zona, avisame y sugerí cambiar de modelo.
- Sí podés: leer, buscar, listar, actualizar state, hacer commits que yo te dicte.
```

---

## Prompt para session handoff (cierre + transferencia)

Antes de cerrar una sesión donde se trabajó, pasale esto al modelo:

```
Vamos a cerrar la sesión. Antes de hacerlo:

1. Actualizá .context/02-state.md con:
   - Fecha de hoy
   - Qué modelo usamos (vos)
   - Qué se hizo en esta sesión
   - Próximo paso concreto
   - Agregá línea a "Sesiones anteriores"

2. Si tomamos decisiones nuevas, agregalas a .context/07-decisions.md.

3. Hacé commit con mensaje claro: "state: [qué se hizo]".

4. Confirmame que está todo guardado antes de cerrar.
```

---

## Prompt para verificar integridad del contexto

Cada cierto tiempo, correlo para asegurar que el contexto sigue coherente:

```
Verificá la coherencia del contexto del proyecto:

1. ¿El estado en .context/02-state.md coincide con los archivos reales en PROD/?
2. ¿Hay números en el artículo o documentación que NO estén en data/verified-metrics.md?
3. ¿Hay archivos duplicados o instrucciones contradictorias entre .context/ y CLAUDE.md?
4. ¿Hay decisiones tomadas en chat reciente que no estén en 07-decisions.md?

Reportame lo que encuentres. No corrijas nada sin confirmar.
```
