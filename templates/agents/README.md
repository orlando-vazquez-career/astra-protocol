# Agentes de ASTRA

Cinco roles. Cada archivo tiene frontmatter (`name`, `description`, `tools`, `model: inherit`) para runtimes con sub-agentes (Claude Code los lee desde `.claude/agents/`; Kimi Code desde `.kimi-code/agents/` con `model_preference`) y un cuerpo que sirve como **prompt de rol** en cualquier otro vendor. `model: inherit` es deliberado: el tier se declara en el cuerpo y el modelo concreto lo decide la configuración del runtime (ver `guides/runtimes.md`).

| Rol | Fases | Permisos | Tier |
|---|---|---|---|
| `navegante` | orquesta las siete; nunca se despacha | plan y coordinación | primary (hilo principal) |
| `cartografo` | Órbita, Carta | lectura, web, `astra` | primary |
| `forjador` | Construcción, Ensayo | edición, toolchain nativo, alias de testnet | secondary (sube a primary en contratos con dinero) |
| `auditor-de-cadena` | Auditoría | solo lectura | primary; otra familia que el forjador si es posible (A9) |
| `oficial-de-lanzamiento` | Lanzamiento, Bitácora | CLI nativo con alias; jamás ve una clave | primary |

## La regla de la caja

Todo despacho lleva **objetivo**, **entradas** (qué archivos leer), **criterio de done** (qué tiene que existir al terminar) y **límite** (turnos o tiempo). El hilo principal (`navegante`) conserva el control; un sub-agente no despacha a otro.

## Uso

- Claude Code: copiar los cinco archivos a `<repo>/.claude/agents/`.
- Kimi Code: convertir `model: inherit` en `model_preference: primary|secondary` según el tier del cuerpo y copiar a `.kimi-code/agents/`.
- Codex, Cursor, Gemini CLI, OpenCode: pegar el cuerpo como prompt del rol cuando se delega la fase.

Ninguno de los cinco lee, pide ni imprime claves (A2). Ninguno mueve fondos de mainnet (A7).
