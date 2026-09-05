# EVM genérico — cualquier otra cadena compatible con Ethereum

La familia `evm` de ASTRA cubre toda cadena que hable JSON-RPC `eth_*`: Ethereum y sus testnets, otras L2 de OP Stack o Arbitrum, sidechains, cadenas de aplicación. Lo que cambia entre ellas es el **chainId**, los RPC, los explorers, la moneda de gas y los riesgos de su arquitectura (L1, L2 optimista, ZK rollup, validium).

## Sondear una cadena que no está en el registro

```bash
astra chain probe base --rpc https://rpc.de.otra.cadena     # usa el perfil de `base` pero el RPC que se pasa
```

El comando reporta el chainId que responde el RPC y **falla** si no coincide con el del perfil elegido; sirve para confirmar el chainId real antes de agregar la cadena. Con Foundry: `cast chain-id --rpc-url <url>`.

## Agregar la cadena al registro

1. Verificar en vivo: chainId (`eth_chainId`), un RPC público que responda, explorer, faucet si es testnet, documentación oficial. El registro público de chainIds (https://chainid.network) ayuda a cruzar datos, pero la sonda en vivo es la que vale.
2. Agregar la entrada a `data/chains.json` de `astra-cli`:

```json
{
  "id": "mi-cadena-testnet",
  "family": "evm",
  "name": "Mi cadena (testnet)",
  "network": "testnet",
  "caip2": "eip155:123456",
  "chainId": 123456,
  "nativeSymbol": "ETH",
  "rpc": ["https://rpc.testnet.mi-cadena.example"],
  "explorers": ["https://explorer.testnet.mi-cadena.example"],
  "faucets": ["https://faucet.mi-cadena.example"],
  "docs": "https://docs.mi-cadena.example",
  "notes": "Qué es, quién la opera, riesgo notable.",
  "verifiedAt": "2026-09-04"
}
```

3. Correr los tests del CLI (`node --test "test/**/*.test.mjs"`): el test de integridad exige `caip2 = eip155:<chainId>`, URLs https y `verifiedAt`.
4. Escribir la guía en `guides/chains/<cadena>.md` con la misma estructura que `base.md`: redes, toolchain, direcciones, ciclo, verificación, costos, estándares, riesgos, enlaces verificados.

## Lo que aplica a toda EVM

- **EIP-155**: el chainId va dentro de la firma; una transacción firmada para una red no vale en otra. Igual, desplegar en la red equivocada cuesta gas real: `astra chain probe` antes de cada deploy.
- **EIP-55**: toda dirección que se pega pasa por `astra address evm 0x...`.
- **EIP-1559**: estimar costos con base fee + propina; en L2 sumar el costo de datos.
- **Alias**: `cast wallet import <alias> --interactive` y `--account <alias>`; nunca `--private-key`.
- **Verificación**: `forge verify-contract` con el verifier del explorer (Etherscan-compatible o Blockscout).
- **Auditoría**: checklist universal y sección EVM de `audit-checklist.md`; si es L2, también la sección Layer 2.

## Otras familias

Una cadena que no sea EVM, Stellar ni UTXO (Solana, Cosmos, Bitcoin puro) necesita además un adaptador en `astra-cli`: validación de direcciones en `lib/address.mjs`, sonda en `lib/probe.mjs` y toolchain en `lib/doctor.mjs`, con vectores públicos en los tests. Entra como sprint propio cuando haga falta.
