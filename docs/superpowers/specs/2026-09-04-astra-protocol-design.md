# ASTRA — diseño de génesis (protocolo + herramientas)

**Fecha**: 2026-09-04 · **Estado**: aprobado para implementación (génesis v0.1.0) · **Autor**: el operador, con asistencia de agente.

## 1. Pregunta y veredicto

**¿Se justifica un protocolo exclusivo de desarrollo Web3, separado de un protocolo general de software?** Sí. Un sprint Web3 tiene invariantes que un protocolo general de software no modela y que, si faltan, cuestan dinero real o reputación irreversible:

1. **Irreversibilidad**: un contrato desplegado en mainnet no se "revierte"; un token mal emitido queda emitido. Hace falta un gate de mainnet más duro que cualquier gate de release.
2. **Custodia de claves**: el agente jamás puede ver, pedir ni imprimir claves privadas o frases semilla; el flujo de firma es humano o vive en el keystore del CLI nativo bajo un nombre.
3. **Dinero en juego**: fees, reservas, gas, trustlines. Toda operación tiene costo y hay que estimarlo antes de firmar.
4. **Toolchains por cadena**: Stellar CLI + Rust/Soroban, Foundry/Hardhat para EVM (Base, Syscoin NEVM, Rollux), `syscoin-cli` para el L1 UTXO. Un capability check previo evita sprints que nacen sin herramientas.
5. **Estándares con gobernanza propia**: SEP/CAP en Stellar, ERC/EIP en EVM, docs de Syscoin/Rollux. Usar el estándar antes que inventar.
6. **Auditoría específica**: reentrancy, control de acceso, aritmética, oráculos, TTL de storage en Soroban, upgradeabilidad, front-running. Con auditor distinto del autor.

Estas seis cosas no encajan en las fases de un protocolo general (donde el deploy es reversible y el secreto es un `.env`), ni en un protocolo de diseño, ni en una membrana de routing. Por eso ASTRA existe como protocolo **exclusivo** de Web3 e **independiente**: no depende de ningún otro protocolo ni de ninguna capa de orquestación para funcionar.

## 2. Nombre y forma

- **ASTRA** — del latín *astra*, plural de *astrum*, "los astros". *Per aspera ad astra*. Pronunciación `/ˈas.tra/`.
- Repos públicos: `astra-protocol` (el protocolo) y `astra-cli` (las herramientas). Locales: `C:/dev/protocols/ASTRA` y `C:/dev/tools/astra-cli`.
- Licencia MIT, copyright 2026 DevZen SpA. Idioma: español latino neutro (sin voseo) en docs; código y CLI con mensajes en español, identificadores en inglés.
- Versión inicial 0.1.0. Promueve a 1.0.0 cuando un contrato o dApp llegue a mainnet siguiendo las siete fases de punta a punta, con Gate 2 firmado y registro en `deployments.json`.

## 3. Restricciones no negociables (del pedido)

| Restricción | Cómo se cumple |
|---|---|
| Windows, macOS y Linux | Herramientas en Node ≥ 20, ESM, **cero dependencias**, sin build step; paths con `node:path`; detección de binarios respetando `PATHEXT`; escritura atómica con rename y reintentos. CI en matriz ubuntu/windows/macos. |
| Cualquier vendor de IA | El protocolo es Markdown. Entrypoints `AGENTS.md` (canónico, estándar abierto que leen Codex, Kimi Code, Cursor, OpenCode, Copilot…), `CLAUDE.md` (Claude Code, importa `@AGENTS.md`) y `GEMINI.md` (Gemini CLI / Antigravity). Skills en formato Agent Skills (`skills/<nombre>/SKILL.md`) sincronizadas a `.claude/skills`, `.agents/skills`, `.kimi-code/skills`, `.cursor/skills` por `astra skills sync`. Servidor MCP stdio (`astra mcp`) para cualquier runtime con MCP. Modelos expresados como **tiers** (primary / secondary), nunca IDs de un vendor. |
| Varias cadenas (Stellar, Syscoin, Base, …) | Registro de cadenas `data/chains.json` con adaptadores por **familia**: `stellar`, `evm` (Base, Syscoin NEVM, Rollux y cualquier chainId), `utxo` (Syscoin L1). Guías por cadena. Agregar una cadena = una entrada en el registro + una guía. |
| Independiente | ASTRA no referencia ni requiere ningún otro protocolo, memoria externa, daemon ni membrana. Funciona con solo el repo y, opcionalmente, el CLI. Otros sistemas pueden invocarlo leyendo `ASTRA-PROTOCOL.md` y llamando al CLI; ASTRA no sabe que existen. |
| Sin secretos ni datos personales en lo público | Axioma A2 + `astra check` (escáner de secretos) corre sobre los dos repos antes de cada push. |

## 4. El protocolo

### 4.1 Fases y gates

```
Órbita → Carta → ⸸ Gate 1: Carta aprobada ⸸ → Construcción → Ensayo → Auditoría → ⸸ Gate 2: Mainnet ⸸ → Lanzamiento → Bitácora
```

| # | Fase | Pregunta | Output | Herramienta |
|---|---|---|---|---|
| 1 | **Órbita** | ¿Qué cadena, qué red, qué estándar, qué herramientas hay? | `docs/astra/orbit.md` (perfil de cadena + veredicto de capacidad + modelo de amenazas inicial) | `astra doctor`, `astra chain info/probe`, `astra standards search` |
| 2 | **Carta** | ¿Qué construimos exactamente? | `docs/astra/chart.md` (interfaz, storage, roles, invariantes, economía, estándares, plan de tests, plan de upgrade/pausa) | plantilla `chart.template.md` |
| G1 | **Gate 1** | El humano aprueba la Carta | marca `aprobada: <fecha>` en `chart.md` | — |
| 3 | **Construcción** | Implementar + tests locales | código + tests verdes (unit, property/fuzz donde exista) | toolchain nativo de la cadena |
| 4 | **Ensayo** | ¿Funciona contra una red real? | deploy a **testnet** registrado en `.astra/deployments.json`, tests de integración, conformidad | `astra deployments add`, `astra address`, `astra check` |
| 5 | **Auditoría** | ¿Qué puede salir mal con dinero real? | `docs/astra/audit.md` (checklist por cadena + hallazgos + severidad + estado) | `astra check --gate mainnet`, sub-agente auditor |
| G2 | **Gate 2** | El humano firma el lanzamiento | `docs/astra/launch.md` con checklist firmado, costos estimados, claves nombradas, plan de rollback/pausa | `astra check --gate mainnet` en verde |
| 6 | **Lanzamiento** | Deploy a mainnet + verificación pública | entrada mainnet en `deployments.json` con tx, commit y verificación en explorer | `astra deployments add --verified`, CLI nativo (firma humana) |
| 7 | **Bitácora** | ¿Qué aprendimos y qué queda escrito? | `docs/astra/devlogs/YYYY-MM-DD-<slug>.md` + `deployments.json` al día | plantilla `devlog.template.md` |

### 4.2 Axiomas

- **A1 — Mainnet es irreversible.** Nada llega a mainnet sin Gate 2 firmado por un humano en `launch.md`.
- **A2 — Las claves no existen para el agente.** Nunca lee, pide, imprime ni registra claves privadas, seeds ni mnemónicos. Firma el humano o el keystore del CLI nativo bajo un nombre (`--source-account <alias>`). Claves de testnet y mainnet viven en alias distintos.
- **A3 — Testnet primero, siempre.** Ningún artefacto se lanza a mainnet sin haber pasado Ensayo en la testnet de su cadena.
- **A4 — Toda dirección desplegada queda registrada** en `.astra/deployments.json` con cadena, red, dirección, tx, commit, fecha y estado de verificación.
- **A5 — Estándar antes que invento.** Si existe SEP/CAP/ERC/EIP o guía oficial para el problema, se usa y se cita.
- **A6 — Verificación pública antes de anunciar.** Código fuente verificado en el explorer (o hash de WASM publicado) antes de comunicar una dirección.
- **A7 — Los agentes no mueven fondos de mainnet.** Ni con permiso. Solo el humano firma en mainnet.
- **A8 — Local-first, cero telemetría, cualquier vendor.** Las herramientas no llaman a ningún servidor salvo los RPC/explorers que el usuario pide probar.
- **A9 — Auditor distinto del autor.** La Auditoría la corre un sub-agente o modelo distinto del que construyó; si hay una sola familia de modelos, se declara en `audit.md`.
- **A10 — Costo visible.** Fees, reservas y gas estimados y escritos antes de Gate 2.

### 4.3 Roles (sub-agentes)

| Rol | Fase | Permisos | Tier |
|---|---|---|---|
| `navegante` | orquesta las siete fases; nunca se despacha | plan + coordinación | primary (hilo principal) |
| `cartografo` | Órbita, Carta (investigación de cadena y estándares) | lectura + web | primary |
| `forjador` | Construcción, Ensayo | edición + toolchain | secondary (escala a primary en contratos) |
| `auditor-de-cadena` | Auditoría | solo lectura | primary, familia distinta al forjador si se puede |
| `oficial-de-lanzamiento` | Lanzamiento, Bitácora | CLI nativo con alias de claves; jamás ve secretos | primary |

Regla de la caja: todo despacho lleva objetivo, entradas, criterio de done y límite de turnos. El hilo principal conserva el control.

### 4.4 Multi-cadena: el perfil de cadena

Cada entrada de `data/chains.json` tiene: `id`, `family` (`stellar` | `evm` | `utxo`), `name`, `network` (`mainnet` | `testnet`), `caip2`, `chainId` (EVM), `nativeSymbol`, `rpc[]`, `horizon` (stellar), `passphrase` (stellar), `explorers[]`, `faucets[]`, `docs`, `notes`, `verifiedAt`. Cadenas de génesis: `stellar-mainnet`, `stellar-testnet`, `base`, `base-sepolia`, `syscoin-nevm`, `syscoin-tanenbaum`, `rollux`, `rollux-tanenbaum`, `syscoin-utxo`. Cualquier otra EVM entra por `--rpc <url>` o por una entrada nueva.

Guías por cadena (`guides/chains/`): `stellar.md`, `syscoin.md` (L1 UTXO + NEVM + Rollux), `base.md`, `evm-generico.md`. Cada guía cubre: toolchain, redes y faucets, formato de direcciones, deploy y verificación, estándares, riesgos específicos para la Auditoría.

### 4.5 Multi-vendor

- `AGENTS.md` es la fuente única de reglas del repo. `CLAUDE.md` importa `@AGENTS.md`. `GEMINI.md` remite a `AGENTS.md`.
- Skills canónicas en `skills/`; copias generadas por `astra skills sync` en `.claude/skills` y `.agents/skills` (commiteadas) y en `.kimi-code/skills`, `.cursor/skills` (bajo demanda). Cada copia lleva cabecera "GENERADO … no editar a mano" y `--check` verifica el sync.
- Agentes en `templates/agents/` en formato frontmatter (name/description/tools/model: inherit) con el tier declarado en el cuerpo; cualquier vendor puede usarlos como prompt de rol.
- MCP: `astra mcp` expone las mismas capacidades del CLI como tools; registrable en Claude Code, Codex, Cursor, Kimi Code, Gemini CLI, OpenCode.

## 5. Las herramientas: `astra-cli`

Un solo paquete Node (≥ 20, ESM, cero dependencias) con binario `astra`.

| Comando | Qué hace | Red |
|---|---|---|
| `astra doctor [--chain <id>] [--json]` | Detecta toolchains (node, git, stellar, cargo, forge, cast, anvil, hardhat, syscoin-cli, docker) y emite veredicto por familia: SAFE / CAUTION / AVOID | no |
| `astra chain list / info <id>` | Registro de cadenas | no |
| `astra chain probe <id> [--rpc <url>]` | Salud en vivo: EVM `eth_chainId` + `eth_blockNumber` (valida que el chainId coincida); Stellar `getNetwork` + `getLatestLedger` + raíz de Horizon (valida passphrase) | sí (explícita) |
| `astra address <chain-id\|family> <address>` | Valida formato y checksum: StrKey (SEP-0023, G/C/M) con CRC16; EIP-55 con keccak-256 propio; bech32/bech32m (BIP-173/350) y base58check para Syscoin UTXO. Si detecta una **clave secreta** (StrKey `S…`, hex de 64, mnemónico) se niega a imprimirla y avisa | no |
| `astra init [--chain <id>...] [--runtimes …] [--protocol-dir <p>]` | Crea `.astra/astra.json`, `.astra/deployments.json`, `docs/astra/*.md` desde plantillas, agrega `.env*` a `.gitignore`, inserta bloque ASTRA idempotente en `AGENTS.md`/`CLAUDE.md`/`GEMINI.md` y sincroniza skills a los runtimes elegidos | no |
| `astra check [--gate mainnet] [--json]` | Escáner de secretos (seeds StrKey, claves hex EVM, mnemónicos, `.env` versionados, keystores), higiene (`.gitignore`), validez de `deployments.json`; con `--gate mainnet` verifica además Carta aprobada, entrada testnet, `audit.md` sin hallazgos críticos abiertos y `launch.md` firmado. Exit 1 si falla | no |
| `astra deployments list / add …` | Registro append de despliegues, escritura atómica, validación de dirección por familia | no |
| `astra standards search <q> [--family …]` | Busca en `data/standards.json` (SEP, CAP, ERC, EIP, CAIP, x402, MPP, docs de Syscoin/Rollux/Base) | no |
| `astra skills sync [--from <dir>] [--runtimes …] [--check]` | Copia skills a los directorios de cada runtime; también instala packs externos | no |
| `astra protocol fetch / path` | Clona el repo público del protocolo en `~/.astra/protocol` (con `git`) o muestra dónde lo encontró | sí (explícita) |
| `astra mcp` | Servidor MCP stdio (JSON-RPC 2.0, protocolVersion 2024-11-05): `astra_doctor`, `astra_chain_list`, `astra_chain_info`, `astra_chain_probe`, `astra_address_validate`, `astra_check`, `astra_deployments_list`, `astra_deployments_add`, `astra_standards_search` | solo si la tool lo pide |

Resolución del directorio del protocolo: `--protocol-dir` → `ASTRA_PROTOCOL_DIR` → `~/.astra/protocol` → `../../protocols/ASTRA` relativo al CLI → `./ASTRA`.

Lo que el CLI **no hace**, por diseño: no firma, no despliega, no guarda claves, no llama a APIs de IA, no envía telemetría.

## 6. Tests y calidad

- `node --test test/` con vectores públicos: SEP-0023 (StrKey), direcciones SAC de XLM emitidas por `stellar contract id asset` (testnet `CDLZFC3S…`, mainnet `CAS3J7GY…`), EIP-55 (vectores del EIP), BIP-173 (`bc1qw508d6…`), base58check, keccak-256 de cadena vacía.
- Tests de integración con temporales: `init` idempotente, `check` con secretos plantados (construidos en runtime para que el propio escáner no los encuentre en el repo), `deployments add` atómico, `skills sync --check`, `doctor` con PATH falso, `mcp` end-to-end por stdio.
- CI GitHub Actions: matriz ubuntu-latest / windows-latest / macos-latest × Node 20 / 22.

## 7. Estructura de repos

```
ASTRA/                                   astra-cli/
├─ ASTRA-PROTOCOL.md                      ├─ bin/astra.mjs
├─ README.md · AGENTS.md · CLAUDE.md      ├─ lib/{cli,registry,probe,doctor,address,keccak,
├─ GEMINI.md · LICENSE                    │        check,deployments,init,skills,standards,
├─ skills/<9 skills>/SKILL.md             │        protocol,mcp,util}.mjs
├─ .claude/skills · .agents/skills (gen.) ├─ data/{chains,standards}.json
├─ templates/{orbit,chart,audit,launch,   ├─ test/*.test.mjs
│   devlog}.template.md · deployments     ├─ .github/workflows/ci.yml
│   .schema.json · agents/<5>.md          ├─ package.json · README.md · LICENSE
├─ guides/{gates,keys-and-secrets,        ├─ AGENTS.md · CLAUDE.md
│   audit-checklist,standards-map,        └─ docs/superpowers/…
│   agentic-payments,runtimes,
│   skills-externas}.md · chains/<4>.md
├─ docs/{GLOSSARY,MAPA}.md · superpowers/
└─ genesis/{README.md,devlogs/…}
```

## 8. Integración de skills externas (evaluación pedida)

Las 34 skills de `stellar-build` se reparten así (no se copian a ASTRA: licencias Apache-2.0 / MIT propias; se instalan con `astra skills sync --from <dir>` en el proyecto que las quiera):

- **ASTRA** (Web3): `smart-contracts`, `dapp`, `assets`, `data`, `agentic-payments`, `zk-proofs`, `standards`, `deploy-stellar-mainnet`, `find-stellar-idea`, `stellar-competitive-landscape`, `scf-round-watcher`, `stellar-help`, `navigate-skills`. Mapeadas a fases en `guides/skills-externas.md`.
- **Protocolo general de software** (AEGIS en este ecosistema): `code-review`, `review-edge-case-hunter`, `investigate`, `dev-story`, `create-architecture`, `create-epics-and-stories`, `prd`, `product-brief`, `prfaq`, `brainstorming`, `advanced-elicitation`, `party-mode`, `reprompt` y las personas `elliot-dev`, `tyler-architect`, `justin-analyst`, `nicole-pm`, `bri-tech-writer`.
- **Protocolo de diseño** (LUMEN): `create-ux-design`, `kaan-ux-designer`.
- **Excluida**: `remove-ai-marks` (política de trazabilidad: borrar marcas de IA contradice la bitácora honesta de cualquiera de los protocolos).

## 9. Fuera de alcance de la génesis

- Firmar o enviar transacciones desde el CLI (A2/A7).
- Indexers, bases de datos de eventos, monitoreo continuo post-lanzamiento.
- Adaptadores para cadenas fuera de las tres familias (Solana, Cosmos, Bitcoin L1 puro) — entran por la misma vía (registro + guía) cuando haga falta.
- Formato de agentes Kimi Code / Cursor generado automáticamente (los agentes se usan como prompts de rol; solo las skills se sincronizan).
