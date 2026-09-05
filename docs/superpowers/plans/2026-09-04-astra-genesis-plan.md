# ASTRA génesis — plan de implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Construir el protocolo ASTRA (repo `astra-protocol`), su herramienta `astra` (repo `astra-cli`, CLI + servidor MCP), publicarlos como repos públicos en GitHub y enseñar a KAIROS a enrutar hacia ASTRA sin que ASTRA sepa de KAIROS.

**Architecture:** Dos repos independientes. `astra-cli` es un paquete Node ≥ 20, ESM, cero dependencias, con un módulo por responsabilidad en `lib/` y datos en `data/` (registro de cadenas y catálogo de estándares); el CLI y el servidor MCP son dos fachadas sobre las mismas funciones. `astra-protocol` es Markdown: protocolo, guías, plantillas, skills canónicas en `skills/` y copias generadas por `astra skills sync`. KAIROS recibe una fila de routing, el enum `astra` y el directorio hermano; nada de ASTRA apunta a KAIROS.

**Tech Stack:** Node 22 (compatible 20), `node --test`, `node:crypto` (sha256), `fetch` nativo, Git, GitHub CLI (`gh`), GitHub Actions. Sin npm install en ningún momento.

**Spec:** `C:/dev/protocols/ASTRA/docs/superpowers/specs/2026-09-04-astra-protocol-design.md`

## Global Constraints

- Node ≥ 20, ESM (`"type": "module"`), **cero dependencias** de runtime y de desarrollo. Tests con `node --test`.
- Windows, macOS y Linux: nunca hardcodear `/` ni `\\`; usar `node:path`; ejecutables buscados con `PATHEXT`; `.cmd/.bat` se lanzan con `shell: true`; escritura atómica con `rename` y reintentos ante `EPERM/EBUSY/EACCES`.
- Cualquier vendor de IA: entrypoints `AGENTS.md` (canónico), `CLAUDE.md` (importa `@AGENTS.md`), `GEMINI.md`; skills en `skills/<nombre>/SKILL.md`; tiers `primary`/`secondary`, nunca IDs de modelo de un vendor.
- Cadenas de génesis: `stellar-mainnet`, `stellar-testnet`, `base`, `base-sepolia`, `syscoin-nevm`, `syscoin-tanenbaum`, `rollux`, `rollux-tanenbaum`, `syscoin-utxo`.
- ASTRA **no menciona** KAIROS, SEELE, MNEMA, PEITHO ni LUMEN. Puede nombrar "un protocolo general de software (por ejemplo AEGIS)" solo en `guides/skills-externas.md` y en la spec.
- Nada de secretos ni datos personales en ningún archivo: `astra check` debe pasar en ambos repos antes de cada push. Los tests construyen secretos de prueba en runtime (nunca literales).
- Idioma: español latino neutro, sin voseo, en docs y mensajes del CLI. Identificadores de código en inglés.
- Licencia MIT, `Copyright (c) 2026 DevZen SpA`. Commits con prefijos `feat:`, `docs:`, `test:`, `chore:` y cuerpo en español.
- Fecha de génesis: `2026-09-04`. Versión de ambos repos: `0.1.0`.

---

## Mapa de archivos

### `C:/dev/tools/astra-cli` (repo `astra-cli`)

| Archivo | Responsabilidad |
|---|---|
| `bin/astra.mjs` | Shebang + `main(process.argv.slice(2))`. |
| `lib/cli.mjs` | `parseArgs`, tabla de comandos, ayuda, formateo texto/JSON, códigos de salida (0 ok, 1 fallo, 2 uso). |
| `lib/util.mjs` | `isWindows`, `fechaLocalISO`, `findExecutable`, `runVersion`, `writeAtomic`, `readJson`, `homeDir`, `gitHead`, `AstraError`. |
| `lib/registry.mjs` | `loadChains`, `listChains`, `getChain`, `resolveFamily`, `FAMILIES`. |
| `lib/keccak.mjs` | `keccak256`, `keccak256Hex`, `toHex`. |
| `lib/address.mjs` | StrKey (SEP-0023), EIP-55, bech32/bech32m, base58check, `validateAddress`. |
| `lib/probe.mjs` | `probeChain` (EVM / Stellar / UTXO) con `fetch` inyectable. |
| `lib/doctor.mjs` | `TOOLS`, `detectTools`, `verdictFor`, `doctor`. |
| `lib/deployments.mjs` | registro `.astra/deployments.json`: `readDeployments`, `validateEntry`, `addDeployment`. |
| `lib/check.mjs` | escáner de secretos, higiene, gate mainnet: `scanFiles`, `checkRepo`. |
| `lib/standards.mjs` | `loadStandards`, `searchStandards`. |
| `lib/protocol.mjs` | `resolveProtocolDir`, `fetchProtocol`, `PROTOCOL_REPO_URL`. |
| `lib/skills.mjs` | `RUNTIME_DIRS`, `expandRuntimes`, `renderGenerated`, `syncSkills`. |
| `lib/init.mjs` | `upsertBlock`, `astraBlock`, `initProject`. |
| `lib/mcp.mjs` | `MCP_TOOLS`, `handleMessage`, `startMcpServer`. |
| `data/chains.json` | registro de cadenas (verificado 2026-09-04). |
| `data/standards.json` | catálogo SEP/CAP/ERC/EIP/CAIP/pagos/docs. |
| `test/*.test.mjs` | un archivo por módulo + `cli.test.mjs` (e2e por proceso hijo). |
| `.github/workflows/ci.yml` | matriz ubuntu/windows/macos × node 20/22. |
| `package.json`, `README.md`, `LICENSE`, `AGENTS.md`, `CLAUDE.md`, `.gitignore`, `.gitattributes`, `.editorconfig` | paquete y docs. |

### `C:/dev/protocols/ASTRA` (repo `astra-protocol`)

`ASTRA-PROTOCOL.md`, `README.md`, `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `LICENSE`, `.gitignore`, `.gitattributes`, `skills/<9>/SKILL.md`, `.claude/skills/*` y `.agents/skills/*` (generados), `templates/{orbit,chart,audit,launch,devlog}.template.md`, `templates/deployments.schema.json`, `templates/agents/<5>.md`, `guides/{gates,keys-and-secrets,audit-checklist,standards-map,agentic-payments,runtimes,skills-externas}.md`, `guides/chains/{stellar,syscoin,base,evm-generico}.md`, `docs/GLOSSARY.md`, `docs/MAPA.md`, `docs/superpowers/{specs,plans}/…`, `genesis/README.md`, `genesis/devlogs/2026-09-04-genesis.md`.

### `C:/dev/protocols/KAIROS` (modificaciones)

`daemon/kairos-daimonion.mjs:265`, `daemon/test/kairos-daimonion.test.mjs` (test nuevo), `guides/routing-matrix.md`, `KAIROS-PROTOCOL.md` (cabecera, §hueco, diagrama, TL;DR, Fase 2, Fase 3 enum, §Relación, §Referencias, version history), `.claude/commands/kairos.md`, `.kimi-code/skills/kairos/SKILL.md`, `templates/wager.template.md`, `templates/proposal.template.md`, `observatory/index.html`, `observatory/cosmografia.html`, `CLAUDE.md`, `README.md`, `guides/observatory.md`, `docs/devlogs/2026-09-04-astra-hermano.md`.

---

## Vectores de prueba oficiales (usados en las tareas 3 y 7)

- StrKey válidos (SEP-0023 §Tests): G `GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVSGZ`; M id 0 `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUQ`; M id 9223372036854775808 `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAJLK`; C `CA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUWDA`; L `LA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUPJN`; B `BAAD6DBUX6J22DMZOHIEZTEQ64CVCHEDRKWZONFEUL5Q26QD7R76RGR4TU`; P `PA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAQACAQDAQCQMBYIBEFAWDANBYHRAEISCMKBKFQXDAMRUGY4DUPB6IBZGM`. SAC de XLM (salida de `stellar contract id asset --asset native`): testnet `CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC`, mainnet `CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA`.
- StrKey inválidos (SEP-0023): `GAAAAAAAACGC6`, `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUR`, `GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVSGZA`, `GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUACUSI`, `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAJLKA`, `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAAV75I`, `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUK===`, `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUO`.
- EIP-55: `0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed`, `0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359`, `0xdbF03B407c01E7cD3CBea99509d93f8DDDC8C6FB`, `0xD1220A0cf47c7B9Be7A2E6BA89F429762e7b9aDb` válidos; `0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAeD` (última letra alterada) inválido; `0x52908400098527886E0F7030069857D2E4169EE7` (todo mayúsculas) válido sin checksum.
- keccak-256: `keccak256("")` = `c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470`.
- bech32 (BIP-173): `bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4` → v0, programa `751e76e8199196d454941c45d1b3a323f1433bd6`; `bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t5` checksum inválido. bech32m (BIP-350): `bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqzk5jj0` → v1, programa `79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798`; `bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqh2y7hd` inválido (bech32 en vez de bech32m); `bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kemeawh` inválido (bech32m en vez de bech32).
- base58check: `1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2` → versión 0, hash 20 bytes; `3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy` → versión 5.
- Chain IDs en vivo (2026-09-04): base `0x2105` (8453), base-sepolia `0x14a34` (84532), syscoin-nevm `0x39` (57), syscoin-tanenbaum `0x1644` (5700), rollux `0x23a` (570). Passphrases: mainnet `Public Global Stellar Network ; September 2015`, testnet `Test SDF Network ; September 2015`.

---

### Task 1: esqueleto de `astra-cli` (paquete, dispatcher, util)

**Files:**
- Create: `C:/dev/tools/astra-cli/package.json`, `bin/astra.mjs`, `lib/cli.mjs`, `lib/util.mjs`, `.gitignore`, `.gitattributes`, `.editorconfig`, `LICENSE`
- Test: `test/util.test.mjs`, `test/cli.test.mjs`

**Interfaces:**
- Produces: `parseArgs(argv, {booleans}) → {_: string[], flags: Record<string, string|true|string[]>}`; `main(argv) → Promise<number>`; `findExecutable(name, env?) → string|null`; `runVersion(file, args?) → string|null`; `writeAtomic(file, text)`; `readJson(file, fallback?)`; `fechaLocalISO(date?)`; `homeDir(env?)`; `gitHead(cwd) → string|null`; `class AstraError extends Error { code, exitCode }`.

- [ ] **Step 1: escribir `package.json`**

```json
{
  "name": "astra-cli",
  "version": "0.1.0",
  "description": "Herramientas del protocolo ASTRA (desarrollo Web3 agentico, multi-cadena, multi-vendor): doctor, registro de cadenas, validacion de direcciones, escaner de secretos, gate de mainnet, sync de skills y servidor MCP.",
  "type": "module",
  "bin": { "astra": "bin/astra.mjs" },
  "engines": { "node": ">=20" },
  "scripts": { "test": "node --test test/", "check": "node bin/astra.mjs check" },
  "files": ["bin", "lib", "data", "README.md", "LICENSE"],
  "keywords": ["web3", "stellar", "soroban", "syscoin", "base", "evm", "mcp", "ai-agents", "protocol"],
  "license": "MIT",
  "repository": { "type": "git", "url": "https://github.com/orlando-vazquez-career/astra-cli.git" }
}
```

- [ ] **Step 2: escribir el test de `util`**

```js
// test/util.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { findExecutable, runVersion, writeAtomic, readJson, fechaLocalISO, isWindows } from '../lib/util.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-util-'));

test('findExecutable respeta PATH y PATHEXT', () => {
  const dir = tmp();
  const name = isWindows ? 'fakestellar.cmd' : 'fakestellar';
  fs.writeFileSync(path.join(dir, name), isWindows ? '@echo off\r\necho fakestellar 9.9.9\r\n' : '#!/bin/sh\necho "fakestellar 9.9.9"\n', { mode: 0o755 });
  const env = { PATH: dir, PATHEXT: '.COM;.EXE;.BAT;.CMD' };
  assert.equal(findExecutable('fakestellar', env), path.join(dir, name));
  assert.equal(findExecutable('no-existe-seguro', env), null);
  assert.equal(runVersion(path.join(dir, name)), 'fakestellar 9.9.9');
});

test('writeAtomic escribe completo y readJson lee con fallback', () => {
  const dir = tmp();
  const f = path.join(dir, 'a', 'b.json');
  writeAtomic(f, JSON.stringify({ ok: 1 }));
  assert.deepEqual(readJson(f), { ok: 1 });
  assert.deepEqual(readJson(path.join(dir, 'nope.json'), { d: true }), { d: true });
  assert.equal(fs.readdirSync(path.dirname(f)).filter(n => n.endsWith('.tmp')).length, 0);
});

test('fechaLocalISO usa el calendario local', () => {
  assert.equal(fechaLocalISO(new Date(2026, 8, 4, 23, 59)), '2026-09-04');
});
```

- [ ] **Step 3: correr y ver fallar** — `cd C:/dev/tools/astra-cli && node --test test/util.test.mjs` → falla con "Cannot find module '../lib/util.mjs'".

- [ ] **Step 4: escribir `lib/util.mjs`**

```js
// util.mjs — utilidades sin estado compartidas por todos los comandos. Cero dependencias.
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { spawnSync } from 'node:child_process';

export const isWindows = process.platform === 'win32';

export class AstraError extends Error {
  constructor(message, { code = 'ASTRA_ERROR', exitCode = 1 } = {}) { super(message); this.code = code; this.exitCode = exitCode; }
}

export function fechaLocalISO(d = new Date()) {
  const p = n => String(n).padStart(2, '0');
  return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())}`;
}

// Busca un ejecutable en PATH. En Windows prueba cada extension de PATHEXT.
export function findExecutable(name, env = process.env) {
  const pathVar = env.PATH ?? env.Path ?? '';
  const dirs = pathVar.split(path.delimiter).filter(Boolean);
  const exts = isWindows ? (env.PATHEXT || '.COM;.EXE;.BAT;.CMD').split(';').filter(Boolean) : [''];
  const alreadyHasExt = isWindows && exts.some(e => name.toLowerCase().endsWith(e.toLowerCase()));
  for (const dir of dirs) {
    const candidates = (!isWindows || alreadyHasExt) ? [path.join(dir, name)] : exts.map(e => path.join(dir, name + e.toLowerCase()));
    for (const c of candidates) {
      try {
        const st = fs.statSync(c);
        if (!st.isFile()) continue;
        if (!isWindows) fs.accessSync(c, fs.constants.X_OK);
        return c;
      } catch { /* siguiente candidato */ }
    }
  }
  return null;
}

// Primera linea no vacia de `<file> --version` (stdout o stderr). null si no se pudo ejecutar.
export function runVersion(file, args = ['--version'], { timeoutMs = 8000 } = {}) {
  const ext = path.extname(file).toLowerCase();
  const needsShell = isWindows && (ext === '.cmd' || ext === '.bat');
  const r = needsShell
    ? spawnSync(`"${file}"`, args, { encoding: 'utf8', timeout: timeoutMs, windowsHide: true, shell: true })
    : spawnSync(file, args, { encoding: 'utf8', timeout: timeoutMs, windowsHide: true });
  if (r.error) return null;
  return `${r.stdout || ''}\n${r.stderr || ''}`.split(/\r?\n/).map(s => s.trim()).find(Boolean) || null;
}

function esperar(ms) { Atomics.wait(new Int32Array(new SharedArrayBuffer(4)), 0, 0, ms); }

// Escritura atomica: temporal propio + rename, con reintentos porque en Windows
// el rename falla si otro proceso tiene el destino abierto.
export function writeAtomic(file, text) {
  fs.mkdirSync(path.dirname(file), { recursive: true });
  const tmp = `${file}.${process.pid}.${Date.now()}.tmp`;
  fs.writeFileSync(tmp, text);
  for (let intento = 1; ; intento++) {
    try { fs.renameSync(tmp, file); return; }
    catch (err) {
      const transitorio = err.code === 'EPERM' || err.code === 'EBUSY' || err.code === 'EACCES';
      if (!transitorio || intento >= 5) { try { fs.rmSync(tmp, { force: true }); } catch { /* nada */ } throw err; }
      esperar(40 * intento);
    }
  }
}

export function readJson(file, fallback) {
  try { return JSON.parse(fs.readFileSync(file, 'utf8')); }
  catch (err) { if (err.code === 'ENOENT' && fallback !== undefined) return fallback; throw err; }
}

export function homeDir(env = process.env) { return env.ASTRA_HOME || os.homedir(); }

export function gitHead(cwd) {
  const r = spawnSync('git', ['rev-parse', 'HEAD'], { cwd, encoding: 'utf8', windowsHide: true });
  return r.status === 0 ? r.stdout.trim() : null;
}
```

- [ ] **Step 5: correr el test de util** — `node --test test/util.test.mjs` → PASS (3).

- [ ] **Step 6: escribir el test e2e del CLI**

```js
// test/cli.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { spawnSync } from 'node:child_process';
import { fileURLToPath } from 'node:url';

const BIN = fileURLToPath(new URL('../bin/astra.mjs', import.meta.url));
export const astra = (args, opts = {}) => spawnSync(process.execPath, [BIN, ...args], { encoding: 'utf8', ...opts });

test('astra --version imprime la version del package.json', () => {
  const r = astra(['--version']);
  assert.equal(r.status, 0);
  assert.match(r.stdout.trim(), /^astra 0\.1\.0$/);
});

test('astra sin argumentos imprime ayuda y sale 2; --help sale 0', () => {
  assert.equal(astra([]).status, 2);
  const r = astra(['--help']);
  assert.equal(r.status, 0);
  assert.match(r.stdout, /doctor/);
  assert.match(r.stdout, /mcp/);
});

test('un comando desconocido sale 2 con mensaje en stderr', () => {
  const r = astra(['nadaquever']);
  assert.equal(r.status, 2);
  assert.match(r.stderr, /desconocido/);
});
```

- [ ] **Step 7: escribir `bin/astra.mjs` y `lib/cli.mjs`**

```js
#!/usr/bin/env node
// bin/astra.mjs — punto de entrada del CLI `astra`.
import { main } from '../lib/cli.mjs';
process.exitCode = await main(process.argv.slice(2));
```

```js
// lib/cli.mjs — parseo de argumentos, tabla de comandos, ayuda y salida (texto o --json).
import { createRequire } from 'node:module';
import { AstraError } from './util.mjs';

const pkg = createRequire(import.meta.url)('../package.json');
export const VERSION = pkg.version;

const BOOLEANS = new Set(['json', 'check', 'force', 'verified', 'help', 'version', 'fetch', 'no-open']);

export function parseArgs(argv, { booleans = BOOLEANS } = {}) {
  const out = { _: [], flags: {} };
  const push = (k, v) => { if (k in out.flags) out.flags[k] = [].concat(out.flags[k], v); else out.flags[k] = v; };
  for (let i = 0; i < argv.length; i++) {
    const a = argv[i];
    if (a === '--') { out._.push(...argv.slice(i + 1)); break; }
    if (!a.startsWith('--')) { out._.push(a); continue; }
    const eq = a.indexOf('=');
    if (eq > 0) { push(a.slice(2, eq), a.slice(eq + 1)); continue; }
    const key = a.slice(2);
    if (booleans.has(key) || i + 1 >= argv.length || argv[i + 1].startsWith('--')) push(key, true);
    else push(key, argv[++i]);
  }
  return out;
}

export const HELP = `astra ${VERSION} — herramientas del protocolo ASTRA (Web3 agentico, multi-cadena, multi-vendor)

Uso: astra <comando> [opciones]

  doctor [--chain <id>]                  toolchains presentes y veredicto SAFE/CAUTION/AVOID por familia
  chain list | info <id> | probe <id> [--rpc <url>]
                                         registro de cadenas y salud en vivo
  address <cadena|familia> <direccion>   valida formato y checksum (nunca imprime claves secretas)
  init [--chain <id>]... [--runtimes claude,codex,kimi,cursor,antigravity,all] [--protocol-dir <p>]
                                         prepara un repo para ASTRA (.astra/, docs/astra/, bloque en AGENTS.md)
  check [--gate mainnet]                 escaner de secretos, higiene y gate de mainnet (exit 1 si falla)
  deployments list | add --chain <id> --address <a> [--kind contract] [--label <l>] [--tx <t>] [--verified]
  standards search <consulta> [--family stellar|evm|syscoin|payments|caip]
  skills sync [--from <dir>] [--runtimes ...] [--check]
  protocol path | fetch [--dir <p>]      donde vive el protocolo / clonarlo
  mcp                                    servidor MCP por stdio

Opciones globales: --json (salida JSON), --help, --version
`;

const COMMANDS = {
  doctor: () => import('./commands/doctor.mjs'),
  chain: () => import('./commands/chain.mjs'),
  address: () => import('./commands/address.mjs'),
  init: () => import('./commands/init.mjs'),
  check: () => import('./commands/check.mjs'),
  deployments: () => import('./commands/deployments.mjs'),
  standards: () => import('./commands/standards.mjs'),
  skills: () => import('./commands/skills.mjs'),
  protocol: () => import('./commands/protocol.mjs'),
  mcp: () => import('./commands/mcp.mjs'),
};

export async function main(argv, { stdout = process.stdout, stderr = process.stderr } = {}) {
  const args = parseArgs(argv);
  if (args.flags.version) { stdout.write(`astra ${VERSION}\n`); return 0; }
  const [cmd, ...rest] = args._;
  if (!cmd || args.flags.help && !cmd) { stdout.write(HELP); return cmd ? 0 : (args.flags.help ? 0 : 2); }
  const loader = COMMANDS[cmd];
  if (!loader) { stderr.write(`astra: comando desconocido '${cmd}'. Proba 'astra --help'.\n`); return 2; }
  try {
    const mod = await loader();
    return await mod.run({ args: rest, flags: args.flags, stdout, stderr }) ?? 0;
  } catch (err) {
    if (err instanceof AstraError) { stderr.write(`astra: ${err.message}\n`); return err.exitCode; }
    stderr.write(`astra: error inesperado: ${err && err.stack ? err.stack : err}\n`);
    return 1;
  }
}
```

Cada comando vive en `lib/commands/<nombre>.mjs` y exporta `run({args, flags, stdout, stderr}) → number`. Convención de salida: si `flags.json`, `stdout.write(JSON.stringify(result, null, 2) + '\n')`; si no, texto con marcadores ASCII `OK` / `WARN` / `FAIL`.

- [ ] **Step 8: correr `node --test test/`** → PASS (util 3 + cli 3). `astra --help` debe salir 0: en `main`, si `cmd` es undefined y `flags.help`, devolver 0; si no hay comando ni `--help`, devolver 2.

- [ ] **Step 9: archivos de repo** — `.gitignore` (`node_modules/`, `.env`, `.env.*`, `!.env.example`, `*.tmp`, `.DS_Store`), `.gitattributes` (`* text=auto eol=lf` + `*.cmd text eol=crlf`), `.editorconfig` (utf-8, lf, 2 espacios), `LICENSE` (MIT DevZen SpA, texto identico al de los protocolos hermanos).

- [ ] **Step 10: commit** — `git init -b main && git add -A && git commit -m "feat: esqueleto del CLI astra (dispatcher, util, tests e2e)"`.

---

### Task 2: registro de cadenas y `astra chain list/info`

**Files:**
- Create: `data/chains.json`, `lib/registry.mjs`, `lib/commands/chain.mjs`
- Test: `test/registry.test.mjs`

**Interfaces:**
- Produces: `loadChains() → {version, verifiedAt, chains: Chain[]}`; `listChains() → Chain[]`; `getChain(id) → Chain|null`; `resolveFamily(idOrFamily) → {family, chain}|null`; `FAMILIES = ['stellar','evm','utxo']`.
- `Chain = { id, family, name, network: 'mainnet'|'testnet', caip2, chainId?: number, nativeSymbol, rpc: string[], horizon?: string, passphrase?: string, friendbot?: string, explorers: string[], faucets: string[], docs: string, notes: string, verifiedAt }`.

- [ ] **Step 1: test de integridad del registro**

```js
// test/registry.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { listChains, getChain, resolveFamily, FAMILIES } from '../lib/registry.mjs';

test('el registro tiene las cadenas de genesis con ids unicos y familias conocidas', () => {
  const ids = listChains().map(c => c.id);
  for (const id of ['stellar-mainnet', 'stellar-testnet', 'base', 'base-sepolia', 'syscoin-nevm', 'syscoin-tanenbaum', 'rollux', 'rollux-tanenbaum', 'syscoin-utxo']) assert.ok(ids.includes(id), id);
  assert.equal(new Set(ids).size, ids.length);
  for (const c of listChains()) {
    assert.ok(FAMILIES.includes(c.family), c.id);
    assert.ok(['mainnet', 'testnet'].includes(c.network), c.id);
    for (const u of [...c.rpc, ...c.explorers, ...c.faucets, c.docs]) assert.match(u, /^https:\/\//, `${c.id} ${u}`);
    if (c.family === 'evm') { assert.equal(typeof c.chainId, 'number'); assert.equal(c.caip2, `eip155:${c.chainId}`); }
    if (c.family === 'stellar') { assert.ok(c.passphrase); assert.ok(c.horizon); }
  }
});

test('getChain y resolveFamily', () => {
  assert.equal(getChain('base').chainId, 8453);
  assert.equal(getChain('nope'), null);
  assert.deepEqual(resolveFamily('evm').family, 'evm');
  assert.equal(resolveFamily('syscoin-nevm').chain.chainId, 57);
  assert.equal(resolveFamily('marte'), null);
});
```

- [ ] **Step 2: correr y ver fallar.**

- [ ] **Step 3: escribir `data/chains.json`** con exactamente estos datos (verificados en vivo el 2026-09-04):

| id | family | network | caip2 | chainId | symbol | rpc | explorers | faucets |
|---|---|---|---|---|---|---|---|---|
| stellar-mainnet | stellar | mainnet | stellar:pubnet | — | XLM | `[]` (nota: usar proveedor propio, ver developers.stellar.org/docs/data/apis/rpc/providers); horizon `https://horizon.stellar.org`; passphrase `Public Global Stellar Network ; September 2015` | `https://stellar.expert/explorer/public`, `https://lab.stellar.org` | `[]` |
| stellar-testnet | stellar | testnet | stellar:testnet | — | XLM | `https://soroban-testnet.stellar.org`; horizon `https://horizon-testnet.stellar.org`; friendbot `https://friendbot.stellar.org`; passphrase `Test SDF Network ; September 2015` | `https://stellar.expert/explorer/testnet`, `https://lab.stellar.org` | `https://friendbot.stellar.org` |
| base | evm | mainnet | eip155:8453 | 8453 | ETH | `https://mainnet.base.org` | `https://basescan.org`, `https://base.blockscout.com` | `[]` |
| base-sepolia | evm | testnet | eip155:84532 | 84532 | ETH | `https://sepolia.base.org`, `https://base-sepolia-rpc.publicnode.com` | `https://sepolia.basescan.org`, `https://base-sepolia.blockscout.com` | `https://docs.base.org/base-chain/tools/network-faucets` |
| syscoin-nevm | evm | mainnet | eip155:57 | 57 | SYS | `https://rpc.syscoin.org`, `https://syscoin.public-rpc.com` | `https://explorer.syscoin.org` | `[]` |
| syscoin-tanenbaum | evm | testnet | eip155:5700 | 5700 | tSYS | `https://rpc.tanenbaum.io`, `https://syscoin-tanenbaum-evm.publicnode.com` | `https://explorer.tanenbaum.io` | `https://faucet.tanenbaum.io` |
| rollux | evm | mainnet | eip155:570 | 570 | SYS | `https://rpc.rollux.com`, `https://rpc.ankr.com/rollux` | `https://explorer.rollux.com` | `[]` |
| rollux-tanenbaum | evm | testnet | eip155:57000 | 57000 | TSYS | `https://rpc-tanenbaum.rollux.com` (nota: la sonda del 2026-09-04 no respondio; verificar en docs.rollux.com) | `[]` | `[]` |
| syscoin-utxo | utxo | mainnet | — | — | SYS | `[]` (requiere nodo local: `syscoin-cli`) ; `addressHrp: "sys"`, `testnetHrp: "tsys"`, `base58Versions: {p2pkh: 63, p2sh: 5, testnetP2pkh: 65, testnetP2sh: 196}` | `https://explorer.syscoin.org` | `[]` |

`docs`: stellar → `https://developers.stellar.org`; base → `https://docs.base.org`; syscoin (nevm, tanenbaum, utxo) → `https://docs.syscoin.org`; rollux → `https://docs.rollux.com`. `notes` en español (una frase por cadena: que es, quien la opera, riesgo notable).

- [ ] **Step 4: escribir `lib/registry.mjs`**

```js
// registry.mjs — registro de cadenas (data/chains.json). Solo lectura.
import { fileURLToPath } from 'node:url';
import { readJson } from './util.mjs';

export const FAMILIES = ['stellar', 'evm', 'utxo'];
let cache = null;
export function loadChains() { if (!cache) cache = readJson(fileURLToPath(new URL('../data/chains.json', import.meta.url))); return cache; }
export function listChains() { return loadChains().chains; }
export function getChain(id) { return listChains().find(c => c.id === id) || null; }
export function resolveFamily(idOrFamily) {
  if (FAMILIES.includes(idOrFamily)) return { family: idOrFamily, chain: null };
  const chain = getChain(idOrFamily);
  return chain ? { family: chain.family, chain } : null;
}
```

- [ ] **Step 5: escribir `lib/commands/chain.mjs`** — subcomandos `list` (tabla id/family/network/chainId/symbol), `info <id>` (todas las propiedades), `probe <id>` (se implementa en Task 4; hasta entonces lanza `AstraError('probe llega en la Task 4', {exitCode: 2})`). `--json` devuelve el objeto crudo.

- [ ] **Step 6: correr `node --test test/`** → PASS. Probar `node bin/astra.mjs chain info base --json`.

- [ ] **Step 7: commit** — `git add -A && git commit -m "feat(chain): registro de cadenas verificado (stellar, base, syscoin nevm/rollux/utxo) y comandos list/info"`.

---

### Task 3: keccak-256, StrKey, EIP-55, bech32, base58check y `astra address`

**Files:**
- Create: `lib/keccak.mjs`, `lib/address.mjs`, `lib/commands/address.mjs`
- Test: `test/keccak.test.mjs`, `test/address.test.mjs`

**Interfaces:**
- Produces: `keccak256(input: Uint8Array|string) → Uint8Array(32)`; `keccak256Hex(input) → string`; `toHex(bytes) → string`.
- `decodeStrKey(s) → {valid:true, prefix, kind, payload:Uint8Array, secret:boolean, ed25519?, id?:bigint} | {valid:false, reason}`; `encodeStrKey(versionByte, payload) → string`; `validateEvmAddress(addr) → {valid, checksum:'ok'|'none', normalized, reason?}`; `bech32Decode(str)`, `decodeSegwit(hrps, addr)`; `base58checkDecode(s) → {version, hash}|null`; `validateAddress(target, address) → {valid, family, chain?, kind?, normalized?, checksum?, secret, reason?}`.

- [ ] **Step 1: tests de keccak**

```js
// test/keccak.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { keccak256Hex } from '../lib/keccak.mjs';
test('keccak256 de cadena vacia', () => {
  assert.equal(keccak256Hex(''), 'c5d2460186f7233c927e7db2dcc703c0e500b653ca82273b7bfad8045d85a470');
});
test('keccak256 de 200 bytes (dos bloques) es determinista y de 64 hex', () => {
  const h = keccak256Hex('a'.repeat(200));
  assert.match(h, /^[0-9a-f]{64}$/);
  assert.equal(h, keccak256Hex(new TextEncoder().encode('a'.repeat(200))));
});
```

- [ ] **Step 2: escribir `lib/keccak.mjs`** (Keccak-f[1600] con BigInt, rate 136, padding `0x01 … 0x80`)

```js
// keccak.mjs — Keccak-256 (la variante pre-SHA3 que usa Ethereum). Implementacion
// con BigInt: lenta pero corta y verificable; solo se usa para checksums EIP-55.
const RC = [0x0000000000000001n, 0x0000000000008082n, 0x800000000000808an, 0x8000000080008000n,
  0x000000000000808bn, 0x0000000080000001n, 0x8000000080008081n, 0x8000000000008009n,
  0x000000000000008an, 0x0000000000000088n, 0x0000000080008009n, 0x000000008000000an,
  0x000000008000808bn, 0x800000000000008bn, 0x8000000000008089n, 0x8000000000008003n,
  0x8000000000008002n, 0x8000000000000080n, 0x000000000000800an, 0x800000008000000an,
  0x8000000080008081n, 0x8000000000008080n, 0x0000000080000001n, 0x8000000080008008n];
const ROT = [[0, 36, 3, 41, 18], [1, 44, 10, 45, 2], [62, 6, 43, 15, 61], [28, 55, 25, 21, 56], [27, 20, 39, 8, 14]];
const M64 = (1n << 64n) - 1n;
const rotl = (x, n) => n === 0 ? x : (((x << BigInt(n)) | (x >> BigInt(64 - n))) & M64);

function keccakF(A) {
  for (let round = 0; round < 24; round++) {
    const C = [0, 1, 2, 3, 4].map(x => A[x] ^ A[x + 5] ^ A[x + 10] ^ A[x + 15] ^ A[x + 20]);
    const D = [0, 1, 2, 3, 4].map(x => C[(x + 4) % 5] ^ rotl(C[(x + 1) % 5], 1));
    for (let i = 0; i < 25; i++) A[i] ^= D[i % 5];
    const B = new Array(25).fill(0n);
    for (let x = 0; x < 5; x++) for (let y = 0; y < 5; y++) B[y + 5 * ((2 * x + 3 * y) % 5)] = rotl(A[x + 5 * y], ROT[x][y]);
    for (let x = 0; x < 5; x++) for (let y = 0; y < 5; y++) A[x + 5 * y] = B[x + 5 * y] ^ ((~B[(x + 1) % 5 + 5 * y] & M64) & B[(x + 2) % 5 + 5 * y]);
    A[0] ^= RC[round];
  }
}

export function keccak256(input) {
  const bytes = typeof input === 'string' ? new TextEncoder().encode(input) : input;
  const rate = 136;
  const padded = new Uint8Array(Math.ceil((bytes.length + 1) / rate) * rate);
  padded.set(bytes);
  padded[bytes.length] ^= 0x01;
  padded[padded.length - 1] ^= 0x80;
  const A = new Array(25).fill(0n);
  for (let off = 0; off < padded.length; off += rate) {
    for (let i = 0; i < rate / 8; i++) {
      let lane = 0n;
      for (let b = 7; b >= 0; b--) lane = (lane << 8n) | BigInt(padded[off + i * 8 + b]);
      A[i] ^= lane;
    }
    keccakF(A);
  }
  const out = new Uint8Array(32);
  for (let i = 0; i < 4; i++) { let lane = A[i]; for (let b = 0; b < 8; b++) { out[i * 8 + b] = Number(lane & 0xffn); lane >>= 8n; } }
  return out;
}
export const toHex = bytes => Array.from(bytes, b => b.toString(16).padStart(2, '0')).join('');
export const keccak256Hex = input => toHex(keccak256(input));
```

- [ ] **Step 3: `node --test test/keccak.test.mjs`** → PASS. Si el vector de cadena vacia falla, revisar en este orden: constantes RC, tabla ROT (`ROT[x][y]`), indice `B[y + 5*((2x+3y)%5)]`, padding.

- [ ] **Step 4: tests de direcciones** (usar los vectores de la seccion "Vectores de prueba oficiales"; la clave secreta de prueba se construye en runtime con `encodeStrKey(18 << 3, new Uint8Array(32))`):

```js
// test/address.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { decodeStrKey, encodeStrKey, validateEvmAddress, decodeSegwit, base58checkDecode, validateAddress } from '../lib/address.mjs';
import { toHex } from '../lib/keccak.mjs';

const G = 'GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVSGZ';

test('StrKey acepta los casos validos de SEP-0023 y las SAC reales', () => {
  for (const [s, kind] of [[G, 'ed25519_public_key'], ['MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUQ', 'muxed_account'],
    ['MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAJLK', 'muxed_account'],
    ['CA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUWDA', 'contract'], ['LA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUPJN', 'liquidity_pool'],
    ['BAAD6DBUX6J22DMZOHIEZTEQ64CVCHEDRKWZONFEUL5Q26QD7R76RGR4TU', 'claimable_balance'],
    ['PA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAQACAQDAQCQMBYIBEFAWDANBYHRAEISCMKBKFQXDAMRUGY4DUPB6IBZGM', 'signed_payload'],
    ['CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC', 'contract'], ['CAS3J7GYLGXMF6TDJBBYYSE3HQ6BBSMLNUQ34T6TZMYMW2EVH34XOWMA', 'contract']]) {
    const r = decodeStrKey(s);
    assert.equal(r.valid, true, `${s}: ${r.reason}`);
    assert.equal(r.kind, kind, s);
  }
  const m = decodeStrKey('MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAJLK');
  assert.equal(m.ed25519, G);
  assert.equal(m.id, 9223372036854775808n);
});

test('StrKey rechaza los casos invalidos de SEP-0023', () => {
  for (const s of ['GAAAAAAAACGC6', 'MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUR', 'GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVSGZA',
    'GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUACUSI', 'MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAJLKA',
    'MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVAAAAAAAAAAAAAAV75I', 'MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUK===',
    'MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUO', '', 'ga7qynf7']) assert.equal(decodeStrKey(s).valid, false, s);
});

test('una semilla secreta se detecta y nunca se devuelve normalizada', () => {
  const seed = encodeStrKey(18 << 3, new Uint8Array(32));
  assert.equal(seed[0], 'S');
  const r = validateAddress('stellar', seed);
  assert.equal(r.valid, false); assert.equal(r.secret, true); assert.ok(!('normalized' in r));
});

test('EIP-55', () => {
  for (const a of ['0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed', '0xfB6916095ca1df60bB79Ce92cE3Ea74c37c5d359', '0xdbF03B407c01E7cD3CBea99509d93f8DDDC8C6FB', '0xD1220A0cf47c7B9Be7A2E6BA89F429762e7b9aDb']) {
    const r = validateEvmAddress(a); assert.equal(r.valid, true, a); assert.equal(r.checksum, 'ok'); assert.equal(r.normalized, a);
  }
  assert.equal(validateEvmAddress('0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAeD').valid, false);
  const upper = validateEvmAddress('0x52908400098527886E0F7030069857D2E4169EE7'); assert.equal(upper.valid, true); assert.equal(upper.checksum, 'none');
  assert.equal(validateEvmAddress('0x1234').valid, false);
});

test('bech32 y bech32m (BIP-173/350)', () => {
  const v0 = decodeSegwit(['bc'], 'bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4');
  assert.equal(v0.version, 0); assert.equal(toHex(v0.program), '751e76e8199196d454941c45d1b3a323f1433bd6');
  const v1 = decodeSegwit(['bc'], 'bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqzk5jj0');
  assert.equal(v1.version, 1); assert.equal(toHex(v1.program), '79be667ef9dcbbac55a06295ce870b07029bfcdb2dce28d959f2815b16f81798');
  for (const bad of ['bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t5', 'bc1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vqh2y7hd', 'bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kemeawh', 'tb1p0xlxvlhemja6c4dqv22uapctqupfhlxm9h8z3k2e72q4k9hcz7vq47Zagq']) assert.ok(decodeSegwit(['bc', 'tb'], bad).error, bad);
  assert.ok(decodeSegwit(['sys'], 'bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4').error, 'hrp ajeno');
});

test('base58check', () => {
  assert.equal(base58checkDecode('1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN2').version, 0);
  assert.equal(base58checkDecode('3J98t1WpEZ73CNmQviecrnyiWrnqRhWNLy').version, 5);
  assert.equal(base58checkDecode('1BvBMSEYstWetqTFn5Au4m4GFg7xJaNVN3'), null);
});

test('validateAddress por familia y por cadena', () => {
  assert.equal(validateAddress('stellar-testnet', 'CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC').kind, 'contract');
  assert.equal(validateAddress('base', '0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed').valid, true);
  const pk = validateAddress('evm', '0x' + 'ab'.repeat(32)); assert.equal(pk.valid, false); assert.equal(pk.secret, true);
  assert.equal(validateAddress('syscoin-utxo', 'bc1qw508d6qejxtdg4y5r3zarvary0c5xw7kv8f3t4').valid, false);
  assert.equal(validateAddress('marte', 'x').valid, false);
});
```

- [ ] **Step 5: escribir `lib/address.mjs`** — base32 RFC 4648 (decode canonico: bits sobrantes en cero; encode sin padding), `crc16xmodem` (poly 0x1021, init 0, checksum little-endian al final), tabla `STRKEY_VERSIONS` = G 6<<3 (32 bytes), S 18<<3 (32, `secret: true`), M 12<<3 (40: clave + id uint64 BE), T 19<<3 (32), X 23<<3 (32), P 15<<3 (32 + 4 + payload ≤ 64 con padding a multiplo de 4), C 2<<3 (32), L 11<<3 (32), B 1<<3 (33, primer byte 0). `decodeStrKey`: alfabeto, longitud `% 8 ∉ {1,3,6}`, bits sobrantes cero, byte de version conocido, checksum, longitud por tipo, re-encode identico. `validateEvmAddress` (regex `^0x[0-9a-fA-F]{40}$`, keccak del hex en minusculas, EIP-55). `bech32Decode` / `convertBits` / `decodeSegwit` (v0 → bech32 y programa 20|32; v≥1 → bech32m y programa 2..40; mayusculas mezcladas → error). `base58checkDecode` con `sha256(sha256(payload))` de `node:crypto`. `validateAddress(target, address)`: `resolveFamily(target)`; stellar → StrKey (secret → `{valid:false, secret:true, reason}` sin `normalized`); evm → 64 hex (con o sin `0x`) → secreto; si no `validateEvmAddress`; utxo → hrps segun red (`sys`/`tsys`, ambas si es familia) → segwit; si no base58check con versiones `{63,5,65,196}` (kind `p2pkh`/`p2sh`), version `128|239` → secreto WIF. Siempre devolver `secret: boolean`.

- [ ] **Step 6: `node --test test/address.test.mjs`** → PASS (7).

- [ ] **Step 7: `lib/commands/address.mjs`** — `run`: exige 2 positionales (si no, exit 2 con uso). Imprime `OK <kind> <normalized>` o `FAIL <reason>`; si `secret`, imprime `FAIL clave secreta detectada: no se imprime, no se registra, no se pega en un chat` y exit 1. `--json` imprime el objeto (sin la direccion original cuando `secret`).

- [ ] **Step 8: commit** — `git add -A && git commit -m "feat(address): validacion StrKey/EIP-55/bech32/base58check con vectores SEP-0023, EIP-55 y BIP-173/350"`.

---

### Task 4: sonda en vivo `astra chain probe`

**Files:**
- Create: `lib/probe.mjs`; Modify: `lib/commands/chain.mjs`
- Test: `test/probe.test.mjs`

**Interfaces:**
- Produces: `probeChain(chain, {rpc?, timeoutMs=10000, fetchImpl=fetch, findExec=findExecutable}) → Promise<{ok, id, family, endpoint?, chainId?, expectedChainId?, blockNumber?, ledger?, passphrase?, protocolVersion?, latencyMs, warnings: string[], reason?}>`.

- [ ] **Step 1: test con fetch inyectado**

```js
// test/probe.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { probeChain } from '../lib/probe.mjs';
import { getChain } from '../lib/registry.mjs';

const jsonResponse = body => ({ ok: true, status: 200, json: async () => body, text: async () => JSON.stringify(body) });

test('EVM: chainId coincide y trae blockNumber', async () => {
  const fetchImpl = async (url, init) => { const req = JSON.parse(init.body); return jsonResponse({ jsonrpc: '2.0', id: req.id, result: req.method === 'eth_chainId' ? '0x2105' : '0x10' }); };
  const r = await probeChain(getChain('base'), { fetchImpl });
  assert.equal(r.ok, true); assert.equal(r.chainId, 8453); assert.equal(r.blockNumber, 16);
});

test('EVM: chainId distinto al esperado marca ok=false con razon', async () => {
  const fetchImpl = async () => jsonResponse({ jsonrpc: '2.0', id: 1, result: '0x1' });
  const r = await probeChain(getChain('base'), { fetchImpl });
  assert.equal(r.ok, false); assert.match(r.reason, /chainId/);
});

test('EVM: si el primer RPC falla prueba el siguiente y deja warning', async () => {
  let n = 0;
  const fetchImpl = async (url, init) => { n++; if (url.includes('rpc.tanenbaum.io')) throw new Error('ECONNRESET'); const req = JSON.parse(init.body); return jsonResponse({ jsonrpc: '2.0', id: req.id, result: req.method === 'eth_chainId' ? '0x1644' : '0x2a' }); };
  const r = await probeChain(getChain('syscoin-tanenbaum'), { fetchImpl });
  assert.equal(r.ok, true); assert.ok(r.warnings.length >= 1); assert.match(r.endpoint, /publicnode/);
});

test('Stellar: valida passphrase por RPC y Horizon', async () => {
  const fetchImpl = async (url, init) => {
    if (!init || !init.body) return jsonResponse({ network_passphrase: 'Test SDF Network ; September 2015', history_latest_ledger: 4509039 });
    const req = JSON.parse(init.body);
    return jsonResponse({ jsonrpc: '2.0', id: req.id, result: req.method === 'getNetwork' ? { passphrase: 'Test SDF Network ; September 2015', protocolVersion: 28 } : { sequence: 4509039 } });
  };
  const r = await probeChain(getChain('stellar-testnet'), { fetchImpl });
  assert.equal(r.ok, true); assert.equal(r.protocolVersion, 28); assert.equal(r.ledger, 4509039);
});

test('UTXO sin syscoin-cli: ok=false con razon clara', async () => {
  const r = await probeChain(getChain('syscoin-utxo'), { findExec: () => null });
  assert.equal(r.ok, false); assert.match(r.reason, /syscoin-cli/);
});
```

- [ ] **Step 2: fallar; Step 3: escribir `lib/probe.mjs`** — `rpcCall(fetchImpl, url, method, params, timeoutMs)` con `AbortSignal.timeout`; EVM: por cada endpoint (o `--rpc`), `eth_chainId` (parseInt hex) y `eth_blockNumber`; primer exito gana, fallos → `warnings`; si `chainId !== chain.chainId` → `ok:false, reason: 'chainId 1 no coincide con el esperado 8453'`. Stellar: si hay `rpc`, `getNetwork` + `getLatestLedger`; siempre `GET horizon` (JSON) y comparar `network_passphrase` con `chain.passphrase`; desajuste → `ok:false`. UTXO: si `findExec('syscoin-cli')` → `spawnSync(cli, ['getblockchaininfo'])` y parsear `blocks`; si no → `ok:false, reason: 'requiere nodo local (syscoin-cli no encontrado)'`. Medir `latencyMs` con `performance.now()`.

- [ ] **Step 4: conectar en `commands/chain.mjs`** (`probe <id> [--rpc <url>]`, exit 1 si `ok:false`). **Step 5: tests PASS + prueba en vivo** `node bin/astra.mjs chain probe base` y `... probe stellar-testnet` (deben decir OK). **Step 6: commit** `feat(chain): sonda en vivo por familia con fetch inyectable`.

---

### Task 5: `astra doctor`

**Files:**
- Create: `lib/doctor.mjs`, `lib/commands/doctor.mjs`
- Test: `test/doctor.test.mjs`

**Interfaces:**
- Produces: `TOOLS: {name, families:string[], role, hint?}[]`; `detectTools({env, cwd, findExec=findExecutable, runVer=runVersion}) → Tool[]` con `{name, found:boolean, path, version, families, role, hint}` + entradas sinteticas `hardhat (local)` y `wasm-target`; `verdictFor(family, tools) → {verdict:'SAFE'|'CAUTION'|'AVOID', reasons:string[]}`; `doctor(opts) → {tools, verdicts:{stellar, evm, utxo}, chain?}`.

- [ ] **Step 1: test con PATH falso**

```js
// test/doctor.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { detectTools, verdictFor, doctor } from '../lib/doctor.mjs';

const fake = present => ({ findExec: name => present.includes(name) ? `/fake/${name}` : null, runVer: p => `${p.split('/').pop()} 1.0.0`, env: { PATH: '' }, cwd: '/nowhere' });

test('con stellar y cargo la familia stellar es SAFE; sin ninguno AVOID', () => {
  assert.equal(verdictFor('stellar', detectTools(fake(['node', 'git', 'stellar', 'cargo']))).verdict, 'SAFE');
  assert.equal(verdictFor('stellar', detectTools(fake(['node', 'git', 'stellar']))).verdict, 'CAUTION');
  assert.equal(verdictFor('stellar', detectTools(fake(['node', 'git']))).verdict, 'AVOID');
});

test('evm: forge → SAFE; solo node → CAUTION con razon', () => {
  assert.equal(verdictFor('evm', detectTools(fake(['node', 'git', 'forge']))).verdict, 'SAFE');
  const v = verdictFor('evm', detectTools(fake(['node', 'git'])));
  assert.equal(v.verdict, 'CAUTION'); assert.ok(v.reasons.some(r => /forge|hardhat/i.test(r)));
});

test('utxo: syscoin-cli → SAFE; sin el → AVOID', () => {
  assert.equal(verdictFor('utxo', detectTools(fake(['syscoin-cli']))).verdict, 'SAFE');
  assert.equal(verdictFor('utxo', detectTools(fake([]))).verdict, 'AVOID');
});

test('doctor devuelve las tres familias y filtra por cadena', () => {
  const d = doctor({ ...fake(['node', 'git', 'forge']), chain: 'base' });
  assert.deepEqual(Object.keys(d.verdicts).sort(), ['evm', 'stellar', 'utxo']);
  assert.equal(d.chain.id, 'base'); assert.equal(d.chain.verdict, 'SAFE');
});
```

- [ ] **Step 2: fallar; Step 3: implementar** `TOOLS` = node (`*`), git (`*`), stellar (stellar, hint `cargo install --locked stellar-cli`), cargo (stellar, hint `https://rustup.rs`), rustup (stellar), forge (evm, hint `https://getfoundry.sh`), cast (evm), anvil (evm), syscoin-cli (utxo, hint `https://docs.syscoin.org`), docker (`*`, opcional). `detectTools` agrega `hardhat (local)` si existe `<cwd>/node_modules/.bin/hardhat` (o `.cmd`) y, si `rustup` esta, `wasm-target` con `found = /wasm32v1-none|wasm32-unknown-unknown/.test(rustup target list --installed)`. Reglas: stellar SAFE = stellar ∧ cargo (razon extra si falta wasm-target: "instalar target wasm32v1-none o wasm32-unknown-unknown"); CAUTION = uno de los dos; AVOID = ninguno. evm SAFE = forge ∨ hardhat local; CAUTION = solo node ("sin compilador: instalar Foundry o Hardhat"); AVOID = sin node. utxo SAFE = syscoin-cli; AVOID si no ("requiere nodo local"). `commands/doctor.mjs`: tabla `OK/--  nombre  version  ruta` + veredictos con razones; `--chain <id>` resalta la familia de esa cadena; exit 1 si la familia pedida es AVOID.

- [ ] **Step 4: tests PASS; `node bin/astra.mjs doctor`** en esta maquina debe mostrar stellar SAFE (stellar 28 + cargo 1.95), evm CAUTION (sin forge), utxo AVOID. **Step 5: commit** `feat(doctor): deteccion de toolchains y veredicto por familia`.

---

### Task 6: registro de despliegues `astra deployments`

**Files:**
- Create: `lib/deployments.mjs`, `lib/commands/deployments.mjs`
- Test: `test/deployments.test.mjs`

**Interfaces:**
- Produces: `DEPLOYMENTS_FILE = '.astra/deployments.json'`; `readDeployments(cwd) → {version:1, deployments: Entry[]}`; `validateEntry(entry) → {ok, errors:string[], normalized: Entry}`; `addDeployment(cwd, entry, {force=false, today}) → Entry`.
- `Entry = { id, chain, family, network, kind:'contract'|'token'|'account'|'other', label, address, tx?, commit?, wasmHash?, date, verified:boolean, verificationUrl?, notes? }`.

- [ ] **Step 1: tests**

```js
// test/deployments.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { addDeployment, readDeployments, validateEntry } from '../lib/deployments.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-dpl-'));
const SAC = 'CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC';

test('add valida cadena, red y direccion, y persiste con id y fecha', () => {
  const cwd = tmp();
  const e = addDeployment(cwd, { chain: 'stellar-testnet', kind: 'contract', label: 'hello', address: SAC }, { today: '2026-09-04' });
  assert.match(e.id, /^dpl_20260904_1$/); assert.equal(e.family, 'stellar'); assert.equal(e.network, 'testnet'); assert.equal(e.verified, false);
  assert.equal(readDeployments(cwd).deployments.length, 1);
});

test('rechaza direccion invalida para la familia, cadena desconocida y duplicados', () => {
  const cwd = tmp();
  assert.throws(() => addDeployment(cwd, { chain: 'base', kind: 'contract', label: 'x', address: SAC }), /direccion/);
  assert.throws(() => addDeployment(cwd, { chain: 'marte', kind: 'contract', label: 'x', address: SAC }), /cadena/);
  addDeployment(cwd, { chain: 'stellar-testnet', kind: 'contract', label: 'a', address: SAC });
  assert.throws(() => addDeployment(cwd, { chain: 'stellar-testnet', kind: 'contract', label: 'b', address: SAC }), /duplicad/);
  assert.equal(addDeployment(cwd, { chain: 'stellar-testnet', kind: 'contract', label: 'b', address: SAC }, { force: true }).label, 'b');
});

test('validateEntry normaliza EIP-55 y exige tx con formato', () => {
  const v = validateEntry({ chain: 'base', kind: 'contract', label: 'x', address: '0x5aaeb6053f3e94c9b9a09f33669435e7ef1beaed', tx: '0x' + 'ab'.repeat(32) });
  assert.equal(v.ok, true); assert.equal(v.normalized.address, '0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed');
  assert.equal(validateEntry({ chain: 'base', kind: 'contract', label: 'x', address: '0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed', tx: 'nope' }).ok, false);
});
```

- [ ] **Step 2: fallar; Step 3: implementar** (`tx` evm `^0x[0-9a-fA-F]{64}$`, stellar `^[0-9a-fA-F]{64}$`; `commit` default `gitHead(cwd)`; `date` default `fechaLocalISO()`; id `dpl_<YYYYMMDD>_<n>` con `n` = cantidad + 1; duplicado = misma `chain` + `address` salvo `force`; `writeAtomic`). Comando: `list [--json]` (tabla `red chain kind label address verified`), `add --chain --address [--kind] [--label] [--tx] [--commit] [--wasm-hash] [--verified] [--verification-url] [--notes] [--force]`.

- [ ] **Step 4: tests PASS. Step 5: commit** `feat(deployments): registro atomico de despliegues validado por familia`.

---

### Task 7: `astra check` (secretos, higiene, gate de mainnet)

**Files:**
- Create: `lib/check.mjs`, `lib/commands/check.mjs`
- Test: `test/check.test.mjs`

**Interfaces:**
- Produces: `listFiles(dir) → string[]` (git tracked + untracked no ignorados, o walk); `scanFile(file, text) → Finding[]`; `checkRepo(dir, {gate}) → {ok, findings: Finding[], gate?: {ok, items:[{name, ok, detail}]}}`; `Finding = {rule, severity:'error'|'warn', file, line?, hint}` (nunca contiene el valor secreto).

- [ ] **Step 1: tests** (secretos construidos en runtime)

```js
// test/check.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { spawnSync } from 'node:child_process';
import { checkRepo, scanFile } from '../lib/check.mjs';
import { encodeStrKey } from '../lib/address.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-check-'));
const git = (cwd, ...a) => spawnSync('git', a, { cwd, encoding: 'utf8' });
const seed = () => encodeStrKey(18 << 3, Uint8Array.from({ length: 32 }, (_, i) => i));
const evmKey = () => '0x' + 'a1'.repeat(32);

test('detecta semilla StrKey, clave EVM con contexto y mnemonico; ignora hashes sin contexto', () => {
  const f = scanFile('x.env', `STELLAR_SECRET=${seed()}\nPRIVATE_KEY=${evmKey()}\nWASM_HASH=${'ab'.repeat(32)}\nMNEMONIC="${Array(12).fill('abandon').join(' ')}"\n`);
  assert.deepEqual(f.map(x => x.rule).sort(), ['evm-private-key', 'mnemonic', 'stellar-secret-seed']);
  assert.ok(f.every(x => !JSON.stringify(x).includes(seed())));
});

test('repo limpio pasa; .env versionado y falta de .gitignore fallan', () => {
  const dir = tmp();
  git(dir, 'init', '-q'); fs.writeFileSync(path.join(dir, '.gitignore'), '.env\n'); fs.writeFileSync(path.join(dir, 'a.txt'), 'hola\n');
  assert.equal(checkRepo(dir).ok, true);
  fs.writeFileSync(path.join(dir, '.env.production'), `KEY=${evmKey()}\n`); git(dir, 'add', '-f', '.env.production');
  const r = checkRepo(dir);
  assert.equal(r.ok, false); assert.ok(r.findings.some(x => x.rule === 'env-tracked'));
  fs.rmSync(path.join(dir, '.gitignore'));
  assert.ok(checkRepo(dir).findings.some(x => x.rule === 'gitignore-env' && x.severity === 'warn'));
});

test('gate mainnet exige carta aprobada, testnet, auditoria apta y launch firmado', () => {
  const dir = tmp(); fs.writeFileSync(path.join(dir, '.gitignore'), '.env\n');
  let g = checkRepo(dir, { gate: 'mainnet' }).gate; assert.equal(g.ok, false); assert.equal(g.items.filter(i => i.ok).length, 0);
  fs.mkdirSync(path.join(dir, 'docs', 'astra'), { recursive: true }); fs.mkdirSync(path.join(dir, '.astra'));
  fs.writeFileSync(path.join(dir, 'docs', 'astra', 'chart.md'), '# Carta\naprobada: 2026-09-04\n');
  fs.writeFileSync(path.join(dir, '.astra', 'deployments.json'), JSON.stringify({ version: 1, deployments: [{ id: 'dpl_1', chain: 'stellar-testnet', family: 'stellar', network: 'testnet', kind: 'contract', label: 'x', address: 'CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC', date: '2026-09-04', verified: false }] }));
  fs.writeFileSync(path.join(dir, 'docs', 'astra', 'audit.md'), '# Auditoria\nveredicto: apto\n| ID | Severidad | Estado |\n|---|---|---|\n| A-1 | alta | cerrada |\n');
  fs.writeFileSync(path.join(dir, 'docs', 'astra', 'launch.md'), '# Launch\nfirmado_por: Operador\nfecha_firma: 2026-09-04\ncosto_estimado: 12 XLM\n');
  g = checkRepo(dir, { gate: 'mainnet' }).gate; assert.equal(g.ok, true, JSON.stringify(g));
  fs.writeFileSync(path.join(dir, 'docs', 'astra', 'audit.md'), '# Auditoria\nveredicto: apto\n| ID | Severidad | Estado |\n|---|---|---|\n| A-1 | critica | abierta |\n');
  assert.equal(checkRepo(dir, { gate: 'mainnet' }).gate.ok, false);
});
```

- [ ] **Step 2: fallar; Step 3: implementar `lib/check.mjs`**
  - `listFiles(dir)`: si `git rev-parse --is-inside-work-tree` ok → `git ls-files -z` ∪ `git ls-files -z -o --exclude-standard`; si no, walk recursivo. Saltar `node_modules/`, `target/`, `dist/`, `.git/`, archivos > 1 MiB o con byte NUL en los primeros 8 KiB, `*.lock`, `package-lock.json`, `pnpm-lock.yaml`, `Cargo.lock`.
  - Reglas por linea: `stellar-secret-seed` (`/\bS[A-Z2-7]{55}\b/` y `decodeStrKey(m).secret === true`, error); `evm-private-key` (`/\b(?:0x)?[0-9a-fA-F]{64}\b/` **solo** si la linea contiene `/(private[_-]?key|privkey|priv_key|secret|mnemonic|seed|deployer|signer)/i`, error); `mnemonic` (linea con `/(mnemonic|seed[ _-]?phrase|frase semilla)/i` y ≥ 12 palabras `[a-z]{3,8}`, error); `secret-assignment` (`/^\s*(?:export\s+)?[A-Z0-9_]*(PRIVATE_KEY|SECRET_KEY|SECRET|MNEMONIC|SEED)[A-Z0-9_]*\s*[=:]\s*["']?([^"'\s]{16,})/` con valor que no sea placeholder `/(your|xxx|changeme|example|<|\$\{|placeholder)/i`, warn; nunca en `.env.example|.env.sample|.env.template`).
  - Reglas por archivo: `env-tracked` (tracked y basename `/^\.env(\..+)?$/` salvo `example|sample|template`, error); `key-material-tracked` (tracked y `/(^|\/)(\.stellar|\.soroban)\/identity\/|\.keystore$|(^|\/)keystore\/.*\.json$|\.pem$|(^|\/)id_(rsa|ed25519)$/`, error); `gitignore-env` (`.gitignore` sin linea `.env`, `.env*`, `.env.*` o `*.env`, warn); `deployments-invalid` (si existe `.astra/deployments.json`: parsea y `validateEntry` por entrada, error).
  - Gate `mainnet` items: `carta-aprobada` (`docs/astra/chart.md` con `/^aprobada:\s*\d{4}-\d{2}-\d{2}\s*$/m`), `testnet-desplegada` (alguna entrada `network==='testnet'`), `auditoria-apta` (`docs/astra/audit.md` con `/^veredicto:\s*apto\s*$/m` y ninguna fila de tabla con severidad `critica|alta` y estado `abierta|abierto`), `launch-firmado` (`docs/astra/launch.md` con `firmado_por:` no vacio ni `<…>`, `fecha_firma:` fecha, `costo_estimado:` no vacio), `sin-secretos` (0 findings `error`). `gate.ok` = todos ok.
  - `checkRepo(dir, {gate})` → `ok = !findings.some(f => f.severity === 'error') && (!gate || gate.ok)`.
  - Comando: imprime findings (`FAIL/WARN rule file:line hint`), luego el gate (`OK/FAIL item detail`), resumen y exit 1 si `!ok`.

- [ ] **Step 4: tests PASS; correr `node bin/astra.mjs check` sobre el propio repo del CLI (debe pasar). Step 5: commit** `feat(check): escaner de secretos, higiene y gate de mainnet`.

---

### Task 8: catalogo de estandares y `astra standards search`

**Files:**
- Create: `data/standards.json`, `lib/standards.mjs`, `lib/commands/standards.mjs`
- Test: `test/standards.test.mjs`

**Interfaces:**
- Produces: `loadStandards() → Standard[]`; `searchStandards(query, {family?, limit=10}) → (Standard & {score})[]`; `Standard = {id, family:'stellar'|'evm'|'syscoin'|'base'|'payments'|'caip', title, url, status, tags:string[], summary}`.

- [ ] **Step 1: test**

```js
// test/standards.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { loadStandards, searchStandards } from '../lib/standards.mjs';

test('el catalogo tiene ids unicos, urls https y familias validas', () => {
  const all = loadStandards(); const ids = all.map(s => s.id);
  assert.equal(new Set(ids).size, ids.length); assert.ok(all.length >= 50);
  for (const s of all) { assert.match(s.url, /^https:\/\//, s.id); assert.ok(['stellar', 'evm', 'syscoin', 'base', 'payments', 'caip'].includes(s.family), s.id); assert.ok(s.tags.length > 0, s.id); }
});

test('busca por token, titulo y tags sin distinguir acentos', () => {
  assert.equal(searchStandards('token interface soroban')[0].id, 'SEP-0041');
  assert.equal(searchStandards('checksum direccion')[0].id, 'ERC-55');
  assert.equal(searchStandards('autenticación web')[0].id, 'SEP-0010');
  assert.ok(searchStandards('token', { family: 'evm' }).every(s => s.family === 'evm'));
  assert.deepEqual(searchStandards('zzzz-nada'), []);
});
```

- [ ] **Step 2: fallar; Step 3: escribir `data/standards.json`** con estas entradas (titulo y estado tomados de la fuente oficial el 2026-09-04; `summary` en español, 1 frase; tags en español e ingles):
  - Stellar SEP (url `https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-XXXX.md`): 0001 Stellar Info File (Active); 0006 Deposit and Withdrawal API (Active); 0007 URI Scheme to facilitate delegated signing (Active); 0010 Stellar Web Authentication (Active); 0011 Txrep (Active); 0012 KYC API (Active); 0023 Strkeys (Active); 0024 Hosted Deposit and Withdrawal (Active); 0030 Account Recovery (Draft); 0031 Cross-Border Payments API (Active); 0038 Anchor RFQ API (Draft); 0041 Soroban Token Interface (Draft); 0045 Stellar Web Authentication for Contract Accounts (Draft); 0046 Contract Meta (Active); 0048 Contract Interface Specification (Active); 0049 Upgradeable Contracts (Draft); 0050 Non-Fungible Tokens (Draft); 0054 Ledger Metadata Storage (Draft); 0055 Contract Build Verification (Draft); 0056 Tokenized Vault Standard (Draft); 0057 T-REX (Token for Regulated EXchanges) (Draft).
  - Stellar CAP (url `https://github.com/stellar/stellar-protocol/blob/master/core/cap-XXXX.md`): 0021 Preconditions (Final); 0040 Signed-Payload (Final); 0046 Soroban smart contract system overview (Final); 0051 Secp256r1 Verification (Final); 0053 TTL host functions (Final); 0058 Constructors for Soroban contracts (Final); 0059 Host functions for BLS12-381 (Final); 0067 Unified Asset Events (Final); 0074 Host functions for BN254 (Final); 0075 Poseidon/Poseidon2 (Final); 0079 muxed address strkey conversions (Implemented).
  - EVM (url `https://eips.ethereum.org/EIPS/eip-N`): ERC-20 Token Standard; ERC-55 Mixed-case checksum address encoding; ERC-165 Standard Interface Detection; ERC-721 Non-Fungible Token Standard; ERC-1155 Multi Token Standard; ERC-1167 Minimal Proxy Contract; ERC-1967 Proxy Storage Slots; ERC-2612 Permit; ERC-2981 NFT Royalty Standard; ERC-4337 Account Abstraction Using Alt Mempool; ERC-4626 Tokenized Vaults; ERC-6551 Token Bound Accounts (Review); ERC-7579 Minimal Modular Smart Accounts (Draft); EIP-155 Simple replay attack protection; EIP-712 Typed structured data hashing and signing; EIP-1559 Fee market change; EIP-2930 Optional access lists; EIP-4844 Shard Blob Transactions; EIP-7702 Set Code for EOAs. Todos `Final` salvo los marcados.
  - CAIP (url `https://github.com/ChainAgnostic/CAIPs/blob/main/CAIPs/caip-N.md`): CAIP-2 Blockchain ID Specification (Final); CAIP-10 Account ID Specification (Final); CAIP-19 Asset Type and Asset ID (Review).
  - Pagos: `x402` (`https://www.x402.org`, HTTP 402 para pagos maquina-a-maquina, repo `https://github.com/coinbase/x402`); `x402-stellar` (`https://developers.stellar.org/docs/build/apps/x402`); `MPP` Machine Payments Protocol (`https://mpp.dev`).
  - Docs Syscoin/Rollux/Base: `SYSCOIN-DOCS` (`https://docs.syscoin.org`, L1 UTXO merge-mined + NEVM chainId 57 + PoDA), `ROLLUX-DOCS` (`https://docs.rollux.com`, L2 OP Stack chainId 570), `BASE-DOCS` (`https://docs.base.org`, L2 OP Stack de Coinbase chainId 8453), `BASE-FAUCETS` (`https://docs.base.org/base-chain/tools/network-faucets`).

- [ ] **Step 4: `lib/standards.mjs`** — normalizar con `s.normalize('NFD').replace(/\p{M}/gu, '').toLowerCase()`; tokens de la consulta (≥ 2 chars); puntaje = id exacto 5, token en id 3, en title 2, en tags 2, en summary 1; ordenar por puntaje desc, id asc; `family` filtra; devolver `[]` si puntaje 0. Comando: `search <consulta...> [--family] [--json]` imprime `ID  titulo  (estado)  url`.

- [ ] **Step 5: tests PASS. Step 6: commit** `feat(standards): catalogo verificado SEP/CAP/ERC/EIP/CAIP/pagos y busqueda`.

---

### Task 9: servidor MCP `astra mcp`

**Files:**
- Create: `lib/mcp.mjs`, `lib/commands/mcp.mjs`
- Test: `test/mcp.test.mjs`

**Interfaces:**
- Produces: `MCP_TOOLS: {name, description, inputSchema, handler(args) → Promise<any>}[]`; `handleMessage(msg, {version}) → Promise<response|null>`; `startMcpServer({input, output, version})`.
- Tools: `astra_doctor {chain?}`, `astra_chain_list {}`, `astra_chain_info {id}`, `astra_chain_probe {id, rpc?}`, `astra_address_validate {target, address}`, `astra_check {cwd?, gate?}`, `astra_deployments_list {cwd?}`, `astra_deployments_add {cwd?, chain, address, kind?, label?, tx?, commit?, verified?, verificationUrl?, notes?}`, `astra_standards_search {query, family?, limit?}`.

- [ ] **Step 1: test e2e por stdio**

```js
// test/mcp.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import { spawn } from 'node:child_process';
import { fileURLToPath } from 'node:url';

const BIN = fileURLToPath(new URL('../bin/astra.mjs', import.meta.url));

function rpcSession(messages) {
  return new Promise((resolve, reject) => {
    const child = spawn(process.execPath, [BIN, 'mcp'], { stdio: ['pipe', 'pipe', 'pipe'] });
    let out = ''; child.stdout.on('data', d => { out += d; });
    child.on('error', reject);
    child.on('close', () => resolve(out.split('\n').filter(Boolean).map(l => JSON.parse(l))));
    for (const m of messages) child.stdin.write(JSON.stringify(m) + '\n');
    child.stdin.end();
  });
}

test('initialize, tools/list y tools/call responden por stdio', async () => {
  const res = await rpcSession([
    { jsonrpc: '2.0', id: 1, method: 'initialize', params: { protocolVersion: '2024-11-05', capabilities: {}, clientInfo: { name: 't', version: '0' } } },
    { jsonrpc: '2.0', method: 'notifications/initialized' },
    { jsonrpc: '2.0', id: 2, method: 'tools/list' },
    { jsonrpc: '2.0', id: 3, method: 'tools/call', params: { name: 'astra_chain_info', arguments: { id: 'base' } } },
    { jsonrpc: '2.0', id: 4, method: 'tools/call', params: { name: 'astra_address_validate', arguments: { target: 'evm', address: '0x' + 'ab'.repeat(32) } } },
    { jsonrpc: '2.0', id: 5, method: 'nope' },
  ]);
  const byId = Object.fromEntries(res.map(r => [r.id, r]));
  assert.equal(byId[1].result.protocolVersion, '2024-11-05'); assert.equal(byId[1].result.serverInfo.name, 'astra');
  assert.ok(byId[2].result.tools.map(t => t.name).includes('astra_check'));
  assert.equal(JSON.parse(byId[3].result.content[0].text).chainId, 8453);
  const secret = JSON.parse(byId[4].result.content[0].text); assert.equal(secret.secret, true); assert.ok(!byId[4].result.content[0].text.includes('abab'));
  assert.equal(byId[5].error.code, -32601);
  assert.equal(res.length, 5, 'la notificacion no genera respuesta');
});
```

- [ ] **Step 2: fallar; Step 3: implementar** — lector de lineas sobre `input` (buffer, split `\n`), cada linea JSON → `handleMessage`; sin `id` (notificacion) → no responder; `initialize` → `{protocolVersion:'2024-11-05', capabilities:{tools:{}}, serverInfo:{name:'astra', version}}`; `ping` → `{}`; `tools/list` → `{tools:[{name, description, inputSchema}]}`; `tools/call` → `{content:[{type:'text', text: JSON.stringify(result, null, 2)}], isError: !!result.isError}`; herramienta desconocida → `error -32602`; metodo desconocido → `-32601`; JSON invalido → `-32700` sin id. Todo log a `stderr`. `commands/mcp.mjs`: `startMcpServer({input: process.stdin, output: process.stdout, version: VERSION})` y mantener el proceso vivo hasta EOF.

- [ ] **Step 4: tests PASS. Step 5: commit** `feat(mcp): servidor MCP stdio con las 9 herramientas`.

---

### Task 10: `astra protocol` y `astra skills sync`

**Files:**
- Create: `lib/protocol.mjs`, `lib/skills.mjs`, `lib/commands/protocol.mjs`, `lib/commands/skills.mjs`
- Test: `test/protocol.test.mjs`, `test/skills.test.mjs`

**Interfaces:**
- Produces: `PROTOCOL_REPO_URL = 'https://github.com/orlando-vazquez-career/astra-protocol.git'`; `resolveProtocolDir({explicit?, env?, home?, cliRoot?, cwd?}) → {dir, source}|null` (candidato valido = contiene `ASTRA-PROTOCOL.md`); `fetchProtocol({dest, findExec?}) → {dir, action:'cloned'|'updated'}`.
- `RUNTIME_DIRS = {claude:'.claude/skills', codex:'.agents/skills', antigravity:'.agents/skills', kimi:'.kimi-code/skills', cursor:'.cursor/skills'}`; `expandRuntimes(list|string) → string[]` (directorios unicos; `all` = todos); `renderGenerated(text, relSource) → string` (inserta `<!-- GENERADO por astra skills sync desde <relSource> — no editar a mano; editar el canonico y re-correr el sync -->` justo despues del frontmatter, o al inicio si no hay); `syncSkills({from, to, runtimes, check}) → {skills:string[], targets:string[], written:string[], stale:string[], ok:boolean}`.

- [ ] **Step 1: tests**

```js
// test/skills.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { syncSkills, expandRuntimes, renderGenerated } from '../lib/skills.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-skills-'));
function fakeProtocol() {
  const dir = tmp(); fs.mkdirSync(path.join(dir, 'skills', 'astra-orbit'), { recursive: true });
  fs.writeFileSync(path.join(dir, 'skills', 'astra-orbit', 'SKILL.md'), '---\nname: astra-orbit\ndescription: fase 1\n---\n\nCuerpo.\n');
  fs.writeFileSync(path.join(dir, 'skills', 'astra-orbit', 'notas.md'), 'extra\n');
  return dir;
}

test('expandRuntimes deduplica directorios y entiende all', () => {
  assert.deepEqual(expandRuntimes('codex,antigravity,claude'), ['.agents/skills', '.claude/skills']);
  assert.equal(expandRuntimes('all').length, 4);
});

test('sync copia con cabecera de generado y --check detecta desvio', () => {
  const from = path.join(fakeProtocol(), 'skills'); const to = tmp();
  const r = syncSkills({ from, to, runtimes: ['claude', 'codex'] });
  assert.equal(r.ok, true); assert.equal(r.written.length, 4);
  const copy = fs.readFileSync(path.join(to, '.claude', 'skills', 'astra-orbit', 'SKILL.md'), 'utf8');
  assert.ok(copy.startsWith('---\nname: astra-orbit')); assert.match(copy, /GENERADO por astra skills sync/);
  assert.equal(syncSkills({ from, to, runtimes: ['claude', 'codex'], check: true }).ok, true);
  fs.writeFileSync(path.join(to, '.agents', 'skills', 'astra-orbit', 'SKILL.md'), 'editado a mano');
  const c = syncSkills({ from, to, runtimes: ['claude', 'codex'], check: true });
  assert.equal(c.ok, false); assert.equal(c.stale.length, 1);
});

test('renderGenerated sin frontmatter pone la cabecera al inicio', () => {
  assert.ok(renderGenerated('hola\n', 'x/SKILL.md').startsWith('<!-- GENERADO'));
});
```

```js
// test/protocol.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { resolveProtocolDir } from '../lib/protocol.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-proto-'));
const mk = dir => { fs.mkdirSync(dir, { recursive: true }); fs.writeFileSync(path.join(dir, 'ASTRA-PROTOCOL.md'), '# ASTRA\n'); return dir; };

test('orden de resolucion: explicito > env > home > hermano del CLI > ./ASTRA', () => {
  const explicit = mk(path.join(tmp(), 'E')); const env = mk(path.join(tmp(), 'V')); const home = tmp(); mk(path.join(home, '.astra', 'protocol'));
  assert.equal(resolveProtocolDir({ explicit, env: { ASTRA_PROTOCOL_DIR: env }, home }).dir, explicit);
  assert.equal(resolveProtocolDir({ env: { ASTRA_PROTOCOL_DIR: env }, home }).dir, env);
  assert.equal(resolveProtocolDir({ env: {}, home }).source, 'home');
  assert.equal(resolveProtocolDir({ env: {}, home: tmp(), cliRoot: tmp(), cwd: tmp() }), null);
});
```

- [ ] **Step 2: fallar; Step 3: implementar** (`fetchProtocol`: `git clone --depth 1 <url> <dest>` o `git -C <dest> pull --ff-only`; comandos `protocol path` (imprime dir y fuente; exit 1 si no hay), `protocol fetch [--dir]`, `skills sync [--from <dir>] [--runtimes] [--check]` con `from` default `<protocolDir>/skills`). **Step 4: tests PASS. Step 5: commit** `feat(skills,protocol): sync multi-runtime y resolucion del directorio del protocolo`.

---

### Task 11: `astra init`

**Files:**
- Create: `lib/init.mjs`, `lib/commands/init.mjs`
- Test: `test/init.test.mjs`

**Interfaces:**
- Produces: `BLOCK_START = '<!-- astra:start -->'`, `BLOCK_END = '<!-- astra:end -->'`; `upsertBlock(text, block) → string`; `astraBlock({chains, protocolDir}) → string`; `initProject({cwd, chains, runtimes, protocolDir, today}) → {created:string[], updated:string[], skipped:string[], warnings:string[]}`.

- [ ] **Step 1: tests**

```js
// test/init.test.mjs
import test from 'node:test';
import assert from 'node:assert/strict';
import fs from 'node:fs';
import os from 'node:os';
import path from 'node:path';
import { initProject, upsertBlock, BLOCK_START, BLOCK_END } from '../lib/init.mjs';

const tmp = () => fs.mkdtempSync(path.join(os.tmpdir(), 'astra-init-'));
function fakeProtocol() {
  const dir = tmp();
  fs.mkdirSync(path.join(dir, 'templates'), { recursive: true }); fs.mkdirSync(path.join(dir, 'skills', 'astra'), { recursive: true });
  for (const t of ['orbit', 'chart', 'audit', 'launch']) fs.writeFileSync(path.join(dir, 'templates', `${t}.template.md`), `# ${t}\n`);
  fs.writeFileSync(path.join(dir, 'skills', 'astra', 'SKILL.md'), '---\nname: astra\ndescription: entrada\n---\nhola\n');
  fs.writeFileSync(path.join(dir, 'ASTRA-PROTOCOL.md'), '# ASTRA\n');
  return dir;
}

test('upsertBlock agrega y luego reemplaza sin duplicar', () => {
  const a = upsertBlock('# Repo\n', `${BLOCK_START}\nv1\n${BLOCK_END}`);
  const b = upsertBlock(a, `${BLOCK_START}\nv2\n${BLOCK_END}`);
  assert.equal(b.split(BLOCK_START).length, 2); assert.match(b, /v2/); assert.doesNotMatch(b, /v1/);
});

test('init crea estado, docs, gitignore, bloques y skills; es idempotente', () => {
  const cwd = tmp(); const protocolDir = fakeProtocol();
  fs.writeFileSync(path.join(cwd, 'AGENTS.md'), '# Mi repo\n');
  const r = initProject({ cwd, chains: ['stellar-testnet'], runtimes: ['claude', 'codex'], protocolDir, today: '2026-09-04' });
  for (const f of ['.astra/astra.json', '.astra/deployments.json', 'docs/astra/orbit.md', 'docs/astra/chart.md', 'docs/astra/audit.md', 'docs/astra/launch.md', 'docs/astra/devlogs/.gitkeep', '.gitignore', 'CLAUDE.md', 'GEMINI.md', '.claude/skills/astra/SKILL.md', '.agents/skills/astra/SKILL.md']) assert.ok(fs.existsSync(path.join(cwd, f)), f);
  assert.ok(r.updated.includes('AGENTS.md'));
  assert.match(fs.readFileSync(path.join(cwd, 'AGENTS.md'), 'utf8'), /# Mi repo[\s\S]*astra:start/);
  assert.match(fs.readFileSync(path.join(cwd, '.gitignore'), 'utf8'), /^\.env$/m);
  assert.deepEqual(JSON.parse(fs.readFileSync(path.join(cwd, '.astra', 'astra.json'), 'utf8')).chains, ['stellar-testnet']);
  const r2 = initProject({ cwd, chains: ['stellar-testnet'], runtimes: ['claude', 'codex'], protocolDir, today: '2026-09-04' });
  assert.equal(r2.created.length, 0);
  assert.equal(fs.readFileSync(path.join(cwd, 'AGENTS.md'), 'utf8').split('astra:start').length, 2);
});

test('sin protocolo resoluble avisa y crea igual el estado minimo', () => {
  const cwd = tmp();
  const r = initProject({ cwd, chains: [], runtimes: ['claude'], protocolDir: null, today: '2026-09-04' });
  assert.ok(r.warnings.some(w => /astra protocol fetch/.test(w)));
  assert.ok(fs.existsSync(path.join(cwd, '.astra', 'deployments.json')));
});
```

- [ ] **Step 2: fallar; Step 3: implementar** — `astraBlock` contiene: titulo `## ASTRA — protocolo de desarrollo Web3`, cadenas elegidas, las siete fases con sus artefactos (`docs/astra/*.md`, `.astra/deployments.json`), los axiomas A1, A2, A3, A7, y los comandos `astra doctor`, `astra check --gate mainnet`, `astra deployments add`; nunca menciona otros protocolos. Archivos `.md` inexistentes se crean con `# <nombre>\n\n` + bloque; `.gitignore` recibe el bloque `# astra:start` … `# astra:end` con `.env`, `.env.*`, `!.env.example`, `.stellar/identity/`, `.soroban/identity/`. Plantillas copiadas solo si el destino no existe (`skipped` si ya estaba). `skills` via `syncSkills`. Comando `init`: resuelve protocolo con `resolveProtocolDir` (`--protocol-dir`), imprime creados/actualizados/omitidos/avisos.

- [ ] **Step 4: tests PASS. Step 5: commit** `feat(init): scaffolding idempotente de un repo adherente a ASTRA`.

---

### Task 12: CI, README y docs de `astra-cli`

**Files:**
- Create: `.github/workflows/ci.yml`, `README.md`, `AGENTS.md`, `CLAUDE.md`

- [ ] **Step 1: `ci.yml`**

```yaml
name: ci
on: { push: { branches: [main] }, pull_request: {} }
jobs:
  test:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [20, 22]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "${{ matrix.node }}" }
      - run: node --test test/
      - run: node bin/astra.mjs check
      - run: node bin/astra.mjs doctor
```

- [ ] **Step 2: `README.md`** (español): que es, instalacion (`git clone` + `node bin/astra.mjs` o `npm link`; sin dependencias), tabla de comandos, registro MCP para Claude Code (`claude mcp add astra -- node <ruta>/bin/astra.mjs mcp`), Codex (`codex mcp add astra -- node <ruta>/bin/astra.mjs mcp`), Cursor / Kimi Code / Gemini CLI / OpenCode (JSON `{"mcpServers":{"astra":{"command":"node","args":["<ruta>/bin/astra.mjs","mcp"]}}}`), politica de claves (A2/A7), cadenas soportadas y como agregar una, licencia. `AGENTS.md`: reglas del repo (cero deps, `node --test`, un modulo por responsabilidad, mensajes en español, nunca imprimir secretos, verificar URLs en vivo antes de agregarlas al registro). `CLAUDE.md`: `@AGENTS.md`.

- [ ] **Step 3: `node --test test/` y `node bin/astra.mjs check`** → verde. **Step 4: commit** `docs: README, guia de agentes y CI multiplataforma`.

---

### Task 13: repo `astra-protocol` — base, entrypoints y `ASTRA-PROTOCOL.md`

**Files:**
- Create: `C:/dev/protocols/ASTRA/{LICENSE,.gitignore,.gitattributes,README.md,AGENTS.md,CLAUDE.md,GEMINI.md,ASTRA-PROTOCOL.md}`

- [ ] **Step 1: `LICENSE`** identico al de la Task 1. `.gitignore`: `.env`, `.env.*`, `!.env.example`, `.DS_Store`, `*.tmp`. `.gitattributes`: `* text=auto eol=lf`.

- [ ] **Step 2: `AGENTS.md`** (fuente unica de reglas del repo): que es este repo (el protocolo, no un proyecto adherente), donde esta cada cosa (mapa corto), reglas: idioma español neutro sin voseo; nunca escribir claves ni datos personales; toda cadena o URL nueva se verifica en vivo y se anota `verifiedAt`; toda edicion a `skills/` se sigue de `astra skills sync` (o `node <astra-cli>/bin/astra.mjs skills sync --from skills --runtimes claude,codex`) y el CI/`--check` lo verifica; versionado SemVer del protocolo con fila en Version history; commits con prefijo. `CLAUDE.md` = `@AGENTS.md` + una linea. `GEMINI.md` = "Este repositorio se rige por `AGENTS.md`; leerlo completo antes de actuar" + las cinco reglas duras repetidas en 5 vinetas.

- [ ] **Step 3: `ASTRA-PROTOCOL.md`** con esta estructura y contenido (cada seccion completa, sin marcadores):
  1. Titulo `# ASTRA — Protocolo de desarrollo Web3 agentico`; cabecera `**Version**: 0.1.0 · **Genesis**: 2026-09-04`, pronunciacion y etimologia, licencia, cita "Por que 0.1.x" (promocion a 1.0.0: primer lanzamiento a mainnet de punta a punta con Gate 2 firmado).
  2. TL;DR: diagrama de fases y gates (`Orbita → Carta → ⸸ Gate 1 ⸸ → Construccion → Ensayo → Auditoria → ⸸ Gate 2: Mainnet ⸸ → Lanzamiento → Bitacora`) + tabla fase/pregunta/output/herramienta (la de la spec §4.1).
  3. El hueco que ASTRA llena (las seis invariantes de la spec §1).
  4. Axiomas A1–A10 (spec §4.2, texto completo).
  5. Cuando usar ASTRA / cuando no (si: contrato, token, dApp con firma, integracion de pagos on-chain, ZK on-chain, deploy a testnet/mainnet; no: backend sin cadena, UI sin firma, scripts de analisis off-chain de solo lectura → protocolo general de software).
  6. Las siete fases, una subseccion cada una con: objetivo, entradas, mecanismo (pasos numerados), herramientas (`astra …`), output, criterio de done. Gate 1 y Gate 2 con checklist completo (Gate 2: carta aprobada, testnet registrada, auditoria apta sin criticas/altas abiertas, `astra check --gate mainnet` verde, costo estimado escrito, alias de clave de mainnet nombrado y distinto del de testnet, plan de pausa/upgrade/rollback, quien firma y cuando).
  7. Roles (tabla spec §4.3) + regla de la caja (objetivo, entradas, done, limite) + A9.
  8. Multi-cadena: el perfil de cadena (campos de `chains.json`), tabla de las 9 cadenas de genesis con chainId/caip2/red, familias y que cambia por familia (firma, direcciones, explorers, verificacion), como agregar una cadena (entrada + guia + sonda en vivo + `verifiedAt`).
  9. Multi-vendor: entrypoints, skills, agentes, MCP, tiers (`primary` = el modelo mas capaz del vendor para cartografo/auditor/oficial; `secondary` = modelo economico para construccion mecanica), tabla de directorios por runtime.
  10. Herramientas: tabla de comandos `astra` (spec §5) y lista de lo que no hace. El protocolo funciona sin el CLI: cada checklist se puede correr a mano.
  11. Artefactos del repo adherente: arbol `.astra/` + `docs/astra/`.
  12. Integracion de skills externas: remite a `guides/skills-externas.md`.
  13. Independencia: ASTRA no depende de ningun otro protocolo, memoria o daemon; cualquier sistema superior lo invoca leyendo este documento y llamando al CLI. Compone bien con un protocolo general de software para la parte no-cadena (mismo gate humano el mismo dia).
  14. No-objetivos (spec §9). 15. Glosario corto (remite a `docs/GLOSSARY.md`). 16. Referencias (developers.stellar.org, docs.syscoin.org, docs.rollux.com, docs.base.org, eips.ethereum.org, github.com/stellar/stellar-protocol, x402.org, mpp.dev, agentskills.io, modelcontextprotocol.io). 17. Version history: `| 0.1.0 | 2026-09-04 | Genesis | … |`.

- [ ] **Step 4: `README.md`**: que es (3 parrafos), las siete fases (tabla corta), instalacion en un proyecto (`astra init --chain stellar-testnet --runtimes all`), estructura del repo, relacion con `astra-cli`, licencia. Sin badges falsos.

- [ ] **Step 5: `git init -b main`, commit** `docs: genesis del protocolo ASTRA v0.1.0 (protocolo, entrypoints, licencia)`.

---

### Task 14: guias

**Files:**
- Create: `guides/gates.md`, `guides/keys-and-secrets.md`, `guides/audit-checklist.md`, `guides/standards-map.md`, `guides/agentic-payments.md`, `guides/runtimes.md`, `guides/skills-externas.md`, `guides/chains/stellar.md`, `guides/chains/syscoin.md`, `guides/chains/base.md`, `guides/chains/evm-generico.md`, `guides/README.md`

- [ ] **Step 1: guias transversales**
  - `gates.md`: Gate 1 (Carta) y Gate 2 (Mainnet): quien, que revisa, checklist, como se registra (`aprobada:` en `chart.md`; `firmado_por/fecha_firma/costo_estimado` en `launch.md`), que pasa si falla (vuelve a la fase anterior con nota), regla de "el mismo dia" al componer con otro protocolo.
  - `keys-and-secrets.md`: A2/A7 operativos: alias en keystores nativos (`stellar keys generate <alias>`, `stellar keys fund`, `cast wallet import <alias> --interactive`, `syscoin-cli` wallet), separacion testnet/mainnet por alias y por `.env` no versionado, que hacer si un agente ve una clave (rotar de inmediato, registrar en bitacora), lo que `astra check` detecta, lo que no detecta.
  - `audit-checklist.md`: checklist universal (control de acceso, reentrancy, aritmetica y overflow, validacion de entradas, oraculos y manipulacion de precio, front-running/MEV, upgradeabilidad y storage layout, pausas y limites, eventos y observabilidad, dependencias y versiones, tests de propiedad/fuzz) + secciones por familia (Stellar: TTL de storage persistente/temporal, `require_auth` en cada entrada, limites de recursos, `extend_ttl`, SEP-41 completo, SAC vs token custom; EVM: `msg.sender` vs `tx.origin`, `delegatecall`, proxies ERC-1967, aprobaciones ERC-20, reentrancy en `receive`, `selfdestruct`, gas griefing; Syscoin: finalidad y merge-mining, puente L1↔NEVM, PoDA como DA para Rollux; Base: secuenciador centralizado y retiros de 7 dias, paymasters/4337/7702, verificacion en Basescan y Blockscout). Formato de `audit.md`: tabla `| ID | Severidad (critica/alta/media/baja) | Estado (abierta/cerrada/aceptada) | Hallazgo | Fix |` y `veredicto: apto|no-apto`.
  - `standards-map.md`: mapa caso de uso → estandar (token fungible: SEP-0041/SAC en Stellar, ERC-20 + ERC-2612 en EVM; NFT: SEP-0050 / ERC-721/1155/2981; upgrade: SEP-0049 + CAP-0058 / ERC-1967 + ERC-1167; auth web: SEP-0010 / EIP-712 + SIWE; smart accounts: SEP-0045 + CAP-0051 / ERC-4337 + EIP-7702 + ERC-7579; anchors/fiat: SEP-0006/0024/0031/0012; direcciones: SEP-0023 / ERC-55 / bech32; identificadores multi-cadena: CAIP-2/10/19; ZK: CAP-0059/0074/0075; pagos agenticos: x402 y MPP). Todo con `astra standards search "<consulta>"` de ejemplo.
  - `agentic-payments.md`: x402 (HTTP 402, flujo cliente/servidor/facilitador, en Stellar los clientes firman auth entries y el facilitador paga fees; en EVM/Base es el ecosistema de origen de Coinbase) y MPP (Machine Payments Protocol, modo por cargo vs canal), cuando cada uno, riesgos (dependencia del facilitador, limites por sesion, replay), como registrarlo en la Carta.
  - `runtimes.md`: tabla runtime → archivo de reglas → directorio de skills → registro MCP: Claude Code (`CLAUDE.md`, `.claude/skills`, `claude mcp add`), Codex (`AGENTS.md`, `.agents/skills`, `codex mcp add`), Antigravity/Gemini CLI (`GEMINI.md`/`AGENTS.md`, `.agents/skills`, JSON de MCP), Kimi Code (`AGENTS.md`, `.kimi-code/skills`, `~/.kimi-code/mcp.json`), Cursor (`AGENTS.md`, `.cursor/skills`, `.cursor/mcp.json`), OpenCode (`AGENTS.md`, `opencode.json`). Tiers `primary/secondary` y como se ligan en cada uno. Prueba de humo por runtime: `astra doctor`, invocar la skill `astra`.
  - `skills-externas.md`: politica (no se vendorizan; licencia propia; se instalan con `astra skills sync --from <dir> --runtimes …`), tabla skill externa → fase ASTRA para el pack `stellar-build`/`stellar-dev-skill` (smart-contracts → Construccion; dapp → Construccion/Ensayo; assets → Carta/Construccion; data → Ensayo/Bitacora; agentic-payments → Carta/Construccion; zk-proofs → Carta/Construccion; standards → Orbita/Carta; deploy-stellar-mainnet → Auditoria/Lanzamiento; find-stellar-idea, stellar-competitive-landscape, scf-round-watcher → antes de Orbita (descubrimiento); stellar-help, navigate-skills → navegacion), y la lista de skills que **no** pertenecen a ASTRA (las de gestion de producto, revision generica y personas van a un protocolo general de software; UX a uno de diseño; `remove-ai-marks` se excluye por politica de trazabilidad).
- [ ] **Step 2: guias por cadena** (cada una con: resumen, redes y chainId/passphrase, toolchain e instalacion, faucets, formato de direcciones y ejemplo valido (usar los vectores publicos), build/test/deploy con comandos reales, verificacion publica, estandares aplicables, riesgos para la Auditoria, enlaces verificados):
  - `stellar.md`: Stellar CLI 28 (`stellar contract build`, `stellar contract deploy --wasm … --source-account <alias> --network testnet`, `stellar contract invoke`, `stellar contract id asset --asset native --network testnet` → `CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC`), friendbot, Horizon vs RPC, `stellar.expert`, SEP-0055 para verificacion de build, SAC, trustlines y reservas.
  - `syscoin.md`: tres capas (L1 UTXO merge-mined con Bitcoin, `syscoin-cli`, direcciones `sys1…`/`S…`; NEVM chainId 57 / Tanenbaum 5700, EVM completo con Foundry/Hardhat, explorers Blockscout `explorer.syscoin.org` / `explorer.tanenbaum.io`, faucet `faucet.tanenbaum.io`; Rollux L2 OP Stack chainId 570 / 57000, `rpc.rollux.com`, `explorer.rollux.com`, PoDA como capa de datos), puente y riesgos, nota de que el RPC de Rollux testnet no respondio en la verificacion del 2026-09-04.
  - `base.md`: L2 OP Stack de Coinbase, chainId 8453 / Sepolia 84532, RPC publicos, Basescan + Blockscout, verificacion `forge verify-contract`, faucets en docs.base.org, smart wallets/paymasters (ERC-4337, EIP-7702), retiros de 7 dias, secuenciador.
  - `evm-generico.md`: cualquier chainId via `astra chain probe --rpc <url>`, `cast chain-id`, como agregar la entrada al registro, EIP-155 y replay, EIP-1559.
- [ ] **Step 3: `guides/README.md`** indice. **Step 4: commit** `docs: guias de gates, claves, auditoria, estandares, pagos, runtimes, skills externas y cadenas`.

---

### Task 15: plantillas y agentes

**Files:**
- Create: `templates/orbit.template.md`, `templates/chart.template.md`, `templates/audit.template.md`, `templates/launch.template.md`, `templates/devlog.template.md`, `templates/deployments.schema.json`, `templates/agents/{navegante,cartografo,forjador,auditor-de-cadena,oficial-de-lanzamiento}.md`, `templates/agents/README.md`

- [ ] **Step 1: plantillas** con bloque `<!-- INSTRUCCIONES -->` al inicio (como se llena, quien, cuando) y campos parseables por `astra check`: `chart.md` termina su cabecera con `aprobada: <YYYY-MM-DD o vacio>`; `audit.md` con `veredicto: <apto|no-apto>` y tabla `| ID | Severidad | Estado | Hallazgo | Fix |`; `launch.md` con `firmado_por:`, `fecha_firma:`, `costo_estimado:`, `alias_mainnet:`, `plan_rollback:`. `orbit.md`: cadena(s), red, veredicto de `astra doctor`, estandares elegidos, modelo de amenazas inicial (activos, atacantes, superficies), decision go/no-go. `devlog.md`: fecha, fase, que se hizo, direcciones registradas, costos reales, aprendizajes, deuda. `deployments.schema.json`: JSON Schema draft-07 del registro (mismos campos que `Entry` de la Task 6).
- [ ] **Step 2: agentes** (frontmatter `name`, `description`, `tools`, `model: inherit`; cuerpo: tier, lugar en el protocolo, que lee, que produce, que no hace, criterio de done; el auditor y el oficial en solo lectura o CLI nativo respectivamente; ninguno lee claves). `README.md` con la tabla de roles y como usarlos en cualquier vendor (copiar como prompt de rol si el runtime no tiene agentes).
- [ ] **Step 3: commit** `docs: plantillas de artefactos y roster de agentes`.

---

### Task 16: skills, sync multi-runtime, glosario, mapa y genesis

**Files:**
- Create: `skills/{astra,astra-orbit,astra-chart,astra-build,astra-testnet,astra-audit,astra-launch,astra-logbook,astra-standards}/SKILL.md`, `.claude/skills/*` y `.agents/skills/*` (generados), `docs/GLOSSARY.md`, `docs/MAPA.md`, `genesis/README.md`, `genesis/devlogs/2026-09-04-genesis.md`

- [ ] **Step 1: skills** — frontmatter `name`, `description` (con disparadores: "usa cuando el usuario dice …"), cuerpo de 40–90 lineas: cuando aplica, pasos numerados, comandos `astra` exactos, artefacto que produce, criterio de done, que no hacer (A2/A7). `astra` (entrada): lee `.astra/astra.json` y `docs/astra/*.md`, determina la fase actual (regla: sin `orbit.md` → Orbita; `chart.md` sin `aprobada:` → Carta/Gate 1; sin entrada testnet → Construccion/Ensayo; sin `audit.md` apto → Auditoria; sin `launch.md` firmado → Gate 2; sin entrada mainnet → Lanzamiento; si no → Bitacora) y propone el siguiente paso.
- [ ] **Step 2: sync** — `node C:/dev/tools/astra-cli/bin/astra.mjs skills sync --from skills --runtimes claude,codex` y luego `… --check` → OK.
- [ ] **Step 3: `docs/GLOSSARY.md`** (Orbita, Carta, Ensayo, Gate de mainnet, perfil de cadena, familia, alias de clave, StrKey, EIP-55, CAIP-2, SAC, PoDA, OP Stack, facilitador x402, tier), `docs/MAPA.md` (mapa de archivos del repo con proposito), `genesis/README.md` (que es la genesis y que es intocable), `genesis/devlogs/2026-09-04-genesis.md` (decisiones del dia, enlaces a spec y plan, verificaciones en vivo hechas, lo que quedo fuera).
- [ ] **Step 4: commit** `feat: skills canonicas y copias generadas para Claude Code y Codex/Antigravity; glosario, mapa y genesis`.

---

### Task 17: verificacion cruzada y publicacion en GitHub

- [ ] **Step 1: escaneo** — `node C:/dev/tools/astra-cli/bin/astra.mjs check` dentro de `C:/dev/tools/astra-cli` y de `C:/dev/protocols/ASTRA` → ambos OK. `grep -ri "kairos\|seele\|mnema\|peitho\|lumen" C:/dev/protocols/ASTRA C:/dev/tools/astra-cli --exclude-dir=.git -l` → solo `docs/superpowers/**` (spec y plan) y `guides/skills-externas.md` pueden mencionar AEGIS/LUMEN; **ninguno** puede mencionar KAIROS ni SEELE salvo la spec y este plan. Si aparece otro, corregir antes de publicar. `grep -rE "[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[a-z]{2,}"` → sin correos personales.
- [ ] **Step 2: `node --test test/`** en `astra-cli` → todo verde.
- [ ] **Step 3: publicar** (autorizado explicitamente por el operador):

```bash
cd /c/dev/protocols/ASTRA && gh repo create orlando-vazquez-career/astra-protocol --public --source=. --remote=origin --push --description "ASTRA — protocolo de desarrollo Web3 agentico: multi-cadena (Stellar, Syscoin, Base, EVM), multi-vendor de IA, independiente"
gh repo edit orlando-vazquez-career/astra-protocol --add-topic web3 --add-topic stellar --add-topic soroban --add-topic syscoin --add-topic base --add-topic evm --add-topic ai-agents --add-topic protocol --add-topic agent-skills
cd /c/dev/tools/astra-cli && gh repo create orlando-vazquez-career/astra-cli --public --source=. --remote=origin --push --description "astra — CLI y servidor MCP del protocolo ASTRA: doctor, cadenas, direcciones, secretos, gate de mainnet, skills, cero dependencias"
gh repo edit orlando-vazquez-career/astra-cli --add-topic web3 --add-topic stellar --add-topic syscoin --add-topic base --add-topic evm --add-topic mcp --add-topic cli --add-topic ai-agents
```

- [ ] **Step 4: CI** — `gh run list -R orlando-vazquez-career/astra-cli --limit 1` y esperar (`gh run watch`) hasta ver la matriz verde en los tres sistemas operativos. Si falla en Windows o macOS, corregir y volver a pushear antes de reportar.

---

### Task 18: KAIROS enruta a ASTRA (KAIROS → ASTRA, nunca al reves)

**Files:**
- Modify: `C:/dev/protocols/KAIROS/daemon/kairos-daimonion.mjs:265` (lista de protocolos), `daemon/test/kairos-daimonion.test.mjs` (test nuevo al final), `guides/routing-matrix.md`, `KAIROS-PROTOCOL.md`, `.claude/commands/kairos.md`, `.kimi-code/skills/kairos/SKILL.md`, `templates/wager.template.md`, `templates/proposal.template.md`, `observatory/index.html`, `observatory/cosmografia.html`, `CLAUDE.md`, `README.md`, `guides/observatory.md`
- Create: `docs/devlogs/2026-09-04-astra-hermano.md`

- [ ] **Step 1: test primero** (agregar al final de `daemon/test/kairos-daimonion.test.mjs`):

```js
test('ASTRA es hermano conocido: PROTOCOL_DIRS y contextOfWager lo resuelven al directorio hermano', async () => {
  const { PROTOCOL_DIRS, contextOfWager } = await import('../kairos-daimonion.mjs');
  assert.equal(PROTOCOL_DIRS.astra, path.resolve(REPO, '..', 'ASTRA'));
  assert.deepEqual(contextOfWager({ source_protocol: 'astra' }),
    { protocol: 'astra', project: 'astra', directory: path.resolve(REPO, '..', 'ASTRA') });
});
```

  Correr `node --test daemon/test/kairos-daimonion.test.mjs` → falla (`PROTOCOL_DIRS.astra` undefined).

- [ ] **Step 2: `kairos-daimonion.mjs`** — cambiar `['mnema', 'lumen', 'aegis', 'peitho', 'kairos']` por `['mnema', 'lumen', 'aegis', 'peitho', 'astra', 'kairos']` y el comentario "los cinco protocolos hermanos" → "los protocolos hermanos". Test → PASS. Correr la suite completa `node --test "daemon/test/*.test.mjs"` → 182 pass.

- [ ] **Step 3: docs de routing**
  - `guides/routing-matrix.md`: fila nueva despues de PEITHO: `| Contrato, token, dApp con firma, deploy a testnet/mainnet, pagos on-chain (x402/MPP), ZK on-chain — Stellar, Syscoin, Base o cualquier EVM | **ASTRA** | Siete fases con dos gates (Carta, Mainnet). Herramientas: \`astra doctor\`, \`astra check --gate mainnet\`, MCP \`astra mcp\`. Protocolo en \`C:/dev/protocols/ASTRA\`. |` y en anti-patrones: "Rutear un deploy a mainnet como si fuera AEGIS — mainnet es irreversible; va a ASTRA con su Gate 2".
  - `KAIROS-PROTOCOL.md`: version `0.5.0`, cabecera `**ASTRA hermano**: 2026-09-04`, `**Hermano de**` agrega `[ASTRA](../ASTRA/ASTRA-PROTOCOL.md) v0.1.0`; §hueco: lista agrega `- **ASTRA** construye en cadena (contratos, tokens, dApps → testnet → mainnet).` y "cuatro protocolos que producen" → "cinco"; diagrama: agregar `ASTRA` a la fila `MNEMA   LUMEN   AEGIS  PEITHO  ASTRA`; TL;DR: `[MNEMA | LUMEN | AEGIS | PEITHO | ASTRA hacen su trabajo]`; Fase 2 tabla: fila ASTRA (misma señal que la matriz); Fase 3 enum: `"mnema" | "lumen" | "aegis" | "peitho" | "astra" | "kairos"`; §Relacion: fila `| **ASTRA** | ¿Que desplegamos en cadena? | ¿El contrato sobrevivio a mainnet sin incidente ni redeploy? (los wagers de ASTRA nacen en Gate 2: fecha de resolucion = 30 dias post-lanzamiento) |` y "Los cinco hermanos" → "Los seis hermanos"; §Referencias: `- ASTRA-PROTOCOL.md — ../ASTRA/ASTRA-PROTOCOL.md (repo publico github.com/orlando-vazquez-career/astra-protocol).`; Version history: `| 0.5.0 | 2026-09-04 | Los sprints Web3 (contratos, tokens, deploys a mainnet) se ruteaban a AEGIS, cuyo gate de release supone reversibilidad. Nace ASTRA (v0.1.0), protocolo exclusivo de desarrollo Web3, independiente y multi-vendor. | **ASTRA reconocido como hermano.** Fila en la matriz de enrutamiento, enum \`source_protocol\` con \`astra\`, directorio hermano en el Daimonion, planeta en la cosmografia, comando \`/kairos astra\`. La relacion es unidireccional: KAIROS enruta a ASTRA; ASTRA no conoce a KAIROS. |`.
  - `.claude/commands/kairos.md`: `argument-hint` → `mnema|lumen|aegis|peitho|astra|seele`; `allowed-tools` agrega `Bash(astra:*)`; linea 7 lista `(MNEMA · LUMEN · AEGIS · PEITHO · ASTRA · SEELE)`; "Si el argumento es un protocolo (`mnema|lumen|aegis|peitho|astra|seele`)"; en el resumen post-boot `(¿MNEMA? ¿LUMEN? ¿AEGIS? ¿PEITHO? ¿ASTRA? ¿esperar? ¿ninguno?)`; seccion nueva antes de las reglas de la membrana: `## Protocolo ASTRA y herramienta \`astra\` (desarrollo Web3)` con: cuando rutear (señales), lectura de `C:/dev/protocols/ASTRA/ASTRA-PROTOCOL.md`, comandos `node C:/dev/tools/astra-cli/bin/astra.mjs doctor|check --gate mainnet|deployments list|chain probe <id>`, MCP `astra` si esta registrado, regla de wager (al firmar Gate 2 se registra un wager `source_protocol: "astra"` con `resolve_by` 30 dias despues), y la frase "KAIROS referencia a ASTRA; ASTRA no referencia a KAIROS: nunca escribir en su repo cosas de la membrana".
  - `.kimi-code/skills/kairos/SKILL.md`: `whenToUse` agrega `astra`.
  - `templates/wager.template.md`: `<mnema | lumen | aegis | peitho | astra | kairos>`; `templates/proposal.template.md`: `ruta_sugerida ∈ {mnema|lumen|aegis|peitho|astra|esperar|ninguno}`.
- [ ] **Step 4: Observatory**
  - `index.html`: variable `--star: #f0c674;   /* astra — los astros */` junto a las otras; `.tag.astra{color:var(--star);background:rgba(240,198,116,.12)}`; `<option value="astra">astra</option>`; `PROTO = ['mnema','lumen','aegis','kairos','peitho','astra']`; `PCOLOR` agrega `astra:'var(--star)'`; `PROTOCOL_DIRS` agrega `astra:'C:/dev/protocols/ASTRA'`.
  - `cosmografia.html`: `COL.astra:'#f0c674'`; `PROTOCOLS.astra:{name:'ASTRA',color:COL.astra,orbit:0.78,speed:0.038,size:14,glyph:'✦',phase0:4.9, desc:'Desarrollo Web3 · siete fases, dos gates: Carta y Mainnet · multi-cadena, multi-vendor'}`; `ORDER=['mnema','lumen','aegis','peitho','astra','seele']`; `cityPhases` agrega `astra:['Órbita','Carta','Ensayo','Auditoría','Lanzamiento']`; comentario "5 planetas" → "6 planetas (… PEITHO, ASTRA y SEELE)".
  - Abrir `observatory/index.html` y `cosmografia.html` con `node daemon/static-server.mjs` no es necesario: validar sintaxis con `node --check` no aplica a HTML; validar con `node -e` que el JSON de `PROTOCOL_DIRS` copiado es parseable no aplica. Verificacion manual: buscar que no quede ningun `['mnema','lumen','aegis','peitho'` sin astra (`grep -n "peitho'" observatory/*.html`).
- [ ] **Step 5: `CLAUDE.md`** linea de hermanos: `- \`../MNEMA\` \`../LUMEN\` \`../AEGIS\` \`../PEITHO\` \`../ASTRA\` — protocolos hermanos … ASTRA (v0.1.0, publico) es el hermano Web3: contratos, tokens, dApps, deploys a testnet/mainnet en Stellar, Syscoin, Base y cualquier EVM; su CLI vive en \`C:/dev/tools/astra-cli\` (\`astra\`, cero dependencias, tambien MCP). La referencia es unidireccional: KAIROS → ASTRA.`; linea de tools agrega `astra-cli`. `README.md`: "MNEMA decide, LUMEN diseña, AEGIS construye, PEITHO vende, ASTRA despliega en cadena, SEELE recuerda. Los seis…"; cosmografia: "MNEMA, LUMEN, AEGIS, PEITHO, ASTRA y SEELE son planetas". `guides/observatory.md` linea 22 igual.
- [ ] **Step 6: devlog** `docs/devlogs/2026-09-04-astra-hermano.md`: por que, que cambio (lista de archivos), la regla de direccion unica, como probar (`/kairos astra`, `node --test`), pendientes (el `/kairos` global en `~/.claude/commands/kairos.md` sigue desactualizado; decidir si se sincroniza).
- [ ] **Step 7: tests y commit** — `node --test "daemon/test/*.test.mjs"` → 182 pass. Commit **solo** los archivos tocados por esta tarea que estaban limpios (excluir `.claude/commands/kairos.md`, que ya tenia cambios sin commitear del operador, y `genesis/devlogs/kairos.jsonl`): `git add daemon/kairos-daimonion.mjs daemon/test/kairos-daimonion.test.mjs guides/routing-matrix.md KAIROS-PROTOCOL.md .kimi-code/skills/kairos/SKILL.md templates/wager.template.md templates/proposal.template.md observatory/index.html observatory/cosmografia.html CLAUDE.md README.md guides/observatory.md docs/devlogs/2026-09-04-astra-hermano.md && git commit -m "feat(routing): ASTRA reconocido como hermano (v0.5.0) — matriz, enum, daimonion, cosmografia; KAIROS → ASTRA, nunca al reves"`. No pushear KAIROS (no fue pedido). Dejar `.claude/commands/kairos.md` modificado en el working tree y reportarlo.

---

### Task 19: cierre

- [ ] **Step 1: memoria** — actualizar `C:/Users/Orlando/.claude/projects/C--dev-hackathons-stellar-elite-09-2026-clases/memory/` con un archivo `project-astra-protocol.md` (que es ASTRA, repos, decisiones, pendientes) y una linea en `MEMORY.md`.
- [ ] **Step 2: reporte al operador** (español): revision de AEGIS/KAIROS/LUMEN y tools (hallazgos concretos), mapa de las 34 skills → protocolo, justificacion de ASTRA, que se construyo, URLs de los dos repos, estado de CI, cambios en KAIROS (y lo que quedo sin commitear), pendientes y decisiones que le tocan.

---

## Self-review

- **Cobertura de la spec**: §1–§3 → Tasks 13, 17 (independencia y escaneo), §4.1–4.3 → Tasks 13, 15, 16; §4.4 → Tasks 2, 14; §4.5 → Tasks 10, 11, 13, 14 (`runtimes.md`); §5 → Tasks 1–11; §6 → tests en cada task + Task 12 (CI); §7 → mapa de archivos; §8 → Task 14 (`skills-externas.md`) y Task 19; §9 → Task 13 (no-objetivos). KAIROS → Task 18. Publicacion → Task 17.
- **Placeholders**: ninguno; las guias y skills tienen su contenido enumerado; los datos de cadenas y estandares estan tabulados con valores verificados.
- **Consistencia de nombres**: `validateAddress` (Tasks 3, 6, 7, 9), `resolveFamily`/`getChain` (Tasks 2–7), `syncSkills` (Tasks 10, 11, 16), `resolveProtocolDir` (Tasks 10, 11), `checkRepo` (Tasks 7, 9), `addDeployment`/`readDeployments` (Tasks 6, 9), `fechaLocalISO`/`writeAtomic`/`findExecutable`/`runVersion` (Task 1 y consumidores). Comandos en `lib/commands/*.mjs` con `run({args, flags, stdout, stderr})`.
