# Runtimes de IA — cómo corre ASTRA en cada vendor

ASTRA es Markdown más skills en el formato abierto **Agent Skills** (`<nombre>/SKILL.md`, https://agentskills.io) más un servidor **MCP** (`astra mcp`). Ningún archivo del protocolo nombra un modelo concreto: los roles hablan de **tiers**.

## Tiers

| Tier | Para qué | Cómo se liga |
|---|---|---|
| `primary` | Órbita, Carta, Auditoría, Lanzamiento: investigación, diseño, juicio | el modelo más capaz que ofrezca el vendor en uso |
| `secondary` | Construcción mecánica, tests repetitivos, formateo | el modelo económico del vendor |

La regla A9 (auditor distinto del autor) se cumple mejor cruzando **familias**: si el forjador corrió en un vendor, el auditor corre en otro. Si solo hay uno, `audit.md` lo declara.

## Tabla por runtime

| Runtime | Archivo de reglas que lee | Directorio de skills | Registro del MCP | Sub-agentes |
|---|---|---|---|---|
| **Claude Code** | `CLAUDE.md` (importa `@AGENTS.md`) | `.claude/skills/` | `claude mcp add astra -- node <astra-cli>/bin/astra.mjs mcp` | `.claude/agents/*.md` (copiar `templates/agents/`) |
| **Codex** | `AGENTS.md` | `.agents/skills/` | `codex mcp add astra -- node <astra-cli>/bin/astra.mjs mcp` | prompts de rol |
| **Antigravity / Gemini CLI** | `GEMINI.md` (remite a `AGENTS.md`) | `.agents/skills/` | JSON `mcpServers` en la configuración del runtime | prompts de rol |
| **Kimi Code** | `AGENTS.md` | `.kimi-code/skills/` | `~/.kimi-code/mcp.json` | `.kimi-code/agents/` (frontmatter `model_preference: primary\|secondary`) |
| **Cursor** | `AGENTS.md` | `.cursor/skills/` | `.cursor/mcp.json` | prompts de rol |
| **OpenCode** | `AGENTS.md` | `.claude/skills/` o `.agents/skills/` | `opencode.json` | prompts de rol |

JSON de MCP común a Cursor, Kimi Code, Gemini CLI y OpenCode (adaptar la ruta; en Windows con `/` o `\\\\`):

```json
{ "mcpServers": { "astra": { "command": "node", "args": ["/ruta/a/astra-cli/bin/astra.mjs", "mcp"] } } }
```

## Preparar un proyecto

```bash
astra protocol fetch                     # una vez por máquina
astra init --chain base-sepolia --runtimes all
astra skills sync --check                # verifica que las copias estén al día
```

`--runtimes` acepta `claude`, `codex`, `antigravity`, `kimi`, `cursor` o `all`. Codex y Antigravity comparten `.agents/skills/`.

## Prueba de humo por runtime

1. Abrir el proyecto en el runtime.
2. Pedir: "usa la skill astra" → tiene que decir en qué fase está el proyecto y el siguiente paso.
3. Pedir: "corre astra doctor para base-sepolia" → tiene que ejecutar el CLI (o la tool MCP `astra_doctor`) y reportar el veredicto.
4. Pedir: "valida esta dirección 0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed en base" → `OK ... checksum ok`.
5. Pegar una clave de prueba **de testnet recién generada y descartable** y pedir que la valide → tiene que negarse a imprimirla y recomendar rotarla. Después, rotarla igual.

## Reglas para el propio runtime

- El bloque `<!-- astra:start --> ... <!-- astra:end -->` que `astra init` deja en `AGENTS.md` / `CLAUDE.md` / `GEMINI.md` es lo mínimo que todo agente tiene que leer. No se edita a mano: se regenera con `astra init`.
- Las skills copiadas llevan cabecera GENERADO; se editan en `skills/` del protocolo (o en el pack de origen) y se vuelven a sincronizar.
- Un runtime que no soporte skills usa el bloque de reglas y las guías; el protocolo no depende de las skills para funcionar.
