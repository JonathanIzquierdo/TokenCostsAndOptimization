# CLAUDE.md — LEER ANTES DE HACER CUALQUIER COSA

> Este archivo es el **bootstrap obligatorio** del proyecto. Si sos un agente AI (Claude, ChatGPT, Gemini, Copilot, da igual) y no leíste este archivo completo, **parate** y leelo antes de responder cualquier pregunta del usuario.
>
> Si sos un modelo chico (Haiku, GPT-4o-mini, Gemini Flash) y no entendés algo: leé el archivo igualmente y seguí el protocolo paso a paso. No improvises.

---

## 🚨 PROTOCOLO DE ARRANQUE OBLIGATORIO

**Antes de tu primera respuesta sustantiva en este proyecto, ejecutá estos 5 pasos en orden. Sin saltearte ninguno.**

1. **Leé este archivo (`CLAUDE.md`) completo.**
2. **Leé `.context/02-state.md`** — te dice qué se hizo y qué falta. Es el archivo que cambia más seguido.
3. **Leé `.context/01-project.md`** — qué es el proyecto, audiencia, idioma, objetivos.
4. **Leé `.context/03-rules.md`** — las reglas innegociables del proyecto. Si las rompés, rompés el proyecto.
5. **Decí en una línea qué vas a hacer.** Antes de tocar cualquier archivo, confirmá al usuario: "Leí el contexto. El estado actual es X. Voy a hacer Y. ¿Confirmás?"

**No saltees el paso 5.** Aunque parezca obvio. Aunque el usuario te apure. La confirmación previene que un modelo desinformado rompa el repo.

---

## 📌 RESUMEN EJECUTIVO DEL PROYECTO

**Qué es:** Investigación y redacción de un artículo técnico + documentación detallada sobre **costo y gobernanza de tokens en IA**, para el equipo de ingeniería de Visma.

**Idioma:** Español (con versiones EN en `PROD/`).

**Estado actual (resumen — fuente de verdad en `.context/02-state.md`):**
- Investigación: ✅ completada
- Borradores v1 artículo y documentación: ✅ completados, en `PROD/`
- Próximo paso: **review e incorporación de feedback** para producir versión final

**Dueño:** Jonathan Izquierdo.

**Modelo recomendado para redacción/escritura:** `claude-sonnet-4-20250514`.

---

## 🗺️ MAPA DEL REPO — DÓNDE ESTÁ TODO

```
CLAUDE.md                    ← Vos estás acá. Bootstrap obligatorio.
README.md                    ← Overview público del proyecto.

.context/                    ← Ingeniería de contexto. Leer en orden numérico.
  00-START-HERE.md           ← Protocolo de arranque detallado.
  01-project.md              ← Qué, por qué, audiencia, idioma.
  02-state.md                ← Estado vivo del proyecto. ⚠️ Actualizar al cerrar sesión.
  03-rules.md                ← Reglas innegociables.
  04-glossary.md             ← Vocabulario interno (PROD/, Capa 0-4, JON.md, etc.)
  05-model-guide.md          ← Qué modelo usar para qué tarea.
  06-files-map.md            ← Qué archivo leer según lo que el usuario te pida.
  07-decisions.md            ← Decisiones cerradas. No reabrir.
  08-prompts.md              ← Prompts listos para copiar al iniciar sesión.

research/                    ← 9 archivos de investigación por tema. Fuente de verdad temática.
data/verified-metrics.md     ← TODOS los números del proyecto con URL de fuente. SIN EXCEPCIÓN.
article/outline.md           ← Estructura aprobada del artículo.
PROD/                        ← Entregables en producción.
  article-draft-v1.md        ← Artículo ES (borrador en review).
  article-draft-v1-EN.md     ← Artículo EN (borrador en review).
  technical-docs-draft-v1.md ← Documentación técnica ES.
  technical-docs-draft-v1-EN.md
  JON.md / JON-EN.md         ← Índice ejecutivo para el dueño del proyecto.
  claude-design-brief-ES.md  ← Brief de visuales para Claude Design.
  claude-design-brief-EN.md
```

---

## 🚦 LAS 4 REGLAS QUE NO PODÉS ROMPER

Detalle completo en `.context/03-rules.md`. Resumen:

1. **Ningún número sin fuente verificada.** Si no está en `data/verified-metrics.md` con URL real, marcalo `[VERIFICAR]` y buscalo antes de usar.
2. **No resumir documentación.** La doc técnica es exhaustiva por diseño. Si el usuario pide "resumí", confirmá que entendiste que vas contra la regla.
3. **Foco en costo y gobernanza.** Cada cosa que agregues debe responder: ¿esto mueve la factura? ¿es accionable?
4. **No explicar lo básico.** La audiencia sabe qué es un token, qué es un LLM, qué es un modelo. No los traten de tontos.

---

## ⚠️ ANTES DE CERRAR LA SESIÓN

Si hiciste cambios, **actualizá `.context/02-state.md`** con:
- Qué hiciste en esta sesión
- En qué archivo quedaste
- Cuál es el próximo paso concreto

Esto es lo que permite que la próxima sesión (con vos, con otro modelo, con otra cuenta) retome sin perder contexto.

---

## 🆘 SI ALGO NO TE QUEDA CLARO

1. Releé `.context/00-START-HERE.md`.
2. Buscá en `.context/04-glossary.md` cualquier término que no entiendas.
3. Si después de eso seguís sin entender, **preguntá al usuario antes de actuar**. No improvises.

**Improvisar en este proyecto cuesta dinero** (literalmente — el proyecto trata sobre eso). Un agente que inventa números rompe la regla #1 y arruina el entregable.

---

## 🎯 PRINCIPIO RECTOR

> *"Gastar bien, no gastar menos."*

Es el mensaje del artículo y la regla del proyecto. Aplica también a tu trabajo como agente: usá el modelo correcto para la tarea, el contexto justo, las herramientas necesarias. Ni más ni menos.
