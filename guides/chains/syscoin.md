# Syscoin — guía de cadena (L1 UTXO, NEVM y Rollux)

Syscoin son tres capas que ASTRA trata como cadenas distintas del registro. **Verificado en vivo**: 2026-09-04 (RPC de NEVM, Tanenbaum y Rollux respondieron con su chainId; el RPC de Rollux testnet no respondió).

| Capa | id del registro | Familia | chainId / CAIP-2 | RPC | Explorer | Faucet |
|---|---|---|---|---|---|---|
| L1 UTXO (mainnet) | `syscoin-utxo` | `utxo` | — | nodo local (`syscoin-cli`) | https://explorer.syscoin.org | — |
| NEVM mainnet | `syscoin-nevm` | `evm` | 57 · `eip155:57` | https://rpc.syscoin.org · https://syscoin.public-rpc.com | https://explorer.syscoin.org | — |
| NEVM testnet (Tanenbaum) | `syscoin-tanenbaum` | `evm` | 5700 · `eip155:5700` | https://rpc.tanenbaum.io · https://syscoin-tanenbaum-evm.publicnode.com | https://explorer.tanenbaum.io | https://faucet.tanenbaum.io |
| Rollux mainnet | `rollux` | `evm` | 570 · `eip155:570` | https://rpc.rollux.com · https://rpc.ankr.com/rollux | https://explorer.rollux.com | — |
| Rollux testnet | `rollux-tanenbaum` | `evm` | 57000 · `eip155:57000` | https://rpc-tanenbaum.rollux.com (sin respuesta el 2026-09-04) | — | ver https://docs.rollux.com |

Documentación: https://docs.syscoin.org · https://docs.rollux.com.

## Qué es cada capa

- **L1 UTXO**: la cadena base, estilo Bitcoin, **merge-mined con Bitcoin** (AuxPoW): hereda hashrate de los mineros de Bitcoin. Gas nativo SYS. Sin RPC público de uso general: se opera con un nodo propio.
- **NEVM**: la capa EVM de Syscoin, anclada al L1. Compatible con Foundry, Hardhat y cualquier tooling de Ethereum. Bloques lentos (**~2.5 minutos**, medido en el explorer): pensada como capa de asentamiento y de disponibilidad de datos, no para interacción de alta frecuencia. Gas en SYS.
- **PoDA (Proof of Data Availability)**: la capa de datos de Syscoin, alternativa a los blobs de Ethereum; los rollups publican ahí sus datos.
- **Rollux**: L2 optimista construido sobre **OP Stack** que asienta en NEVM y usa PoDA. Bloques de ~2 s. Es donde vive la actividad de usuario.

## Toolchain

| Capa | Herramientas | `astra doctor` |
|---|---|---|
| L1 UTXO | `syscoin-cli` / `syscoind` (nodo completo), wallet del nodo | SAFE con `syscoin-cli`; AVOID sin él |
| NEVM / Rollux | Foundry (`forge`, `cast`, `anvil`) o Hardhat; `ethers`/`viem` | SAFE con `forge` o Hardhat local |

## Direcciones

- **NEVM / Rollux**: EVM, EIP-55. `astra address syscoin-nevm 0x...`.
- **L1 UTXO**: bech32 `sys1...` (mainnet) / `tsys1...` (testnet) para segwit, o base58 `S...` (P2PKH, versión 63) y `3...` (P2SH, versión 5). `astra address syscoin-utxo sys1...` valida checksum, prefijo y versión; un WIF (clave privada base58) se rechaza como secreto.
- Cada capa tiene direcciones propias: un `0x...` de NEVM no recibe fondos del L1 sin pasar por el puente.

## Ciclo de desarrollo (NEVM / Rollux)

```bash
forge init mi-contrato && cd mi-contrato
forge build && forge test
cast wallet import deployer-tanenbaum --interactive                 # alias (lo hace la persona)
forge create src/Token.sol:Token --rpc-url https://rpc.tanenbaum.io --account deployer-tanenbaum --broadcast
#   → Deployed to: 0x...  → astra deployments add --chain syscoin-tanenbaum --address 0x... --tx 0x... --label token
cast call 0x... "totalSupply()(uint256)" --rpc-url https://rpc.tanenbaum.io
```

Para Rollux, el mismo flujo con `--rpc-url https://rpc.rollux.com` (mainnet) y `--chain-id 570`; para NEVM mainnet `--chain-id 57`. `astra chain probe <id>` antes de cada deploy: confirma que el RPC apunta al chainId correcto (EIP-155 hace que una transacción firmada para 57 no valga en 570, pero un deploy en la red equivocada igual cuesta gas real).

## Verificación pública (A6)

Los explorers de Syscoin y Rollux son **Blockscout**:

```bash
forge verify-contract --chain-id 5700 --verifier blockscout --verifier-url https://explorer.tanenbaum.io/api 0x... src/Token.sol:Token
forge verify-contract --chain-id 570  --verifier blockscout --verifier-url https://explorer.rollux.com/api    0x... src/Token.sol:Token
```

Registrar con `--verified --verification-url https://explorer.rollux.com/address/0x...` solo cuando el explorer muestra el código verificado.

## Costos (A10)

- NEVM y Rollux: EIP-1559 (base fee + propina) en SYS. Estimar con `cast estimate` y anotar en la Carta; en Rollux sumar el costo de publicar datos (PoDA).
- L1: fee por byte; estimar con `syscoin-cli estimatesmartfee`.

## Riesgos para la Auditoría

- Bloques de ~2.5 min en NEVM: timeouts de UX, confirmaciones y cualquier lógica que dependa de `block.timestamp` con precisión.
- Puente L1 ↔ NEVM (SYS ↔ SYSX) y Rollux ↔ NEVM: fondos en tránsito, finalidad, custodia.
- OP Stack en Rollux: secuenciador, ventana de disputa para retiros, diferencias de opcodes respecto de L1.
- PoDA: qué datos se publican y cuánto tiempo son recuperables para reconstruir el estado.
- Rollux testnet sin RPC verificado: no auditar contra una testnet que no se pudo sondear; confirmar el endpoint vigente en la documentación primero.

## Enlaces verificados

https://docs.syscoin.org · https://docs.rollux.com · https://syscoin.org · https://explorer.syscoin.org · https://explorer.tanenbaum.io · https://explorer.rollux.com · https://faucet.tanenbaum.io
