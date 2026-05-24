# CLAUDE.md — LEER ANTES DE HACER CUALQUIER COSA

> Este archivo es el **bootstrap obligatorio** del proyecto. Si sos un agente AI (Claude, ChatGPT, Gemini, Copilot, da igual) y no leíste este archivo completo, **parate** y leelo antes de responder cualquier pregunta del usuario.
>
> Si sos un modelo chico (Haiku, GPT-4o-mini, Gemini Flash) y no entendés algo: leé el archivo igualmente y seguí el protocolo paso a paso. No improvises.

---

## 🚨 PROTOCOLO DE ARRANQUE OBLIGATORIO

**Antes de tu primera respuesta sustantiva en este proyecto, ejecutá estos pasos en orden. Sin saltearte ninguno.**

1. **Leé este archivo (`CLAUDE.md`) completo.**
2. **Leé `.context/QUICK-START.md`** — atajo de arranque, te ubica en 30 segundos sobre dónde está todo y cuáles son las decisiones editoriales activas. Es el archivo más importante para retomar rápido.
3. **Leé `.context/03-rules.md`** — las reglas innegociables del proyecto. Si las rompés, rompés el proyecto.
4. **Solo si necesitás el log detallado**, leé `.context/02-state.md`. Para la mayoría de las sesiones no hace falta.
5. **Decí en una línea qué vas a hacer.** Antes de tocar cualquier archivo, confirmá al usuario: "Leí el contexto. El estado actual es X. ¿Qué querés ajustar hoy?" y esperá su pedido. No abras archivos grandes hasta saber qué se va a tocar.

**No saltees el paso 5.** Aunque parezca obvio. Aunque el usuario te apure. La confirmación previene que un modelo desinformado rompa el repo.

---

## 📌 RESUMEN EJECUTIVO DEL PROYECTO

**Qué es:** Investigación y redacción de un artículo técnico + documentación detallada sobre **costo y gobernanza de tokens en IA**, para el equipo de ingeniería de Visma.

**Idiomas:** Español (originales en `PRODV2/`) y English (traducciones en `PRODV2/EN/`).

**Estado actual (resumen — fuente de verdad en `.context/QUICK-START.md` y `.context/02-state.md`):**

- Investigación: ✅ completada
- Entregables finales: ✅ 6 archivos en `PRODV2/` (3 ES + 3 EN)
  - `articulo-final.md` / `articulo-final-EN.md` — artículo largo (~34 KB)
  - `articulo-final-short.md` / `articulo-final-short-EN.md` — versión corta (~14 KB)
  - `doc-tecnico-final.md` / `doc-tecnico-final-EN.md` — doc técnico completo (~120 KB)
- Próximo paso: a definir por el usuario al inicio de la próxima sesión

**Dueño:** Jonathan Izquierdo.

**Modelo recomendado para redacción/escritura:** `claude-sonnet-4-6` o superior. Para ajustes chicos sobre archivos existentes, Haiku 4.5 alcanza.

---

## 🗺️ MAPA DEL REPO — DÓNDE ESTÁ TODO

```
CLAUDE.md                    ← Vos estás acá. Bootstrap obligatorio.
README.md                    ← Overview público del proyecto.

.context/                    ← Ingeniería de contexto. Empezar por QUICK-START.
  QUICK-START.md             ← ⭐ Atajo de arranque. Leer SIEMPRE primero.
  00-START-HERE.md           ← Protocolo de arranque histórico (más extenso que QUICK-START).
  01-project.md              ← Qué, por qué, audiencia, idioma.
  02-state.md                ← Log detallado del estado del proyecto. Actualizar al cerrar sesión.
  03-rules.md                ← Reglas innegociables.
  04-glossary.md             ← Vocabulario interno.
  05-model-guide.md          ← Qué modelo usar para qué tarea.
  06-files-map.md            ← Qué archivo leer según el pedido.
  07-decisions.md            ← Decisiones cerradas. No reabrir.
  08-prompts.md              ← Prompts listos para iniciar sesión.

research/                    ← 9 archivos de investigación. Fuente de verdad temática.
data/verified-metrics.md     ← Todos los números del proyecto con URL. SIN EXCEPCIÓN.
article/outline.md           ← Estructura aprobada del artículo (referencia histórica).

PRODV2/                      ← ⭐ ENTREGABLES FINALES. Carpeta activa.
  articulo-final.md          ← Artículo largo ES (34 KB, 25-30 min lectura)
  articulo-final-short.md    ← Artículo corto ES (14 KB, 10-12 min lectura)
  doc-tecnico-final.md       ← Doc técnico ES (120 KB, biblia completa con script Python + HTML)
  EN/
    articulo-final-EN.md
    articulo-final-short-EN.md
    doc-tecnico-final-EN.md
  hyperscaler-discount-vs-gdpr-ES.md  ← Doc standalone histórico (contenido absorbido en sección 3 del técnico final)

PROD/                        ← ⚠️ DRAFTS HISTÓRICOS. No tocar salvo pedido explícito.
  article-draft-v1.md, technical-docs-draft-v1.md, etc.
  JON.md / JON-EN.md         ← Índice ejecutivo para el dueño (puede necesitar update si se referencia PRODV2)
  claude-design-brief-ES.md / EN.md  ← Briefs de visuales para Claude Design.

Examples/                    ← Referencia de estilo. Solo lectura.
Agreements/                  ← Acuerdos. NO mencionar por nombre ni cifras en entregables.
```

---

## 🚦 LAS 4 REGLAS QUE NO PODÉS ROMPER

Detalle completo en `.context/03-rules.md`. Resumen:

1. **Ningún número sin fuente verificada.** Si no está en `data/verified-metrics.md` con URL real, marcalo `[VERIFICAR]` y buscalo antes de usar.
2. **No resumir documentación técnica.** La doc técnica es exhaustiva por diseño. Si el usuario pide "resumí", confirmá que entendiste que vas contra la regla.
3. **Foco en costo y gobernanza.** Cada cosa que agregues debe responder: ¿esto mueve la factura? ¿es accionable?
4. **No explicar lo básico.** La audiencia sabe qué es un token, qué es un LLM, qué es un modelo. No los trates de tontos.

---

## ✏️ DECISIONES EDITORIALES ACTIVAS (lista corta)

Detalle en `.context/QUICK-START.md` sección "DECISIONES EDITORIALES YA APLICADAS". Las críticas:

- **Sin guiones largos (—)** en ningún texto generado. Reemplazar por coma, punto, dos puntos o paréntesis.
- **Palabra "palanca" eliminada**, reemplazada por ajuste/optimización/control.
- **Siglas con glosa inline** la primera vez (BU, MACC, EA, TCO, MDM, JSONL, ZDR, RAG, OTel, BYOK).
- **Frase guía:** *"Gastar bien, no gastar menos"* (ES) / *"Spend well, not spend less"* (EN).
- **NO mencionar Anthropic por nombre** en el acuerdo Visma-Anthropic ni cifras del mismo.

---

## ⚠️ ANTES DE CERRAR LA SESIÓN

Si hiciste cambios, **actualizá `.context/02-state.md`** con:
- Qué hiciste en esta sesión
- En qué archivo quedaste
- Cuál es el próximo paso concreto

Si cambiaron decisiones editoriales o aparecieron nuevas, sincronizar también `.context/QUICK-START.md`.

Esto es lo que permite que la próxima sesión retome sin perder contexto.

---

## 🆘 SI ALGO NO TE QUEDA CLARO

1. Releé `.context/QUICK-START.md`.
2. Buscá en `.context/04-glossary.md` cualquier término que no entiendas.
3. Si después de eso seguís sin entender, **preguntá al usuario antes de actuar**. No improvises.

**Improvisar en este proyecto cuesta dinero** (literalmente — el proyecto trata sobre eso). Un agente que inventa números rompe la regla #1 y arruina el entregable.

---

## 🎯 PRINCIPIO RECTOR

> *"Gastar bien, no gastar menos."*

Es el mensaje del artículo y la regla del proyecto. Aplica también a tu trabajo como agente: usá el modelo correcto para la tarea, el contexto justo, las herramientas necesarias. Ni más ni menos. **No leas archivos grandes hasta saber qué vas a tocar.**
