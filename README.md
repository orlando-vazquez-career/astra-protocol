# ASTRA

**Protocolo de desarrollo Web3 agéntico** — multi-cadena, multi-vendor, independiente.

> *astra*: los astros. *Per aspera ad astra.*

## Qué es

ASTRA es un protocolo de fases para construir en blockchain con agentes de IA: contratos, tokens, dApps que firman, pagos on-chain para agentes, pruebas ZK verificadas en cadena, y sobre todo **despliegues a mainnet que no se pueden deshacer**. Un protocolo de software general modela planificación, ejecución y revisión; no modela la irreversibilidad, la custodia de claves, el dinero en juego, los toolchains por cadena, los estándares con gobernanza propia ni la auditoría específica. ASTRA sí.

Funciona en **Stellar** (Soroban), **Syscoin** (L1 UTXO, NEVM y Rollux), **Base** y cualquier EVM, por familias con adaptadores. Corre con **cualquier runtime de IA** (Claude Code, Codex, Kimi Code, Cursor, Gemini CLI, Antigravity, OpenCode) porque es Markdown más skills en formato abierto y un servidor MCP. Y es **independiente**: no requiere ningún otro protocolo, memoria ni daemon.

## Las siete fases

```
Órbita → Carta → ⸸ Gate 1: Carta aprobada ⸸ → Construcción → Ensayo → Auditoría → ⸸ Gate 2: Mainnet ⸸ → Lanzamiento → Bitácora
```

| Fase | Qué responde |
|---|---|
| **Órbita** | ¿Qué cadena, qué red, qué estándares, qué herramientas hay? (`astra doctor`, `astra chain probe`) |
| **Carta** | ¿Qué construimos exactamente? Diseño que el humano aprueba en el Gate 1. |
| **Construcción** | Implementación con tests locales verdes. |
| **Ensayo** | Deploy a testnet, integración, conformidad, costos medidos. |
| **Auditoría** | Qué puede salir mal con dinero real, con ojos distintos de los que construyeron. |
| **Lanzamiento** | Mainnet una sola vez, bien: firma humana, verificación pública, registro. Gate 2 antes. |
| **Bitácora** | Lo aprendido, escrito donde el próximo sprint lo encuentre. |

Diez axiomas sostienen todo; los dos primeros: **mainnet es irreversible** y **las claves no existen para el agente**. Documento canónico: [`ASTRA-PROTOCOL.md`](./ASTRA-PROTOCOL.md).

## Usarlo en un proyecto

1. Instalar la herramienta: [`astra-cli`](https://github.com/orlando-vazquez-career/astra-cli) (Node ≥ 20, cero dependencias).
2. Traer el protocolo: `astra protocol fetch` (clona este repo en `~/.astra/protocol`).
3. En el repo del proyecto: `astra init --chain stellar-testnet --runtimes all` (o `base-sepolia`, `syscoin-tanenbaum`, ...). Crea `.astra/`, `docs/astra/`, el bloque ASTRA en `AGENTS.md`/`CLAUDE.md`/`GEMINI.md` y las skills en cada runtime.
4. Abrir el runtime de IA e invocar la skill `astra`: dice en qué fase está el proyecto y cuál es el siguiente paso.
5. Antes de mainnet: `astra check --gate mainnet` tiene que salir en verde y el humano firma `docs/astra/launch.md`.

## Estructura del repositorio

```
ASTRA/
├─ ASTRA-PROTOCOL.md          ← documento maestro (fases, gates, axiomas, roles, multi-cadena, multi-vendor)
├─ AGENTS.md · CLAUDE.md · GEMINI.md   ← reglas para los agentes que trabajan en ESTE repo
├─ guides/                    ← gates, claves, auditoría, estándares, pagos agénticos, runtimes, skills externas
│  └─ chains/                 ← stellar, syscoin (L1 + NEVM + Rollux), base, evm-genérico
├─ templates/                 ← orbit, chart, audit, launch, devlog, deployments.schema.json, agents/
├─ skills/                    ← skills canónicas (formato Agent Skills)
├─ .claude/skills · .agents/skills   ← copias generadas por `astra skills sync` (no editar a mano)
├─ docs/                      ← GLOSSARY, MAPA, superpowers/ (spec y plan de la génesis)
└─ genesis/                   ← historia fundacional
```

## Relación con `astra-cli`

El protocolo funciona sin el CLI (cada checklist se puede recorrer a mano). El CLI vuelve mecánico lo que a mano se olvida: capacidad por cadena, sonda en vivo, validación de direcciones, escáner de secretos, registro de despliegues, gate de mainnet, sync de skills y MCP. Nunca firma ni custodia claves.

## Licencia

MIT — DevZen SpA, 2026.
