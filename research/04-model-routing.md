# 04 — Model Routing: De Auto Mode a Gateway Enterprise

No elegir el modelo correcto manualmente es un problema humano. La solución no es disciplina — es automatización. Tres niveles de sofisticación.

---

## Por qué el routing es infraestructura, no feature

> El 37% de las enterprises usan 5 o más modelos en producción en 2026.

Fuente: [Swfte AI — Intelligent LLM Routing 2026](https://www.swfte.com/blog/intelligent-llm-routing-multi-model-ai)

> IDC FutureScape 2026: para 2028 el 70% de las top enterprises usarán arquitecturas multi-tool para model routing dinámico y autónomo.

Fuente: [IDC — The Future of AI is Model Routing 2025](https://blogs.idc.com/2025/11/17/the-future-of-ai-is-model-routing/)

---

## Nivel 1 — Auto Mode: cero configuración, impacto inmediato

### GitHub Copilot Auto Mode

> Con "auto", Copilot elige el modelo más eficiente — ruteando entre GPT-5.4, GPT-5.3-Codex, Sonnet 4.6 y Haiku 4.5. Todos los suscriptores pagos reciben un **10% de descuento** en el multiplicador del modelo cuando usan auto.

Fuente: [GitHub Changelog — Copilot CLI Auto Model Selection Abril 2026](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/)

**Acción inmediata:** cambiar el selector a "Auto" en VSCode. Cero configuración, 10% de descuento automático.

### OpenRouter Auto Router

> El Auto Router (`openrouter/auto`) selecciona el mejor modelo por prompt, powered by NotDiamond. No agrega markup — pagás la tarifa estándar del modelo seleccionado.

Fuente: [OpenRouter — Auto Router Documentation](https://openrouter.ai/docs/guides/routing/routers/auto-router)

OpenRouter se integra directamente con GitHub Copilot como provider alternativo.

Fuente: [OpenRouter — Works With GitHub Copilot](https://openrouter.ai/works-with-openrouter/github-copilot)

---

## Nivel 2 — RouteLLM: routing inteligente open source

Framework de UC Berkeley / LMSYS que usa ML para predecir qué modelo performará mejor por query.

### Resultados verificados (ICLR 2025)

> Router de matrix factorization: **95% de la performance de GPT-4 usando solo el 26% de llamadas a GPT-4** — **48% más barato** que baseline aleatorio.

> Con data augmentation: 95% calidad con **14% de llamadas al modelo fuerte** — reducción de costo del **75%**.

Fuente: [LMSYS Blog — RouteLLM](https://www.lmsys.org/blog/2024-07-01-routellm/)

### Drop-in replacement del cliente OpenAI

```python
# Antes
from openai import OpenAI
client = OpenAI(api_key="sk-...")

# Después (routing automático)
from routellm.controller import Controller
client = Controller(
    routers=["mf"],
    strong_model="gpt-4-turbo",
    weak_model="mixtral-8x7b"
)
```

Fuente: [GitHub — lm-sys/RouteLLM](https://github.com/lm-sys/RouteLLM)

No requiere reescribir la aplicación. Se configura el umbral de costo-calidad y el sistema enruta automáticamente.

---

## Nivel 3 — AI Gateways Enterprise

> Sin un gateway, un solo loop descontrolado puede quemar todo el presupuesto de API en una noche. Los controles de budget son el argumento más sólido para usar un gateway, incluso con un solo provider.

Fuente: [Techsy.io — 8 Best LLM Gateway Tools 2026](https://techsy.io/en/blog/best-llm-gateway-tools)

### Comparativa

| Gateway | Tipo | Modelos | Ideal para |
|---------|------|---------|------------|
| LiteLLM | Open source, self-hosted | 100+ providers | Control total, zero fees de plataforma |
| Portkey | Managed | 1.600+ | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | 300+ | Zero setup, prototyping |

Fuente: [EdenAI — Best LLM Routers 2026](https://www.edenai.co/post/best-llm-routers)

### Guía de decisión rápida

| Necesidad | Opción |
|-----------|--------|
| Self-hosted, control total, zero fees | LiteLLM |
| Compliance, guardrails, audit logs, SOC2 | Portkey |
| Zero setup, 300+ modelos inmediato | OpenRouter |
| Experimentar → migrar a producción | OpenRouter → LiteLLM |

---

## Métricas verificadas

| Estrategia | Ahorro | Fuente |
|------------|--------|--------|
| Copilot Auto Mode | 10% descuento en multiplicador | GitHub Changelog |
| RouteLLM matrix factorization | 48% vs baseline | LMSYS ICLR 2025 |
| RouteLLM con data augmentation | 75% reducción llamadas modelo fuerte | LMSYS ICLR 2025 |
| Model routing general | 20–80% OpEx | IDC FutureScape 2026 |

---

## Archivos relacionados
- `02-model-selection.md` — qué modelo para qué tarea
- `05-copilot-billing.md` — por qué el nuevo billing hace esto urgente
