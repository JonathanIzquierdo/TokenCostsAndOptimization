# 04 — Glosario del proyecto

Vocabulario interno que aparece en archivos del repo y en conversaciones. Si te cruzás con un término que no entendés, buscalo acá.

---

## Términos del repo

**`PROD/`**  
Directorio de entregables en producción. Lo que se va a publicar. Distinto de `research/` (notas internas) y `data/` (números).

**`research/`**  
9 archivos numerados con la investigación por tema. Es la fuente de verdad temática — si querés saber sobre prompt caching, vas a `research/03-prompt-caching.md`.

**`data/verified-metrics.md`**  
Única fuente de verdad para números del proyecto. Cada número tiene URL de fuente. **Si un número no está acá, no se usa.**

**`JON.md` / `JON-EN.md`**  
Índice ejecutivo para Jonathan Izquierdo (el dueño del proyecto). Sirve para que él decida rápido qué revisar.

**`claude-design-brief-*.md`**  
Briefs para Claude Design. Contienen las especificaciones de visuales (gráficos, diagramas) que se generan a partir del contenido aprobado.

**`.context/`**  
Ingeniería de contexto del proyecto. Lo que estás leyendo ahora.

**`CLAUDE.md` (raíz)**  
Bootstrap obligatorio. Lo primero que se lee al entrar al repo.

---

## Términos editoriales

**Capa 0 / Capa 1 / Capa 2 / Capa 3 / Capa 4**  
Estructura de la documentación técnica. Cada capa corresponde a un nivel de complejidad/alcance:

- **Capa 0 — Settings individuales** (1 click): VSCode settings, Copilot Auto Mode, etc.
- **Capa 1 — Prácticas de desarrollador** (cambiar hábitos): elegir el modelo correcto, no abrir 15 archivos en contexto, etc.
- **Capa 2 — Configuración de equipo** (decisiones de tech lead): MCPs activos, memory compression, batch jobs.
- **Capa 3 — Arquitectura técnica** (decisiones de architect): RAG, prompt caching, RouteLLM.
- **Capa 4 — Gobernanza enterprise** (decisiones de plataforma): LiteLLM / Portkey gateways, Azure EA, observabilidad oTel.

El artículo recorre estas capas en orden ascendente.

**"El 62%"**  
Referencia al dato de que el 62% de la factura de AI en equipos enterprise corresponde a contexto re-enviado innecesariamente (fuente: LeanOps 2026). Es el hook principal del artículo.

**"De 5K a 200K"**  
Referencia al crecimiento de tokens en una sesión agéntica del turno 1 al turno 50 (fuente: Redblink). Otro dato hook.

**"Gastar bien, no gastar menos"**  
Principio rector del proyecto. El mensaje de fondo: no es minimizar el gasto, es alinearlo con el valor.

---

## Términos técnicos del dominio

**Prompt caching**  
Técnica que cachea partes estables del prompt (system prompt, contexto largo) entre llamadas. Anthropic ofrece 90% de descuento en cache reads. La técnica más subutilizada del ecosistema.

**Memory compression**  
Resumir/comprimir el historial de conversación largo a una versión más corta antes de enviarlo al modelo. Mem0 reporta hasta 80% menos tokens. Trade-off: posible pérdida de detalle.

**Compaction API (Anthropic)**  
Feature beta enero 2026. Resume automáticamente el historial cuando se acerca al límite de contexto. Elegible ZDR.

**Batch API**  
50% de descuento en llamadas asíncronas (resultado en <24h). Anthropic, OpenAI y otros lo ofrecen. Combinado con caching, hasta 95% de ahorro sobre el precio estándar.

**Extended thinking / thinking tokens**  
Tokens que el modelo "piensa" antes de responder (Sonnet 4, Opus 4, o3). Se facturan como **output**, no input. Usar `display:omitted` no ahorra plata — solo oculta los tokens al usuario.

**Auto Mode (Copilot)**  
Feature de GitHub Copilot que rutea automáticamente al modelo más barato que pueda hacer la tarea. 10% de descuento en el multiplicador. Activación: 1 click.

**RouteLLM**  
Proyecto open source de UC Berkeley / LMSYS (ICLR 2025). Drop-in replacement de OpenAI SDK que rutea entre modelos. Reporta hasta 75% de ahorro manteniendo 95% de la calidad de GPT-4.

**LiteLLM**  
Gateway self-hosted multi-modelo. Permite budget controls por equipo, logging unificado, fallbacks. Open source.

**Portkey**  
Gateway managed (SaaS). Soporta 1600+ modelos, guardrails, SOC2. Alternativa enterprise a LiteLLM.

**OpenRouter**  
SaaS que agrega 300+ modelos detrás de una API. Zero infrastructure. Buen punto de entrada al model routing.

**ZDR (Zero Data Retention)**  
Modo en el que el proveedor de AI no retiene los inputs/outputs. Importante para Visma por GDPR. Compaction API es elegible ZDR.

**MACC (Microsoft Azure Consumption Commitment)**  
Compromiso de gasto plurianual con Azure que desbloquea descuentos extra sobre Enterprise Agreement.

**chat.tools.compressOutput.enabled**  
Setting de VSCode 1.120+ que post-procesa el output de terminal antes de enviarlo al modelo. Reduce tokens en flujos agénticos.

**Tool search (VSCode)**  
Feature que reduce los MCPs cargados en contexto buscando solo los relevantes a la query. Visual Studio Magazine reporta hasta 20% de ahorro de tokens.

---

## Personas

**Jonathan Izquierdo**  
Dueño del proyecto. Trabaja en Visma Latam. El idioma primario del proyecto es español por él y su equipo.

**Visma**  
Empresa SaaS europea, multiproducto. Tiene Microsoft Enterprise Agreement, GitHub Enterprise Cloud con Copilot, y equipos en Europa + Latam. La audiencia del artículo es su equipo de ingeniería.
