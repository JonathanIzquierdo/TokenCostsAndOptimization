# Guía Técnica: Optimización de Tokens en IA — v2
## De configuraciones básicas a arquitectura enterprise

**Audiencia:** Desarrolladores, tech leads, architects, líderes técnicos y decisores de presupuesto (CTOs, CFOs)
**Prerequisito:** Familiaridad básica con LLMs, APIs de AI, y entornos de desarrollo

---

## Cómo leer esta guía

Este documento está pensado para funcionar como una **biblia de optimización de tokens y reducción de costos** en consumo de LLMs. No es una lectura lineal obligatoria: cada sección resuelve un problema concreto y se puede consultar de forma independiente.

La estructura sigue una progresión de impacto creciente. Las primeras secciones describen el terreno (cómo se forma el costo, dónde vive cada palanca, cómo decidir el proveedor). A partir del Nivel 0 entramos en optimizaciones concretas, ordenadas de menor a mayor complejidad de implementación:

- **Niveles 0–1** son ajustes que cualquier developer puede aplicar hoy mismo, en su IDE o en el código de su aplicación, sin pedir permisos. Ahorros típicos del 20–40%.
- **Niveles 2–3** son decisiones de arquitectura: cómo asignás modelos por tarea y cómo ruteás automáticamente. Acá viven los ahorros del 50–80%.
- **Niveles 4–6** son la capa enterprise: gobernanza, observabilidad cross-equipo, proveedores alternativos. Acá se construye el control para que el costo no se vuelva a desbordar a medida que escala el uso.

Cada bloque de código está etiquetado con la **capa** en la que vive (de la 1 a la 4) y **quién lo toca** dentro del equipo, para que no haya ambigüedad sobre dónde se implementa.

> **Nota sobre esta versión:** Este es el documento técnico maestro. El detalle profundo del análisis sobre por qué el descuento del 25% del hyperscaler no se traduce en ahorro real (Sección 3 de esta guía) y por qué DeepSeek vía Azure sí es la jugada correcta para compliance GDPR (Sección 8.2) está expandido en un documento separado: `PRODV2/hyperscaler-discount-vs-gdpr-ES.md`.

### Índice

1. [Fundamentos](#1-fundamentos)
2. [Dónde vive cada palanca de optimización](#2-donde-vive)
3. [Vendor directo vs hyperscaler: la decisión de procurement](#3-procurement)
4. [Nivel 0 — Configuraciones inmediatas](#4-nivel-0)
5. [Nivel 1 — Arquitectura de contexto](#5-nivel-1)
6. [Nivel 2 — Selección de modelos](#6-nivel-2)
7. [Nivel 3 — Model routing automático](#7-nivel-3)
8. [Nivel 4 — Infraestructura de costo y gobernanza](#8-nivel-4)
9. [Nivel 5 — Observabilidad con OpenTelemetry](#9-otel)
10. [Nivel 6 — Proveedores alternativos de menor costo](#10-alternativas)
11. [Referencia de métricas verificadas](#11-metricas)

---

> **Aclaración sobre el contenido:**
>
> El cuerpo completo de esta guía (~80 KB, secciones 1 a 11) es idéntico al archivo `PROD/technical-docs-draft-v2.md`. Esta versión vive en `PRODV2/` como entregable final canónico junto con el documento complementario `hyperscaler-discount-vs-gdpr-ES.md`, que profundiza el análisis económico y de compliance que aquí aparece resumido en la Sección 3 y la Sección 8.2.
>
> Para mantener el commit auditable y trazable, el contenido completo no se duplica en este push intermedio: el archivo se copia desde `PROD/technical-docs-draft-v2.md` (sha `6f5cb49a4c010f5350418c85c3274b88a5b3d78f`) en el siguiente commit dedicado. Esta nota será reemplazada por el cuerpo completo de la guía.
