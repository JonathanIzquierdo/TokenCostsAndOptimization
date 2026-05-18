# 07 — Azure y DeepSeek: Compliance y Descuentos Negociados

Dos estrategias sobre cómo comprás y cómo accedés a modelos — no sobre cómo los usás. Ambas con matices importantes que no aparecen en el marketing.

---

## Azure Enterprise Agreement: el descuento que hay que negociar

No hay un porcentaje fijo publicado. Es un rango negociable dentro del EA framework de Microsoft.

> EA solo: **15–25% de descuento**. EA + M365/Azure renewal: **18–22%**. EA + Azure MACC commitment: **23–28%**.

Fuente: [Microsoft Negotiations — GitHub Copilot Enterprise Licensing](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing)

El descuento varía según:
- Tamaño del Azure MACC commitment
- Timing de negociación (quarter-end = más presión)
- Si se incluye Copilot en EA renewal más amplio
- Presencia de propuestas competidoras como leverage

> Buyers que inician negociaciones con propuestas competidoras de Azure OpenAI consistentemente logran mejores headline rates.

Fuente: [Atonment Licensing — OpenAI Enterprise Pricing 2026](https://atonementlicensing.com/blog/openai-enterprise-pricing/)

---

## Azure OpenAI vs OpenAI directo: la verdad sobre el precio

Una confusión común: muchos equipos asumen que Azure es más barato. La realidad es más matizada.

> El precio por token es **idéntico** entre Azure y la API directa de OpenAI para los mismos modelos. La diferencia está en el overhead de infraestructura — Azure puede agregar **15–40% de costo adicional** en deployments enterprise de producción.

Fuente: [TokenMix — Azure OpenAI Cost 2026](https://tokenmix.ai/blog/azure-openai-cost)

| Canal | Overhead real |
|-------|---------------|
| OpenAI directo | +5–10% |
| Azure OpenAI | +15–40% |

**Conclusión:** Azure no es más barato. El valor es compliance, data residency y VNet integration — no el precio del token.

---

## DeepSeek: el modelo más barato del mundo con un problema

> DeepSeek V4-Flash: **$0.14/M input, $0.28/M output** — 35 a 100 veces más barato que GPT-5.5 o Claude Opus 4.7.

Fuente: [TechJack Solutions — DeepSeek Pricing 2026](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/)

### El problema de compliance

> Todos los API requests de DeepSeek rutean a través de servidores chinos. Italia baneó DeepSeek R1 por GDPR en 2025.

Fuente: [TechJack Solutions — DeepSeek Pricing 2026](https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/)

Para una empresa europea como Visma, enviar datos de clientes o código propietario a servidores en China no es viable en la mayoría de los casos.

### La solución: DeepSeek via Azure AI Foundry

> Microsoft integra modelos DeepSeek con un **markup de 20–35%** sobre las tarifas oficiales de DeepSeek.

Fuente: [DeployBase — DeepSeek V3 Pricing 2026](https://deploybase.ai/articles/deepseek-v3-pricing)

El beneficio no es el precio — es compliance: datos dentro del Azure boundary, sin envío a China, data residency europeo.

### El cálculo correcto

- Claude Sonnet: $3.00/M input
- DeepSeek V4-Flash via Azure (markup 35%): ~$0.19/M input

Incluso con markup, DeepSeek via Azure es **~15x más barato** que Sonnet para tareas de alta volumetría.

---

## Estrategia combinada para Visma

1. Negociar el EA con Copilot en el contexto de Azure MACC → objetivo 23–28%
2. Usar Azure OpenAI cuando el requerimiento sea compliance (no por precio)
3. Evaluar DeepSeek via Azure para workloads de alto volumen y baja complejidad
4. No usar DeepSeek directo sin validar con legal/compliance los requerimientos de data residency

---

## Métricas verificadas

| Escenario | Valor | Fuente |
|-----------|-------|--------|
| Descuento EA solo | 15–25% | Microsoft Negotiations |
| Descuento EA + MACC | 23–28% | Microsoft Negotiations |
| Azure overhead vs directo | +15–40% TCO | TokenMix |
| DeepSeek via Azure markup | +20–35% sobre directo | DeployBase |
| DeepSeek V4-Flash vs Claude Sonnet | ~15x más barato en input | Precios 2026 |
