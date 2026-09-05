# ASTRA — Protocolo de desarrollo Web3 agéntico

**Versión**: 0.1.0 · **Génesis**: 2026-09-04
**Pronunciación**: `/ˈas.tra/` — del latín *astra*, plural de *astrum*, "los astros". *Per aspera ad astra*: por lo áspero, hacia las estrellas.
**Licencia**: [MIT](./LICENSE).
**Repositorios**: [`astra-protocol`](https://github.com/orlando-vazquez-career/astra-protocol) (este documento y sus activos) · [`astra-cli`](https://github.com/orlando-vazquez-career/astra-cli) (herramienta `astra`: CLI y servidor MCP).

> **Por qué 0.1.x** — el protocolo nace de una necesidad concreta: los sprints Web3 (contratos, tokens, dApps, despliegues a mainnet) se estaban conduciendo con protocolos de software general, cuyos gates suponen que un deploy se revierte. En cadena no se revierte. ASTRA promueve a 1.0.0 cuando un contrato o dApp llegue a mainnet siguiendo las siete fases de punta a punta, con Gate 2 firmado y su dirección registrada en `deployments.json`. Hasta entonces, todo es bootstrap y se ajusta con lo que la realidad enseñe.

---

## TL;DR

ASTRA es un protocolo de fases para construir en blockchain con agentes de IA. Es **exclusivo de Web3** (no cubre software general), **multi-cadena** (Stellar, Syscoin, Base y cualquier EVM, por familias con adaptadores), **multi-vendor** (funciona con cualquier runtime de IA: Claude Code, Codex, Kimi Code, Cursor, Gemini CLI, Antigravity, OpenCode) e **independiente** (no requiere ningún otro protocolo, memoria ni daemon).

Siete fases, dos gates humanos:

```
Órbita → Carta → ⸸ Gate 1: Carta aprobada ⸸ → Construcción → Ensayo → Auditoría → ⸸ Gate 2: Mainnet ⸸ → Lanzamiento → Bitácora
```

| # | Fase | Pregunta | Output | Herramienta |
|---|---|---|---|---|
| 1 | **Órbita** | ¿Qué cadena, qué red, qué estándar, qué herramientas hay? | `docs/astra/orbit.md`: perfil de cadena, veredicto de capacidad, modelo de amenazas inicial | `astra doctor`, `astra chain info/probe`, `astra standards search` |
| 2 | **Carta** | ¿Qué construimos exactamente? | `docs/astra/chart.md`: interfaz, storage, roles, invariantes, economía, estándares, plan de tests, plan de upgrade/pausa | plantilla `chart.template.md` |
| G1 | **Gate 1** | El humano aprueba la Carta | línea `aprobada: YYYY-MM-DD` en `chart.md` | — |
| 3 | **Construcción** | Implementar con tests locales | código + tests verdes (unitarios, de propiedad/fuzz donde exista) | toolchain nativo de la cadena |
| 4 | **Ensayo** | ¿Funciona contra una red real? | deploy a **testnet** registrado en `.astra/deployments.json`, tests de integración, conformidad con el estándar | `astra deployments add`, `astra address`, `astra check` |
| 5 | **Auditoría** | ¿Qué puede salir mal con dinero real? | `docs/astra/audit.md`: checklist por cadena, hallazgos con severidad y estado, `veredicto: apto` | `astra check --gate mainnet`, sub-agente auditor |
| G2 | **Gate 2** | El humano firma el lanzamiento | `docs/astra/launch.md`: checklist, costo estimado, alias de claves, plan de rollback, `firmado_por` y `fecha_firma` | `astra check --gate mainnet` en verde |
| 6 | **Lanzamiento** | Deploy a mainnet y verificación pública | entrada mainnet en `deployments.json` con tx, commit y verificación en explorer | CLI nativo (firma humana), `astra deployments add --verified` |
| 7 | **Bitácora** | ¿Qué aprendimos y qué queda escrito? | `docs/astra/devlogs/YYYY-MM-DD-<slug>.md`, registro al día | plantilla `devlog.template.md` |

---

## El hueco que ASTRA llena

Un protocolo general de software modela bien la planificación, la ejecución y la revisión de código. No modela seis invariantes que en Web3 cuestan dinero real o reputación irreversible:

1. **Irreversibilidad.** Un contrato desplegado en mainnet no se revierte; un token mal emitido queda emitido; una dirección publicada queda publicada. El gate de mainnet tiene que ser más duro que cualquier gate de release.
2. **Custodia de claves.** El agente jamás puede ver, pedir, imprimir ni registrar claves privadas o frases semilla. El flujo de firma es humano o vive en el keystore del CLI nativo bajo un alias.
3. **Dinero en juego.** Fees, gas, reservas mínimas, trustlines. Toda operación tiene costo, y hay que estimarlo antes de firmar.
4. **Toolchains por cadena.** Stellar CLI + Rust/Soroban; Foundry o Hardhat para EVM (Base, Syscoin NEVM, Rollux); `syscoin-cli` para el L1 UTXO. Sin un capability check previo, el sprint nace sin herramientas.
5. **Estándares con gobernanza propia.** SEP y CAP en Stellar, ERC y EIP en EVM, documentación oficial de Syscoin, Rollux y Base. Se usa el estándar antes de inventar.
6. **Auditoría específica.** Reentrancy, control de acceso, aritmética, oráculos, front-running, upgradeabilidad, TTL del storage en Soroban. Y la corre alguien distinto de quien construyó.

Ninguna de las seis encaja en un protocolo de desarrollo general, en uno de diseño ni en una capa de enrutamiento. Por eso ASTRA es un protocolo aparte, y por eso es independiente: tiene que poder correr en cualquier equipo, con cualquier runtime de IA, sin pedirle nada a nadie.

---

## Axiomas

Los axiomas no se negocian dentro de un sprint. Cambiarlos exige un bump MAJOR del protocolo.

- **A1 — Mainnet es irreversible.** Nada llega a mainnet sin Gate 2 firmado por un humano en `launch.md`.
- **A2 — Las claves no existen para el agente.** Nunca lee, pide, imprime ni registra claves privadas, semillas ni mnemónicos. Firma el humano, o el keystore del CLI nativo bajo un alias (`--source-account <alias>`, `cast wallet <alias>`). Las claves de testnet y mainnet viven en alias distintos.
- **A3 — Testnet primero, siempre.** Ningún artefacto se lanza a mainnet sin haber pasado Ensayo en la testnet de su cadena.
- **A4 — Toda dirección desplegada queda registrada** en `.astra/deployments.json` con cadena, red, dirección, tx, commit, fecha y estado de verificación.
- **A5 — Estándar antes que invento.** Si existe un SEP, CAP, ERC, EIP o guía oficial para el problema, se usa y se cita en la Carta.
- **A6 — Verificación pública antes de anunciar.** Código fuente verificado en el explorer (o hash de WASM publicado y reproducible) antes de comunicar una dirección.
- **A7 — Los agentes no mueven fondos de mainnet.** Ni con permiso. Solo el humano firma en mainnet.
- **A8 — Local-first, cero telemetría, cualquier vendor.** Las herramientas no llaman a ningún servidor salvo los RPC y explorers que el usuario pide sondear.
- **A9 — Auditor distinto del autor.** La Auditoría la corre un sub-agente o modelo distinto del que construyó; si solo hay una familia de modelos disponible, se declara en `audit.md`.
- **A10 — Costo visible.** Fees, reservas y gas estimados y escritos en `launch.md` antes de Gate 2.

---

## Cuándo usar ASTRA

### Sí

- Un contrato inteligente nuevo o una modificación a uno existente (Soroban, Solidity/Vyper en cualquier EVM).
- Emisión de un activo o token (activo clásico de Stellar con SAC, SEP-0041, ERC-20/721/1155).
- Una dApp o backend que **firma o envía transacciones** (wallets, pagos, custodia, indexación con escritura).
- Integración de pagos on-chain para agentes (x402, MPP) o de pruebas ZK verificadas en cadena.
- Cualquier despliegue a testnet o mainnet, aunque el código ya exista.

### No

- Backend, API o frontend sin ninguna interacción con una cadena → protocolo general de software.
- Scripts de análisis off-chain de solo lectura (dashboards sobre Horizon o un explorer) → protocolo general, con una nota de Órbita si se quiere.
- Diseño visual de la dApp sin lógica de firma → protocolo de diseño; ASTRA entra cuando aparece la wallet.

### Composición con otros protocolos

Un sprint mixto (backend general + contrato) corre ASTRA para la parte en cadena y el protocolo general para el resto, con **los gates humanos el mismo día** para que la persona revise los dos planes juntos. ASTRA no sabe cuál es ese otro protocolo ni lo necesita: sus artefactos viven en `docs/astra/` y `.astra/` y no colisionan con nada.

---

## Las siete fases

### Fase 1 — Órbita

**Objetivo**: saber en qué cielo se vuela antes de dibujar nada.

**Entradas**: la idea (una frase), la cadena o cadenas candidatas, la red objetivo.

**Mecanismo**:

1. Elegir la cadena y la red del registro (`astra chain list`, `astra chain info <id>`) y sondearla en vivo (`astra chain probe <id>`): chainId o passphrase correctos, bloque o ledger avanzando.
2. Correr `astra doctor --chain <id>`: veredicto **SAFE / CAUTION / AVOID** por familia. Con AVOID no se planifica: se instala primero.
3. Ubicar los estándares aplicables (`astra standards search "<tema>"`) y leer la guía de la cadena en `guides/chains/`.
4. Escribir el **modelo de amenazas inicial**: activos en juego (fondos, roles, datos), atacantes plausibles (usuario malicioso, operador comprometido, front-runner, oráculo manipulado), superficies (funciones públicas, upgrades, claves).
5. Decisión go / no-go con una línea de justificación.

**Output**: `docs/astra/orbit.md` (plantilla `templates/orbit.template.md`).

**Done**: hay cadena, red, veredicto de capacidad, lista de estándares y modelo de amenazas escritos, y el go/no-go está firmado por la persona.

### Fase 2 — Carta

**Objetivo**: el diseño completo de lo que se construye, en un documento que un auditor pueda revisar después contra el código.

**Mecanismo**: completar `docs/astra/chart.md` con interfaz pública (funciones, eventos, errores), modelo de storage (claves, tipos, TTL si aplica), roles y permisos (quién puede qué; quién administra, pausa, actualiza), invariantes (lo que nunca puede pasar: "el total emitido nunca supera el cap"), economía (fees estimadas por operación, reservas, límites), estándares que se implementan (con enlaces), plan de tests (unitarios, de propiedad, de integración en testnet), plan de upgrade / pausa / rollback, y qué queda explícitamente fuera.

**Output**: `docs/astra/chart.md`.

⸸ **Gate 1 — Carta aprobada** ⸸ La persona lee la Carta completa y, si la aprueba, escribe la línea `aprobada: YYYY-MM-DD` en la cabecera del archivo. Sin esa línea no hay Construcción. Si la rechaza, anota qué cambia y la Carta vuelve a Fase 2. Detalle en `guides/gates.md`.

### Fase 3 — Construcción

**Objetivo**: implementar lo que dice la Carta, con tests locales verdes.

**Mecanismo**:

1. El toolchain nativo de la cadena: `stellar contract build` y `cargo test` en Soroban; `forge build` y `forge test` (o Hardhat) en EVM.
2. Tests unitarios de cada función pública; tests de propiedad o fuzz sobre los invariantes de la Carta donde el toolchain lo permita (`cargo fuzz`, `forge test --fuzz-runs`).
3. Nada de claves en el código ni en los tests: las cuentas de prueba se generan en runtime.
4. Cada desviación respecto de la Carta se anota en la propia Carta (sección "Cambios durante la construcción"), no se deja implícita.

**Output**: código y tests en el repo; commits con prefijo `astra:`.

**Done**: tests locales verdes, `astra check` sin errores, Carta al día.

### Fase 4 — Ensayo

**Objetivo**: probar contra una red real sin dinero real.

**Mecanismo**:

1. Fondos de testnet (friendbot en Stellar; faucets en Base Sepolia, Syscoin Tanenbaum) a un alias de testnet.
2. Deploy a testnet con el CLI nativo firmando con ese alias. La dirección resultante se valida (`astra address <cadena> <dirección>`) y se registra (`astra deployments add --chain <id-testnet> --address ... --tx ... --label ...`).
3. Tests de integración contra la testnet: cada función de la interfaz, cada rol, cada invariante, incluyendo los caminos de error.
4. Conformidad con el estándar elegido (por ejemplo, que un token SEP-0041 responda a toda la interfaz; que un ERC-20 pase la suite de referencia).
5. Medir costos reales por operación en testnet y anotarlos: son el insumo de A10.

**Output**: entrada `network: testnet` en `deployments.json`, resultados de integración y costos medidos en `orbit.md` o en la Carta.

**Done**: `astra check --gate mainnet` muestra `testnet-desplegada` en OK.

### Fase 5 — Auditoría

**Objetivo**: encontrar lo que puede salir mal con dinero real, con ojos distintos de los que construyeron.

**Mecanismo**:

1. Un sub-agente `auditor-de-cadena` (o un modelo de otra familia; A9) recibe la Carta, el código, los tests y el registro de testnet. Solo lectura.
2. Recorre el checklist universal y el de la familia (`guides/audit-checklist.md`): control de acceso, reentrancy, aritmética, validación de entradas, oráculos, front-running, upgradeabilidad, pausas y límites, eventos, dependencias; en Soroban además TTL del storage y `require_auth`; en EVM `delegatecall`, proxies y aprobaciones; en L2 el secuenciador y los retiros.
3. Cada hallazgo entra a la tabla de `audit.md` con ID, severidad (`crítica`, `alta`, `media`, `baja`), estado (`abierta`, `cerrada`, `aceptada`), descripción y fix.
4. Los hallazgos críticos y altos se cierran en Construcción y se vuelven a probar en Ensayo antes de seguir.
5. El auditor escribe `veredicto: apto` o `veredicto: no-apto` y declara qué modelo o familia usó.

**Output**: `docs/astra/audit.md`.

⸸ **Gate 2 — Mainnet** ⸸ El gate más duro del protocolo. `astra check --gate mainnet` verifica mecánicamente: Carta aprobada, testnet registrada, auditoría apta sin críticas ni altas abiertas, `launch.md` firmado y cero secretos en el repo. La persona completa `docs/astra/launch.md`: checklist recorrido, costo estimado (A10), alias de la clave de mainnet (distinto del de testnet, A2), plan de pausa / upgrade / rollback, quién firma y cuándo. Sin `firmado_por` y `fecha_firma` no hay Lanzamiento. Detalle en `guides/gates.md`.

### Fase 6 — Lanzamiento

**Objetivo**: llegar a mainnet una sola vez, bien.

**Mecanismo**:

1. La persona (o el `oficial-de-lanzamiento` con el alias de mainnet, nunca con la clave) ejecuta el deploy con el CLI nativo. Los agentes no mueven fondos de mainnet (A7).
2. La dirección se valida y se registra: `astra deployments add --chain <id-mainnet> --address ... --tx ... --commit <sha> --label ...`.
3. Verificación pública (A6): código fuente verificado en el explorer (`forge verify-contract`, Blockscout, Basescan) o hash de WASM publicado con build reproducible (SEP-0055). Recién entonces `--verified --verification-url <url>`.
4. Prueba de humo en mainnet con el monto mínimo posible, ejecutada por la persona.
5. Anuncio de la dirección solo después de 3 y 4.

**Output**: entrada `network: mainnet, verified: true` en `deployments.json`.

### Fase 7 — Bitácora

**Objetivo**: que lo aprendido quede escrito donde el próximo sprint lo encuentre.

**Mecanismo**: un devlog (`templates/devlog.template.md`) con fecha, fases recorridas, direcciones registradas, costos reales versus estimados, hallazgos de auditoría y cómo se cerraron, lo que se dejó fuera y la deuda que queda. Se revisa que `deployments.json`, `orbit.md`, `chart.md` y `audit.md` digan la verdad del estado final.

**Output**: `docs/astra/devlogs/YYYY-MM-DD-<slug>.md`.

**Done**: nada de lo escrito contradice lo desplegado.

---

## Roles

| Rol | Fases | Permisos | Tier |
|---|---|---|---|
| `navegante` | orquesta las siete fases; nunca se despacha | plan y coordinación | primary (hilo principal) |
| `cartografo` | Órbita, Carta | lectura + web + `astra` | primary |
| `forjador` | Construcción, Ensayo | edición + toolchain nativo + alias de testnet | secondary (sube a primary en contratos con dinero) |
| `auditor-de-cadena` | Auditoría | solo lectura | primary, de otra familia que el forjador si es posible (A9) |
| `oficial-de-lanzamiento` | Lanzamiento, Bitácora | CLI nativo con alias; jamás ve una clave | primary |

Definiciones en `templates/agents/`, en formato frontmatter (`name`, `description`, `tools`, `model: inherit`) con el tier declarado en el cuerpo. Un runtime sin sub-agentes las usa como prompts de rol.

**La regla de la caja**: todo despacho a un sub-agente lleva cuatro cosas: objetivo, entradas, criterio de done y límite (turnos o tiempo). El hilo principal conserva el control; un sub-agente no despacha a otro.

**Tiers**: `primary` es el modelo más capaz que ofrece el vendor en uso (investigación, diseño, auditoría, lanzamiento); `secondary` es el modelo económico (construcción mecánica, tests repetitivos). Los IDs concretos viven en la configuración de cada runtime (`guides/runtimes.md`), nunca en el protocolo.

---

## Multi-cadena: el perfil de cadena

Cada cadena que ASTRA conoce es una entrada del registro `data/chains.json` de `astra-cli`, con estos campos: `id`, `family` (`stellar` | `evm` | `utxo`), `name`, `network` (`mainnet` | `testnet`), `caip2` (CAIP-2), `chainId` (EVM), `nativeSymbol`, `rpc[]`, `horizon` y `passphrase` (Stellar), `explorers[]`, `faucets[]`, `docs`, `notes`, `verifiedAt`.

La **familia** decide lo que cambia: cómo se validan direcciones (StrKey; EIP-55; bech32/base58), cómo se firma (Stellar CLI con alias; `cast`/Foundry o Hardhat; `syscoin-cli`), cómo se sondea la red (passphrase; chainId; nodo local), cómo se verifica públicamente (hash de WASM y SEP-0055; verificación de fuente en Blockscout/Basescan) y qué riesgos mira la Auditoría.

Cadenas de génesis (verificadas en vivo el 2026-09-04):

| id | familia | red | chainId / CAIP-2 |
|---|---|---|---|
| `stellar-mainnet` | stellar | mainnet | `stellar:pubnet` |
| `stellar-testnet` | stellar | testnet | `stellar:testnet` |
| `base` | evm | mainnet | 8453 · `eip155:8453` |
| `base-sepolia` | evm | testnet | 84532 · `eip155:84532` |
| `syscoin-nevm` | evm | mainnet | 57 · `eip155:57` |
| `syscoin-tanenbaum` | evm | testnet | 5700 · `eip155:5700` |
| `rollux` | evm | mainnet | 570 · `eip155:570` |
| `rollux-tanenbaum` | evm | testnet | 57000 · `eip155:57000` |
| `syscoin-utxo` | utxo | mainnet | nodo local |

Guías por cadena en `guides/chains/`: `stellar.md`, `syscoin.md` (L1 UTXO, NEVM y Rollux), `base.md`, `evm-generico.md`.

**Agregar una cadena** = una entrada en el registro con sus URLs sondeadas en vivo y `verifiedAt`, más una guía en `guides/chains/`. Si es EVM, funciona de inmediato con todo el CLI; si es una familia nueva (Solana, Cosmos, Bitcoin puro), hace falta además un adaptador de direcciones y de sonda en `astra-cli`.

---

## Multi-vendor: cualquier runtime de IA

ASTRA es Markdown. Lo que cambia por runtime es dónde se leen las reglas y las skills, y cómo se registra el MCP:

| Runtime | Reglas | Skills | MCP |
|---|---|---|---|
| Claude Code | `CLAUDE.md` (importa `@AGENTS.md`) | `.claude/skills/` | `claude mcp add astra -- node <astra-cli>/bin/astra.mjs mcp` |
| Codex | `AGENTS.md` | `.agents/skills/` | `codex mcp add astra -- node <astra-cli>/bin/astra.mjs mcp` |
| Antigravity / Gemini CLI | `GEMINI.md` → `AGENTS.md` | `.agents/skills/` | JSON `mcpServers` |
| Kimi Code | `AGENTS.md` | `.kimi-code/skills/` | `~/.kimi-code/mcp.json` |
| Cursor | `AGENTS.md` | `.cursor/skills/` | `.cursor/mcp.json` |
| OpenCode | `AGENTS.md` | `.claude/skills/` o `.agents/skills/` | `opencode.json` |

- `AGENTS.md` es la única fuente de reglas de un repo adherente; `astra init` inserta en él (y en `CLAUDE.md` y `GEMINI.md`) un bloque idempotente entre `<!-- astra:start -->` y `<!-- astra:end -->`.
- Las skills canónicas viven en `skills/` de este repo, en el formato abierto Agent Skills (`<nombre>/SKILL.md` con frontmatter `name` y `description`). `astra skills sync` las copia a los directorios de cada runtime con una cabecera GENERADO; `--check` detecta desvíos. Sin symlinks, para que funcione en Windows.
- Los agentes de `templates/agents/` usan `model: inherit` y declaran el tier en el cuerpo, así ningún ID de modelo de un vendor queda escrito en el protocolo.
- El servidor MCP (`astra mcp`) expone las mismas capacidades del CLI como tools: `astra_doctor`, `astra_chain_list`, `astra_chain_info`, `astra_chain_probe`, `astra_address_validate`, `astra_check`, `astra_deployments_list`, `astra_deployments_add`, `astra_standards_search`.

Detalle y pruebas de humo por runtime en `guides/runtimes.md`.

---

## Herramientas: `astra`

| Comando | Fase | Qué hace |
|---|---|---|
| `astra doctor [--chain <id>]` | Órbita | Toolchains presentes y veredicto SAFE / CAUTION / AVOID por familia |
| `astra chain list` · `info <id>` · `probe <id> [--rpc <url>]` | Órbita | Registro de cadenas y salud en vivo (valida chainId o passphrase) |
| `astra standards search "<tema>" [--family ...]` | Órbita, Carta | Catálogo local de SEP, CAP, ERC, EIP, CAIP, x402, MPP y docs de cadena |
| `astra init [--chain <id>]... [--runtimes ...]` | inicio | Prepara el repo: `.astra/`, `docs/astra/`, `.gitignore`, bloque en `AGENTS.md`, skills |
| `astra address <cadena\|familia> <dirección>` | Ensayo, Lanzamiento | Valida formato y checksum; detecta claves secretas y no las imprime |
| `astra deployments list` · `add ...` | Ensayo, Lanzamiento | Registro atómico y validado de todo lo desplegado (A4) |
| `astra check` | todas | Escáner de secretos e higiene del repo (A2) |
| `astra check --gate mainnet` | Gate 2 | Checklist mecánico del Gate 2 (A1, A3, A9, A10) |
| `astra skills sync` · `protocol path` · `protocol fetch` | inicio | Sincroniza skills; localiza o clona este protocolo |
| `astra mcp` | todas | Las mismas capacidades como servidor MCP por stdio |

Lo que `astra` **no hace**, por diseño: no firma, no despliega, no guarda claves, no llama a APIs de IA, no envía telemetría. Cero dependencias, Node ≥ 20, Windows/macOS/Linux.

El protocolo funciona sin el CLI: cada checklist se puede recorrer a mano con los documentos de `guides/`. El CLI vuelve mecánico lo que a mano se olvida.

---

## Artefactos de un repo adherente

```
<repo>/
├─ .astra/
│  ├─ astra.json              ← cadenas y runtimes del proyecto
│  └─ deployments.json        ← todo lo desplegado (A4); esquema en templates/deployments.schema.json
├─ docs/astra/
│  ├─ orbit.md                ← Fase 1
│  ├─ chart.md                ← Fase 2; Gate 1 = línea "aprobada: YYYY-MM-DD"
│  ├─ audit.md                ← Fase 5; "veredicto: apto" y tabla de hallazgos
│  ├─ launch.md               ← Gate 2; firmado_por, fecha_firma, costo_estimado, alias_mainnet, plan_rollback
│  └─ devlogs/                ← Fase 7
├─ AGENTS.md · CLAUDE.md · GEMINI.md   ← bloque <!-- astra:start --> … <!-- astra:end -->
├─ .claude/skills/ · .agents/skills/ · .kimi-code/skills/ · .cursor/skills/   ← copias generadas
└─ .gitignore                 ← .env, .env.*, .stellar/identity/, .soroban/identity/
```

---

## Skills externas

ASTRA no vendoriza skills de terceros: tienen licencia y ritmo propios. Se instalan en el proyecto que las quiera con `astra skills sync --from <directorio-de-skills> --runtimes ...`. La guía `guides/skills-externas.md` mapea a fases de ASTRA el pack de skills de desarrollo Stellar (`stellar-dev-skill` / `stellar.new`) y dice qué skills de ese pack **no** pertenecen a ASTRA.

---

## Independencia

ASTRA no referencia ni requiere ningún otro protocolo, sistema de memoria, daemon ni capa de enrutamiento. Funciona con este repositorio y, opcionalmente, con `astra-cli`. Cualquier sistema superior que quiera invocarlo lo hace leyendo este documento y llamando al CLI o al MCP; ASTRA no sabe que ese sistema existe y no lo necesita para cerrar un sprint. Esta asimetría es deliberada: un protocolo de despliegue a mainnet tiene que poder auditarse completo desde adentro.

---

## No-objetivos

- Firmar o enviar transacciones desde el CLI (A2, A7).
- Indexers, bases de eventos, monitoreo continuo después del lanzamiento (entran como sprint aparte, con su propia Carta).
- Cadenas fuera de las tres familias de génesis (Solana, Cosmos, Bitcoin L1 puro): entran por la misma vía (registro + guía + adaptador) cuando haga falta.
- Generación automática de agentes en formato Kimi Code o Cursor: los agentes se usan como prompts de rol; solo las skills se sincronizan.
- Reemplazar un protocolo general de software: ASTRA cubre lo que toca la cadena y compone con el resto.

---

## Glosario

Órbita (perfil de cadena y capacidad) · Carta (diseño aprobado por Gate 1) · Ensayo (testnet) · Gate de mainnet (Gate 2) · familia de cadena · alias de clave · StrKey · EIP-55 · CAIP-2 · SAC · PoDA · OP Stack · facilitador x402 · tier. Definiciones completas en `docs/GLOSSARY.md`.

---

## Referencias

- Stellar: https://developers.stellar.org · estándares SEP/CAP: https://github.com/stellar/stellar-protocol · SEP-0023 (StrKey), SEP-0041 (token), SEP-0055 (verificación de build).
- Syscoin y Rollux: https://docs.syscoin.org · https://docs.rollux.com.
- Base: https://docs.base.org.
- EVM: https://eips.ethereum.org (ERC-20, ERC-55, ERC-1967, ERC-4337, EIP-155, EIP-712, EIP-1559, EIP-7702).
- Identificadores multi-cadena: CAIP-2 / CAIP-10 (https://github.com/ChainAgnostic/CAIPs).
- Pagos agénticos: x402 (https://www.x402.org, https://github.com/coinbase/x402, https://developers.stellar.org/docs/build/apps/x402) · MPP (https://mpp.dev).
- Formato de skills: https://agentskills.io · MCP: https://modelcontextprotocol.io.
- Diseño de la génesis: `docs/superpowers/specs/2026-09-04-astra-protocol-design.md` y plan `docs/superpowers/plans/2026-09-04-astra-genesis-plan.md`.

---

## Version history

| Versión | Fecha | Disparador | Cambios |
|---|---|---|---|
| 0.1.0 | 2026-09-04 | Génesis. Los sprints Web3 se conducían con protocolos de software general cuyos gates suponen reversibilidad; hacía falta un protocolo exclusivo, multi-cadena, multi-vendor e independiente. | Protocolo inicial: 7 fases (Órbita · Carta · Construcción · Ensayo · Auditoría · Lanzamiento · Bitácora), 2 gates humanos (Carta, Mainnet), 10 axiomas, 5 roles, 3 familias de cadena con 9 cadenas verificadas, entrypoints para 7 runtimes, 9 skills, plantillas de artefactos y la herramienta `astra` (CLI + MCP, cero dependencias). |
