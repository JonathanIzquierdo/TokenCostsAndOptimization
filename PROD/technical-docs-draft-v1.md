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
7. [Referencia de métricas verificadas](#7-metricas)

---

## 1. Fundamentos: entender el modelo de costo

Antes de optimizar, hay que entender exactamente qué se cobra y por qué. La mayoría de los problemas de costo de AI vienen de no tener este modelo claro.

### 1.1 La anatomía de un request de LLM

Cada request a un modelo de lenguaje tiene tres componentes que se facturan por separado:

**Input tokens:** todo lo que enviás al modelo — system prompt, historial de conversación, tool definitions, documentos de contexto, y el mensaje actual del usuario.

**Output tokens:** todo lo que el modelo genera — la respuesta, el razonamiento (en modelos con extended thinking), y las llamadas a herramientas.

**Cache reads (cuando aplica):** la porción del input que se sirve desde cache en lugar de procesarse nuevamente.

La regla más importante: **los output tokens cuestan significativamente más que los input tokens**, y los cache reads cuestan una fracción mínima del input normal.

### 1.2 Precios de referencia 2026

| Modelo | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|------------|-------------|-----------------|-------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| DeepSeek V4-Flash (directo) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |

Fuentes: Anthropic oficial (verificado abril 2026), DeployBase (DeepSeek via Azure markup estimado 35%)

### 1.3 El problema del contexto acumulado

El costo real de una sesión larga no es lineal — es cuadrático sin gestión activa.

**Sin gestión de contexto:**
- Turno 1: pagás 5.000 tokens de input
- Turno 10: pagás ~15.000 tokens de input (historial de los 9 anteriores + nuevo mensaje)
- Turno 50: podés estar pagando 200.000 tokens de input por llamada

Una auditoría de 30 equipos en producción (LeanOps, mayo 2026) encontró que el **62% de la factura de AI** corresponde a contexto re-enviado — no a trabajo nuevo.

### 1.4 El multiplicador de output

Los output tokens cuestan 5x más que los input tokens en Anthropic. Reducir outputs tiene más impacto en el costo que reducir inputs.

**Regla práctica:** siempre setear `max_tokens` explícitamente. No dejar que el modelo decida cuánto generar.

### 1.5 Extended thinking: el costo que no se ve

Los modelos con razonamiento extendido generan tokens de thinking interno que se facturan como output tokens.

**El error más común:** usar `display: "omitted"` creyendo que evita el costo. No lo hace.

```python
# INCORRECTO — asume que omitir display evita el costo
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={
        "type": "enabled",
        "budget_tokens": 5000,
        "display": "omitted"  # Los 5000 tokens se siguen facturando
    }
)

# CORRECTO — limitar el budget según la complejidad real
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={
        "type": "adaptive",
        "effort": "low"  # low | medium | high
    }
)
```

**Impacto cuantificado:** un response con 500 tokens de output visible y 2.000 de thinking cuesta 5x más que sin thinking. (Fuente: PECollective, 2026)

---

## 2. Nivel 0 — Configuraciones inmediatas

### 2.1 Auto Mode en GitHub Copilot

**Qué hace:** selecciona dinámicamente el modelo más eficiente por request — ruteando entre GPT-5.4, GPT-5.3-Codex, Claude Sonnet 4.6 y Haiku 4.5.

**Cómo activarlo:** en el panel de Copilot en VSCode, cambiar el selector a "Auto".

**Beneficio verificado:** 10% de descuento en el multiplicador del modelo. (Fuente: GitHub Changelog, abril 2026)

### 2.2 chat.tools.compressOutput.enabled

**Qué hace:** VSCode post-procesa el output de terminal antes de enviarlo al modelo:
- Colapsa hunks sin cambios en diffs de git
- Descarta lockfile diffs completos
- Reduce `ls -l` a solo nombres de archivo
- Elimina barras de progreso y advertencias de deprecación

**Cómo activarlo:**
```json
{
  "chat.tools.compressOutput.enabled": true
}
```

Disponible desde VSCode 1.120. (Fuente: VSCode 1.120 Release Notes)

### 2.3 Tool search — MCPs diferidos

**Qué hace:** divide el toolset en herramientas core (~30, siempre disponibles) y diferidas (se cargan bajo demanda). Reduce el overhead de schemas en el contexto.

**Estado:** activo por defecto para modelos Anthropic (Sonnet 4.5+, Opus 4.5+).

**Para modelos GPT en Copilot:**
```json
{
  "github.copilot.chat.responsesApi.toolSearchTool.enabled": true
}
```

**Beneficio verificado:** hasta 20% de ahorro en tokens por request. (Fuente: Visual Studio Magazine, abril 2026)

### 2.4 Visibilidad de token usage para BYOK

VSCode 1.120 corrige el token accounting para modelos con API key propia — antes siempre mostraba 0%. Actualizar a VSCode 1.120 o superior.

### 2.5 Auditar MCPs activos

**Cálculo de impacto:**
- 10 MCPs × 500 tokens/schema = 5.000 tokens overhead/request
- A $3/MTok Sonnet × 100K requests/mes = **$1.500/mes** en herramientas no usadas

**Acción:** revisar extensiones de Copilot en VSCode y desactivar MCPs que no se usan frecuentemente.

---

## 3. Nivel 1 — Arquitectura de contexto

### 3.1 Prompt caching — implementación completa

#### Mecánica de pricing

| Operación | Costo relativo |
|-----------|---------------|
| Cache write TTL 5 minutos | 1.25x precio base input |
| Cache write TTL 1 hora | 2.0x precio base input |
| Cache read | 0.10x precio base input (90% descuento) |

Break-even: un único cache hit con TTL de 5 minutos cubre el costo del write.

Fuente: MetaCTO / Anthropic docs

#### Qué cachear y qué no

**Siempre cachear:** system prompt, tool definitions, knowledge base estático, few-shot examples, políticas de compliance.

**Nunca cachear:** input del usuario, historial reciente, resultados de tool calls del turno actual, timestamps.

#### Implementación en Python

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "Sos un asistente de ingeniería de Visma...",
            "cache_control": {"type": "ephemeral"}  # TTL 5 minutos
        },
        {
            "type": "text",
            "text": knowledge_base_content,  # 50K tokens
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[
        {
            "role": "user",
            "content": user_message  # NO se cachea
        }
    ]
)

# Verificar que el cache funcionó
usage = response.usage
print(f"Cache creation: {usage.cache_creation_input_tokens}")
print(f"Cache read: {usage.cache_read_input_tokens}")
print(f"Input no cacheado: {usage.input_tokens}")
```

#### Impacto con números reales

Escenario: app RAG, 50.000 tokens en system prompt, 1.000 queries/día.

- Sin caching: 50.000 × $3/MTok × 1.000 = **$150/día** en system prompt
- Con caching (95% hit rate): ~**$23.63/día** → 84% de ahorro

Fuente: Finout, Anthropic API Pricing 2026

### 3.2 Gestión de historial de conversación

#### Estrategia A: ventana deslizante fija

```python
def get_windowed_history(messages: list, window_size: int = 10) -> list:
    """Mantiene solo los últimos N turnos."""
    if len(messages) <= window_size:
        return messages
    return messages[-window_size:]
```

**Limitación:** pierde contexto de turnos anteriores importantes.

#### Estrategia B: resumen progresivo

```python
def compress_old_history(
    messages: list,
    recent_window: int = 5,
    summary_model: str = "claude-haiku-4-5"  # Haiku para summarizar — más barato
) -> list:
    if len(messages) <= recent_window:
        return messages
    
    old_messages = messages[:-recent_window]
    recent_messages = messages[-recent_window:]
    
    # Resumir con el modelo más barato
    summary_response = client.messages.create(
        model=summary_model,
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"""Resumí esta conversación en máximo 3 párrafos,
            preservando hechos y decisiones clave:\n\n{format_messages(old_messages)}"""
        }]
    )
    
    summary_text = summary_response.content[0].text
    
    compressed = [
        {"role": "user", "content": f"[Resumen anterior]\n{summary_text}"},
        {"role": "assistant", "content": "Entendido, continúo desde donde estábamos."}
    ] + recent_messages
    
    return compressed
```

#### Estrategia C: Compaction API de Anthropic (recomendada para agentes)

Automatiza completamente la gestión del historial. Beta desde enero 2026. Elegible para ZDR.

```python
client = anthropic.Anthropic()
messages = [{"role": "user", "content": "Empecemos con el feature X"}]

while True:
    response = client.beta.messages.create(
        betas=["compact-2026-01-12"],
        model="claude-sonnet-4-6",
        max_tokens=4096,
        messages=messages,
        context_management={
            "edits": [{
                "type": "compact_20260112",
                "context_token_threshold": 50_000  # Default: 100K. Mínimo: 50K
            }]
        }
    )
    
    messages.append({"role": "assistant", "content": response.content})
    
    # Monitorear compactions
    if hasattr(response, 'context_compact_usage'):
        before = response.context_compact_usage.tokens_before
        after = response.context_compact_usage.tokens_after
        print(f"Compaction: {before} → {after} tokens ({(1-after/before)*100:.0f}% reducción)")
    
    user_input = get_next_user_input()
    if not user_input:
        break
    messages.append({"role": "user", "content": user_input})
```

**Nota:** el system prompt permanece cacheado a través de múltiples compactions. Solo el resumen se escribe como nuevo cache entry.

Fuente: Claude Lab — Context Compaction API Guide, Anthropic Docs

### 3.3 RAG — Recuperación de contexto precisa

```python
def query_with_rag(
    user_query: str,
    vector_store,
    top_k: int = 5,
    max_tokens_per_chunk: int = 500
) -> str:
    client = anthropic.Anthropic()
    
    # Solo los chunks más relevantes
    relevant_chunks = vector_store.search(query=user_query, top_k=top_k)
    
    context = "\n\n".join([
        f"[Doc {i+1}: {chunk.source}]\n{chunk.text[:max_tokens_per_chunk]}"
        for i, chunk in enumerate(relevant_chunks)
    ])
    
    response = client.messages.create(
        model="claude-haiku-4-5",  # Haiku para RAG simple
        max_tokens=500,
        system="Respondé basándote únicamente en el contexto provisto.",
        messages=[{"role": "user", "content": f"Contexto:\n{context}\n\nPregunta: {user_query}"}]
    )
    
    return response.content[0].text
```

**Reducción verificada:** RAG puede reducir tokens de contexto en hasta 70% vs context stuffing. (Fuente: Koombea, 2026)

### 3.4 Output budgets por tipo de tarea

```python
OUTPUT_BUDGETS = {
    "classification": 50,
    "extraction": 200,
    "summarization": 500,
    "code_review": 1000,
    "code_generation": 2000,
    "analysis": 1500,
    "architecture": 3000,
}
```

---

## 4. Nivel 2 — Selección y asignación de modelos

### 4.1 Mapa de modelos por tipo de tarea

```python
MODEL_ASSIGNMENT = {
    # Haiku — tareas simples
    "classification": "claude-haiku-4-5",
    "entity_extraction": "claude-haiku-4-5",
    "data_formatting": "claude-haiku-4-5",
    "file_navigation": "claude-haiku-4-5",
    "simple_qa": "claude-haiku-4-5",
    "translation": "claude-haiku-4-5",
    
    # Sonnet — complejidad media
    "code_generation": "claude-sonnet-4-6",
    "code_review": "claude-sonnet-4-6",
    "summarization": "claude-sonnet-4-6",
    "documentation": "claude-sonnet-4-6",
    "general_chat": "claude-sonnet-4-6",
    "debugging": "claude-sonnet-4-6",
    
    # Opus — solo cuando el razonamiento complejo lo justifica
    "architecture_design": "claude-opus-4-7",
    "complex_reasoning": "claude-opus-4-7",
    "multi_step_planning": "claude-opus-4-7",
    "agent_coordination": "claude-opus-4-7",
}
```

### 4.2 Clasificación dinámica de tareas

```python
def classify_and_route(user_input: str, client) -> tuple[str, str]:
    """
    Clasifica la tarea y retorna (tarea, modelo_recomendado).
    Usa Haiku para clasificar — barato y rápido.
    """
    classification_response = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=50,
        system="""Clasificá el mensaje en UNA categoría:
        classification, extraction, summarization, code_generation, code_review,
        debugging, architecture_design, complex_reasoning, general_chat.
        Respondé SOLO el nombre de la categoría.""",
        messages=[{"role": "user", "content": user_input}]
    )
    
    task_type = classification_response.content[0].text.strip().lower()
    model = MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
    
    return task_type, model
```

---

## 5. Nivel 3 — Model routing automático

### 5.1 OpenRouter Auto Router

```json
{
  "openrouter": {
    "apiKey": "sk-or-...",
    "baseUrl": "https://openrouter.ai/api/v1",
    "defaultModel": "openrouter/auto"
  }
}
```

Análisis de complejidad por prompt, sin markup adicional, powered by NotDiamond.

Fuente: OpenRouter docs, Works With GitHub Copilot

### 5.2 RouteLLM

**Resultados verificados (ICLR 2025):**
- 95% calidad GPT-4 con 26% llamadas → 48% ahorro vs baseline
- Con data augmentation: 95% calidad con 14% llamadas → 75% reducción de costo

Fuente: LMSYS Blog — RouteLLM, github.com/lm-sys/RouteLLM

#### Instalación

```bash
pip install routellm
```

#### Uso básico (drop-in replacement)

```python
from routellm.controller import Controller

client = Controller(
    routers=["mf"],
    strong_model="gpt-4-turbo",
    weak_model="mixtral-8x7b"
)

response = client.chat.completions.create(
    model="router",
    messages=[{"role": "user", "content": user_message}]
)
print(f"Modelo usado: {response.model}")
```

#### Calibración del umbral

```python
from routellm.calibration import calibrate

threshold = calibrate(
    router="mf",
    strong_model="gpt-4-turbo",
    weak_model="mixtral-8x7b",
    strong_model_pct=0.15  # 15% de queries al modelo fuerte
)

client = Controller(
    routers=["mf"],
    strong_model="gpt-4-turbo",
    weak_model="mixtral-8x7b",
    config={"mf": {"threshold": threshold}}
)
```

#### Con modelos Anthropic

```python
client = Controller(
    routers=["mf"],
    strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5",
    config={"mf": {"threshold": 0.3}}
)
```

### 5.3 AI Gateways enterprise

#### LiteLLM — Self-hosted, open source

```bash
pip install litellm
litellm --model claude-sonnet-4-6
```

```python
from litellm import Router

router = Router(
    model_list=[
        {
            "model_name": "production-model",
            "litellm_params": {
                "model": "claude-sonnet-4-6",
                "api_key": os.getenv("ANTHROPIC_API_KEY"),
            },
            "tpm": 100000,
            "rpm": 1000,
        },
        {
            "model_name": "production-model",
            "litellm_params": {
                "model": "azure/claude-sonnet-4-6",  # Azure fallback
                "api_key": os.getenv("AZURE_API_KEY"),
                "api_base": os.getenv("AZURE_API_BASE"),
            },
            "tpm": 100000,
            "rpm": 1000,
        }
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)

await router.acompletion(
    model="production-model",
    messages=messages,
    metadata={"user_api_key": user_id, "budget": 10.0}
)
```

**Config yaml con alertas:**

```yaml
model_list:
  - model_name: claude-sonnet
    litellm_params:
      model: claude-sonnet-4-6
      api_key: sk-...

litellm_settings:
  max_budget: 100
  budget_duration: "1mo"
  alerting:
    - slack
    - email
```

#### Portkey — Managed, compliance y observabilidad

```python
from portkey_ai import Portkey

client = Portkey(
    api_key="PORTKEY_API_KEY",
    virtual_key="ANTHROPIC_VIRTUAL_KEY"
)

response = client.chat.completions.create(
    model="claude-sonnet-4-6",
    messages=[{"role": "user", "content": "Hello"}],
    metadata={"team": "engineering", "feature": "code-review", "user": user_id}
)
```

**Guardrails:**

```python
config = {
    "guardrails": [
        {"type": "pii_detection", "action": "mask"},
        {"type": "budget_limit", "monthly_limit": 500, "alert_at": 0.8}
    ],
    "retry": {"attempts": 3, "on_status_codes": [429, 500, 502, 503]},
    "cache": {"mode": "semantic", "max_age": 3600}
}
```

#### Tabla comparativa

| Característica | LiteLLM | Portkey | OpenRouter |
|---------------|---------|---------|------------|
| Tipo | Open source | Managed SaaS | Managed SaaS |
| Modelos | 100+ | 1.600+ | 300+ |
| Self-hosted | Sí | Opcional | No |
| Budget controls | Por equipo/API key | Por virtual key | Limitado |
| Compliance/SOC2 | No | Sí | No |
| Costo | Free | Free → $49/mes | 5.5% markup |
| Mejor para | Control total, DevOps | Industrias reguladas | Experimentación |

---

## 6. Nivel 4 — Infraestructura de costo y gobernanza

### 6.1 Batch API

**Descuento:** 50% en input y output. Calidad idéntica. (Fuente: Anthropic oficial)

**Stack acumulado:** Base 100% → Batch 50% → Con caching ~5% = hasta 95% de ahorro.

```python
import anthropic

client = anthropic.Anthropic()

requests = []
for document in documents_to_process:
    requests.append({
        "custom_id": f"doc-{document.id}",
        "params": {
            "model": "claude-sonnet-4-6",
            "max_tokens": 200,
            "system": [
                {
                    "type": "text",
                    "text": system_prompt,
                    "cache_control": {"type": "ephemeral"}  # Caching en batch
                }
            ],
            "messages": [{"role": "user", "content": f"Clasificá: {document.text[:1000]}"}]
        }
    })

batch = client.messages.batches.create(requests=requests)
print(f"Batch ID: {batch.id}")

# Polling
import time
while True:
    status = client.messages.batches.retrieve(batch.id)
    if status.processing_status == "ended":
        break
    time.sleep(60)

# Recuperar resultados
results = {}
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        results[result.custom_id] = result.result.message.content[0].text
```

**300K output tokens en Batch (beta):**

```python
batch = client.messages.batches.create(
    requests=requests,
    extra_headers={"anthropic-beta": "output-300k-2026-03-24"}
)
```

Fuente: Finout, Anthropic API Pricing 2026

**Workloads ideales:** clasificación masiva, generación de tests, análisis offline, procesamiento nocturno, evaluaciones de modelos.

### 6.2 DeepSeek via Azure

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_API_KEY"),
    api_version="2024-12-01-preview",
    azure_endpoint=os.getenv("AZURE_ENDPOINT")  # Región EU configurada
)

response = client.chat.completions.create(
    model="deepseek-v4-flash",
    messages=[{"role": "user", "content": text_to_classify}],
    max_tokens=50
)
```

**Cuándo tiene sentido:** clasificación masiva de tickets, extracción de entidades a escala, summarización en lote, pipelines de alta volumetría donde el costo por token es crítico.

**Cuándo NO usarlo:** razonamiento complejo, código crítico, análisis arquitectural.

**Costo comparativo:**
- Claude Sonnet: $3.00/MTok input
- DeepSeek via Azure: ~$0.19/MTok input
- Diferencia: **~15x más barato** para tareas equivalentes de baja complejidad

Fuente: DeployBase, TechJack Solutions

### 6.3 Negociación del EA de Microsoft

- EA solo: 15-25% de descuento
- EA + Azure MACC: 23-28% de descuento

Incluir Copilot en la negociación del EA renewal junto con M365 y Azure produce mejores resultados que negociar por separado. Los MACC son el mecanismo más efectivo para el tier más alto de descuento.

Fuente: Microsoft Negotiations, 2026

### 6.4 Observabilidad y gobernanza

```python
from dataclasses import dataclass
from typing import Optional
import time

@dataclass
class TokenUsageEvent:
    timestamp: float
    user_id: str
    team: str
    feature: str
    model: str
    input_tokens: int
    output_tokens: int
    cache_read_tokens: int
    cache_write_tokens: int
    cost_usd: float
    task_type: str
    session_id: Optional[str] = None

def calculate_cost(model, input_tokens, output_tokens,
                   cache_read_tokens=0, cache_write_tokens=0):
    PRICING = {
        "claude-haiku-4-5": {"input": 1e-6, "output": 5e-6,
                              "cache_read": 0.1e-6, "cache_write": 1.25e-6},
        "claude-sonnet-4-6": {"input": 3e-6, "output": 15e-6,
                               "cache_read": 0.3e-6, "cache_write": 3.75e-6},
        "claude-opus-4-7": {"input": 5e-6, "output": 25e-6,
                             "cache_read": 0.5e-6, "cache_write": 6.25e-6},
    }
    if model not in PRICING:
        return 0.0
    p = PRICING[model]
    return round(
        input_tokens * p["input"] + output_tokens * p["output"] +
        cache_read_tokens * p["cache_read"] + cache_write_tokens * p["cache_write"],
        6
    )

class BudgetMonitor:
    def __init__(self, monthly_budget_usd: float, user_id: str):
        self.monthly_budget = monthly_budget_usd
        self.user_id = user_id
        self.current_spend = self._load_current_spend()
        self.alert_thresholds = [0.5, 0.8, 1.0]
        self.alerts_sent = self._load_alerts_sent()
    
    def check_and_alert(self, new_spend: float):
        self.current_spend += new_spend
        usage_ratio = self.current_spend / self.monthly_budget
        
        for threshold in self.alert_thresholds:
            if usage_ratio >= threshold and threshold not in self.alerts_sent:
                self._send_alert(threshold, usage_ratio)
                self.alerts_sent.add(threshold)
    
    def _send_alert(self, threshold, actual_ratio):
        message = (
            f"⚠️ Budget Alert — {self.user_id}\n"
            f"Consumo: {actual_ratio:.1%} del presupuesto mensual\n"
            f"Gasto: ${self.current_spend:.2f} / ${self.monthly_budget:.2f}"
        )
        send_notification(message, channel="ai-costs-alerts")
```

**Métricas clave a monitorear:**

```
POR EQUIPO:
- Gasto total / mes
- Top 10 usuarios por consumo
- Distribución de modelos (% Haiku vs Sonnet vs Opus)
- Cache hit rate promedio
- Workloads candidatos a migrar a Batch

SEÑALES DE ALERTA:
- Usuario consumiendo >3x la mediana
- Sesión individual > $X
- Burn rate que agota crédito antes de fin de mes
- Bajo cache hit rate en sistemas con caching configurado
- Uso sostenido de Opus para tareas que Sonnet resuelve
```

---

## 7. Referencia de métricas verificadas

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
| Thinking 2000t + output 500t vs sin thinking | 5x más caro | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Azure EA solo descuento | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC descuento | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x más barato input | Precios 2026 | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % gasto IT enterprise | hasta 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Subestimación costos AI | 30% hacia 2027 | IDC | https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing |
