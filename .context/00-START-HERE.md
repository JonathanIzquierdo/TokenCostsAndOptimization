# 00 — START HERE

## Protocolo de arranque detallado

Este archivo expande el protocolo del `CLAUDE.md` raíz con la lógica detrás de cada paso. Si sos un modelo nuevo en el proyecto, leé esto antes de actuar.

---

## Paso 1: Leer `CLAUDE.md` (raíz del repo)

**Por qué:** Es el bootstrap. Te ubica en el proyecto en menos de 3 minutos.

**Qué buscás ahí:**
- Qué es el proyecto en 2 líneas
- El mapa del repo
- Las 4 reglas innegociables
- El principio rector

---

## Paso 2: Leer `.context/02-state.md`

**Por qué:** Es el único archivo del repo que cambia entre sesiones. Te dice **en qué punto exacto** estamos.

**Qué buscás ahí:**
- Última sesión: qué se hizo
- Estado de cada entregable
- Próximo paso concreto y accionable
- Bloqueos o decisiones pendientes

**Si este archivo está desactualizado** (la última entrada es de hace muchos días o no menciona el trabajo reciente), preguntale al usuario qué se hizo en el medio antes de continuar.

---

## Paso 3: Leer `.context/01-project.md`

**Por qué:** Te da el *por qué* del proyecto. Sin esto vas a tomar decisiones editoriales equivocadas.

**Qué buscás ahí:**
- Audiencia exacta (no es "gente técnica" — son developers y tech leads de Visma)
- Por qué existe ahora (Copilot billing junio 2026 + VSCode features)
- Qué se considera éxito
- Tono y formato

---

## Paso 4: Leer `.context/03-rules.md`

**Por qué:** Hay 4 reglas que no se pueden romper. Romperlas arruina el entregable.

**La regla más importante:**
> Ningún número aparece en el proyecto sin URL de fuente verificada en `data/verified-metrics.md`.

Si tenés que poner un número y no lo tenés con fuente: marcalo `[VERIFICAR]` y avisalo al usuario. **No inventes ni interpoles.**

---

## Paso 5: Confirmar al usuario antes de actuar

**Por qué:** Este paso previene desastres. Un modelo que arranca sin confirmar puede:
- Trabajar sobre un archivo equivocado
- Repetir trabajo ya hecho
- Inventar números porque no entendió el contexto
- Cambiar el idioma del entregable

**Formato de confirmación esperado:**

```
Leí el contexto del proyecto.

Estado actual: [una línea desde 02-state.md]
Lo que entiendo que pedís: [tu interpretación]
Lo que voy a hacer: [paso concreto]
Archivos que voy a tocar: [lista]

¿Confirmás antes de arrancar?
```

---

## Cuándo SÍ podés saltarte la confirmación

Solo si el usuario:
- Te dijo explícitamente "no me preguntes, hacelo"
- Te está pidiendo una respuesta puntual que no toca archivos del repo
- Ya estás en el medio de una tarea aprobada

---

## Cuándo NO podés saltarte la confirmación

- Cuando vas a tocar `PROD/` (entregables en producción)
- Cuando vas a agregar números o métricas
- Cuando vas a cambiar la estructura del repo
- Cuando vas a eliminar archivos
- Cuando el usuario te pide algo en una sesión nueva

---

## Si sos Haiku, GPT-mini, Flash o un modelo chico

**Bienvenido.** Este proyecto está diseñado para que vos también puedas trabajar.

Reglas extra para vos:
1. **No interpretes — preguntá.** Si una instrucción tiene 2 interpretaciones posibles, listalas y pedile al usuario que elija.
2. **No condenses contexto.** Si necesitás algo de un archivo grande, leé el archivo entero — no resumas mentalmente lo que leíste.
3. **No tomes decisiones editoriales.** Cambios de estructura, de tono, de tema → siempre confirmar.
4. **Para escritura del artículo:** redirigí al usuario a usar Sonnet 4. La calidad narrativa del artículo lo exige.

Leé `.context/05-model-guide.md` para ver qué tareas SÍ podés hacer bien.
