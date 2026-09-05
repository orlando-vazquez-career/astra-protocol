# AGENTS.md — ASTRA (repositorio del protocolo)

Reglas para cualquier agente de IA que trabaje en este repositorio: Claude Code (lee esto vía `CLAUDE.md`), Codex, Kimi Code, Cursor, Gemini CLI / Antigravity (vía `GEMINI.md`), OpenCode, Copilot. No hay otra fuente de reglas.

## Qué es este repo

Es **el protocolo**, no un proyecto adherente. Contiene el documento maestro `ASTRA-PROTOCOL.md`, las guías, las plantillas de artefactos, el roster de agentes y las skills canónicas. Un proyecto Web3 adhiere a ASTRA copiando estos activos con `astra init` (herramienta en el repo `astra-cli`). ASTRA es independiente: no depende de ningún otro protocolo, memoria externa ni daemon.

## Dónde está cada cosa

- `ASTRA-PROTOCOL.md` — fases, gates, axiomas, roles, multi-cadena, multi-vendor. La verdad del protocolo.
- `guides/` — cómo ejecutar cada parte (gates, claves, auditoría, estándares, pagos agénticos, runtimes, skills externas) y una guía por cadena en `guides/chains/`.
- `templates/` — plantillas de `orbit.md`, `chart.md`, `audit.md`, `launch.md`, devlog, esquema JSON del registro de despliegues y agentes (`templates/agents/`).
- `skills/` — skills canónicas (formato Agent Skills, `<nombre>/SKILL.md`). `.claude/skills/` y `.agents/skills/` son **copias generadas**.
- `docs/` — glosario, mapa del repo, `docs/superpowers/` (spec y plan de la génesis).
- `genesis/` — historia fundacional; no se reescribe.

## Reglas duras

1. **Idioma**: español latino neutro, sin voseo, con acentos. Identificadores y comandos en inglés.
2. **Nunca** escribir claves, semillas, mnemónicos, direcciones de mainnet de terceros ni datos personales. Los ejemplos usan los vectores públicos de SEP-0023, EIP-55 y BIP-173 o las direcciones del SAC de XLM.
3. **Toda URL, chainId, RPC o estándar nuevo se verifica en vivo** antes de escribirlo, y la guía anota la fecha de verificación. Título y estado de un estándar se copian de la fuente oficial.
4. **Skills**: se edita `skills/<nombre>/SKILL.md` y después se corre `astra skills sync --from skills --runtimes claude,codex` (o `node <astra-cli>/bin/astra.mjs skills sync ...`). Las copias en `.claude/skills` y `.agents/skills` llevan cabecera GENERADO y no se editan a mano; `--check` verifica el sync.
5. **Independencia**: este repo no menciona ni depende de otros protocolos, sistemas de memoria, daemons ni membranas. Si otro sistema quiere invocar a ASTRA, lo hace leyendo `ASTRA-PROTOCOL.md` y llamando al CLI.
6. **Versionado**: SemVer del protocolo en la cabecera de `ASTRA-PROTOCOL.md` + fila en *Version history* por cada cambio de comportamiento. Los axiomas solo cambian con bump MAJOR.
7. **Commits**: prefijos `docs:`, `feat:`, `fix:`, `chore:`; cuerpo en español; un cambio por commit.
8. Antes de proponer un commit: `astra check` en verde sobre este repo y `astra skills sync --check` en verde.
