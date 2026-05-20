# Guía Técnica: Optimización de Tokens en IA
## De configuraciones básicas a arquitectura enterprise

**Audiencia:** Desarrolladores, tech leads y architects
**Prerequisito:** Familiaridad básica con LLMs, APIs de AI, y entornos de desarrollo

---

## Índice

1. [Fundamentos](#1-fundamentos)
2. [Dónde vive cada palanca de optimización](#2-donde-vive)
3. [Vendor directo vs hyperscaler: la decisión de procurement](#3-procurement)
4. [Nivel 0 — Configuraciones inmediatas](#4-nivel-0)
5. [Nivel 1 — Arquitectura de contexto](#5-nivel-1)
6. [Nivel 2 — Selección de modelos](#6-nivel-2)
7. [Nivel 3 — Model routing automático](#7-nivel-3)
8. [Nivel 4 — Infraestructura de costo y gobernanza](#8-nivel-4)
9. [Nivel 5 — Observabilidad con OpenTelemetry](#9-otel)
10. [Nivel 6 — Proveedores alternativos de menor costo](#10-alternativas)
11. [Referencia de métricas verificadas](#11-metricas)

---

## Convención visual para los scripts

Cada bloque de código de aplicación lleva una micro-cabecera para que sepas dónde vive y quién lo toca:

> 🔧 **Capa X** (descripción del lugar físico) · **Tecnología** · **Quién lo toca**

Por ejemplo:
> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a `api.anthropic.com`

Los snippets sin esta cabecera son **settings de IDE** (VSCode, Copilot) — cualquier dev individual los puede tocar, no necesitan apps custom.

---

## 1. Fundamentos

### 1.1 Anatomía de un request

**Input tokens:** system prompt, historial, tool definitions, documentos, mensaje del usuario.
**Output tokens:** respuesta, razonamiento (extended thinking), tool calls. **Cuestan 5x más.**
**Cache reads:** 10% del precio normal de input.

### 1.2 Precios de referencia 2026

| Modelo | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|-------------|---------------|--------------------|---------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| GPT-5.4 | $2.50 | $15.00 | $0.25 | — |
| GPT-5.5 (flagship) | $5.00 | $30.00 | $0.50 | — |
| GPT-5 Nano | $0.05 | $0.40 | — | — |
| DeepSeek V4-Flash (directo) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |

Fuentes: [Anthropic oficial mayo 2026](https://platform.claude.com/docs/en/about-claude/pricing), [OpenAI oficial](https://openai.com/api/pricing/), [Finout 2026](https://www.finout.io/blog/openai-pricing-in-2026), [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing), [ToolHalla](https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026)

> **Nota Opus 4.7:** Nuevo tokenizer que genera hasta 35% más tokens para el mismo input vs Opus 4.6. El precio por token no cambió, pero el costo efectivo por request puede ser hasta 35% mayor. Benchmarkear antes de migrar.
> Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 El contexto acumulado

Turno 1: 5K tokens. Turno 10: ~15K. Turno 50: **200K tokens por llamada.**
**El 62% de la factura de AI es contexto re-enviado.** ([LeanOps, 30 equipos, 2026](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/))

### 1.4 Extended thinking: el costo invisible

> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a `api.anthropic.com/v1/messages`

```python
# INCORRECTO — display: omitted NO reduce el costo
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "enabled", "budget_tokens": 5000, "display": "omitted"}
    # Los 5000 tokens se siguen facturando como output
)
# CORRECTO — limitar el budget
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```
500 tokens output + 2.000 thinking = **5x más caro** que sin thinking. ([PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/))

---

## 2. Dónde vive cada palanca de optimización <a id="2-donde-vive"></a>

Esta es la sección que conviene leer **antes** de tocar cualquier configuración. La mayoría de los equipos pierde tiempo buscando palancas en el lugar equivocado: intentan optimizar desde un settings del IDE algo que solo se configura en código de aplicación, o piensan que un cambio en el código va a afectar a un usuario que solo usa Claude.ai.

Hay cuatro lugares físicos donde se toca optimización de tokens. Cada uno tiene una audiencia distinta dentro del equipo, sus propios parámetros, y su propio techo de impacto.

### 2.1 Las cuatro capas de control

| Capa | Lugar físico | Quién lo toca | Palancas que viven aquí | Techo de ahorro |
|------|---------------|----------------|---------------------------|-------------------|
| **Capa 1** | Código de aplicación (Python/JS/etc) que llama directo a la API | Developers que construyen apps internas con LLM | Prompt caching, thinking budgets, max_tokens, modelo por tarea, batch API, output budgets, headers de beta | **Hasta 95% combinado** |
| **Capa 2** | Settings de IDE (VSCode, Copilot, Claude Code, Cursor) | Cualquier dev individual o admin vía MDM | Auto Mode, compressOutput, tool search, debug panels, OTel local | **Hasta 30% combinado** |
| **Capa 3** | Consola de billing / admin de plataforma | Lead técnico, admin del tenant | Modelos permitidos/bloqueados, presupuestos por workspace, alertas, audit logs | **Indirecto: gobernanza** |
| **Capa 4** | Gateway / routing layer (LiteLLM, Portkey, OpenRouter, RouteLLM) | Plataforma / infra | Routing automático entre modelos, fallbacks, retries, presupuestos por equipo, guardrails | **20–80% según [IDC](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing)** |

### 2.2 El paradigma anterior: vendor cerrado vs API key

Antes que cualquiera de las cuatro capas, hay una decisión binaria que define a cuáles capas tenés acceso:

**Si consumís IA como producto cerrado** (Claude.ai, ChatGPT, Copilot, Cursor):
- Tenés acceso a Capa 2 (settings de IDE) y a una versión limitada de Capa 3 (config del producto)
- **No tenés acceso a Capa 1**: no podés tocar caching, thinking, max_tokens, batch, ni ningún parámetro de la llamada
- **No tenés acceso a Capa 4**: el routing lo decide el proveedor (Auto Mode en Copilot es lo más cercano)
- Techo total realista: ~30% de ahorro sobre el consumo no optimizado

**Si construís con API key** (Anthropic, OpenAI, etc., consumidos desde código propio):
- Tenés acceso a las cuatro capas
- Las palancas grandes (caching 90%, batch 50%, routing 20–80%) viven en Capa 1 y Capa 4
- Techo total realista: 90%+ de ahorro sobre el consumo no optimizado

**Este documento asume que estás construyendo, o yendo hacia ahí.** Las capas que siguen dan por sentado que tu equipo escribe código de aplicación contra una API.

### 2.3 Cómo se conecta cada script con su capa

Para que sea fácil ubicar visualmente cada bloque de código en su lugar:

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

Hay una observación operativa importante sobre la Capa 1: si tu equipo tiene 10 microservicios distintos llamando directo al SDK de Anthropic, cada uno decide por su cuenta thinking, caching, max_tokens, modelo, etc. Vas a tener 10 niveles distintos de "optimización" y una factura impredecible.

Las dos formas de gobernarlo:

**Opción A — Wrapper interno compartido**: una librería interna que envuelve el SDK, con defaults sanos hardcodeados (`adaptive thinking effort=low`, modelo Haiku salvo override explícito, prompt caching activo por defecto, max_tokens según el tipo de tarea). Todos los servicios consumen el wrapper, no el SDK directo. Una sola decisión, replicada en todos lados.

**Opción B — Gateway compartido (LiteLLM/Portkey)**: las llamadas pasan por un gateway que aplica policy: "si nadie especificó thinking, forzar low", "si nadie especificó modelo, usar Haiku", "si el costo del request supera $X, alertar". El código de los servicios no cambia — el gateway intercepta y normaliza.

Para equipos hasta ~5 servicios, wrapper alcanza. A partir de ahí, gateway empieza a justificarse por el budget control multi-equipo.

---

## 3. Vendor directo vs hyperscaler: la decisión de procurement <a id="3-procurement"></a>

Esta sección documenta los datos verificados que conviene tener encima de la mesa cuando se evalúa o se negocia un acuerdo de consumo de LLM con un proveedor.

### 3.1 La estructura del precio

Cuando se consume un LLM con API key, hay tres elementos en el costo total:

1. **Precio nominal por token**: lo que figura en la pricing page del modelo
2. **Modificadores de request**: caching, batch, long context, fast mode, data residency
3. **Overhead de proveedor**: support plans, data transfer, hosting de fine-tuned models, monitoring infra

El precio nominal es donde **la mayoría de las empresas comparan** y donde aparentemente "el hyperscaler con descuento gana". El overhead es donde la cuenta se da vuelta.

### 3.2 Pricing parity nominal entre proveedores (mayo 2026)

Anthropic mantiene **pricing parity per-token** entre su API directa y los hyperscalers principales:

| Modelo Claude | Anthropic directo | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|----------------|---------------------|----------------------|--------------------|---------------------|
| Sonnet 4.6 | $3 / $15 | $3 / $15 | $3 / $15 | $3 / $15 |
| Opus 4.7 | $5 / $25 | $5 / $25 | $5 / $25 | $5 / $25 |
| Haiku 4.5 | $1 / $5 | $1 / $5 | $1 / $5 | $1 / $5 |

Fuente: [Anthropic Pricing Docs oficial](https://platform.claude.com/docs/en/about-claude/pricing), [Anthropic Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)

**Multiplicadores explícitos publicados por Anthropic:**
- Bedrock regional endpoints: +10% sobre global
- Vertex regional/multi-region: +10% sobre global
- Claude API directa con `inference_geo: "us"`: +10% (1.1x multiplier)

Lo mismo pasa con OpenAI vs Azure OpenAI: pricing per-token idéntico.

### 3.3 TCO efectivo: el overhead real

**Azure OpenAI vs OpenAI directo — 15–40% overhead efectivo, avg +22%**

Tres fuentes independientes 2026 coinciden:

| Fuente | Rango documentado | Cita textual |
|--------|---------------------|---------------|
| [TokenMix mayo 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15–40%, avg 22% | "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [Inference.net enero 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15–40% | "Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [CloudZero mayo 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20–40% | "Production deployments typically add 20-40% above listed token rates." |

Componentes del overhead Azure:
- **Support plans**: $100–$1.000+/mes (Standard support es de facto obligatorio para producción)
- **Data egress**: gratis primeros 100GB outbound, después $0.087/GB
- **Fine-tuned model hosting**: $1.70–$3/hora **aun sin uso** = $1.224–$2.160/mes por modelo
- **VNet integration, Private Link, content filtering**: $200–$2.000/mes según config
- **Log Analytics y monitoring infra**: variable

**AWS Bedrock para Claude — 20–35% overhead efectivo**

Pricing nominal idéntico a Anthropic directo, pero ([TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)):
- Regional endpoints: +10% (oficial Anthropic)
- Cross-region inference: +10% adicional
- Bedrock cuenta data transfer (egress)
- Bedrock factura todos los errores HTTP 500 (la API directa tiene 3% forgiveness buffer)
- CloudWatch / CloudTrail logging mandatorio

Cita textual: *"TokenMix.ai cost tracking shows enterprises running Claude on Bedrock pay an average of 20-35% more than those using Anthropic's direct API — and most do not realize it."*

**Microsoft Foundry para Claude — pricing idéntico pero trampa documentada**

Claude se factura en Foundry como Marketplace de terceros, no como recurso nativo de Azure. Implicación crítica documentada por [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) y [AZ365.ai marzo 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> "Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."

**Casos reales documentados** ([The Register / AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/)):
- Founder japonés (Leach): ¥237.081 (~$1.600 USD) cargados directo a tarjeta, credits no se aplicaron
- Founder alemán (Bogdan Sevriukov): €999.60 cargados directo
- Otro founder japonés: ¥2.000.000 (~$13.000 USD) en un mes
- Caso adicional reportado por The Register: ~$3.000 USD

Si tu empresa está negociando un acuerdo Microsoft asumiendo que el MACC absorbe el consumo de Claude vía Foundry, **pedir aclaración escrita explícita** sobre la elegibilidad de credits para Claude.

### 3.4 Descuentos enterprise reales

| Tipo de descuento | Rango documentado | Fuente |
|--------------------|--------------------|---------|
| Azure EA solo | 15–25% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Azure EA + MACC | 23–28% negociado | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Anthropic Enterprise / volume | Disponible, custom | [CloudZero](https://www.cloudzero.com/blog/claude-pricing/) |
| OpenAI Enterprise tier | Custom pricing | [Finout, OpenAI](https://www.finout.io/blog/openai-pricing-in-2026) |
| AWS Bedrock private offers | Disponible | Anthropic docs |

### 3.5 La cuenta neta: descuento vs overhead

Suponiendo workload base = 100 unidades de costo en vendor directo:

| Escenario | Costo directo | Costo Azure | Costo Bedrock | Neto vs directo |
|-----------|----------------|---------------|----------------|-------------------|
| GPT-5.4, sin compliance, sin descuento | 100 | 115–140 | n/a | **+15% a +40%** |
| GPT-5.4, EA solo (-20%) sobre 115–140 | 100 | 92–112 | n/a | **-8% a +12%** |
| GPT-5.4, EA + MACC (-25%) sobre 115–140 | 100 | 86–105 | n/a | **-14% a +5%** |
| Claude Sonnet, sin compliance | 100 | n/a (Foundry sin desc.) | 120–135 | **+20% a +35%** |
| DeepSeek vs Azure (sin compliance) | 100 | 120–135 | n/a | **+20% a +35%** |
| DeepSeek vs Azure (con GDPR EU) | inviable | 120–135 | n/a | **prima de compliance** |

**Conclusión:** el descuento EA/MACC típicamente compensa pero no supera el overhead. La factura final queda entre -14% y +12% vs el vendor directo. La decisión real **se gana o se pierde por compliance**, no por descuento.

### 3.6 Cuándo el hyperscaler vale la pena (matriz de decisión)

| Criterio dominante | Camino recomendado |
|---------------------|----------------------|
| Modelo del propio vendor enterprise-maduro (Anthropic, OpenAI), sin mandato Azure-only | **Vendor directo** — features primero, mejor precio neto, soporte de fábrica |
| Modelo geopolíticamente sensible (DeepSeek, Qwen, modelos chinos) | **Hyperscaler (Azure preferido)** — la prima compra compliance |
| Mandato corporativo "todo en Azure/AWS" | **Hyperscaler** — asumir overhead 15–40% como costo de gobernanza |
| Necesidad de MACC burn / facturación unificada Microsoft | **Hyperscaler** — decisión contable, **excluir Claude en Foundry** |
| Modelo open-source self-hosted (Llama, Mistral) | **Hyperscaler con managed endpoint o self-host** — Bedrock/Vertex o GPUs propias |
| Equipo en exploración, prototipos, POCs | **Vendor directo** — onboarding rápido, menos burocracia |
| Compliance GDPR / data residency obligatorio | **Hyperscaler** o **Mistral** (única opción major europea directa) |

### 3.7 Input para negociación

Datos verificados para llevar a la mesa con cualquier vendor (Microsoft, AWS, Google, OpenAI, Anthropic):

| Dato | Valor | Fuente | Uso en negociación |
|------|-------|--------|---------------------|
| Azure overhead vs OpenAI directo | 15–40% (avg 22%) | TokenMix, Inference.net, CloudZero | "Necesitamos descuento que cubra al menos 25% para break-even contra directo" |
| Bedrock overhead vs Anthropic directo | 20–35% efectivo | TokenMix Bedrock | "Foundry o directo nos da el mismo precio sin ese markup" |
| Foundry Claude credits exclusion | Documentado | Microsoft Q&A, AZ365.ai | "Aclaración escrita explícita sobre qué credits aplican a Claude" |
| Anthropic Enterprise volume discount | Disponible | CloudZero | "Solicitar términos comparables a hyperscaler con MACC" |
| OpenAI Enterprise custom pricing | Disponible | Finout | "Solicitar mismo régimen que Azure pero sin overhead" |
| Pricing parity Claude en hyperscalers | Confirmado | Anthropic docs | "El hyperscaler no nos da mejor precio nominal — solo procurement" |

---

## 4. Nivel 0 — Configuraciones inmediatas <a id="4-nivel-0"></a>

> 🔧 **Capa 2** (settings de IDE) · Todos los snippets de esta sección son JSON de VSCode `settings.json` · Cualquier dev individual los puede activar

### 4.1 Auto Mode en GitHub Copilot
Cambiar el selector a "Auto" en VSCode. 10% de descuento automático en el multiplicador. ([GitHub Changelog, abril 2026](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/))

### 4.2 chat.tools.compressOutput.enabled
```json
{ "chat.tools.compressOutput.enabled": true }
```
Post-procesa output de terminal: colapsa diffs sin cambios, descarta lockfiles, reduce `ls -l` a nombres, elimina barras de progreso. VSCode 1.120.

### 4.3 Tool search — MCPs diferidos
Default para Anthropic (Sonnet 4.5+). Para GPT:
```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```
Hasta 20% ahorro en tokens/request. ([Visual Studio Magazine, 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx))

### 4.4 Agent Debug Log Panel — El trace completo dentro de VSCode

La forma más directa de ver qué está pasando en una sesión de agente sin configurar ningún backend externo. Muestra tokens consumidos, tool calls ejecutadas, model turns, subagentes, errores y un **flow chart visual del agente** — todo renderizado directamente en el editor.

**Cómo abrir:**
- Menú overflow `(...)` en el panel de Copilot Chat → **"Show Agent Debug Logs"**
- O Command Palette (`Cmd/Ctrl+Shift+P`) → **"Developer: Open Agent Debug Logs"**

**Requiere habilitar:**
```json
{ "github.copilot.chat.agentDebugLog.enabled": true }
```

**Qué muestra por sesión:**
- **Overview:** model turns totales, tool calls ejecutadas, total tokens consumidos, errores
- **View Logs:** lista cronológica de todos los eventos con timestamps y detalles. Filtración por tipo
- **Agent Flow Chart:** grafo visual del flujo de ejecución. Muestra cómo se relacionan model turns, tool calls y subagentes anidados

Fuente: [VSCode Docs — Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

**Persistir sesiones pasadas en disco:**
```json
{ "github.copilot.chat.agentDebugLog.fileLogging.enabled": true }
```
Con esto activo podés revisar cualquier sesión anterior — tool calls, LLM requests, token usage, errores.
Fuente: [dvlprlife.com — Quick Tips: Debug Copilot Agent Session Logs, mayo 2026](https://www.dvlprlife.com/2026/05/quick-tips-debug-copilot-agent-session-logs-in-vs-code/)

**Export a OTLP JSON:** Cada sesión puede exportarse para compartir, archivar o ingestar en cualquier backend OTel (Jaeger, Grafana, Langfuse, etc.). Click en el ícono Export (download) en el toolbar del panel.

**El comando /troubleshoot:** Con file logging habilitado, podés pedirle al mismo Copilot que analice sus propios logs:
```
/troubleshoot how many tokens did I use?
/troubleshoot why did it skip that tool?
```
Fuente: [VSCode Docs — Troubleshoot AI in Visual Studio Code](https://code.visualstudio.com/docs/copilot/troubleshooting)

### 4.5 Chat Debug View — Inspección request a request

Muestra el detalle de cada request individual: system prompt completo enviado, user prompt, contexto incluido (archivos, snippets, historial), y respuesta del modelo.

**Cómo abrir:** Menú `(...)` → **"Show Chat Debug View"** o Command Palette → **"Developer: Show Chat Debug View"**

Fuente: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, mayo 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 4.6 Token usage visibility para BYOK (VSCode 1.120)
Corrige el token accounting para modelos con API key propia — antes siempre mostraba 0%.

### 4.7 Auditar MCPs activos
10 MCPs × 500 tokens = 5.000 tokens overhead/request = **$1.500/mes** en 100K requests a Sonnet sin usar nada.

---

## 5. Nivel 1 — Arquitectura de contexto <a id="5-nivel-1"></a>

### 5.1 Prompt caching

| Operación | Costo |
|-----------|-------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hora | 2.0x base |
| Cache read | 0.10x base (90% descuento) |

> 🔧 **Capa 1** (código de app) · SDK Anthropic Python · Developers de aplicación que llaman directo a la API y diseñan el shape del prompt

```python
response = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
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
App RAG 50K tokens × 1.000 queries/día → **84% de ahorro**. ([Finout, 2026](https://www.finout.io/blog/anthropic-api-pricing))

### 5.2 Gestión de historial

> 🔧 **Capa 1** (código de app) · Helpers Python sobre el SDK · Developers de aplicación que diseñan el manejo de sesiones agénticas

**Ventana deslizante:**
```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**Resumen progresivo** (usar Haiku para resumir):
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

**Compaction API** (recomendada para agentes, beta enero 2026, elegible ZDR):
```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"], model="claude-sonnet-4-6", max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112", "context_token_threshold": 50_000}]}
)
```

### 5.3 RAG y output budgets

> 🔧 **Capa 1** (código de app) · Pipeline RAG + diccionario de budgets · Developers de aplicación que diseñan el flujo de retrieval y los topes por tipo de tarea

```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```
RAG reduce tokens hasta 70% vs context stuffing. ([Koombea, 2026](https://ai.koombea.com/blog/llm-cost-optimization))

---

## 6. Nivel 2 — Selección y asignación de modelos <a id="6-nivel-2"></a>

> 🔧 **Capa 1** (código de app) o **Capa 4** (en gateway si se centraliza) · Diccionario de asignación + router clasificador · Developers de aplicación, o plataforma si se mueve al gateway compartido

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

---

## 7. Nivel 3 — Model routing automático <a id="7-nivel-3"></a>

### 7.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)
95% calidad GPT-4 con 26% llamadas → 48% ahorro. Con augmentation: **75% reducción**. ([LMSYS ICLR 2025](https://www.lmsys.org/blog/2024-07-01-routellm/))

> 🔧 **Capa 4** (gateway / routing layer) · Open source UC Berkeley/LMSYS · Plataforma o equipo de infra que pone una capa entre los servicios y los proveedores

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

### 7.2 AI Gateways enterprise

> 🔧 **Capa 4** (gateway compartido) · LiteLLM self-hosted / Portkey managed · Plataforma o equipo de infra. El código de los servicios consumidores NO cambia — el gateway intercepta

```python
# LiteLLM — self-hosted, budget controls
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"}, "tpm": 100000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"}, "tpm": 100000}
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)

# Portkey — managed, compliance
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6", messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

| Gateway | Tipo | Mejor para |
|---------|------|------------|
| LiteLLM | Open source | Control total, zero fees |
| Portkey | Managed | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | Experimentación rápida |

---

## 8. Nivel 4 — Infraestructura de costo y gobernanza <a id="8-nivel-4"></a>

### 8.1 Batch API (50% descuento, calidad idéntica)

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
Con caching: hasta **95% de ahorro**. ([PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/))

### 8.2 DeepSeek via Azure

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

**Contexto de procurement:** ~15x más barato que Sonnet en precio nominal, con compliance europeo asegurado. El acceso directo a DeepSeek (`api.deepseek.com`) no es viable para empresas con datos EU porque rutea todos los requests por servidores chinos. Azure agrega un markup de **20–35% sobre el directo** ([DeployBase, 2026](https://deploybase.ai/articles/deepseek-v3-pricing)), pero ese markup es **prima de compliance, no overhead**: los datos quedan en la región Azure que elegiste, el contrato es con Microsoft, y el compliance europeo está cubierto. Aun con ese 20–35% encima, DeepSeek vía Azure sigue costando ~1/15 lo que Sonnet, así que para clasificación masiva, extracción estructurada o batch processing offline, sigue siendo la mejor relación precio/compliance del mercado.

**Nota crítica:** este es uno de los pocos casos donde el hyperscaler es la elección correcta por razones de modelo, no de procurement general. Para Claude o GPT, el mismo cálculo no aplica (ver sección 3 — pricing parity nominal).

### 8.3 Observabilidad y alertas

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

---

## 9. Nivel 5 — Observabilidad con OpenTelemetry <a id="9-otel"></a>

### 9.1 OTel en Claude Code (CLI)

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

### 9.2 OTel en Copilot Chat
```json
{ "github.copilot.chat.otel.enabled": true, "github.copilot.chat.otel.endpoint": "http://localhost:4317" }
```

### 9.3 Stack local en 30 segundos

> 🔧 **Capa 2** (estación de trabajo del dev) · Docker container del Aspire Dashboard · Dev individual para uso local sin cuenta cloud

```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```
Abrir http://localhost:18888 — trazas en tiempo real.

### 9.4 Stack producción (Grafana + Prometheus)

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

### 9.5 Configuración centralizada (MDM)

> 🔧 **Capa 3** (managed settings enterprise) · JSON distribuible vía MDM (Intune, Jamf) · Admin de plataforma de developers

```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

---

## 10. Nivel 6 — Proveedores alternativos de menor costo <a id="10-alternativas"></a>

Los precios varían **625x**. La mayoría paga 4–30x más de lo necesario.

### Groq — LPU hardware, velocidad extrema

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan latencia sub-100ms en tareas simples

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```
Llama 4 ~$0.11/MTok. 4–10x más barato que GPT-4o. Sub-100ms.

### Together AI — 100+ modelos

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan batch high-volume con fine-tuning

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, 500))
```

### Mistral — El único provider major europeo

> 🔧 **Capa 1** (código de app) · SDK OpenAI con base_url alternativa · Developers de aplicación que necesitan GDPR nativo sin pasar por hyperscaler

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("MISTRAL_API_KEY"), base_url="https://api.mistral.ai/v1")
response = client.chat.completions.create(model="mistral-small-latest",
    messages=messages, max_tokens=200)
```
Sede en Francia, GDPR nativo, datos en la UE.

### Guía de decisión

| Tarea | Provider | Razón |
|-------|----------|--------|
| Razonamiento complejo | Claude Opus / GPT-5 | Calidad insustituible |
| Código general | Claude Sonnet | Mejor balance |
| Clasificación masiva | Groq (Llama 4) | 4–10x más barato, sub-100ms |
| Batch offline | Together AI / Fireworks | Precio bajo con SLA |
| Data residency EU | Mistral | Único major europeo |
| Compliance EU + costo mínimo | DeepSeek via Azure | Compliance + 15x más barato |

Self-hosting con vLLM: solo a partir de $5K–10K/mes y 70%+ GPU utilization.

---

## 11. Referencia de métricas verificadas <a id="11-metricas"></a>

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Contexto re-enviado = % factura | 62% | LeanOps 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens turno 1 vs turno 50 | 5K → 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Multiplicador output vs input | 4–6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Descuento cache reads | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Ahorro app RAG con caching | 88–95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| RAG reducción tokens | hasta 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM calidad/llamadas | 95%/26% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM ahorro vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con augmentation | 75% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Tool search VSCode | hasta 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode | 10% descuento | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top usuarios = spend | 10–15% = 60–70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic cost | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API | 50% descuento | Anthropic | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching | hasta 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Thinking omitted = ahorro | Ninguno | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking 2000t + 500t output | 5x más caro | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Opus 4.7 tokenizer overhead | hasta 35% más tokens | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| Azure EA solo | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| **Azure OpenAI overhead vs OpenAI directo** | **15–40% (avg 22%)** | **TokenMix** | **https://tokenmix.ai/blog/azure-openai-cost** |
| **Azure OpenAI overhead vs OpenAI directo** | **15–40%** | **Inference.net** | **https://inference.net/content/azure-openai-pricing-explained/** |
| **Azure OpenAI overhead vs OpenAI directo** | **20–40%** | **CloudZero** | **https://www.cloudzero.com/blog/azure-openai-pricing/** |
| **Bedrock Claude overhead vs Anthropic directo** | **20–35% efectivo** | **TokenMix Bedrock** | **https://tokenmix.ai/blog/aws-bedrock-pricing** |
| **Bedrock regional endpoints premium** | **+10% sobre global** | **Anthropic docs oficial** | **https://platform.claude.com/docs/en/about-claude/pricing** |
| **Foundry Claude credits exclusion** | **No aplican Azure credits** | **Microsoft Q&A** | **https://learn.microsoft.com/en-us/answers/questions/5851352** |
| **Caso documentado Foundry trap** | **~$13K USD/mes una startup** | **AZ365.ai marzo 2026** | **https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/** |
| **Pricing parity Claude Bedrock/Vertex/Foundry** | **Idéntico al directo** | **Anthropic docs oficial** | **https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry** |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x más barato | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % gasto IT | hasta 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4–10x más barato | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K–10K/mes | morphllm.com | https://www.morphllm.com/llm-api |
| Anthropic pricing oficial mayo 2026 | Sonnet $3/$15, Opus $5/$25, Haiku $1/$5 | Anthropic | https://platform.claude.com/docs/en/about-claude/pricing |
| OpenAI pricing oficial 2026 | GPT-5.4 $2.50/$15, GPT-5.5 $5/$30, Nano $0.05/$0.40 | OpenAI / Finout | https://openai.com/api/pricing/ |
