# PROJECT INSTRUCTIONS
## Instrucciones completas para continuar el proyecto de forma independiente

Este archivo contiene todo el contexto necesario para que cualquier persona (o sesión de Claude) pueda continuar el proyecto sin depender de conversaciones anteriores.

---

## Descripción del proyecto

**Qué es:** Un artículo técnico sobre uso inteligente de tokens en IA — consumo, optimización, costos y gobernanza — dirigido al equipo de ingeniería de Visma.

**Por qué existe:** El costo de usar IA en producción está creciendo silenciosamente. En junio 2026, GitHub Copilot migró de flat fee a usage-based billing basado en tokens. Esto hace urgente que todo el equipo entienda cómo se generan los costos y qué pueden hacer al respecto — desde cambiar un setting en VSCode hasta implementar routing enterprise.

**Qué se quiere lograr:**
1. Generar conciencia colectiva en el equipo técnico
2. Proveer estrategias accionables en distintos niveles de complejidad
3. Publicar como artículo interno con storytelling + documentación técnica linkada

---

## Formato y entregables

| Entregable | Descripción |
|-----------|-------------|
| Artículo | 2.500–3.500 palabras, storytelling técnico, en español |
| Documentación | Archivos de research detallados por tema (este repositorio) |

El artículo linkea a los archivos de research para quien quiera profundizar en cada tema.

---

## Audiencia

**Quiénes:** Desarrolladores y tech leads del equipo de Visma (empresa SaaS enterprise europea)
**Qué saben:** Ya conocen qué es un LLM, qué es un token, y usan herramientas como GitHub Copilot y Claude Code en el trabajo diario
**Qué NO necesitan:** Explicación de conceptos básicos de IA
**Contexto relevante:** Visma tiene Microsoft Enterprise Agreement, usa Azure, y tiene GitHub Enterprise Cloud con Copilot

---

## Pasos del proyecto

| # | Paso | Estado |
|---|------|--------|
| 1 | Recolección de información | ✅ Completado |
| 2 | Organización y estructura | ✅ Completado |
| 3 | Optimización con data adicional (si aparece info nueva) | ⏳ Pendiente |
| 4 | Redacción del artículo | ⏳ Pendiente |
| 5 | Redacción de documentación técnica final | ⏳ Pendiente |

---

## Temas confirmados del artículo

1. **Selección de modelo según tarea y costo** — no todos los modelos cuestan lo mismo ni sirven para lo mismo
2. **MCPs activos innecesariamente** — cada herramienta activa agrega tokens de overhead aunque no se use
3. **Ingeniería de contexto para agentes** — el contexto re-enviado es el 62% de la factura
4. **Prompt caching y memory compression** — 90% de descuento que nadie usa
5. **Azure EA discounts** — rango negociado 15-28% (no número fijo publicado)
6. **DeepSeek via Azure** — el modelo más barato con compliance europeo (20-35% markup vs directo, pero 15x más barato que Sonnet)
7. **Herramientas de medición VSCode + oTel** — visibilidad de token consumption
8. **chat.tools.compressOutput.enabled** — setting VSCode 1.120 que comprime terminal output
9. **Copilot usage-based billing junio 2026** — el cambio que lo hace todo urgente
10. **Output tokens 4-6x más caros que input** — la asimetría que pocos conocen
11. **Auto model routing** — Copilot Auto Mode (10% descuento) + OpenRouter
12. **RouteLLM + AI Gateways** — LiteLLM / Portkey para routing enterprise
13. **Compaction API Anthropic** — beta enero 2026, resume historial automáticamente
14. **Batch API Anthropic** — 50% descuento, sin diferencia de calidad
15. **Extended thinking tokens** — se facturan como output aunque uses display:omitted

---

## Reglas editoriales CRÍTICAS

### Regla #1: Ningún número sin fuente
Ningún dato numérico o porcentaje puede usarse en el artículo sin URL de fuente verificada. Ver `data/verified-metrics.md` para el inventario completo. Si un número no está ahí, se marca `[VERIFICAR]` y se busca antes de escribir.

### Regla #2: Foco en costo
Cada tema debe responder directamente: **¿esto mueve la factura? ¿es accionable?**
Temas de accuracy o calidad de respuesta solo se incluyen si el vínculo con el costo es directo e inevitable. No incluir prompt engineering o performance de modelo por sí solos.

### Regla #3: No interpolar métricas
No estimar ni calcular números propios. Solo datos extraídos de artículos o estudios reales.

---

## Temas descartados (y por qué)

| Tema | Razón del descarte |
|------|-------------------|
| "Lost in the middle" problem | Es un tema de accuracy/calidad, no de costo directo. El vínculo con la factura es indirecto. |

---

## Datos verificados clave (resumen rápido)

| Dato | Valor | Fuente |
|------|-------|--------|
| Contexto re-enviado = % de la factura | 62% | LeanOps, auditoría 30 equipos 2026 |
| Tokens en conversación 20 turnos | 5K-10K innecesarios | Redis |
| Turno 1 vs turno 50 (agentes) | 5K → 200K tokens | Redblink |
| Output vs input (multiplicador) | 4-6x | Redis |
| Prompt caching descuento | 90% | Anthropic docs |
| RouteLLM: calidad GPT-4 con menos llamadas | 95% calidad con 26% llamadas | LMSYS ICLR 2025 |
| RouteLLM ahorro máximo | 75% reducción (con data augmentation) | LMSYS ICLR 2025 |
| Model routing ahorro OpEx | 20-80% | IDC FutureScape 2026 |
| RAG reduce contexto | hasta 70% | Koombea |
| Copilot Business precio | $19/usuario/mes = $19 AI Credits | GitHub Docs |
| Top usuarios concentran spend | 10-15% = 60-70% del total | Synapx |
| Workflows agénticos costo | ~3.5x flat fee anterior | Synapx |
| Auto Mode Copilot descuento | 10% en multiplicador | GitHub Changelog |
| Memory compression | hasta 80% menos tokens | Mem0 |
| Azure EA discount | 15-25% solo EA; 23-28% con MACC | Microsoft Negotiations |
| DeepSeek via Azure markup | +20-35% sobre DeepSeek directo | DeployBase |
| chat.tools.compressOutput | VSCode 1.120, comprime terminal output | VSCode release notes |
| Tool search VSCode | hasta 20% token savings | Visual Studio Magazine |
| Batch API Anthropic | 50% descuento, sin pérdida de calidad | Anthropic oficial |
| Batch + caching combinados | hasta 95% ahorro vs estándar | PECollective |
| Compaction API | Beta enero 2026, elegible ZDR | Claude Lab |
| Extended thinking billing | Se factura como output (display:omitted no ahorra) | CheckThat.ai |
| AI % gasto IT enterprise | hasta 50% (algunas firmas) | Deloitte 2026 via Redblink |

---

## Modelo elegido para redacción

**Claude Sonnet 4** — `claude-sonnet-4-20250514`

**Por qué Sonnet 4 y no Opus 4:**
Opus 4 es más caro y no aporta valor diferencial para escritura técnica con estructura clara y datos ya recolectados. Usar Opus sería exactamente el anti-patrón que el artículo critica: modelo más caro para tarea que no lo justifica.

**Por qué no Haiku:**
La exigencia de calidad narrativa del artículo necesita más capacidad de la que Haiku ofrece consistentemente.

---

## Estructura del artículo (outline v1)

Ver `article/outline.md` para la estructura completa. Resumen:

1. **Apertura** — Hook con el 62% de la factura
2. **Sección 1** — El problema invisible (cómo crece el costo)
3. **Sección 2** — El cambio urgente (Copilot billing junio 2026)
4. **Sección 3** — Las capas de optimización (Capa 0 a Capa 4)
5. **Cierre** — De práctica individual a cultura de equipo

---

## Routing de model routing (3 niveles técnicos)

| Nivel | Herramienta | Descripción |
|-------|-------------|-------------|
| 1 — Básico | Copilot Auto Mode + OpenRouter | Zero config, 10% descuento, disponible hoy |
| 2 — Técnico | RouteLLM (UC Berkeley/LMSYS, ICLR 2025) | Open source, drop-in OpenAI replacement, 75% ahorro potencial |
| 3 — Enterprise | LiteLLM (self-hosted) / Portkey (managed) / OpenRouter (SaaS) | Budget controls, observabilidad, guardrails, SOC2 |

---

## Instrucciones para continuar con Claude

Si retomás el proyecto en una nueva sesión de Claude:

1. **Leer este archivo primero** — contiene todo el contexto
2. **Leer `data/verified-metrics.md`** antes de redactar cualquier sección
3. **Usar Claude Sonnet 4** (`claude-sonnet-4-20250514`)
4. **Próximo paso:** revisar `article/outline.md` y arrancar la redacción del artículo
5. **Antes de usar cualquier número:** verificar que está en `data/verified-metrics.md` con URL

### Prompt sugerido para nueva sesión:
```
Estoy trabajando en un artículo técnico sobre optimización de tokens en IA para el equipo de Visma.
Todo el contexto, datos verificados e instrucciones están en el repositorio:
https://github.com/JonathanIzquierdo/TokenCostsAndOptimization

Lee PROJECT_INSTRUCTIONS.md y data/verified-metrics.md antes de continuar.
El próximo paso es: [describir qué necesitás hacer].
Modelo a usar: claude-sonnet-4-20250514
```

---

## Historial de decisiones

| Decisión | Razonamiento |
|----------|-------------|
| Español como idioma del artículo | Audiencia interna Visma, más impacto en lengua nativa |
| Sonnet 4 para redacción | Calidad suficiente, consistente con mensaje del artículo |
| "Lost in the middle" descartado | No conecta directamente con costo — es accuracy, no billing |
| Azure 24% → rango 15-28% | El 24% es un acuerdo negociado específico, no número publicado |
| DeepSeek via Azure = compliance, no precio | Azure cobra 20-35% más que directo — el beneficio es seguridad |
| Batch API incluida | 50% descuento verificado, workload relevante para el equipo |
| Extended thinking incluida | Costo oculto directo — thinking tokens = output tokens aunque uses omitted |
