# CLAUDE MULTIPROJECT — Instrucciones para Agente AI

Este archivo es el PRIMERO que debe leer cualquier agente AI (Claude, ChatGPT, Gemini u otro) antes de trabajar en este repositorio. Contiene todo lo necesario para retomar el proyecto sin contexto previo, desde cualquier cuenta.

---

## QUIÉN SOS Y QUÉ ESTÁS HACIENDO

Sos un agente AI trabajando en un proyecto de escritura técnica para el equipo de Visma. El proyecto está 100% documentado en este repositorio. No dependés de ninguna conversación previa — todo lo que necesitás está acá.

El dueño del proyecto es Jonathan Izquierdo. Cuando arranques una sesión nueva, confirmá que leíste este archivo y los archivos clave del repo antes de hacer cualquier cosa.

---

## QUÉ ES EL PROYECTO

### El objetivo
Producir DOS entregables completos:

**Entregable 1 — Artículo**
Un artículo con storytelling técnico. No tan largo pero completo. Que atrape al lector y lo lleve por distintos niveles de conciencia sobre el consumo de tokens en IA. Tono: un colega del equipo que descubrió algo importante y lo comparte. Incluye links a la documentación técnica para quien quiera profundizar. El artículo tiene que poder leerse de corrido sin sentirse como un manual.

**Entregable 2 — Documentación técnica**
Documentación detallada, sin resumir, con toda la profundidad necesaria. Cada tema cubierto en detalle. Con métricas verificadas, ejemplos de código donde aplica, y guías accionables. El lector debe poder abrir cualquier sección y saber exactamente qué hacer. Jonathan fue explícito: "Si hay mucha data no me molesta, no trates de resumir."

### La audiencia
Equipo técnico de Visma — desarrolladores, tech leads y architects. La audiencia es mixta: algunos solo usan Copilot, otros construyen agentes y pipelines. El artículo va dirigido a TODOS los niveles. La documentación técnica va dirigida a los perfiles más técnicos.

**No explicar:** qué es un LLM, qué es un token, qué es un modelo de AI. La audiencia ya lo sabe.

### El idioma
**Español.** Todo el artículo y la documentación en español.

### Por qué este proyecto existe ahora
Dos eventos concretos lo dispararon:

1. **GitHub Copilot cambia a usage-based billing el 1 de junio 2026.** Ya no es un flat fee. Cada token consumido tiene un costo directo. La estimación interna es que la factura enterprise sube ~3x. Esto hace que la optimización de tokens pase de "buena práctica" a "necesidad operacional".

2. **VSCode lanzó `chat.tools.compressOutput.enabled`** — una feature que post-procesa el output de terminal antes de enviarlo al modelo. Esto motivó explorar qué más existe en el ecosistema.

---

## ESTADO ACTUAL DEL PROYECTO

| Paso | Descripción | Estado |
|------|-------------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional | 🔄 Opcional si hay gaps |
| 4 | Redacción del artículo | ⏳ PRÓXIMO PASO |
| 5 | Redacción de documentación técnica | ⏳ Pendiente |

**Próxima acción concreta:** Empezar a redactar el artículo siguiendo el outline en `article/outline.md`. Usar los datos de `data/verified-metrics.md`. No inventar métricas — solo usar datos con fuente verificada.

---

## ARCHIVOS DEL REPOSITORIO Y PARA QUÉ SIRVE CADA UNO

```
README.md                          — Overview general del proyecto
PROJECT-INSTRUCTIONS.md            — Contexto completo, decisiones tomadas, reglas
claude-multiproject.md             — Este archivo (leer primero)
research/
  01-context-and-token-waste.md    — El problema base: contexto desperdiciado (62% de la factura)
  02-model-selection.md            — Selección de modelo + MCPs innecesarios
  03-prompt-caching.md             — Prompt caching (90% descuento) + memory compression
  04-model-routing.md              — Routing en 3 niveles: Auto Mode → RouteLLM → Gateways
  05-copilot-billing.md            — Nuevo billing Copilot junio 2026
  06-vscode-tools.md               — Herramientas VSCode para reducir tokens
  07-azure-deepseek.md             — Azure EA discounts + DeepSeek via Azure (compliance)
  08-batch-and-compaction.md       — Batch API (50% off) + Compaction API Anthropic
  09-extended-thinking-costs.md    — El costo oculto del extended thinking
data/
  verified-metrics.md              — TODOS los números con fuente y URL verificada
article/
  outline.md                       — Estructura del artículo (borrador aprobado)
  draft-v1.md                      — Borrador del artículo (se crea al redactar)
  draft-final.md                   — Versión final del artículo
docs/
  (se crean al redactar la documentación técnica)
```

---

## REGLAS QUE NO SE PUEDEN ROMPER

### Regla 1: Ningún número sin fuente
Ningún dato numérico puede aparecer en el artículo o la documentación sin una URL verificada en `data/verified-metrics.md`. Si necesitás un número que no está ahí, buscalo primero en la web y agregalo al archivo de métricas con su fuente antes de usarlo. Nunca inventar ni estimar porcentajes.

### Regla 2: No resumir innecesariamente
Jonathan pidió explícitamente: "Si hay mucha data no me molesta, no trates de resumir." La documentación técnica debe ser detallada y completa. No recortar por longitud.

### Regla 3: Foco en costo y gobernanza
Cada tema debe conectar con: ¿esto mueve la factura? ¿es accionable para el equipo? No incluir temas de accuracy o calidad de respuesta a menos que el vínculo con el costo sea directo e inevitable.

### Regla 4: Tono correcto para el artículo
El artículo debe sonar como un colega del equipo que descubrió algo importante — no como documentación oficial, no como post de marketing, no como manual corporativo. Directo, técnico, sin condescendencia.

### Regla 5: No explicar lo básico
La audiencia sabe qué es un token, un modelo, un LLM, un agente. No explicar conceptos básicos.

### Regla 6: Guardar el progreso en el repo
Al terminar cada sesión o sección importante, subir el progreso al repo. No dejar trabajo solo en el chat. Usar commits descriptivos como "Add article draft: apertura y sección 1".

---

## DECISIONES YA TOMADAS — NO REABRIR

### Modelo para redacción
**Claude Sonnet 4** (`claude-sonnet-4-20250514`). No usar Opus 4 — sería el anti-patrón que el artículo critica.

### El número "24% de descuento Azure"
Es un acuerdo negociado específico de Visma, no un precio publicado. El rango verificado es 15-28% según leverage comercial. En el artículo presentarlo como rango negociable, nunca como número fijo.

### DeepSeek via Azure
El ángulo correcto es compliance, no precio. Azure cobra 20-35% más que DeepSeek directo. El valor es poder usar un modelo muy barato sin enviar datos a China. Incluso con el markup, es ~15x más barato que Sonnet para tareas de alta volumetría.

### Tema descartado: "Lost in the middle"
Vínculo con el costo demasiado indirecto. Descartado como punto standalone.

---

## RESUMEN DE LOS DATOS MÁS IMPORTANTES

Datos completos con URLs en `data/verified-metrics.md`. Los más críticos para el artículo:

| Dato | Valor | Fuente |
|------|-------|--------|
| Contexto re-enviado = % factura | 62% | LeanOps 2026 (30 equipos) |
| Tokens turno 1 vs turno 50 (agentes) | 5K → 200K | Redblink |
| Multiplicador output vs input | 4–6x | Redis 2026 |
| Descuento prompt caching Anthropic | 90% en cache reads | Anthropic oficial |
| RouteLLM: calidad GPT-4 con % llamadas | 95% calidad con 26% llamadas | LMSYS ICLR 2025 |
| RouteLLM con augmentation | 75% reducción de costo | LMSYS ICLR 2025 |
| Copilot top usuarios = % del spend | 10-15% = 60-70% del total | Synapx |
| Multiplicador workflows agénticos | ~3.5x flat fee anterior | Synapx |
| Auto Mode Copilot descuento | 10% en multiplicador | GitHub Changelog |
| Batch API descuento | 50% en input y output | Anthropic oficial |
| Batch + caching combinados | hasta 95% de ahorro | PECollective |
| Extended thinking display:omitted | No ahorra — se cobra igual | CheckThat.ai |
| DeepSeek V4-Flash vs Claude Sonnet | ~15x más barato en input | Precios 2026 |
| Azure EA discount rango | 15-28% negociado | Microsoft Negotiations |

---

## ESTRUCTURA DEL ARTÍCULO (outline aprobado)

Ver `article/outline.md` para la versión completa. Resumen ejecutivo:

**Título:** "El dinero que se pierde en cada conversación con la IA"

1. **Apertura** — Hook: el 62% de la factura es contexto re-enviado. La analogía: pagamos por el trabajo del modelo, pero en realidad pagamos por lo que le contamos antes de trabajar.

2. **El problema invisible** — Cómo crece el costo sin que nadie lo vea: la sesión que va de 5K a 200K tokens en el turno 50. Output tokens 4-6x más caros. Extended thinking invisible. El 10-15% de usuarios que concentra el 60-70% del gasto.

3. **El cambio urgente** — Copilot usage-based billing junio 2026. Multiplicador agéntico ~3.5x. Deloitte: AI hasta 50% del gasto IT.

4. **Las capas de optimización** — De fácil a avanzado:
   - Capa 0: hoy, sin cambiar nada (Auto Mode, compressOutput, tool search)
   - Capa 1: arquitectura de contexto (prompt caching, compaction)
   - Capa 2: selección de modelo correcta (Haiku/Sonnet/Opus + MCPs)
   - Capa 3: routing inteligente (RouteLLM, LiteLLM, Portkey)
   - Capa 4: palancas de infraestructura (Batch API, DeepSeek via Azure)

5. **Cierre** — De práctica individual a cultura de equipo. Gobernanza = visibilidad. El objetivo no es gastar menos, es gastar bien.

---

## CÓMO ARRANCAR UNA SESIÓN NUEVA

### Mensaje para pegarle a Claude al inicio de sesión

> "Trabajá en el repositorio GitHub JonathanIzquierdo/TokenCostsAndOptimization. Leé primero `claude-multiproject.md`, luego `article/outline.md` y `data/verified-metrics.md`. El estado: pasos 1 y 2 completados. Próximo paso: redactar el artículo en español siguiendo el outline aprobado, guardando el progreso en `article/draft-v1.md`. La audiencia es el equipo técnico de Visma (todos los niveles). Tono: colega que comparte algo importante, no manual corporativo. Ningún número sin fuente verificada en verified-metrics.md."

### Flujo de trabajo recomendado para continuar
1. Leer este archivo
2. Leer `article/outline.md`
3. Leer `data/verified-metrics.md`
4. Redactar el artículo sección por sección, guardando en `article/draft-v1.md`
5. Una vez aprobado el artículo, redactar la documentación técnica basándose en los archivos de `research/`
6. Review final: verificar números con fuente, tono correcto, links funcionan

---

## CONTEXTO ADICIONAL QUE UN AGENTE DEBE SABER

### Cómo se recolectó la información
Toda la información fue recolectada en mayo 2026 buscando en web en tiempo real. Las fuentes son artículos de 2025-2026. Ningún número fue estimado ni interpolado — todo viene de artículos o estudios reales.

### El contexto europeo importa
Visma es una empresa europea con requerimientos de compliance estrictos (GDPR, data residency). Eso hace que temas como DeepSeek via Azure (compliance vs precio) y las negociaciones del EA de Microsoft sean especialmente relevantes. El artículo debe reflejar ese contexto cuando corresponda.

### El mensaje de fondo del proyecto
Este no es solo un proyecto de reducción de costos. Es un proyecto de cultura de ingeniería. El mensaje: gastar bien, no gastar menos. Visibilidad, gobernanza, conciencia colectiva. El equipo que entiende cómo funcionan los tokens va a tomar mejores decisiones técnicas en general.

### Tono del artículo — ejemplos
**Bien:** "El 62% de lo que pagás en AI no es por el trabajo que hace el modelo — es por lo que ya le contaste antes."
**Bien:** "La sesión que empieza en 5.000 tokens llega a 200.000 en el turno 50. Nadie lo ve. Nadie lo para. Hasta que llega la factura."
**Mal:** "En este artículo exploraremos las mejores prácticas para optimizar el consumo de tokens en entornos de producción."
**Mal:** "Es importante destacar que los output tokens tienen un costo significativamente mayor."
