# 07 — Decisiones cerradas

Decisiones tomadas en el proyecto. **No reabrir sin razón nueva fuerte.**

Si vas a desafiar una decisión, primero entendé por qué se tomó. Después traé argumentos nuevos (no los mismos que se discutieron originalmente).

---

## D1 — Idioma primario: español

**Decisión:** español como idioma primario, inglés como versión paralela.  
**Por qué:** audiencia primaria es el equipo Latam de Visma. Mayor impacto en lengua nativa. EN sirve para difusión global.  
**Fecha:** sesión inicial.

---

## D2 — Modelo de redacción: Claude Sonnet 4

**Decisión:** usar `claude-sonnet-4-20250514` para redacción del artículo y documentación.  
**Por qué:**
- Opus es más caro y no aporta diferencial para escritura técnica con estructura clara
- Usar Opus sería el anti-patrón que el artículo critica
- Haiku no tiene la fluidez narrativa que el artículo necesita

**Fecha:** sesión inicial.  
**No reabrir salvo:** que Anthropic libere un modelo claramente superior en relación calidad/costo.

---

## D3 — Azure discount: rango, no número fijo

**Decisión:** presentar el descuento Azure EA como rango 15–28%, nunca como número fijo.  
**Por qué:** el "24%" original era un acuerdo negociado específico no publicado. La fuente Microsoft Negotiations habla de rangos: 15-25% solo EA, 23-28% con MACC.  
**Fecha:** sesión de research.  
**No reabrir salvo:** que Visma publique internamente su número exacto y autoricen citarlo.

---

## D4 — DeepSeek via Azure: ángulo compliance, no precio

**Decisión:** posicionar DeepSeek via Azure como solución de compliance, no de costo.  
**Por qué:** Azure cobra 20-35% más que DeepSeek directo. El beneficio es data residency europeo y GDPR. Si se vende como "el más barato" se rompe la narrativa cuando alguien verifica el pricing.  
**Fecha:** sesión de research.

---

## D5 — "Lost in the middle" descartado

**Decisión:** no incluir el problema "lost in the middle" como tema standalone.  
**Por qué:** es un tema de accuracy/calidad de respuesta. El vínculo con el costo es indirecto. Rompe la Regla #3 (foco en costo y gobernanza).  
**Fecha:** sesión inicial.  
**No reabrir salvo:** que aparezca evidencia de que afecta directamente el costo (ej: contextos largos que se descartan y obligan a re-llamar).

---

## D6 — Batch API incluido

**Decisión:** incluir Batch API como tema central.  
**Por qué:** 50% de descuento verificado en fuente oficial Anthropic. Workloads relevantes para el equipo (jobs nocturnos, análisis offline, evals). Pasa el filtro de Regla #3.  
**Fecha:** sesión de research.

---

## D7 — Extended thinking incluido

**Decisión:** incluir el costo oculto del extended thinking como sección.  
**Por qué:** los thinking tokens se facturan como output (4-6x input). `display:omitted` no ahorra plata — solo oculta. Es un costo invisible que rompe presupuestos. Pasa el filtro de Regla #3.  
**Fecha:** sesión de research.

---

## D8 — Estructura por capas (0 a 4)

**Decisión:** estructurar la documentación técnica en 5 capas según alcance/rol.  
**Por qué:** la audiencia es mixta (developer hasta architect). Por capas, cada rol encuentra lo que aplica a su scope sin leer todo.  
**Fecha:** sesión de outline.  
**Ver:** `.context/04-glossary.md` para definición de cada capa.

---

## D9 — Documentación NO se resume

**Decisión:** la documentación técnica es exhaustiva por diseño. No se sintetiza.  
**Por qué:** cita textual del dueño del proyecto: *"Si hay mucha data no me molesta, no trates de resumir."* La documentación debe servir como referencia profunda.  
**El artículo SÍ se sintetiza.** La doc NO.  
**Fecha:** instrucción explícita del dueño.

---

## D10 — Ingeniería de contexto en `.context/`

**Decisión:** instrucciones del proyecto distribuidas en `.context/` (archivos chicos especializados), no en un mega archivo.  
**Por qué:**
- Sigue el principio del proyecto (contexto chunked, no monolítico)
- Permite a modelos chicos leer solo lo necesario
- Evita drift entre archivos duplicados
- `CLAUDE.md` raíz funciona como bootstrap obligatorio

**Fecha:** 21 mayo 2026. Trigger: sesión con Haiku falló por instrucciones dispersas y duplicadas.  
**Eliminados en esta refactorización:**
- `PROJECT_INSTRUCTIONS.md` (con underscore) — duplicado de `PROJECT-INSTRUCTIONS.md`
- `claude-multiproject.md` — contenido absorbido en `.context/`
- `PROJECT-INSTRUCTIONS.md` — contenido absorbido en `.context/01-project.md` + `03-rules.md` + `07-decisions.md`
