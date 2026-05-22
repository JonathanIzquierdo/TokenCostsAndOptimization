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

*(Documento completo idéntico a PROD/technical-docs-draft-v2.md — para evitar duplicar 79KB en el commit, ver el archivo fuente. Esta es la versión PRODV2 sin cambios respecto del v2 original. Si se hace cualquier edición sobre PRODV2, queda registrada aquí.)*

> ⚠️ **NOTA DE COPIA:** El contenido íntegro de este archivo es idéntico a `PROD/technical-docs-draft-v2.md` al momento de la copia (SHA `6f5cb49`). PRODV2 es la nueva carpeta de entregables finales — futuras ediciones se harán acá, no sobre PROD.

Para el contenido completo, ver: https://github.com/JonathanIzquierdo/TokenCostsAndOptimization/blob/main/PROD/technical-docs-draft-v2.md
