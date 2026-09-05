# Mapa de estándares — caso de uso → estándar (A5)

Antes de inventar, buscar. `astra standards search "<tema>"` busca en el catálogo local; esta guía dice qué buscar según lo que se construye. Título y estado de cada estándar se copiaron de la fuente oficial el 2026-09-04: confirmar el estado en la URL antes de implementar (los Draft cambian).

| Quiero construir... | Stellar | EVM (Base, Syscoin NEVM, Rollux) | Ejemplo de búsqueda |
|---|---|---|---|
| Un token fungible | Activo clásico + **SAC** (Stellar Asset Contract) si no hace falta lógica custom; **SEP-0041** si la hace | **ERC-20**, con **ERC-2612** (permit) si se quiere aprobar por firma | `astra standards search "token fungible"` |
| Un NFT | **SEP-0050** | **ERC-721** o **ERC-1155**; **ERC-2981** para regalías | `astra standards search "nft"` |
| Un contrato actualizable | **SEP-0049** + constructores **CAP-0058** para inicialización atómica; TTL **CAP-0053** | **ERC-1967** (slots de proxy) + **ERC-1167** (clones) | `astra standards search "upgrade proxy"` |
| Autenticación web con la wallet | **SEP-0010**; para cuentas contrato / passkeys **SEP-0045** + **CAP-0051** | **EIP-712** (typed data) y el patrón Sign-In with Ethereum | `astra standards search "autenticación web"` |
| Una smart wallet / cuenta inteligente | **SEP-0045**, **CAP-0051** (secp256r1 / passkeys) | **ERC-4337**, **EIP-7702**, **ERC-7579** | `astra standards search "smart account"` |
| Rampa fiat (anchor) | **SEP-0006** (API), **SEP-0024** (interactiva), **SEP-0031** (transfronteriza), **SEP-0012** (KYC), **SEP-0038** (cotizaciones) | fuera del alcance de los estándares EVM; proveedores | `astra standards search "anchor deposito"` |
| Deep links / pedir una firma desde un QR | **SEP-0007** | EIP-681 (no incluido en el catálogo; ver eips.ethereum.org) | `astra standards search "uri firma"` |
| Validar o normalizar direcciones | **SEP-0023** (StrKey) | **ERC-55** (checksum) | `astra address` ya lo hace |
| Identificar cadenas, cuentas y activos entre ecosistemas | **CAIP-2** (`stellar:pubnet`), **CAIP-10**, **CAIP-19** | **CAIP-2** (`eip155:8453`), **CAIP-10**, **CAIP-19** | `astra standards search "caip"` |
| Bóvedas / yield | **SEP-0056** | **ERC-4626** | `astra standards search "vault"` |
| Tokens regulados / RWA | **SEP-0057** (T-REX) | (patrones sobre ERC-20 con listas de permitidos) | `astra standards search "regulado"` |
| Pruebas ZK verificadas en cadena | **CAP-0059** (BLS12-381, disponible), **CAP-0074** (BN254) y **CAP-0075** (Poseidon) según versión de protocolo de la red | precompilados de Ethereum (bn254) heredados por las L2; verificar en la doc de la cadena | `astra standards search "zk"` |
| Cobrar por request a agentes (máquina a máquina) | **x402** (guía oficial en Stellar) o **MPP** | **x402** (ecosistema de origen, Base) | `astra standards search "x402"` · ver `agentic-payments.md` |
| Verificar públicamente el código desplegado | **SEP-0055** (build verification) + hash de WASM en el explorer | verificación de fuente en Blockscout / Basescan | `astra standards search "verificacion build"` |
| Eventos para indexar balances | **CAP-0067** (eventos unificados de activos) | eventos `Transfer` / `Approval` de ERC-20/721 | `astra standards search "eventos indexer"` |
| Estimar costos | fees base y de recursos de Soroban; reservas por entrada de ledger | **EIP-1559** (base fee + propina); **EIP-4844** explica por qué el gas de L2 es barato | `astra standards search "fee gas"` |

## Cómo se cita un estándar en la Carta

```
Estándares:
- SEP-0041 Soroban Token Interface (Draft, verificado 2026-09-04) — se implementa completo.
- SEP-0049 Upgradeable Contracts (Draft) — se sigue el proceso de upgrade con timelock de 24 h.
```

Un estándar citado sin fecha de verificación ni decisión (completo / parcial / solo inspiración) no cuenta como cumplido el axioma A5.

## Fuentes

- SEP: https://github.com/stellar/stellar-protocol/tree/master/ecosystem · CAP: https://github.com/stellar/stellar-protocol/tree/master/core
- ERC/EIP: https://eips.ethereum.org
- CAIP: https://github.com/ChainAgnostic/CAIPs
- x402: https://www.x402.org · MPP: https://mpp.dev
