---
name: astra-testnet
description: Fase 4 de ASTRA (Ensayo). Despliega a TESTNET con un alias de claves, valida y registra la direccion en .astra/deployments.json, corre tests de integracion contra la red real y mide costos. Usar cuando el usuario dice "desplegar a testnet", "probar en la red", "ensayo", "friendbot", "faucet" o "registrar la direccion".
---

# astra-testnet — Fase 4: Ensayo

## Cuándo

Con Construcción verde. Produce la entrada `network: testnet` en `.astra/deployments.json` y los costos medidos.

## Pasos

1. `astra chain probe <id-testnet>`: chainId o passphrase confirmados hoy. Si falla, no despliegues.
2. Alias de testnet (lo crea la persona; tú solo usas el nombre): Stellar `stellar keys generate --global <alias> --network testnet --fund`; EVM `cast wallet import <alias> --interactive`. Fondos: friendbot (`stellar keys fund`) o faucet de `astra chain info <id>`.
3. Deploy con el CLI nativo, firmando con el alias:
   - Soroban: `stellar contract deploy --wasm <ruta.wasm> --source-account <alias> --network testnet [-- --arg valor]`
   - EVM: `forge create <ruta:Contrato> --rpc-url <rpc> --account <alias> --broadcast`
4. Valida la dirección: `astra address <id-testnet> <dirección>`. Regístrala: `astra deployments add --chain <id-testnet> --address <dirección> --tx <hash> --label <nombre> [--wasm-hash <hash>]`.
5. Integración contra testnet: cada función de la interfaz, cada rol, cada invariante, caminos de error. Conformidad con el estándar (interfaz completa de SEP-0041 / ERC-20, etc.).
6. Mide el costo real por operación (fees, recursos, gas) y anótalo en la Carta, sección Economía (A10).
7. `astra check` en OK; `astra check --gate mainnet` tiene que mostrar `testnet-desplegada` en OK.

## No hacer

- Nada de mainnet (A1, A3, A7).
- No pegar una clave en ningún comando ni archivo (A2). Si la testnet de Stellar se reseteó, se redespliega; las entradas viejas no prueban nada.

## Done

Entrada testnet registrada y validada, integración documentada, costos medidos, `testnet-desplegada` OK.
