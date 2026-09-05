---
name: astra-build
description: Fase 3 de ASTRA (Construccion). Implementa lo que dice la Carta aprobada con el toolchain nativo de la cadena (Stellar CLI + cargo para Soroban; Foundry o Hardhat para EVM), con tests unitarios y de propiedad verdes y sin claves en el codigo. Usar cuando el usuario dice "construir", "implementar el contrato", "escribir el token", "codear la dApp" con una Carta ya aprobada.
---
<!-- GENERADO por astra skills sync desde skills/astra-build/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-build — Fase 3: Construcción

## Cuándo

Solo con `docs/astra/chart.md` que tenga `aprobada: YYYY-MM-DD`. Si no la tiene, la respuesta es "falta el Gate 1" (skill `astra-chart`).

## Pasos

1. Lee la Carta completa y `guides/chains/<familia>.md`. Confirma `astra doctor --chain <id>` en SAFE.
2. Esqueleto con el toolchain nativo:
   - Soroban: `stellar contract init` o `cargo new --lib`; `soroban-sdk` fijado; `stellar contract build`.
   - EVM: `forge init` (o Hardhat); versión de Solidity fijada; `forge build`.
3. Implementa función por función siguiendo la tabla de interfaz de la Carta: control de acceso (`require_auth` / modifier) en cada función que cambia estado; errores distinguibles; eventos por cada cambio relevante.
4. Tests unitarios por función y por camino de error (`cargo test`, `forge test`). Tests de propiedad o fuzz sobre cada invariante de la Carta (`forge test --fuzz-runs 1000`, `cargo fuzz` o proptest).
5. Cuentas y claves de prueba generadas en runtime por el framework de tests; nada de claves en fixtures. `astra check` sin errores.
6. Si te desvías de la Carta, anótalo en su sección "Cambios durante la construcción" en el mismo commit. Commits con prefijo `astra:`.
7. Cuando todo está verde, sigue con `astra-testnet`.

## No hacer

- No desplegar en mainnet ni preparar nada de mainnet en esta fase (A1, A3).
- No leer, pedir ni imprimir claves (A2). Si una herramienta exige `--private-key`, no se usa.
- No inventar estándares parciales: si la Carta dice "SEP-0041 completo", es completo.

## Done

Build y tests locales verdes (unitarios + propiedad), `astra check` OK, Carta al día con los cambios.
