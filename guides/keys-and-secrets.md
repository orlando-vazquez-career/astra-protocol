# Claves y secretos — A2 y A7 en la práctica

**A2**: las claves no existen para el agente. **A7**: los agentes no mueven fondos de mainnet, ni con permiso.

## El principio: alias, no claves

Toda firma se hace a través de un **alias** de un keystore nativo. El agente conoce el nombre del alias; el keystore conoce la clave. Ningún comando que ejecute un agente recibe una clave por argumento, por variable de entorno visible en el transcript ni por archivo del repo.

| Cadena | Crear alias (lo hace la persona) | Firmar con el alias (puede hacerlo un agente en testnet) | Dónde queda la clave |
|---|---|---|---|
| Stellar | `stellar keys generate --global alice --network testnet --fund` (testnet) · para mainnet: `stellar keys generate --global prod-deployer` y fondear a mano | `stellar contract deploy --source-account alice --network testnet ...` | `~/.config/stellar/identity/<alias>.toml` (global) o `.stellar/identity/` (local al proyecto: **debe estar en .gitignore**; `astra init` lo agrega) |
| EVM (Foundry) | `cast wallet import deployer-testnet --interactive` (pide la clave una vez, la guarda cifrada) | `forge create --account deployer-testnet --rpc-url https://sepolia.base.org ...` · `forge script --account ...` | `~/.foundry/keystores/<alias>` (cifrado con contraseña) |
| EVM (Hardhat) | variables de configuración de Hardhat fuera del repo (`npx hardhat vars set DEPLOYER_KEY`), o un keystore/hardware wallet | scripts que leen la variable; nunca la imprimen | fuera del repo, en el almacén de Hardhat del usuario |
| Syscoin L1 | `syscoin-cli createwallet prod` + `syscoin-cli -rpcwallet=prod encryptwallet "<frase>"` (lo hace la persona) | `syscoin-cli -rpcwallet=prod sendtoaddress ...` (solo la persona en mainnet) | el `wallet.dat` del nodo |

Reglas:

1. **Alias de testnet y de mainnet distintos**, con nombres que no se confundan (`alice-testnet`, `prod-deployer`). Un agente solo recibe alias de testnet.
2. **Mainnet firma la persona.** El `oficial-de-lanzamiento` prepara el comando exacto y la persona lo ejecuta, o lo ejecuta el agente con el alias de mainnet **solo** cuando la persona lo pidió en ese momento y para ese comando, y aun así nunca para mover fondos (A7).
3. Nunca `--secret-key`, `--private-key`, `PRIVATE_KEY=` inline. Si una herramienta solo acepta la clave por argumento, no la usa un agente.

## Qué hacer si un agente vio una clave

1. Considerarla comprometida. Rotar: nuevo alias, nueva clave, mover fondos si es mainnet.
2. Registrar el incidente en el devlog (`docs/astra/devlogs/`): qué se vio, dónde, qué se rotó. Sin escribir la clave.
3. Borrar la clave del lugar donde apareció (archivo, transcript, log) y, si estaba en git, reescribir la historia solo si el repo es privado y todos los clones se coordinan; si era público, la rotación es lo único que sirve.

## Qué detecta `astra check`

| Regla | Qué busca | Severidad |
|---|---|---|
| `stellar-secret-seed` | `S` + 55 caracteres base32 con checksum StrKey válido (una semilla real, no un texto que se le parece) | error |
| `evm-private-key` | 64 hex en una línea que menciona private key, secret, mnemonic, seed, deployer o signer | error |
| `mnemonic` | 12 a 24 palabras consecutivas de 3 a 8 letras junto a la palabra mnemonic / seed phrase | error |
| `secret-assignment` | `X_SECRET=...`, `PRIVATE_KEY=...`, `MNEMONIC=...` con un valor real (no placeholder) fuera de `.env.example` | aviso |
| `env-tracked` | `.env`, `.env.local`, `.env.production` en el índice de git | error |
| `key-material-tracked` | `.stellar/identity/`, `.soroban/identity/`, `*.keystore`, `keystore/*.json`, `*.pem`, `id_rsa` en git | error |
| `gitignore-env` | `.gitignore` sin `.env` | aviso |

Los hallazgos dicen archivo y línea, **nunca el valor**.

Qué **no** detecta: claves en formatos que no reconoce (JSON de keystore cifrado no se flaggea si no está en git; un hex de 64 sin contexto se considera hash), secretos en binarios, secretos que ya salieron del repo hacia un chat. Por eso el escáner es la red, no la política.

## Variables de entorno

- `.env` para valores locales, nunca versionado; `.env.example` con placeholders (`<tu-alias>`, `your_key_here`) versionado.
- Las passphrases de red de Stellar **no son secretos** (`Public Global Stellar Network ; September 2015`); el escáner las excluye.
- Los RPC con API key (Alchemy, QuickNode, proveedores de RPC de Stellar) sí son secretos: van a `.env`.

## Direcciones que se pegan

Toda dirección que entra a un documento o a un comando pasa por `astra address <cadena> <dirección>`: un carácter cambiado en una dirección de mainnet manda fondos a la nada. El comando además rechaza claves secretas disfrazadas de dirección.
