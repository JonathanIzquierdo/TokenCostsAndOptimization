# QUICK-START — Lo primero que leo al retomar el proyecto

> Archivo de **arranque rápido** pensado para minimizar tokens al retomar.
> Si lo leés vos (agente IA) al inicio de una sesión: con esto + `CLAUDE.md` + `03-rules.md` ya tenés contexto suficiente para el 80% de los pedidos.
> Solo bajá a `02-state.md` si necesitás el log detallado de sesiones anteriores.

**Última sesión:** 25 mayo 2026 — Opus 4.7.

---

## ESTADO AL CIERRE

**Carpeta activa de entregables:** `PRODV2/`. Adentro hay 4 archivos en español y una subcarpeta `EN/` con sus traducciones (4 archivos también).

```
PRODV2/
├── articulo-final.md                  ← Artículo largo ES (34 KB, lectura 25-30 min)
├── articulo-final-short.md            ← Artículo corto ES (14 KB, lectura 10-12 min, estilo magazine)
├── articulo-final-short-align.md      ← 🆕 Short alineada con feedback del lead (11.5 KB, executive comms, por rol)
├── doc-tecnico-final.md               ← Doc técnico ES (120 KB, biblia completa)
└── EN/
    ├── articulo-final-EN.md
    ├── articulo-final-short-EN.md
    ├── articulo-final-short-align-EN.md  ← 🆕 Traducción de la short-align
    └── doc-tecnico-final-EN.md
```

**Carpetas históricas (NO tocar salvo que el usuario lo pida explícito):** `PROD/` (drafts viejos v1), `Examples/` (referencia de estilo), `Agreements/` (acuerdo Anthropic-Visma, NO mencionar por nombre ni cifras en entregables).

---

## QUÉ ESTÁ TERMINADO

- ✅ Investigación 9 temas (`research/01..09`)
- ✅ Métricas verificadas (`data/verified-metrics.md`)
- ✅ Artículo final ES + EN (largo y short)
- ✅ Doc técnico final ES + EN con script Python local + HTML de dashboard personal incluido en sección 9.6
- ✅ Traducciones al inglés de los 4 archivos en `PRODV2/EN/`
- ✅ Versión short-align ES + EN (executive comms, lead with business stake, diferenciada por rol dev/tech lead/BU MD, con mención al Accelerator)

---

## DIFERENCIA ENTRE LAS DOS VERSIONES SHORT (importante)

- **`articulo-final-short.md`** — versión condensada estilo magazine. Misma narrativa que la larga pero recortada. Audiencia general, mismo registro.
- **`articulo-final-short-align.md`** — versión alineada con feedback de leadership. Estructura distinta: abre con business stake + top 3 acciones, después bloques diferenciados por rol (developer / tech lead / BU MD), cierra con mención al Accelerator interno. Es la versión recomendada para audiencias ejecutivas o cuando hay que distribuir un brief operativo. No es "mejor" que la otra, es para distinto público.

---

## DECISIONES EDITORIALES QUE YA ESTÁN APLICADAS (no reabrir)

1. **Sin guiones largos (—).** El usuario no los quiere en ningún texto generado por Claude. Reemplazar siempre por coma, punto, dos puntos o paréntesis. Guiones cortos en palabras compuestas técnicas (pay-per-token, usage-based) sí permitidos.
2. **Palabra "palanca" reemplazada** en todo el material por `ajuste` / `optimización` / `control` según contexto.
3. **Siglas glosadas inline la primera vez que aparecen**, no en glosario aparte. Aplicado a: BU, MACC, EA, TCO, MDM, JSONL, ZDR, RAG, OTel, BYOK.
4. **Sección stoppers reescrita.** Encuadre correcto: **ningún vendor serio te explota la factura por default**. Rate limits + cuotas del plan + spending limits configurables paran antes. El riesgo solo aparece si el usuario activa overage conscientemente.
5. **Gobernanza personal reescrita.** Camino principal A: **script Python + HTML estático local** con Chart.js. Camino B: Aspire Dashboard local (docker). Grafana Cloud solo mención secundaria.
6. **Espectro de 3 niveles:** "productos cerrados / herramientas con configuración / API directa". No usar metáfora caja negra/gris.
7. **Placeholders de links cruzados:** cada vez que un archivo menciona otro entregable, usar `(Link al documento técnico)` o `(Link al artículo extendido)` textual entre paréntesis. El usuario los reemplaza después.
8. **Frase guía:** *"Gastar bien, no gastar menos"* en ES y *"Spend well, not spend less"* en EN.
9. **NO mencionar Anthropic por nombre** en el acuerdo Visma-Anthropic ni cifras del mismo.
10. **Cifra ~3.5x para uso agéntico**: usar el dato SIN atribuir a Synapx en el texto. Si alguien pregunta por la fuente, el usuario la pasa aparte.
11. **Accelerator (programa interno):** se puede mencionar como cierre de la versión short-align. **Cuidar wording:** no prometer workshops para todos. El programa está en piloto y la capacidad se asigna por impacto, no por demanda. En esta iteración el usuario pidió sacar los detalles operativos del piloto (waitlist, sign-up). Solo queda la mención al programa, al módulo de tokens en preparación y a los teaching modules self-service.

---

## TERMINOLOGÍA EN INGLÉS (decisiones aplicadas)

Mantener en inglés tal cual (jerga universal): prompt caching, output/input tokens, system prompt, tool calls, extended thinking, rate limit, spending limit, overage, flat fee, usage-based billing, seat, AI Credits, MTok, flagship, caching, routing, batch, pay-per-token, fine-tuning, wrapper, gateway, endpoint, hyperscaler. Productos: Copilot, Claude Code, Cursor, Bedrock, Vertex, Foundry sin traducir.

---

## SI EL USUARIO PIDE...

| Pedido | Archivo a tocar |
|---|---|
| "Ajustar el artículo largo" | `PRODV2/articulo-final.md` (+ replicar en EN) |
| "Ajustar el artículo corto" (estilo magazine) | `PRODV2/articulo-final-short.md` (+ EN) |
| "Ajustar la versión short-align / exec brief" | `PRODV2/articulo-final-short-align.md` (+ EN) |
| "Ajustar el técnico" | `PRODV2/doc-tecnico-final.md` (+ EN) |
| "Cambiar una cifra" | Primero verificar en `data/verified-metrics.md`. Si no está, buscar fuente; no inventar. |
| "Generar versión Word/docx" | Ya se hizo manualmente fuera del repo. Si pide de nuevo: usar skill docx con docx-js. |
| "Cambio en una decisión editorial de la lista de arriba" | Confirmar antes con el usuario que quiere reabrir esa decisión, no asumir. |

---

## POSIBLES PRÓXIMOS PASOS (sin orden, lo que el usuario quiera)

- Pulir o iterar contenido sobre la base existente (cualquiera de los 8 archivos: 4 ES + 4 EN).
- Generar archivos en formato Word/docx si quiere distribuir por mail.
- Reemplazar los placeholders `(Link al documento técnico)` por URLs reales del repo cuando estén los archivos en el sistema final.
- Crear un README específico de PRODV2 que explique cuándo usar cada variante (largo, short magazine, short-align exec, técnico).
- Iniciar una pieza nueva (teaching module mencionado en `.context/01-project.md` como entregable futuro).

---

## PROTOCOLO DE ARRANQUE ABREVIADO

Si ya leíste este archivo + `CLAUDE.md` + `03-rules.md`, decile al usuario en una línea: *"Leí el contexto. Tenemos los 8 archivos finales en PRODV2 (4 ES + 4 EN), incluyendo la versión short-align para executive comms. ¿Qué querés ajustar hoy?"* y esperá. No abras ningún archivo grande hasta que el usuario diga qué tocar.

**Regla de oro de tokens:** no leas el archivo de PRODV2 hasta tener confirmación de que se va a tocar. Son 11 a 120 KB cada uno.
