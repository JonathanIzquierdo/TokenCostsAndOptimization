# 01 — Project

## Qué es

Un proyecto de **escritura técnica con investigación verificada** sobre el costo, optimización y gobernanza del uso de tokens en IA, dirigido al equipo de ingeniería de Visma.

Produce dos entregables complementarios:

1. **Un artículo** — storytelling técnico, atractivo, no muy largo (2.500–3.500 palabras). Que el lector quiera leerlo de corrido y termine sabiendo qué hacer mañana.
2. **Documentación técnica detallada** — sin resumir, exhaustiva, con código y ejemplos, organizada para que cada persona del equipo aplique las capas de optimización según su rol.

El artículo linkea a la documentación para profundizar.

---

## Por qué existe AHORA

Dos eventos concretos lo dispararon:

### Evento 1 — GitHub Copilot cambió a usage-based billing el 1 de junio 2026

Antes era flat fee ($19/seat Business, $39/seat Enterprise). Ahora cada seat incluye "AI Credits" equivalentes al precio del seat + bonus 3 meses. Después se paga por token consumido. Los seat tokens son **pooled** a nivel enterprise Visma.

Estimación interna: la factura enterprise sube aproximadamente 3x bajo el nuevo esquema sin cambios de comportamiento.

### Evento 2 — VSCode 1.120 lanzó `chat.tools.compressOutput.enabled`

Un setting que post-procesa el output de terminal antes de enviarlo al modelo. Mediano impacto solo, pero abrió la pregunta: *¿qué más existe en el ecosistema que podemos activar hoy mismo?*

---

## Audiencia

**Quiénes son:**
- Desarrolladores del equipo Visma (Latam y global)
- Tech leads
- Architects
- Algunos product managers técnicos

**Qué saben ya — NO explicar:**
- Qué es un LLM, un token, un modelo de AI
- Qué es prompt engineering
- Qué es un contexto / contexto de un modelo
- Qué hace GitHub Copilot, Claude Code, Cursor
- Diferencias básicas entre modelos (que existen tiers, que cuestan diferente)

**Qué probablemente NO saben (y sí explicamos):**
- Que el output cuesta 4-6x más que el input
- Que el extended thinking se factura como output aunque uses display:omitted
- Que prompt caching tiene 90% de descuento y casi nadie lo aprovecha
- Que RouteLLM puede dar 95% de la calidad de GPT-4 con el 26% de las llamadas
- Que Copilot Auto Mode da 10% de descuento solo por activarlo
- Que Batch API + caching combinados pueden llegar a 95% de ahorro

**Contexto enterprise específico:**
- Visma es empresa europea — GDPR y data residency importan
- Tiene Microsoft Enterprise Agreement (relevante para Azure pricing)
- Tiene GitHub Enterprise Cloud con Copilot
- Multiproducto, multiequipo, escala enterprise

---

## Idioma

**Español** para el artículo y documentación primaria.
**Inglés** como versión paralela disponible en `PROD/*-EN.md`.

El español es la audiencia primaria (equipo Latam). El inglés permite compartir con equipos globales.

---

## Qué se considera éxito

1. El equipo lee el artículo entero (no lo abandona en el medio)
2. Después de leerlo, cambian al menos UNA cosa en cómo trabajan con AI
3. La documentación técnica se vuelve referencia (gente la consulta más de una vez)
4. Los tech leads la comparten con sus equipos
5. Aparece en conversaciones de planning como "según el artículo de tokens..."

**Qué NO es éxito:**
- Que sea "completo" pero nadie lo lea
- Que parezca un white paper de consultora
- Que el equipo lo lea y piense "sí, ya sabía todo esto"

---

## Tono

**El tono correcto:** un colega del equipo que descubrió algo importante mientras debuggeaba algo aburrido, y lo cuenta porque te puede ahorrar plata y dolor.

**Lo que SÍ va:**
- ✅ "El 62% de lo que pagás en AI no es por el trabajo que hace el modelo. Es por mandar de vuelta el mismo contexto."
- ✅ "La sesión que empieza en 5.000 tokens llega a 200.000 en el turno 50. Nadie lo ve hasta que llega la factura."
- ✅ "Hay un setting de VSCode que activás en 10 segundos y ahorra ~15% del consumo de terminal."

**Lo que NO va:**
- ❌ "En este artículo exploraremos las mejores prácticas para la optimización..."
- ❌ "Es importante destacar que los output tokens tienen un costo significativamente mayor..."
- ❌ "En el contexto actual de la inteligencia artificial empresarial..."

---

## Formato del artículo

- Storytelling técnico, no manual corporativo
- 2.500–3.500 palabras
- Headers cortos y accionables
- Datos duros con fuente al lado o linkeada
- Bloques de código cuando aplica
- Cierre con "qué hacer esta semana"
- Links a la documentación para profundizar

---

## Formato de la documentación técnica

- Exhaustiva — no resumir
- Por capas de optimización (Capa 0 a Capa 4, ver `04-glossary.md`)
- Con ejemplos de código donde aplica
- Con tablas de pricing y multiplicadores
- Con "cuándo aplicar" y "cuándo no" para cada técnica
