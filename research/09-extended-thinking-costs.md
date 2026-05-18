# 09 — Extended Thinking: El Costo Oculto que Nadie Monitorea

Extended thinking mejora la calidad del razonamiento. También puede multiplicar silenciosamente la factura de output tokens — especialmente cuando se usa `display: "omitted"`.

---

## La regla fundamental

> Los extended thinking tokens se facturan como **output tokens** a tarifas de output. Configurar `display: "omitted"` reduce la latencia pero **NO el costo** — los thinking tokens completos se siguen facturando igual.

Fuente: [CheckThat.ai — Anthropic Pricing 2026](https://checkthat.ai/brands/anthropic/pricing)

Este es el punto crítico que la mayoría de los equipos no conoce. Cuando usás `display: "omitted"`, el modelo piensa internamente, no te muestra el razonamiento, pero **pagás por cada token de ese razonamiento invisible**.

---

## El multiplicador de costo real

> Response con 500 tokens de output visible + 2.000 tokens de thinking:
> - Con extended thinking: 2.500 tokens × $25/MTok = $0.0625
> - Sin extended thinking: 500 tokens × $25/MTok = $0.0125
> - **5x más caro** por el mismo output visible.

Fuente: [PECollective — Anthropic API Pricing 2026](https://pecollective.com/tools/anthropic-api-pricing/)

En tareas muy complejas el ratio puede ser aún mayor: el modelo puede generar 5–10x más thinking tokens que output tokens visibles.

---

## Cuándo el extended thinking justifica el costo

**Vale la pena:**
- Razonamiento matemático complejo
- Code review de múltiples archivos
- Debugging de problemas difíciles
- Análisis arquitectural
- Tareas donde la calidad del output impacta directamente el tiempo de desarrollo

**No vale la pena:**
- Completions simples de código
- Búsqueda y recuperación de información
- Formateo y transformación de datos
- Clasificación y etiquetado
- Cualquier tarea que Haiku resuelve bien

---

## El problema de Opus 4.7 por defecto

> Opus 4.7 usa adaptive thinking automáticamente — el modelo decide cuánto razonar solo. Sin configurar effort levels, puede decidir pensar extensamente en queries simples.

Fuente: [Anthropic Docs — Building with Extended Thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

### El truco del "omitted" que no ahorra dinero

```python
# Esto NO reduce el costo — solo oculta el razonamiento al usuario
thinking={
    "type": "enabled",
    "budget_tokens": 5000,
    "display": "omitted"  # ⚠️ Los tokens se siguen cobrando igual
}
```

---

## Cómo controlar el costo

### 1. Setear budget_tokens explícitamente

```python
thinking={
    "type": "enabled",
    "budget_tokens": 1024  # Mínimo posible para tareas moderadas
}
```

El modelo puede usar menos del budget si la tarea no lo requiere, pero no usará más de lo configurado.

### 2. Adaptive thinking con effort bajo

```python
# Para Opus 4.7
thinking={
    "type": "adaptive",
    "effort": "low"  # low | medium | high
}
```

### 3. Routing: reservar extended thinking solo para lo que lo requiere

La estrategia más efectiva: rutear tareas simples a Haiku o Sonnet sin thinking, y reservar Opus con thinking para los casos que genuinamente lo justifican.

---

## En contexto de Copilot

Con usage-based billing, usar modelos con thinking activo (Claude Opus, o1/o3 de OpenAI) en tareas simples puede costar 5–10x más por interacción que Haiku sin thinking.

---

## Métricas verificadas

| Escenario | Costo relativo | Fuente |
|-----------|---------------|--------|
| Sin extended thinking (500 tokens output) | 1x base | Anthropic pricing |
| Con thinking (500 visible + 2000 thinking) | 5x base | PECollective |
| display: "omitted" vs display: "shown" | Mismo costo — solo cambia visibilidad | CheckThat.ai |
