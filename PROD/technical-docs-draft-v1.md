# Guía Técnica: Optimización de Tokens en IA
## De configuraciones básicas a arquitectura enterprise

**Audiencia:** Desarrolladores, tech leads y architects de Visma
**Prerequisito:** Familiaridad básica con LLMs, APIs de AI, y entornos de desarrollo

---

## Índice

1. [Fundamentos](#1-fundamentos)
2. [Nivel 0 — Configuraciones inmediatas](#2-nivel-0)
3. [Nivel 1 — Arquitectura de contexto](#3-nivel-1)
4. [Nivel 2 — Selección de modelos](#4-nivel-2)
5. [Nivel 3 — Model routing automático](#5-nivel-3)
6. [Nivel 4 — Infraestructura de costo y gobernanza](#6-nivel-4)
7. [Nivel 5 — Observabilidad con OpenTelemetry](#7-otel)
8. [Nivel 6 — Proveedores alternativos de menor costo](#8-alternativas)
9. [Referencia de métricas verificadas](#9-metricas)

---

## 1. Fundamentos

### 1.1 Anatomía de un request

**Input tokens:** system prompt, historial, tool definitions, documentos, mensaje del usuario.
**Output tokens:** respuesta, razonamiento (extended thinking), tool calls. **Cuestan 5x más.**
**Cache reads:** 10% del precio normal de input.

### 1.2 Precios de referencia 2026

| Modelo | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|------------|-------------|-----------------|-------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| DeepSeek V4-Flash (directo) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |

Fuentes: Anthropic oficial (abril 2026), DeployBase, ToolHalla

> **Nota Opus 4.7:** Nuevo tokenizer que genera hasta 35% más tokens para el mismo input vs Opus 4.6. El precio por token no cambió, pero el costo efectivo por request puede ser hasta 35% mayor. Benchmarkear antes de migrar.
> Fuente: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 El contexto acumulado

Turno 1: 5K tokens. Turno 10: ~15K. Turno 50: **200K tokens por llamada.**
**El 62% de la factura de AI es contexto re-enviado.** (LeanOps, 30 equipos, 2026)

### 1.4 Extended thinking: el costo invisible

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
500 tokens output + 2.000 thinking = **5x más caro** que sin thinking. (PECollective, 2026)

---

## 2. Nivel 0 — Configuraciones inmediatas

### 2.1 Auto Mode en GitHub Copilot
Cambiar el selector a "Auto" en VSCode. 10% de descuento automático en el multiplicador. (GitHub Changelog, abril 2026)

### 2.2 chat.tools.compressOutput.enabled
```json
{ "chat.tools.compressOutput.enabled": true }
```
Post-procesa output de terminal: colapsa diffs sin cambios, descarta lockfiles, reduce `ls -l` a nombres, elimina barras de progreso. VSCode 1.120.

### 2.3 Tool search — MCPs diferidos
Default para Anthropic (Sonnet 4.5+). Para GPT:
```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```
Hasta 20% ahorro en tokens/request. (Visual Studio Magazine, 2026)

### 2.4 Agent Debug Log Panel — El trace completo dentro de VSCode

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

### 2.5 Chat Debug View — Inspección request a request

Muestra el detalle de cada request individual: system prompt completo enviado, user prompt, contexto incluido (archivos, snippets, historial), y respuesta del modelo.

**Cómo abrir:** Menú `(...)` → **"Show Chat Debug View"** o Command Palette → **"Developer: Show Chat Debug View"**

Fuente: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, mayo 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 2.6 Token usage visibility para BYOK (VSCode 1.120)
Corrige el token accounting para modelos con API key propia — antes siempre mostraba 0%.

### 2.7 Auditar MCPs activos
10 MCPs × 500 tokens = 5.000 tokens overhead/request = **$1.500/mes** en 100K requests a Sonnet sin usar nada.

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
    model="claude-sonnet-4-6", max_tokens=1024,
    system=[
        {"type": "text", "text": "Sos un asistente de Visma...",
         "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": knowledge_base_content,
         "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": user_message}]
)
print(f"Cache read: {response.usage.cache_read_input_tokens}")
```
App RAG 50K tokens × 1.000 queries/día → **84% de ahorro**. (Finout, 2026)

### 3.2 Gestión de historial

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

### 3.3 RAG y output budgets
```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```
RAG reduce tokens hasta 70% vs context stuffing. (Koombea, 2026)

---

## 4. Nivel 2 — Selección y asignación de modelos

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

## 5. Nivel 3 — Model routing automático

### 5.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)
95% calidad GPT-4 con 26% llamadas → 48% ahorro. Con augmentation: **75% reducción**.

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

### 5.2 AI Gateways enterprise

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

## 6. Nivel 4 — Infraestructura de costo y gobernanza

### 6.1 Batch API (50% descuento, calidad idéntica)

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
Con caching: hasta **95% de ahorro**. (PECollective, 2026)

### 6.2 DeepSeek via Azure
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
~15x más barato que Sonnet con compliance europeo. (DeployBase, 2026)

### 6.3 Observabilidad y alertas
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

## 7. Nivel 5 — Observabilidad con OpenTelemetry

### 7.1 OTel en Claude Code (CLI)
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

### 7.2 OTel en Copilot Chat
```json
{ "github.copilot.chat.otel.enabled": true, "github.copilot.chat.otel.endpoint": "http://localhost:4317" }
```

### 7.3 Stack local en 30 segundos
```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```
Abrir http://localhost:18888 — trazas en tiempo real.

### 7.4 Stack producción (Grafana + Prometheus)
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

### 7.5 Configuración centralizada (MDM)
```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.visma.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

---

## 8. Nivel 6 — Proveedores alternativos de menor costo

Los precios varían **625x**. La mayoría paga 4–30x más de lo necesario.

### Groq — LPU hardware, velocidad extrema
```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```
Llama 4 ~$0.11/MTok. 4–10x más barato que GPT-4o. Sub-100ms.

### Together AI — 100+ modelos
```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, 500))
```

### Mistral — El único provider major europeo
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

## 9. Referencia de métricas verificadas

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
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x más barato | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % gasto IT | hasta 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4–10x más barato | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K–10K/mes | morphllm.com | https://www.morphllm.com/llm-api |
