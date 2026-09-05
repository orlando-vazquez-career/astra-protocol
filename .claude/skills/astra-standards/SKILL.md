---
name: astra-standards
description: Busca el estandar correcto antes de inventar (axioma A5 de ASTRA): SEP y CAP de Stellar, ERC y EIP de EVM, CAIP multi-cadena, x402 y MPP para pagos agenticos, docs oficiales de Syscoin, Rollux y Base. Usar cuando el usuario pregunta "que SEP/ERC uso para...", "hay un estandar para...", "como se hace un token/NFT/upgrade/auth en Stellar o EVM", o cita un numero de SEP, CAP, ERC o EIP.
---
<!-- GENERADO por astra skills sync desde skills/astra-standards/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-standards — estándar antes que invento (A5)

## Cuándo

En Órbita y Carta, y cada vez que aparezca una decisión de diseño que otros ya resolvieron: tokens, NFT, upgrade, autenticación, cuentas inteligentes, rampas fiat, direcciones, identificadores entre cadenas, ZK, pagos entre máquinas, verificación de código.

## Pasos

1. `astra standards search "<tema en español o inglés>" [--family stellar|evm|syscoin|base|payments|caip]`. El catálogo normaliza acentos y busca en id, título, tags y resumen.
2. Cruza con `guides/standards-map.md` (tabla caso de uso → estándar por familia).
3. Para cada candidato, abre la URL oficial y confirma el **estado actual** (Draft / Active / Final) y la versión de protocolo de la red que lo soporta. El catálogo es un mapa de ruteo con fecha; el estado puede haber cambiado.
4. Decide y escribe en la Carta: `<ID> <Título> (<estado>, verificado <fecha>) — se implementa completo | parcial (qué partes) | solo inspiración`.
5. Si no hay estándar, dilo explícitamente en la Carta ("sin estándar aplicable; diseño propio") para que la Auditoría lo mire con más cuidado.

## Atajos

- Token fungible: Stellar → activo clásico + SAC o SEP-0041; EVM → ERC-20 (+ ERC-2612).
- NFT: SEP-0050; ERC-721 / ERC-1155 (+ ERC-2981).
- Upgrade: SEP-0049 + CAP-0058; ERC-1967 + ERC-1167.
- Auth web / smart wallet: SEP-0010, SEP-0045 + CAP-0051; EIP-712, ERC-4337, EIP-7702, ERC-7579.
- Direcciones: SEP-0023; ERC-55 (`astra address` lo aplica).
- Multi-cadena: CAIP-2 / CAIP-10 / CAIP-19.
- Pagos entre máquinas: x402, MPP (`guides/agentic-payments.md`).
- Verificación de código: SEP-0055; verificación de fuente en Blockscout / Basescan.

## Done

La Carta cita cada estándar con id, estado, fecha y decisión, o declara que no hay.
