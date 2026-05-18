# 05 — Copilot Usage-Based Billing: El Cambio que Lo Hace Todo Urgente

Desde el 1 de junio de 2026, GitHub Copilot ya no es una suscripción flat. Cada token tiene un costo directo. Esto transforma la optimización de tokens de "buena práctica" a "necesidad operacional".

---

## El cambio

> "Copilot evolucionó de un asistente en el editor a una plataforma agéntica con sesiones largas y multi-step. El uso agéntico se está convirtiendo en el default, con demandas de compute significativamente más altas."

Fuente: [GitHub Blog — Copilot is Moving to Usage-Based Billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)

---

## Estructura del nuevo modelo

| Plan | Precio mensual | AI Credits incluidos |
|------|---------------|---------------------|
| Copilot Business | $19/usuario/mes | $19 en AI Credits |
| Copilot Enterprise | $39/usuario/mes | $39 en AI Credits |

**1 AI Credit = $0.01 USD**

Los créditos se consumen basados en tokens (input, output y cached) a las tarifas publicadas de cada modelo.

Fuente: [GitHub Docs — Billing for Copilot](https://docs.github.com/en/copilot/concepts/billing/organizations-and-enterprises)

### Créditos promocionales (junio–agosto 2026)
- Business: +$30/usuario/mes en créditos
- Enterprise: +$70/usuario/mes en créditos

Fuente: [GitHub Blog — Usage-Based Billing](https://github.blog/news-insights/company-news/github-copilot-is-moving-to-usage-based-billing/)

---

## El riesgo del tail de usuarios

> El **top 10–15% de usuarios concentra el 60–70%** del consumo de features avanzadas.

Fuente: [Synapx — Copilot Usage-Based Billing Executive Guide 2026](https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/)

### El multiplicador agéntico

> Estimaciones ilustrativas para workflows agénticos intensivos: ~£52/developer/mes — **3.5x el flat fee anterior**.

Fuente: [Synapx](https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/)

---

## Qué sigue siendo gratuito

Code completions e inline suggestions **no consumen AI Credits** — permanecen ilimitadas.

Lo que sí consume créditos:
- Chat (editor y GitHub.com)
- Copilot Workspace / Cloud Agents
- Code Review agéntico (también consume GitHub Actions minutes — **doble facturación**)

Fuente: [GitHub Docs — Models and Pricing](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing)

---

## El descuento EA de Microsoft

> EA solo: **15–25% de descuento** bajo lista. EA + Azure MACC: **23–28% de descuento**.

Fuente: [Microsoft Negotiations — GitHub Copilot Enterprise Licensing](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing)

**Nota crítica:** No es un número fijo publicado — es un rango negociado. Requiere conversación con el account team de Microsoft en el contexto del EA renewal o Azure MACC commitment.

---

## Por qué es un problema de gobernanza, no solo de pricing

> Las empresas que construyan infraestructura de AI cost governance ahora usarán esa misma infraestructura para cada nuevo servicio de AI. Las que lo traten como un cambio de billing puntual lo reconstruirán desde cero cada vez.

Fuente: [Open Empower — Copilot Token-Based Billing Enterprise](https://www.openempower.com/blog/github-copilot-token-based-billing-enterprise-ai-cost-governance)

### Señales de alerta a monitorear
- Usuarios consumiendo >3x la mediana de tokens
- Uso sostenido de modelos alto tier más allá de planning
- Sesiones individuales superando umbrales de costo
- Burn rate que agotaría créditos antes de mediados de mes

---

## Acciones inmediatas

1. Activar el preview de billing en GitHub Billing Overview antes de junio
2. Identificar el top 10–15% de usuarios por consumo
3. Setear alertas en 50%, 80% y 100% del crédito mensual
4. Activar Auto Mode (10% descuento, zero config)
5. Revisar uso de Workspace/Cloud Agents

---

## Métricas verificadas

| Dato | Valor | Fuente |
|------|-------|--------|
| AI Credits en Business | $19/usuario/mes | GitHub Docs |
| Top usuarios = % del spend | 10–15% = 60–70% | Synapx |
| Multiplicador agéntico | ~3.5x flat fee | Synapx |
| Descuento Auto Mode | 10% en multiplicador | GitHub Changelog |
| Descuento EA negociado | 15–28% según leverage | Microsoft Negotiations |
