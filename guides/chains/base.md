# Base — guía de cadena

**Familia**: `evm` · **Verificado en vivo**: 2026-09-04 (RPC de mainnet y Sepolia respondieron con su chainId; explorers Blockscout en línea).

| id | Red | chainId / CAIP-2 | RPC | Explorers | Faucet |
|---|---|---|---|---|---|
| `base` | mainnet | 8453 · `eip155:8453` | https://mainnet.base.org | https://basescan.org · https://base.blockscout.com | — |
| `base-sepolia` | testnet | 84532 · `eip155:84532` | https://sepolia.base.org · https://base-sepolia-rpc.publicnode.com | https://sepolia.basescan.org · https://base-sepolia.blockscout.com | https://docs.base.org/base-chain/tools/network-faucets |

Documentación: https://docs.base.org.

## Qué es

L2 optimista de Coinbase construido sobre **OP Stack** (la misma base que Optimism), anclado a Ethereum (Sepolia para la testnet). Bloques de ~2 s, gas en ETH, EVM completo. El **secuenciador** lo opera Coinbase; los **retiros a L1** pasan por la ventana de disputa (7 días). Los RPC públicos tienen límite de tasa: en producción se usa un proveedor propio (la API key va a `.env`, nunca al repo).

## Toolchain

Foundry (`forge`, `cast`, `anvil`) o Hardhat; `ethers` / `viem`; OnchainKit y Smart Wallet de Coinbase para dApps con cuentas inteligentes. `astra doctor --chain base-sepolia` da SAFE con `forge` o Hardhat local.

## Direcciones

EVM con checksum EIP-55: `astra address base 0x5aAeb6053F3E94C9b9A09f33669435E7Ef1BeAed` → `OK ... checksum ok`. Un hex de 64 caracteres se rechaza como clave privada.

## Ciclo de desarrollo

```bash
forge build && forge test --fuzz-runs 1000
cast wallet import deployer-sepolia --interactive                    # alias de testnet (lo hace la persona)
astra chain probe base-sepolia                                        # chainId 84532 confirmado
forge create src/Vault.sol:Vault --rpc-url https://sepolia.base.org --account deployer-sepolia --broadcast
#   → astra deployments add --chain base-sepolia --address 0x... --tx 0x... --label vault
forge verify-contract --chain-id 84532 --verifier blockscout --verifier-url https://base-sepolia.blockscout.com/api 0x... src/Vault.sol:Vault
```

Para mainnet: alias distinto (`prod-deployer`), `astra check --gate mainnet` en verde, `--rpc-url https://mainnet.base.org --chain-id 8453`, y la persona ejecuta el comando (A7). Verificación en Basescan (`--verifier etherscan` con API key en `.env`) o Blockscout; registrar con `--verified --verification-url`.

## Costos (A10)

EIP-1559 en ETH: base fee de L2 (baja) más el costo de publicar los datos en L1 (blobs EIP-4844). `cast estimate` da el gas; el costo de datos varía con la congestión de Ethereum: estimar con margen y anotar en `launch.md`.

## Estándares que más se usan

ERC-20 / ERC-2612, ERC-721 / ERC-1155 / ERC-2981, ERC-1967 / ERC-1167 (proxies y clones), ERC-4337 y EIP-7702 (cuentas inteligentes, paymasters, gas patrocinado), EIP-712 (firmas tipadas), EIP-155 (chainId en la firma). x402 nació en este ecosistema. `astra standards search "<tema>" --family evm`.

## Riesgos para la Auditoría

- Secuenciador centralizado: qué hace la dApp si deja de incluir transacciones (retiro forzado vía L1).
- Ventana de 7 días para retiros a L1: fondos "en tránsito" que el usuario tiene que entender.
- Paymasters: quién paga el gas, límites, y qué pasa si el patrocinio se corta.
- Diferencias de precompilados u opcodes respecto de Ethereum L1: consultar la documentación de OP Stack antes de usar algo exótico.
- RPC público con límite de tasa: tests de integración que fallan por 429 no son bugs del contrato.

## Enlaces verificados

https://docs.base.org · https://www.base.org · https://basescan.org · https://base.blockscout.com · https://base-sepolia.blockscout.com · https://docs.base.org/base-chain/tools/network-faucets
