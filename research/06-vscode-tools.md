# 06 — Herramientas VSCode para Reducir Tokens

VSCode 1.118 y 1.120 llegaron con features específicamente diseñadas para reducir token consumption — publicadas dos días después del anuncio de usage-based billing. No es casualidad.

---

## chat.tools.compressOutput.enabled

> Al activar esta setting, VS Code post-procesa el output de comandos de terminal antes de enviarlo al modelo: colapsa hunks sin cambios en diffs, descarta lockfile diffs, reduce `ls -l` a nombres, elimina barras de progreso de `npm install` y advertencias de deprecación.

Fuente: [Visual Studio Code 1.120 Release Notes](https://code.visualstudio.com/updates/v1_120)

### Cómo activarlo

```json
// settings.json
{
  "chat.tools.compressOutput.enabled": true
}
```

O desde Command Palette: `Preferences: Open Settings (UI)` → buscar `compressOutput`.

---

## Tool Search: MCPs diferidos por defecto

### El problema
Cada MCP activo agrega su schema completo al contexto en cada request — aunque nunca uses esa herramienta en esa sesión.

### La solución (VSCode 1.118)

> VS Code dividió el toolset en:
> - **Core (~30 herramientas):** siempre disponibles, cubren ~88% de las llamadas reales
> - **Diferidas:** schemas no se cargan hasta que se solicitan explícitamente
>
> Genera hasta **20% de ahorro en tokens**. Ya es el default para modelos Anthropic (Sonnet 4.5+ y Opus 4.5+).

Fuente: [Visual Studio Magazine — VS Code Curbs Token Use 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

### Para usuarios de modelos GPT en Copilot

```json
{
  "github.copilot.chat.responsesApi.toolSearchTool.enabled": true
}
```

---

## Prompt caching con 93% de reuse rate

> VS Code 1.118 ships prompt caching con tasas de reuso del **93%** en sesiones típicas.

Fuente: [Visual Studio Magazine](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

El 93% de los tokens del system prompt y tool definitions se sirven desde cache — pagando solo el 10% del precio base en esa porción.

---

## Visibilidad de token usage para BYOK (VSCode 1.120)

> Anteriormente el control mostraba siempre 0% para modelos con API key propia. VSCode 1.120 corrige el token accounting para modelos BYOK (Anthropic, OpenAI, etc.).

Fuente: [Visual Studio Code 1.120 Release Notes](https://code.visualstudio.com/updates/v1_120)

Sin esta visibilidad, los equipos volaban ciegos sobre cuánto contexto consumían en sesiones largas.

---

## OpenTelemetry (oTel) para medición profunda

La combinación VSCode + oTel permite trazar token consumption a nivel de request.

**Qué medir:**
- Tokens de input por tipo de tarea — identificar qué operaciones consumen más
- Tokens de output por modelo — validar output budgets
- Cache hit rate — medir efectividad del prompt caching
- Distribución de modelos — verificar que el routing funciona

---

## Checklist de configuración para el equipo

| Setting | Impacto |
|---------|---------|
| `chat.tools.compressOutput.enabled: true` | Reduce tokens de terminal output |
| Tool search (default en Anthropic models) | Hasta 20% ahorro en tool schemas |
| Auto Mode en selector de modelo | 10% descuento en multiplicador |
| Prompt caching (automático en VSCode 1.118+) | 90% off en tokens cacheados |
| Token usage visibility (VSCode 1.120+) | Visibilidad correcta del contexto |

---

## Métricas verificadas

| Feature | Impacto | Fuente |
|---------|---------|--------|
| chat.tools.compressOutput.enabled | Reduce tokens de terminal output | VSCode 1.120 |
| Tool search (MCPs diferidos) | hasta 20% ahorro por request | VS Magazine |
| Prompt caching en VSCode | 93% reuse rate en sesiones típicas | VS Magazine |
