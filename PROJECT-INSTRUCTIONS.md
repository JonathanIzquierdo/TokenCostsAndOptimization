# [DEPRECATED] PROJECT-INSTRUCTIONS.md

Este archivo fue reemplazado por el sistema de ingeniería de contexto en `.context/` el 21 de mayo 2026.

**No leer este archivo.** Su contenido fue migrado y mejorado en:

- **Bootstrap obligatorio:** [`CLAUDE.md`](./CLAUDE.md) (raíz del repo)
- **Protocolo de arranque:** [`.context/00-START-HERE.md`](./.context/00-START-HERE.md)
- **Qué es el proyecto:** [`.context/01-project.md`](./.context/01-project.md)
- **Estado vivo:** [`.context/02-state.md`](./.context/02-state.md)
- **Reglas:** [`.context/03-rules.md`](./.context/03-rules.md)
- **Decisiones cerradas:** [`.context/07-decisions.md`](./.context/07-decisions.md)
- **Prompts de arranque:** [`.context/08-prompts.md`](./.context/08-prompts.md)

---

Razón del cambio: el archivo monolítico no permitía a modelos chicos (Haiku, GPT-mini, Flash) leer solo el contexto necesario para su tarea. La nueva estructura sigue el principio del proyecto: contexto chunked, no monolítico.
