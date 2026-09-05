# Skills externas — packs de terceros dentro de un proyecto ASTRA

## Política

- ASTRA **no vendoriza** skills de terceros: tienen su licencia (Apache-2.0, MIT u otra), su ritmo de cambios y su mantenedor. Copiarlas dentro del protocolo las congelaría y mezclaría licencias.
- Un proyecto las instala cuando las quiere: `astra skills sync --from <directorio-con-skills> --runtimes claude,codex,...`. Cada copia lleva cabecera GENERADO y se actualiza volviendo a sincronizar desde el pack.
- Antes de instalar un pack se audita: qué scripts ejecuta, qué URLs consulta, si pide claves o escribe fuera del proyecto. Una skill que pide una clave privada no se instala (A2).
- Las skills externas se usan **dentro** de las fases de ASTRA; no reemplazan los gates ni los artefactos.

## Pack de desarrollo Stellar (`stellar-dev-skill` / `stellar.new`)

Skills publicadas por la Stellar Development Foundation y el ecosistema stellar.new para desarrollo en Stellar. Mapa a fases de ASTRA:

| Skill externa | Fase ASTRA | Uso |
|---|---|---|
| `standards` | Órbita, Carta | Elegir SEP/CAP; complementa `astra standards search`. |
| `find-stellar-idea`, `stellar-competitive-landscape`, `scf-round-watcher` | antes de Órbita (descubrimiento) | Qué construir, quién ya lo hizo, qué financia el Stellar Community Fund. |
| `assets` | Carta, Construcción | Activos clásicos, trustlines, flags, SAC. |
| `smart-contracts` | Construcción | Anatomía de un contrato Soroban, build, tests, deploy. |
| `dapp` | Construcción, Ensayo | stellar-sdk en browser/Node, Freighter, Stellar Wallets Kit, smart accounts. |
| `data` | Ensayo, Bitácora | RPC y Horizon: consultas, streaming, verificación de estado en testnet/mainnet. |
| `agentic-payments` | Carta, Construcción | x402 y MPP en Stellar (ver `agentic-payments.md`). |
| `zk-proofs` | Carta, Construcción | Groth16 sobre BLS12-381 (CAP-0059), estado de BN254/Poseidon. |
| `deploy-stellar-mainnet` | Auditoría, Lanzamiento | Checklist de mainnet de la SDF; se corre **además** de `astra check --gate mainnet`, nunca en su lugar. |
| `stellar-help`, `navigate-skills` | navegación | Descubrir qué skill del pack aplica. |

Instalación en un proyecto Stellar:

```bash
astra skills sync --from /ruta/al/pack/skills --runtimes claude,codex
```

## Skills que NO pertenecen a ASTRA

Un pack de desarrollo suele traer también skills de gestión de producto, revisión de código genérica, personas de equipo y diseño de UX. Esas no son Web3 y no se instalan por ASTRA:

- Gestión de producto y planificación (PRD, briefs, épicas e historias, PRFAQ, brainstorming, elicitación, sesiones de grupo, personas de desarrollador/arquitecto/analista/PM/redactor técnico, revisión de código, caza de casos borde, investigación forense, reescritura de prompts) → pertenecen a un **protocolo general de software** (por ejemplo AEGIS), que compone con ASTRA con los gates alineados el mismo día.
- Diseño de UX y la persona de diseñador → pertenecen a un **protocolo de diseño**.
- Skills que borran marcas de autoría de IA de los documentos → **excluidas por política**: contradicen la bitácora honesta que exige la fase 7 y la trazabilidad que exige una auditoría.

## Cómo evaluar un pack nuevo

1. Leer cada `SKILL.md`: frontmatter (`name`, `description`), scripts referenciados, comandos que ejecuta.
2. `grep` por `curl`, `wget`, `fetch`, `http`, `rm -rf`, `eval`, `PRIVATE_KEY`, `secret`, `mnemonic`.
3. Escanear caracteres invisibles o bidireccionales (Unicode) en los archivos.
4. Verificar que las URLs que cita existan y sean las oficiales.
5. Instalar solo las skills que caen en una fase de ASTRA; documentar en `orbit.md` qué pack y qué versión.
