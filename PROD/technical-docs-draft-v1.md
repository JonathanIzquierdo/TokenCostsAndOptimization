# Guía Técnica: Optimización de Tokens en IA
## De configuraciones básicas a arquitectura enterprise

**Audiencia:** Desarrolladores, tech leads y architects de Visma  
**Objetivo:** Guía práctica con todos los niveles de implementación — desde configuraciones de cinco minutos hasta decisiones de arquitectura para sistemas en producción  
**Prerequisito:** Familiaridad básica con LLMs, APIs de AI, y entornos de desarrollo  

---

## Índice

1. [Fundamentos: entender el modelo de costo](#1-fundamentos)
2. [Nivel 0 — Configuraciones inmediatas (sin código)](#2-nivel-0)
3. [Nivel 1 — Arquitectura de contexto](#3-nivel-1)
4. [Nivel 2 — Selección y asignación de modelos](#4-nivel-2)
5. [Nivel 3 — Model routing automático](#5-nivel-3)
6. [Nivel 4 — Infraestructura de costo y gobernanza](#6-nivel-4)
7. [Nivel 5 — Observabilidad con OpenTelemetry](#7-otel)
8. [Nivel 6 — Proveedores alternativos de menor costo](#8-alternativas)
9. [Referencia de métricas verificadas](#9-metricas)

---

## 1. Fundamentos: entender el modelo de costo

Antes de optimizar, hay que entender exactamente qué se cobra y por qué.

### 1.1 La anatomía de un request de LLM

Cada request tiene tres componentes que se facturan por separado:

**Input tokens:** todo lo que enviás al modelo — system prompt, historial, tool definitions, documentos, mensaje del usuario.

**Output tokens:** todo lo que el modelo genera — respuesta, razonamiento (extended thinking), llamadas a herramientas.

**Cache reads (cuando aplica):** la porción del input servida desde cache.

Regla más importante: **los output tokens cuestan significativamente más que los input tokens**, y los cache reads cuestan una fracción mínima.

### 1.2 Precios de referencia 2026

| Modelo | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|------------|-------------|-----------------|-------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| DeepSeek V4-Flash (directo) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |
| Llama 4 via Together AI | ~$0.18 | ~$0.59 | — | — |

Fuentes: Anthropic oficial (abril 2026), DeployBase, ToolHalla, inference.net

### 1.3 El problema del contexto acumulado

El costo de una sesión larga no es lineal — es cuadrático sin gestión activa:
- Turno 1: 5.000 tokens de input
- Turno 10: ~15.000 tokens
- Turno 50: 200.000 tokens por llamada

Auditoría de 30 equipos en producción (LeanOps, mayo 2026): **el 62% de la factura de AI es contexto re-enviado.**

### 1.4 El multiplicador de output

Output tokens cuestan 5x más que input en Anthropic. Reducir outputs tiene más impacto en el costo que reducir inputs.

**Regla práctica:** siempre setear `max_tokens` explícitamente.

### 1.5 Extended thinking: el costo que no se ve

```python
# INCORRECTO — display omitted NO reduce el costo
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={
        "type": "enabled",
        "budget_tokens": 5000,
        "display": "omitted"  # Los 5000 tokens se siguen facturando
    }
)

# CORRECTO — limitar el budget según complejidad real
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```

**Impacto:** 500 tokens output visible + 2.000 tokens thinking = 5x más caro que sin thinking. (Fuente: PECollective, 2026)

---

## 2. Nivel 0 — Configuraciones inmediatas

### 2.1 Auto Mode en GitHub Copilot

Cambiar el selector a "Auto" en VSCode. Selecciona dinámicamente el modelo más eficiente por request.

**Beneficio verificado:** 10% de descuento en el multiplicador del modelo. (Fuente: GitHub Changelog, abril 2026)

### 2.2 chat.tools.compressOutput.enabled

```json
{ "chat.tools.compressOutput.enabled": true }
```

Post-procesa output de terminal: colapsa diffs sin cambios, descarta lockfile diffs, reduce `ls -l` a nombres, elimina barras de progreso. Disponible desde VSCode 1.120.

### 2.3 Tool search — MCPs diferidos

Activo por defecto para modelos Anthropic (Sonnet 4.5+). Para GPT:

```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```

**Beneficio:** hasta 20% ahorro en tokens por request. (Fuente: Visual Studio Magazine, 2026)

### 2.4 Auditar MCPs activos

10 MCPs × 500 tokens/schema = 5.000 tokens overhead/request = **$1.500/mes** en 100K requests a Sonnet sin usar nada.

---

## 3. Nivel 1 — Arquitectura de contexto

### 3.1 Prompt caching

| Operación | Costo |
|-----------|-------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hora | 2.0x base |
| Cache read | 0.10x base (90% descuento) |

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "Sos un asistente de ingeniería de Visma...",
            "cache_control": {"type": "ephemeral"}
        },
        {
            "type": "text",
            "text": knowledge_base_content,
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": user_message}]
)

usage = response.usage
print(f"Cache creation: {usage.cache_creation_input_tokens}")
print(f"Cache read: {usage.cache_read_input_tokens}")
```

**Impacto:** app RAG 50K tokens × 1000 queries/día → 84% de ahorro. (Fuente: Finout, 2026)

### 3.2 Gestión de historial

**Estrategia A: ventana deslizante**
```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**Estrategia B: resumen progresivo** (usar Haiku para resumir — más barato)
```python
def compress_old_history(messages, recent_window=5):
    if len(messages) <= recent_window:
        return messages
    old = messages[:-recent_window]
    recent = messages[-recent_window:]
    summary = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=500,
        messages=[{"role": "user",
            "content": f"Resumí en 3 párrafos:\n{format_messages(old)}"}]
    ).content[0].text
    return [{"role": "user", "content": f"[Resumen]\n{summary}"},
            {"role": "assistant", "content": "Entendido."}] + recent
```

**Estrategia C: Compaction API (recomendada para agentes)**
```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112",
                                    "context_token_threshold": 50_000}]}
)
if hasattr(response, 'context_compact_usage'):
    before = response.context_compact_usage.tokens_before
    after = response.context_compact_usage.tokens_after
    print(f"Compaction: {before} → {after} tokens")
```

Beta desde enero 2026. Elegible para ZDR. (Fuente: Claude Lab, Anthropic Docs)

### 3.3 RAG y output budgets

```python
# Solo top-K chunks relevantes, no todo el knowledge base
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])

# Output budgets por tipo de tarea
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```

RAG reduce tokens de contexto hasta 70% vs context stuffing. (Fuente: Koombea, 2026)

---

## 4. Nivel 2 — Selección y asignación de modelos

```python
MODEL_ASSIGNMENT = {
    # Haiku — pattern matching, no razonamiento
    "classification": "claude-haiku-4-5",
    "entity_extraction": "claude-haiku-4-5",
    "data_formatting": "claude-haiku-4-5",
    "file_navigation": "claude-haiku-4-5",
    "simple_qa": "claude-haiku-4-5",
    "translation": "claude-haiku-4-5",
    # Sonnet — punto óptimo calidad/costo
    "code_generation": "claude-sonnet-4-6",
    "code_review": "claude-sonnet-4-6",
    "summarization": "claude-sonnet-4-6",
    "general_chat": "claude-sonnet-4-6",
    "debugging": "claude-sonnet-4-6",
    # Opus — solo cuando el razonamiento complejo lo justifica
    "architecture_design": "claude-opus-4-7",
    "complex_reasoning": "claude-opus-4-7",
    "agent_coordination": "claude-opus-4-7",
}

def classify_and_route(user_input, client):
    """Usa Haiku para clasificar — barato y rápido."""
    result = client.messages.create(
        model="claude-haiku-4-5", max_tokens=50,
        system="Clasificá en: classification, extraction, summarization, code_generation, "
               "code_review, debugging, architecture_design, complex_reasoning, general_chat. "
               "SOLO el nombre de la categoría.",
        messages=[{"role": "user", "content": user_input}]
    )
    task_type = result.content[0].text.strip().lower()
    return task_type, MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
```

---

## 5. Nivel 3 — Model routing automático

### 5.1 RouteLLM (open source, UC Berkeley/LMSYS)

**Resultados ICLR 2025:** 95% calidad GPT-4 con 26% llamadas → 48% ahorro. Con augmentation: 75% reducción de costo.

```bash
pip install routellm
```

```python
from routellm.controller import Controller

client = Controller(
    routers=["mf"],
    strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5",
    config={"mf": {"threshold": 0.3}}
)

response = client.chat.completions.create(
    model="router",
    messages=[{"role": "user", "content": user_message}]
)
print(f"Modelo usado: {response.model}")
```

### 5.2 AI Gateways enterprise

**LiteLLM (self-hosted, open source):**
```python
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"},
         "tpm": 100000, "rpm": 1000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"},
         "tpm": 100000, "rpm": 1000}  # Azure fallback
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)
```

**Portkey (managed, compliance):**
```python
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6",
    messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

| Gateway | Tipo | Modelos | Mejor para |
|---------|------|---------|------------|
| LiteLLM | Open source | 100+ | Control total, zero fees |
| Portkey | Managed | 1.600+ | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | 300+ | Experimentación rápida |

---

## 6. Nivel 4 — Infraestructura de costo y gobernanza

### 6.1 Batch API

**50% descuento, calidad idéntica.** Combinado con caching: hasta 95% de ahorro. (Fuente: Anthropic oficial, PECollective)

```python
batch = client.messages.batches.create(requests=[
    {
        "custom_id": f"doc-{doc.id}",
        "params": {
            "model": "claude-sonnet-4-6", "max_tokens": 200,
            "system": [{"type": "text", "text": system_prompt,
                        "cache_control": {"type": "ephemeral"}}],
            "messages": [{"role": "user", "content": doc.text[:1000]}]
        }
    } for doc in documents
])

# 300K output tokens en Batch (beta)
batch = client.messages.batches.create(
    requests=requests,
    extra_headers={"anthropic-beta": "output-300k-2026-03-24"}
)

# Polling
import time
while client.messages.batches.retrieve(batch.id).processing_status != "ended":
    time.sleep(60)

results = {r.custom_id: r.result.message.content[0].text
           for r in client.messages.batches.results(batch.id)
           if r.result.type == "succeeded"}
```

### 6.2 DeepSeek via Azure

```python
from openai import AzureOpenAI
client = AzureOpenAI(
    api_key=os.getenv("AZURE_API_KEY"),
    api_version="2024-12-01-preview",
    azure_endpoint=os.getenv("AZURE_ENDPOINT")  # Región EU
)
response = client.chat.completions.create(
    model="deepseek-v4-flash", max_tokens=50,
    messages=[{"role": "user", "content": text_to_classify}]
)
```

~15x más barato que Sonnet para tareas de alta volumetría, con compliance europeo. (Fuente: DeployBase, precios 2026)

### 6.3 Observabilidad básica y alertas

```python
OUTPUT_PRICING = {
    "claude-haiku-4-5": {"input": 1e-6, "output": 5e-6,
                          "cache_read": 0.1e-6, "cache_write": 1.25e-6},
    "claude-sonnet-4-6": {"input": 3e-6, "output": 15e-6,
                           "cache_read": 0.3e-6, "cache_write": 3.75e-6},
    "claude-opus-4-7": {"input": 5e-6, "output": 25e-6,
                         "cache_read": 0.5e-6, "cache_write": 6.25e-6},
}

def calculate_cost(model, input_tokens, output_tokens,
                   cache_read=0, cache_write=0):
    p = OUTPUT_PRICING.get(model, {})
    if not p:
        return 0.0
    return round(
        input_tokens * p["input"] + output_tokens * p["output"] +
        cache_read * p["cache_read"] + cache_write * p["cache_write"], 6
    )

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

## 7. Nivel 5 — Observabilidad con OpenTelemetry

La combinación VSCode + Copilot + Claude Code tiene soporte nativo de OpenTelemetry. Sin instrumentarlo, volás a ciegas: ves el total en el billing pero no sabés qué desarrollador, qué workflow, ni qué operación lo genera.

### 7.1 OTel en Claude Code (CLI)

Claude Code tiene telemetría nativa. Está desactivada por defecto — opt-in con una variable de entorno.

**Activación básica:**

```bash
# Agregar a ~/.zshrc o ~/.bashrc para que persista
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_TRACES_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRIC_EXPORT_INTERVAL=10000  # 10 segundos

# Atributos personalizados para filtrar por equipo en el dashboard
export OTEL_RESOURCE_ATTRIBUTES="team.id=platform,department=engineering"
```

Fuentes: Claude Code Docs — Observability, SigNoz Docs, Bindplane Blog 2026

**Por defecto NO se captura el contenido de los prompts** — solo metadata (modelo, tokens, duración, herramientas). Para capturar contenido completo (prompts, respuestas, tool schemas):

```bash
export COPILOT_OTEL_CAPTURE_CONTENT=true
# O en VSCode settings:
# "github.copilot.chat.otel.captureContent": true
```

> Tratar el stream de telemetría como más seguro que logging de prompts crudo, pero no automáticamente seguro para todos los backends. Evaluar las implicaciones de privacidad antes de activar en producción.

Fuente: VSCode Copilot Chat — Monitoring Agents, Microsoft GitHub

### 7.2 OTel en Copilot Chat (VSCode)

Copilot Chat también tiene soporte nativo de OTel desde VSCode 1.118. Exporta traces, métricas y eventos siguiendo las OTel GenAI Semantic Conventions.

**Activar desde VSCode settings:**

```json
// settings.json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.endpoint": "http://localhost:4317"
}
```

**La jerarquía de spans que produce:**

```
client invoke_agent copilot [~15s]
├── chat gpt-4o [~3s]          ← LLM request con tool calls
├── execute_tool readFile [~50ms]
├── execute_tool runCommand [~2s]
└── chat gpt-4o [~4s]          ← Respuesta final
```

Para sesiones de Copilot CLI:

```
copilot-chat invoke_agent copilotcli [~45s]
└── github-copilot invoke_agent [~42s]
    ├── chat claude-sonnet-4.6 [~16s]
    ├── execute_tool task [~18s]
    │   └── invoke_agent task   ← subagente
    └── chat claude-sonnet-4.6 [~4s]
```

Fuente: Microsoft VSCode Copilot Chat — Agent Monitoring Docs, GitHub

### 7.3 Stack de observabilidad local (quickstart)

Para ver trazas sin cuenta en la nube — el Aspire Dashboard de Microsoft es el camino más rápido:

```bash
# Levantar el dashboard — expone UI en :18888 y endpoint OTLP en :4317
docker run --rm -d \
  -p 18888:18888 \
  -p 4317:18889 \
  --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

Abrir http://localhost:18888 — las trazas de Copilot y Claude Code aparecen en tiempo real.

Fuente: Microsoft VSCode Copilot Chat — Agent Monitoring Docs

### 7.4 Stack completo con Grafana (producción)

```yaml
# docker-compose.yml — OTel Collector + Prometheus + Grafana
version: '3.8'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config", "/etc/otel/config.yaml"]
    volumes:
      - ./otel-config.yaml:/etc/otel/config.yaml
    ports:
      - "4317:4317"   # gRPC
      - "4318:4318"   # HTTP
    depends_on: [prometheus]

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data/prometheus:/prometheus
    command:
      - "--storage.tsdb.retention.time=90d"
      - "--config.file=/etc/prometheus/prometheus.yml"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    depends_on: [prometheus]
```

```yaml
# otel-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  prometheus:
    endpoint: "0.0.0.0:9464"
  logging:
    verbosity: detailed

service:
  pipelines:
    metrics:
      receivers: [otlp]
      processors: [batch]
      exporters: [prometheus]
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [logging]
```

Fuente: AI.cc — Claude Code Monitor Tutorial, Sealos Blog — Claude Code Metrics

### 7.5 Configuración enterprise centralizada (admin-managed)

Para equipos donde no querés que cada desarrollador configure su propio endpoint:

```json
// Archivo de managed settings — distribuible via MDM
// Alta precedencia — los usuarios no pueden sobreescribirlo
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.visma.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>",
    "OTEL_METRIC_EXPORT_INTERVAL": "10000"
  }
}
```

Distribuible via MDM (Mobile Device Management) o herramientas de gestión de dispositivos.

Fuente: SigNoz Docs — Claude Code Monitoring, Bindplane Blog 2026

### 7.6 Métricas clave que expone el stack

**Métricas de tokens y costo:**
- `gen_ai.client.token.usage` — tokens por tipo (input, output, cache read, cache write)
- `gen_ai.client.operation.duration` — latencia por operación
- Costo estimado por sesión, por modelo, y por usuario

**Atributos de filtrado disponibles:**
- `gen_ai.request.model` — modelo utilizado
- `gen_ai.provider.name` — provider (anthropic, openai, etc.)
- `gen_ai.tool.name` — herramienta ejecutada
- `copilot_chat.edit.source` — dónde se originó el edit
- `team.id`, `department` — custom attributes para filtrar por equipo

**Filtrado por equipo en el dashboard:**

```bash
export OTEL_RESOURCE_ATTRIBUTES="team.id=backend,department=engineering,user.id=jonathan"
```

Fuente: Microsoft VSCode — Monitor Agent Usage with OpenTelemetry

### 7.7 Opciones de backend de observabilidad

| Backend | Tipo | Mejor para |
|---------|------|------------|
| Aspire Dashboard | Local, Docker | Quickstart, desarrollo local |
| Grafana + Prometheus | Self-hosted | Control total, zero costo |
| Jaeger | Self-hosted | Tracing detallado |
| Datadog | Managed | Integración con infra existente |
| Honeycomb | Managed | Exploracón y queries ad-hoc |
| Langfuse | Managed/self-hosted | LLM-specific observabilidad |
| SigNoz | Managed/self-hosted | OTel nativo, open source |
| Azure Monitor | Managed | Ecosistema Microsoft |

Todos soportan OTLP — no hay lock-in. El mismo pipeline de Claude Code funciona con cualquiera.

Fuente: Microsoft VSCode Copilot Chat — Agent Monitoring Docs

---

## 8. Nivel 6 — Proveedores alternativos de menor costo

El mercado de inference APIs en 2026 tiene más de 11 providers en producción. Los precios varían 625x entre el más barato y el más caro. La mayoría de los equipos usa solo OpenAI o Anthropic por defecto y paga entre 4 y 30 veces más de lo necesario para muchas tareas.

> "Defaulting to the newest model without measuring whether the quality difference justifies the cost is how teams end up in quarterly reviews they’d rather skip."

Fuente: CloudZero — LLM API Pricing Comparison 2026

### 8.1 El mapa del mercado en 2026

**Tier 1 — Frontier models (premium):**
OpenAI GPT-5, Anthropic Claude Opus 4.7, Google Gemini 3 Pro. Benchmarks top. Precio alto. Justificados para razonamiento complejo y tareas críticas.

**Tier 2 — Mid-tier models (punto óptimo calidad/costo):**
Claude Sonnet 4.6, GPT-5.4 Mini, Gemini 3 Flash. Para la mayoría del trabajo en producción.

**Tier 3 — Open-weight via inference providers (alta volumetría y bajo costo):**
Llama 4, Mixtral, Qwen 2.5, DeepSeek via providers. Para tareas bien definidas y predecibles donde el prompt es maduro. 4-30x más barato que frontier.

**Self-hosted (elimina costo por token):**
Para equipos con $5K-10K/mes en API spend y capacidad de DevOps. vLLM es el framework estándar. Solo rentable a 70%+ de utilización de GPU.

Fuente: inference.net — LLM API Pricing Comparison 2026, morphllm.com

### 8.2 Inference providers: Groq, Together AI, Fireworks AI

Estos providers no desarrollan modelos propios — corren modelos open-weight (Llama, Mixtral, Qwen, DeepSeek) en su infraestructura y compiten en precio y latencia. Sus APIs son compatibles con el formato de OpenAI, por lo que la migración es un cambio de endpoint.

#### Groq — LPU hardware, velocidad extrema

Groq construyó silicon propio — el Language Processing Unit (LPU) — diseñado específicamente para inferencia de transformers. Logra 500-800 tokens/segundo en Llama 4 vs 50-150 tok/s en GPU estándar.

**Precios representativos (Groq, marzo 2026):**
- Llama 3.1 70B: ~$0.59/MTok input (OpenAI GPT-4o es $2.50/MTok → 4x más caro)
- Llama 4 modelos: $0.07-$0.20/MTok input

Fuente: ToolHalla — Groq vs Together AI vs Fireworks 2026

**Integración (API-compatible con OpenAI):**
```python
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1"
)

response = client.chat.completions.create(
    model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text_to_classify}],
    max_tokens=50
)
```

**Cuándo usar Groq:** tareas sensibles a latencia que necesitan respuesta sub-100ms. Clasificación, extracción, FAQs de usuario final. NO para tareas complejas de razonamiento.

#### Together AI — El marketplace más amplio

Más de 100 modelos open-source, soporte de fine-tuning (LoRA y full fine-tuning), y deployments dedicados.

**Precios representativos (Together AI, 2026):**
- Llama 4 Maverick: ~$0.18/MTok input
- DeepSeek V3.2: ~$0.14/MTok input
- Mixtral 8x7B: ~$0.24/MTok input

Fuente: PkgPulse — Groq vs Together AI vs Fireworks 2026

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("TOGETHER_API_KEY"),
    base_url="https://api.together.xyz/v1"
)

response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages,
    max_tokens=OUTPUT_BUDGETS.get(task_type, 500)
)
```

**Cuándo usar Together:** workloads de batch de alta volumetría, fine-tuning en datos propios, o cuando querés el modelo open-source más nuevo disponible.

#### Fireworks AI — Production-grade con SLAs

Enfocado en producción con SLAs formales, function calling optimizado, y deployments dedicados.

**Precios representativos (Fireworks, 2026):**
- Llama 4 Scout: ~$0.15/MTok input
- DeepSeek V3.2: ~$0.27/MTok input

Fuente: ToolHalla — Groq vs Together AI vs Fireworks 2026

**Cuándo usar Fireworks:** producción que necesita SLA formal, workloads con function calling intensivo, pipelines de imagen.

### 8.3 Mistral — La opción europea

Mistral es el único provider importante con sede en Europa (París). Para organizaciones con requerimientos estrictos de data residency europeo, es la opción más directa.

**Ventajas para Visma:**
- Sede y datos en la UE — GDPR nativo
- Precios significativamente por debajo de OpenAI
- Modelos open-weight disponibles para self-hosting si se necesita control total
- API compatible con el formato de OpenAI

**Precios representativos:**
- Mistral Small: ~$0.10/MTok input (una fracción de GPT-4o)
- Mistral Medium: ~$0.40/MTok input

Fuente: PE Collective — Best OpenAI Alternatives 2026

```python
from openai import OpenAI

client = OpenAI(
    api_key=os.getenv("MISTRAL_API_KEY"),
    base_url="https://api.mistral.ai/v1"
)

response = client.chat.completions.create(
    model="mistral-small-latest",
    messages=messages,
    max_tokens=200
)
```

### 8.4 Google Gemini — El más barato del tier alto

Gemini Flash 2.0 a $0.075/MTok input es el modelo de alta calidad más barato de un provider major en 2026. Con contexto de 1M tokens.

**Cuándo evaluar Gemini:** workloads que necesitan contexto muy largo (repositorios enteros, documentos masivos) donde el precio de Anthropic o OpenAI se vuelve prohibitivo. La integración con Google Workspace puede ser relevante si el equipo ya está en el ecosistema Google.

Fuente: buildmvpfast.com — LLM API Pricing Comparison 2026

### 8.5 Guía de decisión: ¿cuándo migrar de Anthropic/OpenAI?

No es una migración total — es una estrategia de routing. La misma lógica de "modelo correcto para la tarea" aplica entre providers.

| Tarea | Provider recomendado | Razón |
|-------|---------------------|--------|
| Razonamiento complejo, arquitectura | Claude Opus / GPT-5 | Calidad insustituible |
| Trabajo general de código | Claude Sonnet | Mejor balance |
| Clasificación y extracción masiva | Groq (Llama 4) | 4-10x más barato, sub-100ms |
| Batch offline alta volumetría | Together AI / Fireworks | Precio bajo con SLA |
| Requerimiento data residency EU | Mistral | Único provider major EU-based |
| Contexto muy largo (>200K tokens) | Gemini Flash | Mejor precio en contexto largo |
| Compliance europeo + modelo barato | DeepSeek via Azure | Compliance + 15x más barato que Sonnet |

**Regla práctica:** empezar con el free tier de Groq, Together, y Fireworks. Benchmarkar en tu workload real específico. Commitear cuando tenés datos, no opiniones.

Fuente: ToolHalla — Groq vs Together AI vs Fireworks 2026, inference.net 2026

### 8.6 Self-hosting: cuándo tiene sentido

Self-hosting con vLLM elimina completamente el costo por token. Pero tiene condiciones.

**Solo tiene sentido cuando:**
- Gasto en API supera $5K-10K/mes
- El equipo tiene capacidad de DevOps para mantener la infraestructura
- Se puede sostener 70%+ de utilización de GPU (debajo de eso, API providers ganan en costo)
- Se necesita privacidad total o custom fine-tuning en datos propios

**El costo oculto del self-hosting:** DevOps engineering time, GPU reserved instances, monitoring, upgrades de modelos. Para la mayoría de equipos, el punto de equilibrio no se alcanza hasta $10K+/mes en API spend.

Fuente: morphllm.com — LLM API Comparison 2026

---

## 9. Referencia de métricas verificadas

| Métrica | Valor | Fuente | URL |
|---------|-------|--------|-----|
| Contexto re-enviado = % factura | 62% | LeanOps 30 equipos 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens 20 turnos (innecesarios) | 5.000–10.000 | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Tokens turno 1 vs turno 50 | 5K → 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Multiplicador output vs input | 4–6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Descuento cache reads Anthropic | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Ahorro app RAG con caching | 88–95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse rate | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| RAG reducción tokens | hasta 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM calidad/llamadas | 95% calidad / 26% llamadas | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM ahorro vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM con augmentation | 75% reducción | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Tool search VSCode ahorro | hasta 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode descuento | 10% multiplicador | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top usuarios = spend | 10–15% = 60–70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Workflows agénticos costo | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API descuento | 50% input y output | Anthropic oficial | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching combinados | hasta 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Extended thinking display omitted | No ahorra nada | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking 2000t + output 500t | 5x más caro que sin thinking | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Azure EA solo descuento | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC descuento | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x más barato input | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % gasto IT enterprise | hasta 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs OpenAI GPT-4o | 4–10x más barato Llama 4 | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Rango de precios mercado 2026 | $0.08–$25/MTok output | morphllm.com | https://www.morphllm.com/llm-api |
| Self-hosting break-even | $5K–10K/mes en API spend | morphllm.com | https://www.morphllm.com/llm-api |
| Precios cayeron | 60–80% en 12 meses | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
