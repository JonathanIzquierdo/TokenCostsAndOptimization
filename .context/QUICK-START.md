# QUICK-START — Lo primero que leo al retomar el proyecto

> Archivo de **arranque rápido** pensado para minimizar tokens al retomar.
> Si lo leés vos (agente IA) al inicio de una sesión: con esto + `CLAUDE.md` + `03-rules.md` ya tenés contexto suficiente para el 80% de los pedidos.
> Solo bajá a `02-state.md` si necesitás el log detallado de sesiones anteriores.

**Última sesión:** 24 mayo 2026 — Opus 4.7.

---

## ESTADO AL CIERRE

**Carpeta activa de entregables:** `PRODV2/`. Adentro hay 3 archivos finales en español y una subcarpeta `EN/` con sus traducciones.

```
PRODV2/
├── articulo-final.md            ← Artículo largo ES (34 KB, lectura 25-30 min)
├── articulo-final-short.md      ← Artículo corto ES (14 KB, lectura 10-12 min)
├── doc-tecnico-final.md         ← Doc técnico ES (120 KB, biblia completa)
└── EN/
    ├── articulo-final-EN.md
    ├── articulo-final-short-EN.md
    └── doc-tecnico-final-EN.md
```

**Carpetas históricas (NO tocar salvo que el usuario lo pida explícito):** `PROD/` (drafts viejos v1), `Examples/` (referencia de estilo), `Agreements/` (acuerdo Anthropic-Visma, NO mencionar por nombre ni cifras en entregables).

---

## QUÉ ESTÁ TERMINADO

- ✅ Investigación 9 temas (`research/01..09`)
- ✅ Métricas verificadas (`data/verified-metrics.md`)
- ✅ Artículo final ES + EN (largo y short)
- ✅ Doc técnico final ES + EN con script Python local + HTML de dashboard personal incluido en sección 9.6
- ✅ Traducciones al inglés de los 3 archivos en `PRODV2/EN/`

---

## DECISIONES EDITORIALES QUE YA ESTÁN APLICADAS (no reabrir)

1. **Sin guiones largos (—).** El usuario no los quiere en ningún texto generado por Claude. Reemplazar siempre por coma, punto, dos puntos o paréntesis. Guiones cortos en palabras compuestas técnicas (pay-per-token, usage-based) sí permitidos.
2. **Palabra "palanca" reemplazada** en todo el material por `ajuste` / `optimización` / `control` según contexto, porque "lever" se confunde con leverage financiero en inglés.
3. **Siglas glosadas inline la primera vez que aparecen**, no en glosario aparte. Aplicado a: BU, MACC, EA, TCO, MDM, JSONL, ZDR, RAG, OTel, BYOK.
4. **Sección stoppers (8.3 del técnico, sección equivalente en artículo) reescrita.** Encuadre correcto: **ningún vendor serio te explota la factura por default**. Rate limits + cuotas del plan + spending limits configurables paran antes. El riesgo solo aparece si el usuario activa overage conscientemente.
5. **Gobernanza personal (9.6 técnico) reescrita.** Camino principal A: **script Python + HTML estático local** con Chart.js, ~150 líneas, sin dependencias. Camino B: Aspire Dashboard local (docker). Grafana Cloud solo mención secundaria.
6. **Espectro de 3 niveles:** "productos cerrados / herramientas con configuración / API directa". No usar metáfora caja negra/gris.
7. **Placeholders de links cruzados:** cada vez que un archivo menciona otro entregable del par, usar `(Link al documento técnico)` o `(Link al artículo extendido)` textual entre paréntesis. El usuario los reemplaza después.
8. **Frase guía:** *"Gastar bien, no gastar menos"* en ES y *"Spend well, not spend less"* en EN. Casi marca registrada del proyecto.
9. **NO mencionar Anthropic por nombre** en el acuerdo Visma-Anthropic ni cifras del mismo. Mantener tono genérico.

---

## TERMINOLOGÍA EN INGLÉS (decisiones aplicadas)

Mantener en inglés tal cual (jerga universal): prompt caching, output/input tokens, system prompt, tool calls, extended thinking, rate limit, spending limit, overage, flat fee, usage-based billing, seat, AI Credits, MTok, flagship, caching, routing, batch, pay-per-token, fine-tuning, wrapper, gateway, endpoint, hyperscaler. Productos: Copilot, Claude Code, Cursor, Bedrock, Vertex, Foundry sin traducir.

---

## SI EL USUARIO PIDE...

| Pedido | Archivo a tocar |
|---|---|
| "Ajustar el artículo largo" | `PRODV2/articulo-final.md` (+ replicar en EN) |
| "Ajustar el artículo corto" | `PRODV2/articulo-final-short.md` (+ EN) |
| "Ajustar el técnico" | `PRODV2/doc-tecnico-final.md` (+ EN) |
| "Cambiar una cifra" | Primero verificar en `data/verified-metrics.md`. Si no está, buscar fuente; no inventar. |
| "Generar versión Word/docx" | Ya se hizo manualmente fuera del repo. Si pide de nuevo: usar skill docx con docx-js. |
| "Cambio en una decisión editorial de la lista de arriba" | Confirmar antes con el usuario que quiere reabrir esa decisión, no asumir. |

---

## POSIBLES PRÓXIMOS PASOS (sin orden, lo que el usuario quiera)

- Pulir o iterar contenido sobre la base existente (cualquiera de los 6 archivos).
- Generar archivos en formato Word/docx si quiere distribuir por mail.
- Reemplazar los placeholders `(Link al documento técnico)` por URLs reales del repo cuando estén los archivos en el sistema final.
- Crear un README específico de PRODV2 que explique el par largo/short/técnico para alguien que llega a la carpeta sin contexto.
- Iniciar una pieza nueva (teaching module mencionado en `.context/01-project.md` como entregable futuro).

---

## PROTOCOLO DE ARRANQUE ABREVIADO

Si ya leíste este archivo + `CLAUDE.md` + `03-rules.md`, decile al usuario en una línea: *"Leí el contexto. Tenemos los 6 archivos finales en PRODV2 (3 ES + 3 EN). ¿Qué querés ajustar hoy?"* y esperá. No abras ningún archivo grande hasta que el usuario diga qué tocar.

**Regla de oro de tokens:** no leas el archivo de PRODV2 hasta tener confirmación de que se va a tocar. Son 14 a 120 KB cada uno.
