# Guía Técnica Final: Optimización de Tokens en IA
## De configuraciones básicas a arquitectura enterprise, con narrativa para todos los niveles técnicos

**Audiencia:** Desarrolladores, tech leads, architects, líderes técnicos, decisores de presupuesto, y también personas con menos background técnico que quieran entender qué implementar y por qué.
**Prerequisito:** Familiaridad básica con LLMs y APIs de IA. No hace falta ser experto en cada herramienta. Cada sección incluye contexto suficiente para seguir.

---

## Cómo leer esta guía

Este documento está pensado como **biblia de optimización de tokens y reducción de costos** en consumo de LLMs. No es una lectura lineal obligatoria: cada sección resuelve un problema concreto y se puede consultar de forma independiente.

A diferencia de una documentación técnica clásica que asume contexto previo, este doc envuelve cada snippet con la explicación de **dónde va, por qué se hace así, cuándo conviene y cómo verificás que está funcionando**. La idea es que tanto un dev senior que solo quiere copiar el snippet, como un tech lead o decisor que necesita entender qué le va a pedir al equipo, encuentren lo que necesitan sin sentirse abrumados o sin contexto.

La estructura sigue una progresión de impacto creciente:

- **Niveles 0 y 1** son ajustes que cualquier developer puede aplicar hoy mismo en su IDE o en código de aplicación. Ahorros típicos del 20 a 40%.
- **Niveles 2 y 3** son decisiones de arquitectura: cómo asignás modelos por tarea y cómo ruteás automáticamente. Ahorros del 50 a 80%.
- **Niveles 4 a 6** son capa enterprise: gobernanza, observabilidad cross-equipo, proveedores alternativos. Aquí se construye el control para que el costo no se desborde a medida que escala el uso.

Cada bloque de código está etiquetado con la **capa** en la que vive (de la 1 a la 4) y **quién lo toca** dentro del equipo, para que no haya ambigüedad sobre dónde se implementa.

### Índice

1. [Fundamentos](#1-fundamentos)
2. [Dónde vive cada palanca de optimización](#2-donde-vive)
3. [Vendor directo vs hyperscaler: la decisión de procurement](#3-procurement)
4. [Nivel 0: configuraciones inmediatas](#4-nivel-0)
5. [Nivel 1: arquitectura de contexto](#5-nivel-1)
6. [Nivel 2: selección de modelos](#6-nivel-2)
7. [Nivel 3: model routing automático](#7-nivel-3)
8. [Nivel 4: infraestructura de costo y gobernanza](#8-nivel-4)
9. [Nivel 5: observabilidad con OpenTelemetry y gobernanza personal](#9-otel)
10. [Nivel 6: proveedores alternativos de menor costo](#10-alternativas)
11. [Referencia de métricas verificadas](#11-metricas)

---

## Convención visual para los scripts

Cada bloque de código de aplicación lleva una **micro-cabecera** que cumple dos funciones: ubicar el snippet en su capa física (Capa 1 a Capa 4, ver Sección 2) e identificar qué rol del equipo es el que lo toca. Esto evita el problema clásico de leer un fragmento de código sin saber si va en el IDE, en la app, en el gateway o en la consola de administración.

El formato es:

> 🔧 **Capa X** (descripción del lugar físico) · **Tecnología** · **Quién lo toca**

Por ejemplo:
> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a `api.anthropic.com`

Los snippets que **no** tienen esta cabecera son **settings de IDE** (VSCode, Copilot, Claude Code, Cursor). Esos los puede tocar cualquier desarrollador individual sobre su propia instalación, no requieren construir apps custom ni pasar por plataforma.

---

## 1. Fundamentos

Antes de optimizar conviene entender exactamente qué se está pagando. La factura de un LLM no es una caja negra: cada componente de un request tiene un precio distinto, y entender la composición del costo es el primer paso para reducirlo.

### 1.1 Anatomía de un request

Toda llamada a un LLM se factura en tres bloques principales que conviene distinguir desde el inicio:

- **Input tokens.** Todo lo que el modelo recibe para procesar: system prompt (instrucciones generales del asistente), historial de la conversación, definiciones de herramientas (tools), documentos adjuntos y el mensaje del usuario. Es la parte "barata" del precio, pero también la que crece sin control si no se gobierna.
- **Output tokens.** Lo que el modelo genera: la respuesta visible al usuario, el razonamiento interno (extended thinking) y las tool calls. **Cuestan típicamente 5x más que los input tokens.** Esto significa que limitar la longitud de la respuesta tiene un impacto desproporcionado en la factura.
- **Cache reads.** Cuando reutilizás un prompt o un bloque grande de contexto vía prompt caching, las lecturas de cache se cobran al **10% del precio normal de input**. Es el descuento más grande que ofrece la plataforma, y la palanca individual con mayor impacto en aplicaciones que reutilizan contexto (RAG, agentes, asistentes con base de conocimiento).

La asimetría 1x input, 5x output, 0.1x cache es la regla mental más útil para diseñar prompts: vale la pena pagar más en input estructurado y cacheable, a cambio de minimizar output libre.

### 1.2 Precios de referencia 2026

Esta tabla resume los precios públicos de los principales modelos a mayo 2026, expresados en USD por millón de tokens (MTok). Es la referencia de partida para cualquier análisis de costo, pero **el precio nominal no es el precio efectivo**: ese se calcula incluyendo caching, batch, y el overhead del proveedor (que se explican en las secciones siguientes).

| Modelo | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|-------------|---------------|--------------------|---------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| GPT-5.4 | $2.50 | $15.00 | $0.25 | n/a |
| GPT-5.5 (flagship) | $5.00 | $30.00 | $0.50 | n/a |
| GPT-5 Nano | $0.05 | $0.40 | n/a | n/a |
| DeepSeek V4-Flash (directo) | $0.14 | $0.28 | n/a | n/a |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | n/a | n/a |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | n/a | n/a |

Fuentes: [Anthropic oficial mayo 2026](https://platform.claude.com/docs/en/about-claude/pricing), [OpenAI oficial](https://openai.com/api/pricing/), [Finout 2026](https://www.finout.io/blog/openai-pricing-in-2026), [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing), [ToolHalla](https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026)

Lo primero que salta a la vista es que **hay una diferencia de hasta 100x entre el modelo más barato (GPT-5 Nano a $0.05) y el más caro (GPT-5.5 a $5)** solo en input. Esa asimetría es la base de toda la estrategia de "asignación de modelo por tarea" que se desarrolla en el Nivel 2: tareas simples (clasificación, extracción, navegación de archivos) no necesitan el modelo flagship, y el ahorro es inmediato.

> **Nota Opus 4.7:** El nuevo tokenizer de Opus 4.7 genera **hasta 35% más tokens para el mismo input** comparado con Opus 4.6. El precio por token no cambió, pero el costo efectivo por request puede ser hasta 35% mayor. Si tu equipo está evaluando migrar de Opus 4.6 a 4.7, conviene benchmarkear con tus propios prompts antes de tomar la decisión: el incremento puede compensar la mejora en capacidad, o no.
> Fuente: [Finout, Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 El contexto acumulado: el problema invisible

Este es el concepto que más se subestima en aplicaciones agénticas y de chat. En una conversación o sesión larga, **el contexto no se mantiene constante: se acumula**. Cada turno se le re-envía al modelo todo lo anterior para que pueda razonar con la conversación completa, y eso multiplica los tokens facturados de forma silenciosa.

La progresión típica en una sesión agéntica es:
- Turno 1: ~5K tokens (system prompt + mensaje inicial)
- Turno 10: ~15K tokens (system prompt + 10 turnos previos)
- Turno 50: **~200K tokens por llamada** (system prompt + 50 turnos + outputs de tools + documentos cargados)

A turno 50 estás pagando 40 veces más por la misma "conversación" que al turno 1. Y como cada llamada vuelve a procesar todo desde cero, la factura crece de forma cuadrática respecto al largo de la sesión.

El dato cuantificado es contundente: según [LeanOps (estudio sobre 30 equipos en 2026)](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/), **el 62% de la factura de AI corresponde a contexto re-enviado**, no a información nueva. Es decir: en promedio, casi 2/3 de lo que gastás en tokens es repagar contenido que ya pagaste antes.

Esta es la razón por la que prompt caching (Sección 5.1) y compactación de historial (Sección 5.2) son las dos optimizaciones de mayor impacto en cualquier app conversacional o agéntica.

### 1.4 Extended thinking: el costo invisible

Los modelos con "extended thinking" (Claude Opus, Sonnet, o5-style en OpenAI) tienen una capa de razonamiento interno que se factura como output, aunque el usuario no la vea. Este es probablemente el error más común y más caro en producción: equipos que dejan thinking habilitado en modo default, pensando que ocultarlo del display lo desactiva.

**No: ocultar el razonamiento no lo desactiva ni lo abarata.** Los tokens de thinking se siguen generando y facturando, simplemente no se muestran.

**Dónde va este snippet:** en el código de tu aplicación, en el archivo donde inicializás el cliente de Anthropic y construís los requests al modelo. Típicamente es un archivo del tipo `services/llm_client.py` o `lib/anthropic.ts`, no algo que se toca desde un IDE.

> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a `api.anthropic.com/v1/messages`

```python
# INCORRECTO: display: omitted NO reduce el costo
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "enabled", "budget_tokens": 5000, "display": "omitted"}
    # Los 5000 tokens se siguen facturando como output
)

# CORRECTO: limitar el budget con modo adaptativo
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```

**Qué hace cada parámetro:**
- `type: "enabled"` activa thinking con un budget fijo de tokens que se reservan siempre, incluso si la tarea no los necesita.
- `display: "omitted"` solo controla la visibilidad del razonamiento en la respuesta. **No afecta el costo.**
- `type: "adaptive"` deja que el modelo decida cuánto razonar según la complejidad de la tarea. Es la opción correcta para producción.
- `effort: "low"` (también `medium`, `high`) le indica al modelo qué tan generoso ser con el budget de razonamiento.

**Cómo verificás:** en la respuesta de la API, el objeto `usage` te trae `output_tokens` desglosado. Si comparás dos llamadas equivalentes con y sin thinking, la diferencia en `output_tokens` es exactamente lo que pagaste de más por habilitar thinking.

El impacto concreto: una respuesta de 500 tokens output sumada a 2.000 tokens de thinking termina costando **5 veces más** que la misma respuesta sin thinking activado. Por eso la recomendación operativa es: usar `adaptive` con `effort: low` por default y subir el effort solo en las tareas que demuestren beneficio real con razonamiento extendido (arquitectura, debugging complejo, razonamiento multi-paso). Para clasificación, extracción o respuestas factuales, thinking es desperdicio puro.

Fuente: [PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/)

---

## 2. Dónde vive cada palanca de optimización <a id="2-donde-vive"></a>

Esta es probablemente la sección más importante del documento, porque resuelve un problema que casi todos los equipos sufren al empezar: **buscar la palanca en el lugar equivocado**. Hay un patrón muy frecuente: un dev intenta optimizar desde un settings del IDE algo que solo se puede configurar en el código de la aplicación, o un líder pide "activar caching para todos" cuando los usuarios consumen IA a través de un producto cerrado donde no existe esa opción.

Para evitar esa confusión, conviene mapear con precisión los lugares físicos donde se toca optimización. Son cuatro, y cada uno tiene una audiencia distinta, sus propios parámetros y su propio techo de impacto.

### 2.1 Las cuatro capas de control

| Capa | Lugar físico | Quién lo toca | Palancas que viven aquí | Techo de ahorro |
|------|---------------|----------------|---------------------------|-------------------|
| **Capa 1** | Código de aplicación (Python/JS/etc) que llama directo a la API | Developers que construyen apps internas con LLM | Prompt caching, thinking budgets, max_tokens, modelo por tarea, batch API, output budgets, headers de beta | **Hasta 95% combinado** |
| **Capa 2** | Settings de IDE (VSCode, Copilot, Claude Code, Cursor) | Cualquier dev individual o admin vía MDM | Auto Mode, compressOutput, tool search, debug panels, OTel local | **Hasta 30% combinado** |
| **Capa 3** | Consola de billing / admin de plataforma | Lead técnico, admin del tenant | Modelos permitidos/bloqueados, presupuestos por workspace, alertas, audit logs | **Indirecto: gobernanza** |
| **Capa 4** | Gateway / routing layer (LiteLLM, Portkey, OpenRouter, RouteLLM) | Plataforma / infra | Routing automático entre modelos, fallbacks, retries, presupuestos por equipo, guardrails | **20 a 80% según [IDC](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing)** |

Cómo leer esta tabla: si tu equipo solo tiene developers usando productos cerrados (Copilot, Cursor), la mayor parte de tu ahorro va a venir de Capa 2 (configurar bien el IDE) y de gobernanza de licencias en Capa 3. Si tu equipo construye apps que llaman directo a la API, las palancas más fuertes están en Capa 1 (caching, batch, asignación de modelo), y eventualmente Capa 4 cuando hay varios servicios y necesitás centralizar gobierno.

### 2.2 El espectro: productos cerrados, herramientas con configuración, API directa

Antes de discutir las cuatro capas hay una distinción anterior, conceptual, que define a cuáles capas tenés acceso. Hace año y medio, el mundo de la IA se dividía claramente en dos: las herramientas (caja negra) y las APIs (transparentes). Hoy esa división binaria quedó obsoleta. Lo que tenemos es un **espectro de tres niveles**, y mucha gente sigue razonando con el modelo viejo, lo que produce expectativas equivocadas.

**Productos cerrados.** Productos consumer sin instrumentación pensada para el usuario. Ejemplos: ChatGPT versión consumer, Gemini versión consumer, Claude.ai en planes Free/Pro. Lo único que ves es el chat, la respuesta y alguna cuota mensual del plan. No tenés model picker, no hay telemetría exportable, no podés modificar parámetros. Techo realista de ahorro: cambiar de plan o cancelar.

**Herramientas con configuración.** Productos enterprise o IDEs con visibilidad, model picker, telemetría OpenTelemetry y settings ricos. Ejemplos: Copilot en VSCode/Visual Studio, Claude Code, Claude Cowork en planes Team/Enterprise. Tenés acceso a Capa 2 (settings del IDE) y a una parte de Capa 3 (consola del producto). El caching lo hace la herramienta por vos. Cursor está en este grupo pero todavía no expone OTel nativo, tiene Admin Dashboard con API propia y exports CSV en Enterprise, y hay hooks de comunidad (cursor-otel-hook, CursorLens) que cierran el gap. Techo realista de ahorro: ~30% sobre consumo no optimizado.

**API directa.** Vos escribís el código que llama al modelo. Ejemplos: Anthropic API directa, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Tenés acceso a las cuatro capas. Las palancas grandes (caching 90%, batch 50%, routing 20 a 80%) viven en Capa 1 y Capa 4. Techo realista de ahorro: más del 90%.

**Una BU puede estar en los tres niveles simultáneamente.** Herramientas con configuración para que los devs usen Copilot, API directa para una feature de producto que llama LLM por debajo, y quizás algún equipo usando ChatGPT consumer informalmente. Eso no es problema, es la realidad. Lo que importa es saber **en qué nivel está cada caso de uso** y no asumir que todo lo que no es API directa es un producto cerrado.

La regla simple para diseñar nuevas implementaciones:
- **Herramientas con configuración para uso humano** (un dev escribiendo código, alguien explorando ideas).
- **API directa para uso programático** (una feature de producto que llama un modelo sin humano del otro lado).

Los productos cerrados son problema sobre todo cuando se usan para casos que ameritan otra cosa: si un equipo está usando ChatGPT consumer para tareas que requieren observabilidad, gobernanza o repetibilidad, ahí hay algo para revisar.

**Este documento asume que estás construyendo, o yendo hacia ahí.** Las capas que siguen dan por sentado que tu equipo escribe código de aplicación contra una API o está evaluando hacerlo. Si hoy estás 100% en productos cerrados y querés bajar costos significativos, el paso anterior es subir al menos a una herramienta con configuración o construir un wrapper interno con API key. Recién a partir de ahí abren las palancas que valen la pena.

### 2.3 Cómo se conecta cada script con su capa

Para que sea fácil ubicar visualmente cada bloque de código en su lugar, esta tabla mapea las tecnologías más comunes a su capa. Es el "mapa rápido" del documento: si más adelante ves un snippet con `client.messages.create(...)` y dudás dónde va, esta es la referencia.

| Tipo de snippet | Capa | Quién lo toca |
|------------------|------|----------------|
| `client.messages.create(...)`, llamadas a SDK | Capa 1 | Developers de app |
| `client.messages.batches.create(...)` | Capa 1 | Developers de app |
| `cache_control: {type: "ephemeral"}` | Capa 1 | Developers de app |
| `thinking={"type": ...}` | Capa 1 | Developers de app |
| `OUTPUT_BUDGETS = {...}` | Capa 1 | Developers de app |
| `MODEL_ASSIGNMENT = {...}` | Capa 1 (en wrapper interno) o Capa 4 (en gateway) | Plataforma o developers |
| `chat.tools.compressOutput.enabled` en `settings.json` | Capa 2 | Cualquier dev |
| `github.copilot.chat.agentDebugLog.enabled` | Capa 2 | Cualquier dev |
| `export CLAUDE_CODE_ENABLE_TELEMETRY=1` | Capa 2 (env vars locales) o Capa 3 (MDM enterprise) | Dev individual o admin |
| `Controller(routers=["mf"], ...)` (RouteLLM) | Capa 4 | Plataforma |
| `Router(model_list=[...], budget_manager=...)` (LiteLLM) | Capa 4 | Plataforma |
| `Portkey(api_key=..., virtual_key=...)` | Capa 4 | Plataforma |

### 2.4 La forma de gobernar Capa 1 a escala

Hay una observación operativa que conviene anticipar: si tu equipo tiene 10 microservicios distintos llamando directo al SDK de Anthropic, cada uno decide por su cuenta thinking, caching, max_tokens, modelo. Resultado típico en producción: 10 niveles distintos de "optimización", drift entre servicios y una factura impredecible que nadie puede explicar bien.

La causa raíz no es técnica, es organizacional: cuando la decisión de configuración vive distribuida en N repositorios, las buenas prácticas no se replican solas. Cada nuevo servicio empieza con defaults distintos según el desarrollador que lo arrancó.

Hay dos formas de gobernarlo, y conviene elegirla **antes** de que el problema aparezca:

**Opción A: wrapper interno compartido.** Una librería interna que envuelve el SDK del proveedor y le aplica defaults sanos hardcodeados: `adaptive thinking effort=low`, modelo Haiku salvo override explícito, prompt caching activo por defecto, max_tokens según el tipo de tarea. Todos los servicios consumen el wrapper en lugar del SDK directo. La ventaja es que es una sola decisión, replicada en todos lados, y cada cambio futuro se propaga con bumps de versión. Es el camino más limpio para equipos chicos o medianos.

**Opción B: gateway compartido (LiteLLM / Portkey).** En vez de envolver el SDK en código, las llamadas pasan por un gateway HTTP que aplica policy: "si nadie especificó thinking, forzar low", "si nadie especificó modelo, usar Haiku", "si el costo de un request supera $X, alertar y/o bloquear". El código de los servicios no cambia, el gateway intercepta y normaliza desde la red. Ventaja: funciona aunque cada servicio esté en un lenguaje distinto y se gobierne presupuestos por equipo de forma centralizada.

Regla práctica: **para equipos hasta ~5 servicios, wrapper alcanza**. A partir de ahí, gateway empieza a justificarse porque permite control multi-equipo, multi-lenguaje, fallbacks automáticos y observabilidad unificada sin tocar el código de las aplicaciones.

---

## 3. Vendor directo vs hyperscaler: la decisión de procurement <a id="3-procurement"></a>

Esta sección documenta los datos verificados que conviene tener encima de la mesa cuando se evalúa o se negocia un acuerdo de consumo de LLM con un proveedor. Es una de las áreas donde más se malinterpreta el costo real: el precio nominal por token es solo una parte del precio efectivo, y la diferencia entre los dos puede llegar al 40%.

### 3.1 La estructura del precio

Cuando se consume un LLM con API key, el costo total se compone de **tres elementos**, y conviene mantenerlos separados al comparar proveedores:

1. **Precio nominal por token.** Lo que figura en la pricing page del modelo. Es el número que todos miran primero y el que aparece en las propuestas comerciales.
2. **Modificadores de request.** Caching, batch, long context, fast mode, data residency. Son factores multiplicativos (algunos hacen bajar el precio, otros lo suben) y dependen de cómo armás cada llamada.
3. **Overhead del proveedor.** Support plans, data transfer (egress), hosting de fine-tuned models, monitoring infra, audit logging. Son **costos fijos o por uso adicionales** que el pricing nominal no captura.

El precio nominal es donde la mayoría de las empresas comparan, y donde aparentemente "el hyperscaler con descuento gana". El overhead es donde la cuenta se da vuelta y muchas comparaciones se demuestran equivocadas después del primer mes de producción.

### 3.2 Pricing parity nominal entre proveedores (mayo 2026)

Un dato que conviene internalizar antes de cualquier negociación: Anthropic mantiene **pricing parity per-token** entre su API directa y los hyperscalers principales. Es decir, el modelo cuesta exactamente lo mismo por token, independientemente de si lo accedés directamente o a través de un cloud provider:

| Modelo Claude | Anthropic directo | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|----------------|---------------------|----------------------|--------------------|---------------------|
| Sonnet 4.6 | $3 / $15 | $3 / $15 | $3 / $15 | $3 / $15 |
| Opus 4.7 | $5 / $25 | $5 / $25 | $5 / $25 | $5 / $25 |
| Haiku 4.5 | $1 / $5 | $1 / $5 | $1 / $5 | $1 / $5 |

Fuente: [Anthropic Pricing Docs oficial](https://platform.claude.com/docs/en/about-claude/pricing), [Anthropic Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)

Esto rompe un mito frecuente en negociaciones enterprise: "vamos a hyperscaler porque nos da mejor precio en Claude". **No es cierto a nivel nominal.** El hyperscaler te puede dar otras cosas (procurement unificado, compliance, créditos), pero no un mejor precio por token.

**Multiplicadores explícitos publicados por Anthropic:**
- Bedrock regional endpoints: +10% sobre global
- Vertex regional/multi-region: +10% sobre global
- Claude API directa con `inference_geo: "us"`: +10% (1.1x multiplier)

Estos son los únicos modificadores oficiales y públicos. Cualquier otro markup que aparezca en la factura corresponde a overhead del proveedor.

Lo mismo pasa con OpenAI vs Azure OpenAI: pricing per-token idéntico, overhead distinto.

### 3.3 TCO efectivo: el overhead real

Acá es donde la conversación se vuelve concreta. Diferentes fuentes independientes han medido el costo total efectivo de consumir LLMs a través de hyperscalers, y los datos son consistentes.

**Azure OpenAI vs OpenAI directo: 15 a 40% overhead efectivo, promedio +22%**

Tres fuentes independientes de 2026 coinciden en el rango:

| Fuente | Rango documentado | Cita textual |
|--------|---------------------|---------------|
| [TokenMix mayo 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15 a 40%, avg 22% | "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [Inference.net enero 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15 a 40% | "Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [CloudZero mayo 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20 a 40% | "Production deployments typically add 20-40% above listed token rates." |

Los componentes del overhead de Azure son los siguientes, y conviene tenerlos identificados para poder pedir cuentas en una negociación:

- **Support plans**: $100 a $1.000+/mes. En producción enterprise, Standard support es de facto obligatorio, así que ese costo entra siempre.
- **Data egress**: los primeros 100GB outbound son gratuitos, después se paga $0.087/GB. En aplicaciones con tráfico considerable, esto se acumula rápido.
- **Fine-tuned model hosting**: $1.70 a $3/hora **aun sin uso**, es decir entre $1.224 y $2.160/mes por modelo, simplemente por tenerlo desplegado. Si tenés varios fine-tuned models, multiplicar.
- **VNet integration, Private Link, content filtering**: $200 a $2.000/mes según configuración. Son costos que muchas empresas necesitan por compliance, pero que la pricing page del modelo no muestra.
- **Log Analytics y monitoring infra**: variable, depende del volumen y la retención.

**AWS Bedrock para Claude: 20 a 35% overhead efectivo**

Pricing nominal idéntico a Anthropic directo, pero según [TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing):
- Regional endpoints: +10% (cifra oficial publicada por Anthropic).
- Cross-region inference: +10% adicional cuando se rutea entre regiones.
- Bedrock cuenta data transfer (egress). La API directa no.
- Bedrock factura **todos** los errores HTTP 500. La API directa tiene un buffer de tolerancia (forgiveness buffer) del 3% en errores del servidor que no se cobran.
- CloudWatch y CloudTrail logging mandatorio, con su costo asociado.

Cita textual: *"TokenMix.ai cost tracking shows enterprises running Claude on Bedrock pay an average of 20-35% more than those using Anthropic's direct API, and most do not realize it."*

**Microsoft Foundry para Claude: pricing idéntico pero trampa documentada**

Este es probablemente el caso más sutil y más caro, porque a primera vista parece el mismo precio. Claude se factura en Foundry como **Marketplace de terceros**, no como recurso nativo de Azure. La implicación crítica está documentada por [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) y [AZ365.ai marzo 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> "Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."

En castellano: **el consumo de Claude en Foundry no descuenta de tus Azure credits ni de tu MACC (Microsoft Azure Consumption Commitment).** Se va directo a la tarjeta de crédito asociada a la cuenta. Si tu plan de procurement asumía que el MACC absorbía el gasto en Claude, te vas a llevar una sorpresa.

**Casos reales documentados** ([The Register / AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/)):
- Founder japonés (Leach): ¥237.081 (~$1.600 USD) cargados directo a tarjeta, con credits no aplicados.
- Founder alemán (Bogdan Sevriukov): €999.60 cargados directo.
- Otro founder japonés: ¥2.000.000 (~$13.000 USD) en un mes.
- Caso adicional reportado por The Register: aproximadamente $3.000 USD.

**Acción concreta:** Si tu empresa está negociando un acuerdo Microsoft asumiendo que el MACC absorbe el consumo de Claude vía Foundry, **pedir aclaración escrita explícita** sobre qué tipos de credits aplican a Claude y bajo qué condiciones. Si la respuesta es ambigua, asumir que no aplican.

### 3.4 Descuentos enterprise reales

Acá los datos verificados sobre qué descuentos efectivamente se logran en negociaciones reales. Sirven como benchmark para no aceptar la primera oferta de un vendor:

| Tipo de descuento | Rango documentado | Fuente |
|--------------------|--------------------|---------|
| Azure EA solo | 15 a 25% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Azure EA + MACC | 23 a 28% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Anthropic Enterprise / volume | Disponible, custom | [CloudZero](https://www.cloudzero.com/blog/claude-pricing/) |
| OpenAI Enterprise tier | Custom pricing | [Finout, OpenAI](https://www.finout.io/blog/openai-pricing-in-2026) |
| AWS Bedrock private offers | Disponible | Anthropic docs |

### 3.5 La cuenta neta: descuento vs overhead

Esta es la tabla que conviene tener a mano cuando alguien pregunta "¿pero no nos conviene Azure porque tenemos descuento?". Supongamos un workload base = 100 unidades de costo cuando se consume desde vendor directo:

| Escenario | Costo directo | Costo Azure | Costo Bedrock | Neto vs directo |
|-----------|----------------|---------------|----------------|-------------------|
| GPT-5.4, sin compliance, sin descuento | 100 | 115 a 140 | n/a | **+15% a +40%** |
| GPT-5.4, EA solo (-20%) sobre 115 a 140 | 100 | 92 a 112 | n/a | **-8% a +12%** |
| GPT-5.4, EA + MACC (-25%) sobre 115 a 140 | 100 | 86 a 105 | n/a | **-14% a +5%** |
| Claude Sonnet, sin compliance | 100 | n/a (Foundry sin desc.) | 120 a 135 | **+20% a +35%** |
| DeepSeek vs Azure (sin compliance) | 100 | 120 a 135 | n/a | **+20% a +35%** |
| DeepSeek vs Azure (con GDPR EU) | inviable | 120 a 135 | n/a | **prima de compliance** |

**Conclusión operativa:** el descuento EA/MACC típicamente compensa pero **no supera** el overhead. La factura final, después de descuentos, queda en una franja entre -14% y +12% comparado con vendor directo. Es decir: cuando hay descuento bueno, el hyperscaler puede ser ligeramente más barato; cuando el descuento es menor o el workload genera mucho overhead, sale más caro. La decisión real **se gana o se pierde por compliance y procurement, no por descuento de precio**.

### 3.6 Cuándo el hyperscaler vale la pena (matriz de decisión)

A partir de los datos anteriores, esta matriz resume cuándo conviene cada camino. No hay una respuesta universal: depende del criterio dominante en tu organización.

| Criterio dominante | Camino recomendado |
|---------------------|----------------------|
| Modelo del propio vendor enterprise-maduro (Anthropic, OpenAI), sin mandato Azure-only | **Vendor directo**, features primero, mejor precio neto, soporte de fábrica |
| Modelo geopolíticamente sensible (DeepSeek, Qwen, modelos chinos) | **Hyperscaler (Azure preferido)**, la prima compra compliance |
| Mandato corporativo "todo en Azure/AWS" | **Hyperscaler**, asumir overhead 15 a 40% como costo de gobernanza |
| Necesidad de MACC burn / facturación unificada Microsoft | **Hyperscaler**, decisión contable, **excluir Claude en Foundry** |
| Modelo open-source self-hosted (Llama, Mistral) | **Hyperscaler con managed endpoint o self-host**, Bedrock/Vertex o GPUs propias |
| Equipo en exploración, prototipos, POCs | **Vendor directo**, onboarding rápido, menos burocracia |
| Compliance GDPR / data residency obligatorio | **Hyperscaler** o **Mistral** (única opción major europea directa) |

### 3.7 Input para negociación

Cuando vayas a sentarte con cualquier vendor (Microsoft, AWS, Google, OpenAI, Anthropic), estos son los datos verificados que conviene tener a mano. Sirven tanto para pedir descuentos como para detectar promesas exageradas en una propuesta comercial:

| Dato | Valor | Fuente | Uso en negociación |
|------|-------|--------|---------------------|
| Azure overhead vs OpenAI directo | 15 a 40% (avg 22%) | TokenMix, Inference.net, CloudZero | "Necesitamos descuento que cubra al menos 25% para break-even contra directo" |
| Bedrock overhead vs Anthropic directo | 20 a 35% efectivo | TokenMix Bedrock | "Foundry o directo nos da el mismo precio sin ese markup" |
| Foundry Claude credits exclusion | Documentado | Microsoft Q&A, AZ365.ai | "Aclaración escrita explícita sobre qué credits aplican a Claude" |
| Anthropic Enterprise volume discount | Disponible | CloudZero | "Solicitar términos comparables a hyperscaler con MACC" |
| OpenAI Enterprise custom pricing | Disponible | Finout | "Solicitar mismo régimen que Azure pero sin overhead" |
| Pricing parity Claude en hyperscalers | Confirmado | Anthropic docs | "El hyperscaler no nos da mejor precio nominal, solo procurement" |

---

## 4. Nivel 0: configuraciones inmediatas <a id="4-nivel-0"></a>

Este nivel reúne todas las optimizaciones que **cualquier developer puede activar hoy mismo sobre su propia instalación de VSCode/Copilot**, sin necesidad de pedir permisos, sin tocar código de aplicación y sin construir infraestructura nueva. Son los "quick wins" del documento: poco esfuerzo, impacto inmediato, baseline obligatorio antes de pasar a niveles más complejos.

> 🔧 **Capa 2** (settings de IDE) · Todos los snippets de esta sección son JSON de VSCode `settings.json` · Cualquier dev individual los puede activar

**Dónde se editan estos settings.** Todo lo que sigue se mete en el `settings.json` de VSCode. La forma más rápida de abrirlo:

1. Abrir Command Palette: `Ctrl+Shift+P` (Linux/Windows) o `Cmd+Shift+P` (Mac).
2. Escribir "Preferences: Open User Settings (JSON)".
3. Se abre un archivo `settings.json` donde podés pegar las líneas de cada subsección.

Si nunca tocaste el `settings.json`, vas a ver un archivo con llaves `{}`. Las opciones se separan con coma. Si no estás seguro, hay también la opción "Preferences: Open Settings" (sin el "(JSON)"), que es una UI buscable donde podés escribir el nombre del setting y activarlo con un toggle.

### 4.1 Auto Mode en GitHub Copilot

Esta es la optimización más rápida que existe en Copilot: cambiar el selector de modelo a **"Auto"** en VSCode. En modo Auto, Copilot elige automáticamente el modelo apropiado para cada request, y la plataforma aplica un **10% de descuento automático** sobre el multiplicador. Sin perder calidad para tareas estándar, dado que Auto Mode rutea las tareas simples a modelos más baratos y reserva el flagship solo para las que lo justifican.

**Cómo se activa:** en el panel de Copilot Chat, arriba del input box hay un selector con el nombre del modelo actual (típicamente "GPT-4" o "Claude Sonnet"). Click ahí, seleccionar "Auto". Listo.

**Cómo verificás que funciona:** después de activar Auto, en la barra inferior de Copilot Chat vas a ver indicado qué modelo eligió para cada respuesta. Si para preguntas simples ves modelos baratos (Haiku, GPT-5 Nano) y para preguntas complejas ves Sonnet u Opus, el routing está haciendo su trabajo.

Fuente: [GitHub Changelog, abril 2026](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/)

### 4.2 chat.tools.compressOutput.enabled

VSCode 1.120 introdujo una opción de post-procesamiento del output de terminal que reduce significativamente el tamaño de los outputs que el agente devuelve. Lo que hace concretamente: colapsa diffs sin cambios, descarta lockfiles voluminosos, reduce `ls -l` a solo nombres de archivo, elimina barras de progreso y outputs ANSI inútiles.

**Cómo se activa:** en `settings.json`, agregar:

```json
{ "chat.tools.compressOutput.enabled": true }
```

**Qué pasa cuando lo activás.** Es transparente para el flujo del agente, la lógica no cambia, simplemente se le ahorra tener que procesar (y pagar) tokens basura. El agente sigue ejecutando los mismos comandos, pero antes de devolverte la salida, VSCode la limpia.

**Cuándo conviene.** Siempre. No tiene downside conocido. Activarlo es lo primero que cualquier dev debería hacer al instalar Copilot.

**Cómo verificás.** Abrí el Agent Debug Log (Sección 4.4). Cuando un comando ejecuta algo voluminoso (un `pnpm install` por ejemplo), comparalo con y sin compressOutput activado: vas a ver outputs mucho más cortos enviados al modelo, y eso se traduce directo en menos tokens facturados.

### 4.3 Tool search: MCPs diferidos

Cuando tenés varios MCPs configurados, cada llamada incluye las definiciones de todas las herramientas disponibles, aunque el modelo no las vaya a usar. Tool search rompe ese patrón: en lugar de mandar todas las definiciones de entrada, el modelo recibe solo un índice y busca las herramientas a demanda cuando las necesita.

Es default desde Anthropic Sonnet 4.5+. Para GPT en Copilot hay que activarlo manualmente.

**Cómo se activa:** en `settings.json`:

```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```

**Qué pasa.** Antes de tool search, si tenías 10 MCPs activos, cada request mandaba las definiciones de los 10 (típicamente 500 tokens por MCP = 5.000 tokens de overhead por request). Con tool search, el modelo recibe solo un índice ligero y solo carga las definiciones de las herramientas que va a usar.

**Cuándo conviene.** Cuando tenés 2 o más MCPs activos. Si tenés uno solo, el ahorro es marginal. Si tenés 5+, el ahorro es importante.

Impacto medido: hasta **20% de ahorro en tokens por request**, dependiendo de cuántos MCPs tengas activos. Fuente: [Visual Studio Magazine, 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

### 4.4 Agent Debug Log Panel: el trace completo dentro de VSCode

Esta es probablemente la herramienta más subutilizada del stack: la forma más directa de ver qué está pasando en una sesión de agente sin configurar ningún backend externo. Muestra tokens consumidos, tool calls ejecutadas, model turns, subagentes, errores y un **flow chart visual del agente**, todo renderizado directamente en el editor.

¿Por qué importa para optimización de costos? Porque hasta que no medís, no podés optimizar. La mayoría de los devs no tienen idea de cuántos tokens consume cada sesión, ni cuántas tool calls se disparan en silencio, ni qué subagentes se generan en cascada. Este panel hace todo eso visible en tiempo real.

**Cómo abrir el panel:**
- Menú overflow `(...)` en el panel de Copilot Chat, **"Show Agent Debug Logs"**
- O Command Palette (`Cmd/Ctrl+Shift+P`), **"Developer: Open Agent Debug Logs"**

**Requiere habilitar el setting:**

```json
{ "github.copilot.chat.agentDebugLog.enabled": true }
```

Sin este setting, el comando del menú no muestra nada.

**Qué muestra por sesión:**
- **Overview:** model turns totales, tool calls ejecutadas, total tokens consumidos, errores
- **View Logs:** lista cronológica de todos los eventos con timestamps y detalles, filtrable por tipo
- **Agent Flow Chart:** grafo visual del flujo de ejecución. Muestra cómo se relacionan model turns, tool calls y subagentes anidados. Es especialmente útil cuando un agente se "va por las ramas", el grafo deja ver claramente dónde se desvió.

Fuente: [VSCode Docs, Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

**Persistir sesiones pasadas en disco:**

```json
{ "github.copilot.chat.agentDebugLog.fileLogging.enabled": true }
```

Con file logging activo, cualquier sesión queda registrada y se puede revisar después: tool calls, LLM requests, token usage, errores. Sirve para análisis postmortem cuando una sesión salió cara y querés entender por qué.

Fuente: [dvlprlife.com, Quick Tips: Debug Copilot Agent Session Logs, mayo 2026](https://www.dvlprlife.com/2026/05/quick-tips-debug-copilot-agent-session-logs-in-vs-code/)

**Export a OTLP JSON:** Cada sesión puede exportarse para compartir, archivar o ingestar en cualquier backend OTel (Jaeger, Grafana, Langfuse, etc.). Click en el ícono Export (download) en el toolbar del panel. Esto es el puente entre la observabilidad local (Capa 2) y la observabilidad centralizada (Capa 3, ver Sección 9).

**El comando /troubleshoot:** Con file logging habilitado, podés pedirle al mismo Copilot que analice sus propios logs:

```
/troubleshoot how many tokens did I use?
/troubleshoot why did it skip that tool?
```

Es meta-debugging: el LLM analiza su propio comportamiento previo. Útil para encontrar patrones de derroche cuando no es obvio a simple vista.

Fuente: [VSCode Docs, Troubleshoot AI in Visual Studio Code](https://code.visualstudio.com/docs/copilot/troubleshooting)

### 4.5 Chat Debug View: inspección request a request

Mientras Agent Debug Log da la vista agregada, el **Chat Debug View** muestra el detalle de cada request individual: system prompt completo enviado al modelo, user prompt, contexto incluido (archivos, snippets, historial), y respuesta del modelo. Es la vista "mecánica" del IDE, equivalente a abrir el inspector de red en un browser.

Sirve sobre todo para auditar **qué se está mandando al modelo en cada turno**. A veces lo que se manda es mucho más de lo que el dev imaginaba (archivos enteros cuando alcanzaba con un fragmento, historial completo cuando se podía resumir).

**Cómo abrir:** Menú `(...)`, **"Show Chat Debug View"** o Command Palette, **"Developer: Show Chat Debug View"**

Fuente: [Medium, GitHub Copilot Token Usage Explained, Simform Engineering, mayo 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 4.6 Token usage visibility para BYOK (VSCode 1.120)

Antes de la versión 1.120 de VSCode, cuando un dev usaba su propia API key (BYOK, Bring Your Own Key), el indicador de uso de tokens en Copilot siempre mostraba 0%, lo cual generaba la falsa impresión de "consumo cero". La versión 1.120 corrige ese accounting: ahora el indicador refleja correctamente el uso real cuando se usa BYOK, con lo cual los devs pueden monitorear su propio costo aunque estén pagando directamente a Anthropic/OpenAI.

### 4.7 Auditar MCPs activos

Una optimización barata y subestimada: cada MCP activo agrega tokens de definiciones al contexto base de cada request. Los números importan cuando hay muchos MCPs activos.

La aritmética típica: **10 MCPs × 500 tokens promedio por definición = 5.000 tokens de overhead por request**. Multiplicado por 100.000 requests al mes con Sonnet ($3/MTok input), eso son **$1.500/mes solo en MCPs que no se están usando**. Y eso sin contar que tool search lo mitiga parcialmente, pero no lo elimina del todo.

**Acción:** hacer una pasada cada cierto tiempo (mensual idealmente) por los MCPs activos y desactivar los que el equipo no usa. Si un MCP solo se necesita para una tarea puntual, activarlo a demanda y desactivarlo después.

**Dónde se administran los MCPs activos.** En VSCode, Command Palette, "MCP: Show Installed Servers". Vas a ver una lista con cada MCP y un toggle para activarlo/desactivarlo individualmente.

---

## 5. Nivel 1: arquitectura de contexto <a id="5-nivel-1"></a>

Mientras el Nivel 0 era todo configuración sobre productos cerrados, el Nivel 1 entra en el territorio donde se gana de verdad: **el diseño del contexto que mandás al modelo en cada request**. Tres palancas, todas en Capa 1:

1. **Prompt caching.** Pagar 10% por contenido repetido.
2. **Gestión de historial.** No re-enviar 200K tokens cuando 10K alcanzan.
3. **RAG y output budgets.** Mandar solo lo relevante y limitar la longitud de la respuesta.

### 5.1 Prompt caching: la palanca individual de mayor impacto

Prompt caching es la palanca individual de mayor impacto en aplicaciones que reutilizan contexto. La idea es simple: si un bloque de contenido se repite entre llamadas (un system prompt largo, una base de conocimiento, una guía de estilo, los primeros N turnos de una sesión), se puede marcar para que el proveedor lo cachee y cobre solo el 10% del precio en las llamadas siguientes.

#### Cómo funciona la economía del caching

El costo se reparte así:

| Operación | Costo |
|-----------|-------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hora | 2.0x base |
| Cache read | 0.10x base (90% descuento) |

**Cómo leer esta tabla:** la primera vez que mandás un bloque cacheable, pagás un poquito más (1.25x si querés que dure 5 minutos, 2x si querés que dure una hora). De ahí en adelante, cada vez que reutilizás ese bloque, pagás solo el 10% de su costo normal. Si lo reutilizás dos veces dentro de la ventana, ya estás ahorrando. Si lo reutilizás cien veces, el ahorro es enorme.

El break-even es de **un solo hit**. Cualquier app con system prompt estable o RAG está dejando dinero en la mesa si no lo activa.

#### Caching cuando usás herramientas con configuración (Claude Code, Copilot, Claude Cowork)

Si usás herramientas como Claude Code, Claude Cowork o Copilot Chat, **el caching lo hace la herramienta por vos**. No tenés que activar nada. Tu `CLAUDE.md`, tu system prompt, los archivos que cargás, todo va al cache automáticamente. VSCode ha publicado que su prompt caching reuse rate dentro del editor es del **93%**.

Lo que sí podés hacer cuando estás en una herramienta con configuración es **medir el efecto** (con OpenTelemetry, ver Sección 9) y **favorecer el cache con buenas prácticas**:

- Mantené estable la estructura de tus prompts. Cambiar el orden de las instrucciones invalida el cache.
- No cierres y abras archivos sin necesidad. Cada vez que cambiás el set de archivos abiertos, podés estar invalidando porciones del cache.
- Una conversación larga en un mismo contexto cachea bien; saltar entre proyectos resetea la cuenta.
- Si Claude Code se está portando lento o consumiendo mucho, abrí una sesión nueva en lugar de seguir la del lunes pasado.

#### Caching cuando consumís API directa de Anthropic

Acá sí lo activás vos, y hay **dos modos**: automatic y explicit. La diferencia tiene que ver con cuánto control querés sobre dónde se ubica el cache breakpoint.

##### Automatic caching: el modo "empezás en 30 segundos"

Anthropic agregó automatic caching a su API en abril 2026. La idea: en lugar de marcar manualmente qué bloque cachear, ponés un solo campo a nivel top-level del request y el sistema ubica el breakpoint solo, en el último bloque cacheable.

**Dónde va este snippet:** en el código de tu aplicación, en el archivo donde inicializás el cliente Anthropic y construís cada `messages.create()`. Típicamente un servicio del tipo `services/llm_client.py`.

> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a `api.anthropic.com`

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},  # automatic caching, top-level
    system="Sos un asistente interno de Visma. ...largo system prompt...",
    messages=[{"role": "user", "content": user_message}]
)
```

**Qué hace este parámetro.** Anthropic recibe el request y automáticamente coloca el cache breakpoint en el último bloque cacheable (el system prompt en este caso). En llamadas siguientes con el mismo contenido, el system prompt se sirve desde cache al 10% del costo.

**Cuándo conviene automatic.** Es el modo recomendado para empezar. Si recién estás incorporando caching a tu app y querés ver el ahorro rápido sin tener que decidir qué cachear, este modo te lleva al 80% del beneficio con cero decisiones de diseño.

##### Explicit caching: el modo "control fino"

Cuando necesitás caching más granular (múltiples bloques cacheables con TTLs distintos, control sobre exactamente qué se cachea y qué no), pasás a explicit. Marcás cada bloque que querés cachear con `cache_control` directamente. Hasta 4 breakpoints por request.

> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que necesitan control fino del caching

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {"type": "text", "text": "Sos un asistente interno...",
         "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": knowledge_base_content,
         "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": user_message}]
)
print(f"Cache read: {response.usage.cache_read_input_tokens}")
```

**Qué hace:**
- El primer bloque del system prompt (las instrucciones del asistente) tiene su propio `cache_control`. Va a cachearse como bloque independiente.
- El segundo bloque (la base de conocimiento) tiene otro `cache_control`. También se cachea como bloque independiente.
- Si en futuras llamadas las instrucciones cambian pero la base de conocimiento no (caso típico cuando iterás sobre el system prompt), el segundo bloque se sigue sirviendo desde cache aunque el primero se invalide.

**Cuándo conviene explicit:**
- Cuando tu prompt tiene partes que cambian con frecuencia y partes que no. Querés que las que no cambian se sigan sirviendo del cache aunque las otras se invaliden.
- Cuando algunos bloques son inmensos (una base de conocimiento de 50K tokens) y otros chicos (un system prompt corto). Caching de ambos separadamente te da más resiliencia.

**TTL extendido (1 hora):** si tus llamadas se distribuyen en ventanas más largas que 5 minutos, podés extender el TTL:

```python
"cache_control": {"type": "ephemeral", "ttl": "1h"}
```

El cache write con TTL de 1h cuesta 2x base (en lugar de 1.25x), pero el cache read sigue siendo 10%. Si tu app hace llamadas en ventanas de 30 a 60 minutos, el TTL extendido es claramente conveniente. Si las llamadas se concentran en ráfagas de minutos, el TTL default de 5 min alcanza.

##### Cómo verificás que el caching está funcionando

Esta es la parte que la mayoría de los devs no hace y por eso no sabe si su caching anda. En la respuesta de cualquier llamada API de Anthropic, el objeto `usage` te trae:

```
"usage": {
    "input_tokens": 21,
    "cache_creation_input_tokens": 188086,
    "cache_read_input_tokens": 0,
    "output_tokens": 393
}
```

**Cómo leer esto:**
- `input_tokens`: tokens nuevos en este request, fuera del cache. En esta llamada, solo el user message (21 tokens).
- `cache_creation_input_tokens`: tokens que se escribieron al cache en esta llamada. La primera vez que mandás un bloque cacheable, todos sus tokens van acá.
- `cache_read_input_tokens`: tokens que se leyeron del cache (al 10% del precio). En la primera llamada de la sesión está en cero. En la segunda llamada con el mismo prefix, debería ser igual a lo que era `cache_creation` la primera vez.
- `output_tokens`: lo que generó el modelo.

**El patrón saludable:** primera llamada, cache_creation alto, cache_read en cero. Segunda llamada (con el mismo prefix), cache_creation en cero o bajo, cache_read alto. Tercera llamada, igual a la segunda. Y así.

**Si nunca ves cache_read distinto de cero**, el cache no está activando. Causas comunes:
- El contenido cacheable está cambiando entre requests (típicamente porque hay un timestamp o un session ID al principio del prompt).
- Las llamadas están separadas por más de 5 minutos (con TTL default) y el cache expiró.
- Hay un bug en tu app que está enviando el prompt sin el `cache_control`.

Una buena práctica es loguear `cache_read_input_tokens / (cache_read_input_tokens + input_tokens)` como métrica de "cache hit ratio" en tus dashboards (Sección 9). Si esa métrica baja, algo se rompió.

##### Caching en Bedrock y Vertex (mismo principio, misma sintaxis)

Si consumís Claude vía AWS Bedrock o Google Vertex, la sintaxis es la misma. Anthropic mantiene paridad entre su API directa y los hyperscalers. Bedrock soporta `cache_control: {"type": "ephemeral"}` igual que la API directa.

> 🔧 **Capa 1** (código de app) · SDK Anthropic via Bedrock · Developers de apps con compliance EU o AWS-only

```python
import boto3, json
client = boto3.client("bedrock-runtime")

body = {
    "anthropic_version": "bedrock-2023-05-31",
    "system": [{"type": "text", "text": "Reply concisely"}],
    "messages": [{"role": "user", "content": [
        {"type": "text", "text": "Describe the best way to learn programming."},
        {"type": "text", "text": "<contenido cacheable>",
         "cache_control": {"type": "ephemeral"}}
    ]}],
    "max_tokens": 2048
}

response = client.invoke_model(
    modelId="anthropic.claude-sonnet-4-6-20260101-v1:0",
    body=json.dumps(body)
)
```

La documentación oficial de cada hyperscaler tiene el detalle por modelo, porque no todos los modelos cachean todo. Para Vertex la sintaxis es equivalente, usando el SDK de Google.

#### Cuándo aplica con más impacto

- Apps RAG con base de conocimiento grande y estable.
- Asistentes con system prompts complejos (instrucciones, ejemplos, guías de tono, herramientas).
- Agentes multi-turn donde los primeros N turnos se mantienen iguales.

**Caso típico de impacto:** una aplicación RAG con system prompt + base de conocimiento de ~50K tokens, llamada 1.000 veces al día. Sin caching, se procesan 50M tokens/día solo en setup. Con caching, esos 50K se cobran una vez al precio completo (cache write) y luego ~999 veces al 10%. El ahorro neto medido es del **84%** sobre la factura total de esa app.

Fuente: [Finout, 2026](https://www.finout.io/blog/anthropic-api-pricing), [Anthropic Prompt Caching Docs oficial](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)

### 5.2 Gestión de historial

En aplicaciones agénticas y de chat el problema crónico, como vimos en la Sección 1.3, es que el contexto se acumula turno a turno. Hay tres estrategias para evitar que cada llamada termine procesando 200K tokens cuando alcanzaban con 10K. Cada estrategia tiene un trade-off distinto entre fidelidad de la memoria y costo.

**Dónde van estos snippets:** en el código que orquesta las llamadas al modelo. Típicamente un módulo del tipo `services/conversation_manager.py` que recibe el array de mensajes, aplica alguna estrategia, y pasa el array procesado al cliente Anthropic.

> 🔧 **Capa 1** (código de app) · Helpers Python sobre el SDK · Developers de aplicación que diseñan el manejo de sesiones agénticas

**Estrategia A: ventana deslizante.** La más simple: mantener solo los últimos N turnos y descartar lo viejo. Es rápida, predecible y útil cuando la conversación no requiere memoria a largo plazo (asistentes de tareas puntuales, chats transaccionales).

```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**Cuándo conviene:** chats transaccionales, asistentes que resuelven una pregunta y siguen.

**El riesgo:** si algo importante se mencionó en el turno 3 y la ventana es 10, al turno 13 el modelo lo "olvida". Para muchos casos eso es aceptable; para otros (asistentes de soporte, sesiones de debugging largas) no.

**Estrategia B: resumen progresivo.** Cuando la conversación crece, se toma todo lo viejo (todo menos los últimos N turnos recientes) y se resume con un modelo barato (Haiku idealmente). El resumen reemplaza el historial en la próxima llamada. Preserva memoria de largo plazo a costo bajo.

```python
def compress_old_history(messages, recent_window=5):
    if len(messages) <= recent_window:
        return messages
    old, recent = messages[:-recent_window], messages[-recent_window:]
    summary = client.messages.create(
        model="claude-haiku-4-5", max_tokens=500,
        messages=[{"role": "user", "content": f"Resumí en 3 párrafos:\n{format_messages(old)}"}]
    ).content[0].text
    return [{"role": "user", "content": f"[Resumen]\n{summary}"},
            {"role": "assistant", "content": "Entendido."}] + recent
```

**Notá la elección de Haiku para el resumen.** Usar el modelo flagship para resumir sería tirar plata. Resumir es una tarea simple donde Haiku rinde igual a 1/3 del precio.

**Cuándo conviene:** asistentes que necesitan recordar el contexto general de una sesión larga (soporte, debugging, planificación) sin pagar el costo de re-enviar todo.

**Estrategia C: Compaction API (beta enero 2026, elegible ZDR).** Esta es la opción más limpia y la recomendada para agentes en producción. En lugar de implementar tu propia lógica de resumen, Anthropic provee una API beta que se encarga de la compactación automáticamente cuando el contexto supera un umbral. Vos definís el threshold, el sistema decide cuándo y cómo compactar.

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112", "context_token_threshold": 50_000}]}
)
```

**Qué hace:** cuando el contexto supera los 50K tokens, Anthropic compacta automáticamente la parte vieja del historial usando un proceso optimizado que mantiene la información relevante y descarta el ruido.

**Cuándo conviene:** agentes en producción donde no querés mantener tu propia lógica de compresión. Es elegible para ZDR (Zero Data Retention), así que no agrega complicaciones de compliance.

### 5.3 RAG y output budgets

Estas son dos prácticas complementarias que casi todos los equipos en producción terminan adoptando. Vale entender cada una por separado.

> 🔧 **Capa 1** (código de app) · Pipeline RAG + diccionario de budgets · Developers de aplicación que diseñan el flujo de retrieval y los topes por tipo de tarea

**RAG (Retrieval-Augmented Generation).** En lugar de meter una base de conocimiento entera al system prompt y pagarla cada vez, se la indexa en una vector store y se recuperan solo los fragmentos relevantes para cada query. El modelo recibe únicamente lo que necesita para responder, no todo el corpus.

```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
```

**Qué pasa.** En lugar de mandar 50K tokens de base de conocimiento al modelo, mandás los 5 fragmentos más relevantes (~2.500 tokens). El modelo responde con la misma calidad porque el resto del corpus no era pertinente para esa query específica.

El impacto típico es alto: **RAG reduce los tokens hasta un 70%** comparado con "context stuffing" (meter toda la documentación al system prompt). El truco operativo está en afinar el `top_k` y el largo máximo por chunk: 5 chunks de 500 caracteres alcanzan para la mayoría de los casos; subirlos solo si el modelo demuestra falta de contexto.

Fuente: [Koombea, 2026](https://ai.koombea.com/blog/llm-cost-optimization)

**Output budgets.** Como vimos en la Sección 1.1, los output tokens cuestan 5x más que los input. La consecuencia operativa es que **limitar la longitud de la respuesta tiene un impacto desproporcionado en el costo**. La forma estándar de hacerlo es no usar un `max_tokens` único para toda la app, sino calibrar el budget según el tipo de tarea:

```python
OUTPUT_BUDGETS = {
    "classification": 50,
    "extraction": 200,
    "summarization": 500,
    "code_review": 1000,
    "code_generation": 2000,
    "architecture": 3000
}
```

**Cómo se aplica.** Cuando armás cada `messages.create()`, en lugar de pasarle un `max_tokens=4096` genérico, le pasás el budget correspondiente al tipo de tarea: `max_tokens=OUTPUT_BUDGETS["classification"]` para clasificaciones, etc.

La idea es que una clasificación binaria nunca necesita 1.000 tokens de output (probablemente alcanza con 5), una extracción de entidades no debería pasar de 200, y solo las tareas que justifican respuesta larga (generación de código, diseño de arquitectura) reciben budgets grandes. Esto evita que el modelo "se explaye" cuando no es necesario, sin perder capacidad cuando sí lo es.

Combinado con RAG, el flujo completo queda: retrieval acotado, contexto chico, output budget acotado al tipo de tarea. Resultado: factura predecible y orden de magnitud menor.

---

## 6. Nivel 2: selección y asignación de modelos <a id="6-nivel-2"></a>

Este nivel resuelve una pregunta que parece trivial pero rara vez se trata con rigor: **¿cuál modelo usás para cada tarea?** La respuesta default en la mayoría de los equipos es "el flagship para todo", y es exactamente el error que infla la factura.

La diferencia de precio entre modelos del mismo proveedor es enorme (Haiku $1/$5 vs Opus $5/$25, es decir 5x), y para muchas tareas Haiku rinde igual o casi igual que Opus. Asignar el modelo correcto por tipo de tarea es probablemente el ajuste de mayor relación costo/esfuerzo del documento.

> 🔧 **Capa 1** (código de app) o **Capa 4** (en gateway si se centraliza) · Diccionario de asignación + router clasificador · Developers de aplicación, o plataforma si se mueve al gateway compartido

La estrategia es simple: mantener un diccionario de qué tipo de tarea va a qué modelo, y un clasificador barato (Haiku) que decide para cada request entrante a cuál tipo pertenece. El clasificador agrega un costo marginal (50 tokens de output a precio de Haiku, es decir, fracciones de un centavo por request) pero permite rutear correctamente todo el tráfico subsiguiente.

```python
MODEL_ASSIGNMENT = {
    "classification": "claude-haiku-4-5", "entity_extraction": "claude-haiku-4-5",
    "data_formatting": "claude-haiku-4-5", "file_navigation": "claude-haiku-4-5",
    "simple_qa": "claude-haiku-4-5",
    "code_generation": "claude-sonnet-4-6", "code_review": "claude-sonnet-4-6",
    "summarization": "claude-sonnet-4-6", "general_chat": "claude-sonnet-4-6",
    "debugging": "claude-sonnet-4-6",
    "architecture_design": "claude-opus-4-7", "complex_reasoning": "claude-opus-4-7",
    "agent_coordination": "claude-opus-4-7",
}

def classify_and_route(user_input, client):
    result = client.messages.create(
        model="claude-haiku-4-5", max_tokens=50,
        system="Clasificá en: classification, extraction, summarization, code_generation, "
               "code_review, debugging, architecture_design, complex_reasoning, general_chat. "
               "SOLO el nombre.",
        messages=[{"role": "user", "content": user_input}]
    )
    task_type = result.content[0].text.strip().lower()
    return task_type, MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
```

**Cómo leer el diccionario:**
- **Haiku** para tareas mecánicas: clasificación, extracción de entidades, formateo de datos, navegación de archivos, Q&A simples. Son tareas donde la diferencia de calidad entre modelos es marginal y el ahorro es 5x.
- **Sonnet** para el grueso del trabajo de desarrollo: generación y revisión de código, resúmenes, chat general, debugging. Es el sweet spot precio/calidad y debería ser el default cuando la tarea no se puede clasificar claramente como simple o muy compleja.
- **Opus** solo para lo realmente complejo: diseño de arquitectura, razonamiento multi-paso, coordinación entre agentes. Si tu aplicación gasta el grueso del presupuesto en Opus, hay una probabilidad alta de que estés sobrecalificando las tareas.

**Una nota sobre el fallback:** la última línea (`MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")`) hace que si el clasificador devuelve algo no esperado, se caiga a Sonnet por default. Es el valor seguro: si dudás, Sonnet. Nunca caer a Opus por default, ese debería ser siempre una decisión explícita y justificada.

**Implementación a escala:** Cuando hay varios servicios, este diccionario y la lógica de routing son exactamente lo que conviene mover al wrapper interno o al gateway (Sección 2.4). De esa forma la "tabla de asignación" es una sola, gobernable y auditable.

---

## 7. Nivel 3: model routing automático <a id="7-nivel-3"></a>

El Nivel 2 resuelve la asignación con reglas explícitas: un diccionario hecho a mano que mapea tipos de tarea a modelos. Funciona muy bien cuando las categorías son claras y el dominio es estable.

El Nivel 3 lleva la idea un paso más allá: **dejar que un router aprenda automáticamente qué requests requieren el modelo flagship y cuáles puede resolver un modelo más barato**, sin necesidad de definir las reglas a mano. Es lo que se conoce como "model routing" o "cascading" en la literatura, y los resultados publicados son notables.

### 7.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)

RouteLLM es un sistema open source publicado en ICLR 2025 por UC Berkeley y LMSYS. La idea central: entrenar un router (un clasificador chico) que, para cada nuevo request, decide si lo resuelve el "modelo fuerte" (flagship) o el "modelo débil" (uno más barato). El router aprende de comparaciones humanas previas qué características del prompt predicen que el modelo barato va a ser suficiente.

Los resultados publicados: **95% de la calidad de GPT-4** usando solo el **26% de las llamadas** ruteadas al modelo fuerte. Es decir, el 74% restante se resuelve con el modelo barato sin pérdida medible de calidad. El **ahorro neto medido es del 48%**, y con técnicas de augmentation puede llegar hasta el **75%**.

Fuente: [LMSYS ICLR 2025](https://www.lmsys.org/blog/2024-07-01-routellm/)

> 🔧 **Capa 4** (gateway / routing layer) · Open source UC Berkeley/LMSYS · Plataforma o equipo de infra que pone una capa entre los servicios y los proveedores

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

**El parámetro clave es el threshold.** Define qué tan agresivo es el router. Un threshold bajo (0.1) manda más cosas al modelo fuerte (más calidad, menos ahorro). Un threshold alto (0.5+) manda más cosas al modelo débil (más ahorro, riesgo de calidad). El paper recomienda calibrarlo midiendo calidad sobre un subset representativo de tu tráfico.

**Cuándo conviene RouteLLM:** cuando ya tenés volumen suficiente (decenas de miles de requests/día) y la heurística de Nivel 2 te queda corta porque tu tráfico es más diverso que las categorías que pudiste definir. Es un step-up natural sobre la asignación explícita: del diccionario hecho a mano al modelo entrenado.

### 7.2 AI Gateways enterprise

Más allá de RouteLLM como solución específica, en producción enterprise se imponen los **AI gateways**: una capa de red por la que pasan todas las llamadas a LLMs y que centraliza routing, retries, fallbacks, presupuestos por equipo, guardrails y observabilidad. Es el equivalente a un API gateway tradicional, pero específico para LLMs.

> 🔧 **Capa 4** (gateway compartido) · LiteLLM self-hosted / Portkey managed · Plataforma o equipo de infra. El código de los servicios consumidores NO cambia, el gateway intercepta

```python
# LiteLLM: self-hosted, budget controls
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"}, "tpm": 100000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"}, "tpm": 100000}
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)

# Portkey: managed, compliance
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6", messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

Hay tres jugadores principales en este espacio y conviene entender en qué se diferencian:

| Gateway | Tipo | Mejor para |
|---------|------|------------|
| LiteLLM | Open source | Control total, zero fees |
| Portkey | Managed | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | Experimentación rápida |

**LiteLLM** se self-hostea (es open source), no agrega fees al token cost, y da control total. Tiene budget manager built-in con Redis, multi-provider routing, fallbacks automáticos. Es la elección típica para equipos que quieren un gateway sin agregar otro vendor a la stack.

**Portkey** es managed (SaaS) y se especializa en compliance: SOC2, guardrails, virtual keys (separar credenciales por equipo sin exponer la API key real), audit logs completos. Cobra fee adicional sobre el tráfico. Es la elección típica para empresas con requerimientos serios de compliance y que valoran no operar un gateway.

**OpenRouter** es la opción para experimentación: catálogo enorme de modelos (100+ providers), una sola API key, paga el costo de cada provider más un margen pequeño. Útil para POCs y para descubrir qué modelo nuevo conviene probar, no tanto para producción enterprise por temas de compliance y observabilidad.

**Regla operativa:** si el equipo es de plataforma y no quiere agregar vendors, LiteLLM. Si compliance es la prioridad, Portkey. OpenRouter rara vez termina siendo la elección de producción a escala, pero es excelente para fase de descubrimiento.

---

## 8. Nivel 4: infraestructura de costo y gobernanza <a id="8-nivel-4"></a>

Este nivel reúne tres palancas que tienen en común no ser optimizaciones de prompt o de modelo, sino **decisiones de infraestructura y procesos** que cambian la estructura del costo. Cada una tiene su lógica:

1. **Batch API.** Para trabajos no-realtime, mitad de precio sin pérdida de calidad.
2. **DeepSeek vía Azure.** Para tareas masivas baratas con compliance europeo.
3. **Stoppers y observabilidad.** Para que el costo no se desborde sin avisar.

### 8.1 Batch API (50% descuento, calidad idéntica)

Una buena parte del consumo de LLM en empresas no necesita responder en tiempo real. Procesar 50.000 documentos para extraer entidades, generar embeddings de un corpus, evaluar miles de respuestas contra una rúbrica, hacer clasificación masiva de tickets históricos: todos esos casos pueden esperar minutos u horas para devolver resultados.

Para esos casos, Anthropic (y OpenAI) ofrecen Batch API: las llamadas se procesan asincrónicamente cuando hay capacidad disponible, con un SLA de hasta 24 horas, **al 50% del precio normal**. No hay pérdida de calidad: es exactamente el mismo modelo, solo con un canal de latencia distinto.

> 🔧 **Capa 1** (código de app) · SDK Anthropic Batches API · Developers de aplicación que procesan jobs no-realtime (ETL, evaluaciones, generación masiva)

```python
batch = client.messages.batches.create(requests=[
    {"custom_id": f"doc-{doc.id}", "params": {
        "model": "claude-sonnet-4-6", "max_tokens": 200,
        "system": [{"type": "text", "text": system_prompt, "cache_control": {"type": "ephemeral"}}],
        "messages": [{"role": "user", "content": doc.text[:1000]}]
    }} for doc in documents
])
import time
while client.messages.batches.retrieve(batch.id).processing_status != "ended":
    time.sleep(60)
results = {r.custom_id: r.result.message.content[0].text
           for r in client.messages.batches.results(batch.id)
           if r.result.type == "succeeded"}
```

Notá una optimización adicional en el snippet: el `cache_control: {"type": "ephemeral"}` aplicado al system prompt. Cuando combinás Batch + caching, los descuentos se acumulan multiplicativamente: 50% de batch sobre 10% de cache read. **Resultado documentado: hasta 95% de ahorro** comparado contra mandar cada doc por separado en modo realtime sin cache. Fuente: [PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/)

**Cuándo aplica:**
- Pipelines ETL nocturnos.
- Evaluación masiva de modelos (eval harnesses).
- Generación de datasets sintéticos.
- Re-procesamiento de datos históricos.

**Cuándo NO aplica:** cualquier cosa que tenga al usuario esperando una respuesta en pantalla. Aunque la latencia mínima son minutos, no segundos.

### 8.2 DeepSeek via Azure

Este es uno de los pocos casos en este documento donde la elección de hyperscaler es la correcta **por razones del modelo, no de procurement general** (ver Sección 3 para la regla general).

> 🔧 **Capa 1** (código de app) · SDK OpenAI apuntando a Azure endpoint · Developers de aplicación que procesan tareas masivas baratas con compliance EU

```python
from openai import AzureOpenAI
client = AzureOpenAI(api_key=os.getenv("AZURE_API_KEY"),
                     api_version="2024-12-01-preview",
                     azure_endpoint=os.getenv("AZURE_ENDPOINT"))  # Región EU
response = client.chat.completions.create(
    model="deepseek-v4-flash", max_tokens=50,
    messages=[{"role": "user", "content": text_to_classify}]
)
```

**Contexto de procurement:** DeepSeek V4-Flash es aproximadamente **15x más barato que Sonnet** en precio nominal (~$0.14/MTok input vs $3.00 de Sonnet), con compliance europeo asegurado al ser servido por Azure. Es decir, te da una calidad razonable para tareas mecánicas a un precio cercano al de los modelos más baratos del mercado.

El acceso directo a DeepSeek (`api.deepseek.com`) **no es viable para empresas con datos EU** porque rutea todos los requests por servidores chinos, lo cual choca de frente con cualquier requerimiento serio de GDPR o data residency.

Azure agrega un markup del **20 a 35% sobre el precio directo** ([DeployBase, 2026](https://deploybase.ai/articles/deepseek-v3-pricing)), pero **ese markup no es overhead general, es prima de compliance**: los datos quedan en la región Azure que elegiste (EU si así lo necesitás), el contrato es con Microsoft, y el compliance europeo está cubierto. Aun con ese 20 a 35% encima, DeepSeek vía Azure sigue costando aproximadamente **1/15 lo que Sonnet**, así que para clasificación masiva, extracción estructurada o batch processing offline, sigue siendo la mejor relación precio/compliance del mercado.

**Nota crítica:** Este es uno de los pocos casos donde el hyperscaler es la elección correcta por razones de modelo, no de procurement general. Para Claude o GPT, el mismo cálculo no aplica (ver Sección 3 sobre pricing parity nominal). La regla es: **si el modelo en sí no es accesible directamente con compliance, el hyperscaler vale el markup; si el modelo ya está disponible directamente del vendor con compliance, el hyperscaler suma overhead sin agregar valor.**

### 8.3 Stoppers y observabilidad de costo

La pregunta que mata equipos: "¿quién consumió $5.000 en Claude esta semana?". Sin instrumentación, la respuesta llega cuando ya pasó. Con instrumentación, llega cuando todavía se puede frenar.

#### Cómo configurar stoppers por vendor

Antes de instrumentar nada custom, **cada herramienta tiene un mecanismo nativo de spending limits que conviene revisar y configurar**. Esto no requiere código: son settings de admin en cada plataforma.

| Herramienta | Hay stopper por default | Dónde se configura |
|---|---|---|
| **GitHub Copilot (nuevo billing junio 2026)** | No. Por encima de los AI Credits del seat, el overage se factura sin límite | Admin organización en GitHub, Billing & Plans, setear "spending limit" en cero o en un monto X |
| **Cursor** | Hay tope por plan, pero permite "additional usage" si lo activás | Settings de la org, desactivar overage |
| **Claude Desktop / Code (planes Pro/Team/Enterprise)** | Sí, te corta cuando se acaba la cuota del plan | Subir de plan o esperar reset mensual |
| **Anthropic API directa** | No. Cobra hasta que llegue el límite de tu tarjeta o tu workspace limit | Consola Anthropic, Limits, Workspace spend limit |
| **OpenAI API directa** | No. Mismo modelo que Anthropic | Dashboard OpenAI, Settings, Limits, Monthly budget |
| **Azure OpenAI / Bedrock** | No. Va a la factura cloud sin tope salvo que se ponga budget alert | Azure Cost Management, Budgets, AWS Budgets, alertas en X% del presupuesto |

**Cómo se lee esta tabla operacionalmente.** Para cada herramienta que use tu BU:
1. Entrá a la consola admin correspondiente.
2. Buscá el setting de spending limit / budget / quota.
3. Configurá un tope o al menos una alerta al 50%, 80% y 100% de tu presupuesto mensual.
4. Documentá quién es el responsable de revisar esos topes cada mes.

**El stopper no es solución, es alarma de incendio.** Si tu factura llega al tope, algo se rompió antes. La pregunta no es "¿cuándo nos cortan?", es "¿quién está mirando la trayectoria y avisando antes de que el tope se acerque?".

#### Budget monitor custom (para apps con API directa)

Si construís apps que llaman directo a la API, además del spending limit del vendor podés instrumentar un **budget monitor** en tu propio código que rastree gasto por usuario/equipo y alerte progresivamente.

> 🔧 **Capa 1** (código de app) o **Capa 4** (gateway centralizado) · Helper Python sobre el SDK o middleware del gateway · Developers de aplicación o plataforma

```python
class BudgetMonitor:
    def __init__(self, monthly_budget_usd, user_id):
        self.budget = monthly_budget_usd
        self.user_id = user_id
        self.spend = 0.0
        self.thresholds = [0.5, 0.8, 1.0]
        self.alerted = set()

    def check(self, new_spend):
        self.spend += new_spend
        ratio = self.spend / self.budget
        for t in self.thresholds:
            if ratio >= t and t not in self.alerted:
                self.alerted.add(t)
                send_notification(
                    f"⚠️ {self.user_id}: {ratio:.0%} del presupuesto "
                    f"(${self.spend:.2f}/${self.budget:.2f})",
                    channel="ai-costs-alerts"
                )
```

**Tres puntos importantes sobre este patrón:**

1. **Los thresholds están graduados.** Al 50% se avisa para que el usuario tenga visibilidad; al 80% para que actúe; al 100% para que escale o se detenga. Avisar solo al 100% es demasiado tarde.
2. **`alerted` evita spam.** Una vez que se cruza un threshold, no se vuelve a notificar. Los usuarios desactivan notificaciones que les llegan cien veces al día.
3. **Donde ponerlo importa.** Si lo metés en cada servicio, cada equipo tiene su propio monitor (drift, datos inconsistentes). Si lo metés en el gateway compartido (Capa 4), una sola implementación gobierna todo el tráfico de la empresa. Cuanto más grande la organización, más conviene la opción del gateway.

A escala enterprise, este patrón básico se complementa con OTel (Sección 9) para tener métricas, traces y logs unificados, y dashboards que muestran consumo por equipo, por modelo, por feature.

---

## 9. Nivel 5: observabilidad con OpenTelemetry y gobernanza personal <a id="9-otel"></a>

Si los niveles anteriores tratan sobre **ahorrar** tokens, este nivel trata sobre **saber qué está pasando**. Sin observabilidad, todas las optimizaciones son apuestas a ciegas: no podés afirmar que el caching está funcionando, no sabés qué equipo gasta más, no podés detectar regresiones de costo cuando se introducen.

OpenTelemetry (OTel) se convirtió en el estándar de facto para observabilidad de IA. Tiene tres ventajas frente a alternativas propietarias: es vendor-neutral (los datos no quedan atrapados en un SaaS), tiene soporte nativo en los CLIs de los principales proveedores (Claude Code, Copilot) y se integra con cualquier backend (Grafana, Prometheus, Jaeger, Langfuse, etc.).

Esta sección tiene dos audiencias: **individuos** que quieren ver su propio consumo (Sección 9.6) y **plataformas** que quieren observar a toda la empresa (Secciones 9.1 a 9.5). Las dos comparten la misma tecnología, cambia solo la escala.

### 9.1 OTel en Claude Code (CLI)

Claude Code expone telemetría OTel completa con solo activar variables de entorno. No hay que instalar nada extra ni modificar el CLI.

**Dónde se setean estas variables.** Las podés poner de dos formas:
- **Locales para tu shell:** las agregás a tu `~/.zshrc` o `~/.bashrc` (Mac/Linux) o a las variables de entorno del usuario en Windows. Quedan activas en cualquier terminal que abras de ahí en adelante.
- **Enterprise vía MDM:** un admin las distribuye automáticamente a todos los devs (ver Sección 9.5).

> 🔧 **Capa 2** (env vars locales) o **Capa 3** (MDM enterprise) · Variables de entorno del shell · Developers individuales o admin via managed settings

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_TRACES_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRIC_EXPORT_INTERVAL=10000
export OTEL_RESOURCE_ATTRIBUTES="team.id=platform,department=engineering"
```

**Qué hace cada variable:**
- `CLAUDE_CODE_ENABLE_TELEMETRY=1` activa el subsystem de telemetría. Sin esta, las demás no hacen nada.
- `OTEL_METRICS_EXPORTER=otlp`, `OTEL_LOGS_EXPORTER=otlp`, `OTEL_TRACES_EXPORTER=otlp` le dicen a Claude Code que exporte métricas, logs y trazas usando el protocolo OTLP (el estándar de OpenTelemetry).
- `OTEL_EXPORTER_OTLP_ENDPOINT` es la dirección del backend al que se mandan los datos. `localhost:4317` significa que hay un backend corriendo en tu propia máquina (típicamente un container, ver Sección 9.3). Para enterprise, esto apunta a un collector interno (`collector.internal:4317`).
- `OTEL_EXPORTER_OTLP_PROTOCOL=grpc` define el formato de transporte. gRPC es el más eficiente; también podés usar `http/protobuf` si gRPC tiene problemas.
- `OTEL_METRIC_EXPORT_INTERVAL=10000` exporta métricas cada 10 segundos (10000 ms). Bajo este valor recargás el backend; arriba perdés granularidad.
- `OTEL_RESOURCE_ATTRIBUTES` etiqueta los datos. Es la más útil de todas para enterprise: te permite filtrar por `team.id` y `department` en el dashboard, comparando consumo entre equipos. Cuando se aplica vía MDM, esta variable se setea automáticamente según el equipo al que pertenece el dev.

**Qué métricas vas a ver.** Claude Code emite seis tipos de métricas, con nombres textuales (estos son los nombres reales que vas a buscar en Grafana o Prometheus):

- `claude_code_token_usage`: desglosado en `input`, `output`, `cache_creation` y `cache_read`. La métrica clave para saber cuánto contexto repetido estás reenviando.
- `claude_code_cost_usage`: costo en USD por API call.
- `claude_code_session_count`: número de sesiones que abriste.
- `claude_code_active_time_total`: tiempo activo de Claude Code.
- `claude_code_lines_of_code_count`: líneas de código agregadas y removidas.
- `claude_code_code_edit_tool_decision`: accept/reject de sugerencias, desglosado por lenguaje.

### 9.2 OTel en Copilot Chat

Para Copilot la activación es por settings, más simple que en Claude Code:

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "otlp",
    "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4317"
}
```

**Qué hace:** activa el export OTel desde Copilot Chat hacia el endpoint que indiques. Si querés algo más simple sin levantar backend, podés exportar a un archivo local:

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "file",
    "github.copilot.chat.otel.outfile": "/tmp/copilot-otel.jsonl"
}
```

**Atributos que exporta.** Copilot Chat sigue las **GenAI Semantic Conventions** de OpenTelemetry, un estándar abierto para nombrar atributos de telemetría de LLMs. Los principales que vas a ver:
- `gen_ai.request.model`: qué modelo se invocó.
- `gen_ai.provider.name`: qué proveedor (anthropic, openai, etc.).
- `gen_ai.tool.name`: qué herramienta se ejecutó.
- `copilot_chat.edit.source`: de dónde vino una edición.
- Token counts y durations por cada span.

**Nota importante:** el OTel monitoring viene **off por default** en Copilot Chat. Hay que activarlo explícitamente. Está bien que sea así porque te da control de qué se exporta y a dónde, especialmente importante por privacidad: por default solo se exportan metadatos (tokens, modelo, duraciones), no el contenido de prompts ni respuestas. Si querés capturar contenido completo:

```json
{ "github.copilot.chat.otel.captureContent": true }
```

Y aclaración del lado del vendor: "Content capture can include sensitive information such as code, file contents, and user prompts". Manejarlo con cuidado.

Tanto Claude Code como Copilot emiten al endpoint OTLP, por lo cual podés tener **un único backend que consolide la observabilidad de todos los productos de IA del equipo**, independientemente de qué herramienta esté usando cada dev.

### 9.3 Stack local en 30 segundos

Para experimentar y validar la captura, no hace falta levantar infraestructura compleja. El Aspire Dashboard de Microsoft es un container que arranca un backend OTLP completo (trazas, métricas, logs) con una UI navegable, todo en un solo `docker run`.

> 🔧 **Capa 2** (estación de trabajo del dev) · Docker container del Aspire Dashboard · Dev individual para uso local sin cuenta cloud

```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

**Qué hace este comando:**
- `--rm` borra el container cuando lo parás (no se acumulan containers viejos).
- `-d` lo corre en segundo plano (detached).
- `-p 18888:18888` mapea el puerto 18888 (UI web) a tu máquina.
- `-p 4317:18889` mapea el puerto 4317 (endpoint OTLP) a tu máquina.
- `--name aspire-dashboard` le pone un nombre para que sea fácil de parar después (`docker stop aspire-dashboard`).
- La última línea es la imagen oficial de Microsoft, mantenida y gratis.

**Cómo lo usás:**
1. Corré el comando.
2. Abrí http://localhost:18888 en el browser.
3. Setea las variables de entorno de la Sección 9.1 con `OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317`.
4. Usá Claude Code normalmente. Cada request que hagas va a aparecer en el dashboard en tiempo real.

Es el camino más rápido para que un dev pruebe localmente "qué se ve" en observabilidad de IA antes de invertir en infraestructura para producción.

### 9.4 Stack producción (Grafana + Prometheus)

Para enterprise, el stack más común es OTel Collector + Prometheus para métricas + Grafana para dashboards. Los tres son open source, ampliamente adoptados, y se orquestan típicamente con docker-compose.

> 🔧 **Capa 3** (infra de observabilidad) · docker-compose para stack OTel + Prometheus + Grafana · Equipo de infra/SRE

```yaml
version: '3.8'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config", "/etc/otel/config.yaml"]
    volumes: ["./otel-config.yaml:/etc/otel/config.yaml"]
    ports: ["4317:4317", "4318:4318"]
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    command: ["--storage.tsdb.retention.time=90d", "--config.file=/etc/prometheus/prometheus.yml"]
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment: ["GF_SECURITY_ADMIN_PASSWORD=admin"]
```

Una vez levantado, las trazas y métricas de Claude Code/Copilot fluyen al collector (puerto 4317), de ahí Prometheus las recolecta (con retención de 90 días en este ejemplo), y Grafana las visualiza. Los dashboards típicos que se arman para AI cost observability incluyen: tokens por equipo/día, costo por feature, ratio cache hit/miss, distribución de modelos por tarea, latencias por endpoint, tasa de error.

### 9.5 Configuración centralizada (MDM)

A escala enterprise no podés pedirle a cada developer que configure su shell con las variables OTel. La forma estándar de hacerlo es distribuir la configuración vía MDM (Intune para Windows, Jamf para Mac). Una sola política push se aplica a todos los devs y queda gobernada centralizadamente.

> 🔧 **Capa 3** (managed settings enterprise) · JSON distribuible vía MDM (Intune, Jamf) · Admin de plataforma de developers

```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

Con esto, **todos los developers del grupo quedan instrumentados automáticamente**, los datos viajan a un collector interno autenticado con bearer token, y desde el primer día tenés visibilidad real de cómo se está usando IA en toda la organización. Es la fundación que habilita las decisiones de optimización de los niveles anteriores: sin observabilidad enterprise, las palancas son apuestas.

### 9.6 Gobernanza personal: tu propio dashboard en 10 minutos

Las secciones 9.1 a 9.5 están pensadas para plataforma. Esta sección es distinta: es para **cualquier persona que use IA todos los días** y quiera ver su propio consumo, sin esperar a que el equipo de plataforma arme infraestructura.

La idea: con las mismas tecnologías que la sección anterior pero a escala individual, en 10 minutos cualquier dev puede tener su dashboard personal que le responde:

- ¿Cuántas sesiones abrí esta semana?
- ¿Cuánto estoy reenviando de contexto repetido?
- ¿Qué modelos estoy usando, en qué proporción?
- ¿Cuántos tokens fueron input vs output?
- ¿Cuánto me cuesta cada día?

#### Camino A: Grafana Cloud (sin servidor, free tier)

Esta es la opción más rápida y más cómoda. Grafana ofrece un tier gratis que incluye un endpoint OTLP, base de datos de métricas y dashboards. Cero infraestructura local, todo desde el browser.

**Pasos:**

1. Te registrás gratis en https://grafana.com/products/cloud/. El free tier es generoso para uso individual.
2. Después del signup, te dan un **endpoint OTLP** específico para tu cuenta. Es una URL del tipo `https://otlp-gateway-prod-us-central-0.grafana.net/otlp`.
3. También te dan credenciales (instance ID + API token) que se pasan como header de autorización.
4. En tu `~/.zshrc` o `~/.bashrc`, poné las variables de entorno apuntando al endpoint de Grafana Cloud:

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp-gateway-prod-us-central-0.grafana.net/otlp
export OTEL_EXPORTER_OTLP_HEADERS="Authorization=Basic <tu-token-base64>"
```

Notá que cambia `OTEL_EXPORTER_OTLP_PROTOCOL` a `http/protobuf` en este caso (Grafana Cloud no acepta gRPC directo).

5. Hacer `source ~/.zshrc` (o reiniciar la terminal). De ahí en adelante, cada sesión de Claude Code va a mandar métricas a Grafana Cloud.
6. En el browser, vas al dashboard de Grafana Cloud y armás las visualizaciones que querés.

#### Camino B: Aspire Dashboard local (si preferís que nada salga de tu máquina)

Si por temas de privacidad preferís que las métricas no salgan de tu computadora, levantás el container de Aspire Dashboard (el mismo de la Sección 9.3). Mismo `docker run`, y las variables apuntan a `localhost:4317` en lugar de Grafana Cloud.

Trade-off: el dashboard solo está disponible cuando el container está corriendo. Si reiniciás la máquina, los datos persisten en el container (mientras no lo borres). Para uso individual está perfecto, pero no es para análisis de tendencias de meses.

#### Cómo armás las visualizaciones útiles

Una vez que el backend está recibiendo datos, las cinco preguntas de gobernanza personal se traducen así en términos de PromQL (el lenguaje de queries de Prometheus, también usado por Grafana Cloud):

- **¿Cuántas sesiones abrí esta semana?** Query: `sum(claude_code_session_count) by (day)`, gráfico de barras agrupado por día.
- **¿Cuánto estoy reenviando de contexto repetido?** Query: ratio entre `claude_code_token_usage{type="cache_read"}` y `claude_code_token_usage{type="input"}`. Cuanto más alto el `cache_read`, mejor.
- **¿Qué modelos estoy usando, en qué proporción?** Query: `sum(claude_code_token_usage) by (model)`, pie chart o barras horizontales.
- **¿Cuántos tokens input vs output?** Query: `sum(claude_code_token_usage) by (type)` filtrando `type="input"` vs `type="output"`. El output cuesta 4 a 6x más, así que ver esta proporción te dice si tus pedidos están sesgados a generación pesada.
- **¿Cuánto me cuesta cada día?** Query: `sum(claude_code_cost_usage) by (day)`, línea temporal.

Si nunca tocaste Grafana, el camino más rápido es importar un dashboard pre-armado: hay varios open source en https://grafana.com/grafana/dashboards/ buscando "Claude Code" o "OpenTelemetry GenAI" que ya tienen estas queries y otras.

#### Por qué importa que cada uno se mida

Esto no es un ejercicio de FinOps clásico. Es algo más directo: **si vos no podés ver tu propio consumo, no podés mejorarlo.** Es la misma lógica de un atleta con su tracker, o de un dev con su profiler. No medirse no es neutral, es ceguera.

Y para una empresa, la gobernanza colectiva no se construye desde arriba pidiendo reportes a las BUs. Se construye desde abajo, con cada persona que entiende su propio uso y ajusta. El tech lead que tiene un dashboard del equipo se basa en datos que existen porque la gente del equipo se midió primero. La BU que reporta consumo razonable a finance lo hace porque los devs midieron antes. Sin el primer paso, lo demás es teatro de presupuesto.

Diez minutos de setup. Cero euros de costo. Las preguntas de gobernanza personal contestadas para siempre.

---

## 10. Nivel 6: proveedores alternativos de menor costo <a id="10-alternativas"></a>

Para cerrar el cuadro: además de optimizar Claude/GPT, hay un universo de proveedores alternativos que para ciertos casos resultan radicalmente más baratos. La franja de precios es enorme: **los precios varían hasta 625x entre el modelo más caro y el más barato del mercado**. La mayoría de los equipos paga entre 4x y 30x más de lo necesario para tareas donde un modelo barato rinde igual.

La estrategia no es "reemplazar todo por el más barato": es **identificar las tareas donde un modelo alternativo da el mismo resultado** y rutear esas tareas allí, dejando Claude/GPT para el trabajo que justifica su precio.

### Groq: LPU hardware, velocidad extrema

Groq tiene hardware custom (LPUs, Language Processing Units) que sirven modelos open source con latencias sub-100ms. Es la elección para tareas que necesitan respuesta instantánea: clasificación inline, autocompletado, moderación de contenido, routing de mensajes.

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan latencia sub-100ms en tareas simples

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```

Llama 4 vía Groq cuesta aproximadamente $0.11/MTok input. Es entre **4 y 10 veces más barato que GPT-4o** para tareas equivalentes, con latencias sub-100ms (frente a 500ms a 1s de los providers tradicionales). Notá el patrón de integración: el SDK de OpenAI funciona apuntando a la URL de Groq, no hay que aprender un nuevo SDK, solo cambiar el `base_url`.

### Together AI: 100+ modelos

Together AI hostea más de 100 modelos open source con SLA empresarial y soporte de fine-tuning. Es la elección para batch processing high-volume y para casos donde necesitás un modelo específico open source que no esté disponible en otras plataformas.

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan batch high-volume con fine-tuning

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, "summarization"))
```

Mismo patrón que Groq: SDK de OpenAI apuntando a otro endpoint. El plus de Together es la diversidad de modelos disponibles y el soporte de fine-tuning gestionado.

### Mistral: el único provider major europeo

Mistral tiene sede en Francia, GDPR nativo, datos en la UE. Es la única alternativa de proveedor de modelos major **europea directa**: sin pasar por hyperscaler, sin overhead de procurement, con compliance europeo asegurado por contrato directo.

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan GDPR nativo sin pasar por hyperscaler

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("MISTRAL_API_KEY"), base_url="https://api.mistral.ai/v1")
response = client.chat.completions.create(model="mistral-small-latest",
    messages=messages, max_tokens=200)
```

Para empresas europeas que necesitan minimizar dependencias americanas y mantener data residency dentro de la UE sin pagar overhead de hyperscaler, Mistral es la primera opción a evaluar.

### Guía de decisión

Esta tabla resume cuál proveedor conviene para cuál tarea. La regla operativa: **cada tarea va al proveedor que mejor balancea calidad, precio y compliance para esa tarea específica**. No hay un proveedor "ganador" universal.

| Tarea | Provider | Razón |
|-------|----------|--------|
| Razonamiento complejo | Claude Opus / GPT-5 | Calidad insustituible |
| Código general | Claude Sonnet | Mejor balance |
| Clasificación masiva | Groq (Llama 4) | 4 a 10x más barato, sub-100ms |
| Batch offline | Together AI / Fireworks | Precio bajo con SLA |
| Data residency EU | Mistral | Único major europeo |
| Compliance EU + costo mínimo | DeepSeek via Azure | Compliance + 15x más barato |

**Una nota sobre self-hosting:** correr modelos open source en GPUs propias (con vLLM por ejemplo) puede ser tentador y aparenta máximo control. La regla práctica documentada: solo se justifica económicamente a partir de **$5K a $10K/mes de consumo sostenido y 70%+ de utilización de GPU**. Por debajo de eso, el costo operativo (GPUs, observabilidad, scaling, fallbacks, on-call) supera el ahorro nominal. Fuente: [morphllm.com](https://www.morphllm.com/llm-api). Para la gran mayoría de los equipos, los providers managed son más baratos en TCO real.

---

## 11. Referencia de métricas verificadas <a id="11-metricas"></a>

Cierre del documento: todas las métricas usadas a lo largo de las secciones, con su fuente y URL para que cualquier número pueda verificarse independientemente. Sirve como anexo para negociaciones, presentaciones internas, propuestas a leadership o cuando alguien cuestiona una cifra.

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Contexto re-enviado = % factura | 62% | LeanOps 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens turno 1 vs turno 50 | 5K a 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Multiplicador output vs input | 4 a 6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Descuento cache reads | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Ahorro app RAG con caching | 88 a 95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Prompt caching automatic vs explicit | Documentado | Anthropic oficial | https://platform.claude.com/docs/en/build-with-claude/prompt-caching |
| Prompt caching break-even | 1 hit | MetaCTO | https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration |
| RAG reducción tokens | hasta 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM calidad/llamadas | 95%/26% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM ahorro vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con augmentation | 75% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Tool search VSCode | hasta 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode | 10% descuento | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top usuarios = spend | 10 a 15% = 60 a 70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic cost | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API | 50% descuento | Anthropic | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching | hasta 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Thinking omitted = ahorro | Ninguno | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking 2000t + 500t output | 5x más caro | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Opus 4.7 tokenizer overhead | hasta 35% más tokens | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| Azure EA solo | 15 a 25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC | 23 a 28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| **Azure OpenAI overhead vs OpenAI directo** | **15 a 40% (avg 22%)** | **TokenMix** | **https://tokenmix.ai/blog/azure-openai-cost** |
| **Azure OpenAI overhead vs OpenAI directo** | **15 a 40%** | **Inference.net** | **https://inference.net/content/azure-openai-pricing-explained/** |
| **Azure OpenAI overhead vs OpenAI directo** | **20 a 40%** | **CloudZero** | **https://www.cloudzero.com/blog/azure-openai-pricing/** |
| **Bedrock Claude overhead vs Anthropic directo** | **20 a 35% efectivo** | **TokenMix Bedrock** | **https://tokenmix.ai/blog/aws-bedrock-pricing** |
| **Bedrock regional endpoints premium** | **+10% sobre global** | **Anthropic docs oficial** | **https://platform.claude.com/docs/en/about-claude/pricing** |
| **Foundry Claude credits exclusion** | **No aplican Azure credits** | **Microsoft Q&A** | **https://learn.microsoft.com/en-us/answers/questions/5851352** |
| **Caso documentado Foundry trap** | **~$13K USD/mes una startup** | **AZ365.ai marzo 2026** | **https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/** |
| **Pricing parity Claude Bedrock/Vertex/Foundry** | **Idéntico al directo** | **Anthropic docs oficial** | **https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry** |
| DeepSeek via Azure markup | +20 a 35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x más barato | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % gasto IT | hasta 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4 a 10x más barato | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K a $10K/mes | morphllm.com | https://www.morphllm.com/llm-api |
| Anthropic pricing oficial mayo 2026 | Sonnet $3/$15, Opus $5/$25, Haiku $1/$5 | Anthropic | https://platform.claude.com/docs/en/about-claude/pricing |
| OpenAI pricing oficial 2026 | GPT-5.4 $2.50/$15, GPT-5.5 $5/$30, Nano $0.05/$0.40 | OpenAI / Finout | https://openai.com/api/pricing/ |
| GenAI Semantic Conventions OTel | Estándar abierto | OpenTelemetry oficial | https://opentelemetry.io/docs/specs/semconv/gen-ai/ |
| Claude Code OTel observability | Documentado | Anthropic oficial | https://code.claude.com/docs/en/agent-sdk/observability |
| Copilot Chat OTel monitoring | Documentado | VSCode oficial | https://code.visualstudio.com/docs/copilot/guides/monitoring-agents |
