# 06 — Herramientas VSCode para Reducir Tokens y Monitorear Consumo

VSCode tiene un conjunto de herramientas para dos objetivos distintos: **reducir** el consumo de tokens y **visualizar** lo que está pasando. Ambas son críticas con el nuevo billing de Copilot.

---

## 1. chat.tools.compressOutput.enabled

> Al activar esta setting, VS Code post-procesa el output de comandos de terminal antes de enviarlo al modelo: colapsa hunks sin cambios en diffs, descarta lockfile diffs, reduce `ls -l` a nombres, elimina barras de progreso de `npm install` y advertencias de deprecación.

Fuente: [Visual Studio Code 1.120 Release Notes](https://code.visualstudio.com/updates/v1_120)

```json
{ "chat.tools.compressOutput.enabled": true }
```

---

## 2. Tool Search: MCPs diferidos por defecto

Cada MCP activo agrega su schema completo al contexto en cada request. VSCode 1.118 dividió el toolset en core (~30 herramientas, 88% de los casos) y diferidas (se cargan bajo demanda).

> Genera hasta **20% de ahorro en tokens**. Default para modelos Anthropic (Sonnet 4.5+, Opus 4.5+).

Fuente: [Visual Studio Magazine — VS Code Curbs Token Use 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

Para modelos GPT en Copilot:
```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```

---

## 3. Prompt caching con 93% de reuse rate

> VS Code 1.118 ships prompt caching con tasas de reuso del **93%** en sesiones típicas.

Fuente: [Visual Studio Magazine](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

---

## 4. Visibilidad de token usage para BYOK (VSCode 1.120)

VSCode 1.120 corrige el token accounting para modelos con API key propia — antes siempre mostraba 0%.

Fuente: [Visual Studio Code 1.120 Release Notes](https://code.visualstudio.com/updates/v1_120)

---

## 5. Agent Debug Log Panel — El trace completo dentro de VSCode

Esta es la herramienta más directa para ver qué está pasando en una sesión de agente: tokens consumidos, tool calls ejecutadas, subagentes invocados, errores — todo sin necesidad de configurar ningún backend externo.

> El Agent Debug Log panel muestra un registro cronológico de todo lo que ocurre durante una sesión de chat: tool calls, LLM requests, descubrimiento de prompt files y errores. Es un trace legible del proceso completo del agente, renderizado directamente dentro de VSCode.

Fuente: [VSCode Docs — Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

### Cómo abrirlo

Opción 1 — desde el panel de Copilot Chat:
- Click en el menú de overflow `(...)` → seleccionar **"Show Agent Debug Logs"**

Opción 2 — desde el Command Palette:
- `Cmd/Ctrl + Shift + P` → ejecutar **"Developer: Open Agent Debug Logs"**

Fuente: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, mayo 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### Qué muestra

El panel tiene tres vistas:

**Overview (por sesión):**
- Model turns totales
- Tool calls ejecutadas
- Total de tokens consumidos
- Total de eventos
- Errores

**View Logs:**
- Lista cronológica de todos los eventos con timestamps, nombres y detalles
- Filtración y búsqueda por tipo de evento

**Agent Flow Chart:**
- Grafo visual del flujo de ejecución del agente
- Muestra cómo se relacionan model turns, tool calls y subagentes
- Hace visible el árbol completo cuando hay subagentes anidados

Fuente: [GitHub VSCode Issues — Chat Agent Debug Log Panel](https://github.com/microsoft/vscode/issues/303786)

### Habilitar persistencia de sesiones en disco

Por defecto el panel solo muestra la sesión activa. Para poder revisar sesiones pasadas:

```json
{ "github.copilot.chat.agentDebugLog.fileLogging.enabled": true }
```

> Con esto habilitado, cada sesión de agente se persiste en disco. Podés volver a cualquier sesión anterior e inspeccionar tool calls, LLM requests, descubrimiento de prompt files, token usage y errores después del hecho.

Fuente: [dvlprlife.com — Quick Tips: Debug Copilot Agent Session Logs in VS Code, mayo 2026](https://www.dvlprlife.com/2026/05/quick-tips-debug-copilot-agent-session-logs-in-vs-code/)

### Export a OTLP JSON

Cada sesión puede exportarse a un archivo OpenTelemetry JSON (formato OTLP) para:
- Compartir con un compañero para debugging colaborativo
- Archivar una sesión problemática para análisis posterior
- Importar en el Agent Debug Panel para revisar offline
- Ingestar en cualquier backend OTel (Jaeger, Grafana, Langfuse, etc.)

Cómo exportar:
1. Abrir el Agent Debug Logs panel
2. Navegar a la sesión que querés exportar
3. Click en el ícono de Export (download) en el toolbar superior derecho
4. Elegir la ubicación para guardar el JSON

Fuente: [VSCode Docs — Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

### El comando /troubleshoot

Una vez habilitado el file logging, podés pedirle al mismo Copilot que analice sus propios logs:

```
/troubleshoot how many tokens did I use?
/troubleshoot why did it skip that tool?
/troubleshoot list all paths you tried to load customizations in #session
```

Fuente: [VSCode Docs — Troubleshoot AI in Visual Studio Code](https://code.visualstudio.com/docs/copilot/troubleshooting)

---

## 6. Chat Debug View — Inspección request a request

Mientras el Agent Debug Log muestra la sesión completa, el Chat Debug View muestra el detalle de cada request individual.

**Cómo abrirlo:**
- Menú overflow `(...)` en el panel de Copilot Chat → **"Show Chat Debug View"**
- O Command Palette → **"Developer: Show Chat Debug View"**

**Qué muestra por request:**
- System prompt completo enviado
- User prompt
- Contexto incluido (archivos, snippets, historial)
- Respuesta del modelo

Fuente: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, mayo 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

---

## 7. OpenTelemetry para observabilidad de equipo

Para equipos que quieren visibilidad centralizada más allá de la sesión individual, ver el archivo `research/` sobre OTel. El Agent Debug Log resuelve el problema individual; OTel resuelve el problema de gobernanza a nivel equipo.

---

## Checklist completo de configuración

| Setting / Herramienta | Propósito | Impacto |
|----------------------|-----------|----------|
| `chat.tools.compressOutput.enabled: true` | Reduce tokens de terminal output | Ahorro en sesiones con terminal frecuente |
| Tool search (default Anthropic) | MCPs diferidos | Hasta 20% ahorro por request |
| Auto Mode en selector de modelo | Routing automático | 10% descuento en multiplicador |
| Prompt caching (VSCode 1.118+) | Cache de contexto estático | 90% off en tokens cacheados |
| Token usage visibility (VSCode 1.120+) | Visibilidad BYOK | Contador correcto del contexto |
| Agent Debug Log panel | Trace de sesión | Ver tokens, tool calls, flow chart |
| `agentDebugLog.fileLogging.enabled: true` | Persistir sesiones en disco | Revisar sesiones pasadas |
| Chat Debug View | Inspección por request | Ver system prompt + contexto enviado |
| Export a OTLP JSON | Compartir / archivar | Análisis offline o en backend OTel |

---

## Métricas verificadas

| Feature | Impacto | Fuente |
|---------|---------|--------|
| chat.tools.compressOutput.enabled | Reduce tokens de terminal output | VSCode 1.120 release notes |
| Tool search (MCPs diferidos) | hasta 20% ahorro por request | Visual Studio Magazine |
| Prompt caching en VSCode | 93% reuse rate | Visual Studio Magazine |
| Agent Debug Log | Visibilidad tokens y trace sin setup externo | VSCode Docs, dvlprlife.com |
