---
name: astra-orbit
description: Fase 1 de ASTRA (Orbita). Elige y sondea la cadena, mide la capacidad del entorno con astra doctor, lista los estandares aplicables y escribe el modelo de amenazas inicial en docs/astra/orbit.md. Usar cuando el usuario dice "orbita", "que cadena usamos", "podemos construir en Stellar/Base/Syscoin desde aca", "perfil de cadena" o empieza un proyecto Web3 nuevo.
---
<!-- GENERADO por astra skills sync desde skills/astra-orbit/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-orbit — Fase 1: Órbita

## Cuándo

Antes de diseñar. Produce `docs/astra/orbit.md` y el go/no-go que firma la persona.

## Pasos

1. Cadena y red: `astra chain list`; `astra chain info <id>`; `astra chain probe <id>`. Anota chainId o passphrase confirmados, bloque/ledger y fecha. Si la sonda falla, no sigas con esa cadena: busca otro RPC (`--rpc <url>`) o corrige el registro.
2. Capacidad: `astra doctor --chain <id>`. Copia el veredicto (SAFE / CAUTION / AVOID) y lo que falta. Con AVOID, la respuesta es "instalar X" y nada más.
3. Estándares: `astra standards search "<tema>"` y `guides/standards-map.md`. Para cada candidato: id, título, estado y fecha; confirma el estado en la URL oficial (A5).
4. Lee `guides/chains/<familia>.md` entera; anota faucets, explorers y costos preliminares.
5. Modelo de amenazas inicial: activos en juego, atacantes plausibles, superficies, supuestos que lo cambian todo.
6. Escribe `docs/astra/orbit.md` desde `templates/orbit.template.md` (si no existe, `astra init` lo crea). Sin marcadores `<...>`.
7. Pide a la persona el go/no-go con nombre y fecha.

## No hacer

- No inventar chainIds, RPC ni estándares: todo verificado en vivo o marcado "pendiente de verificar".
- No pedir claves ni fondos de mainnet: en Órbita solo se necesita, como mucho, un alias de testnet.

## Done

`orbit.md` completo, veredicto de capacidad distinto de AVOID, estándares fechados, modelo de amenazas escrito, go firmado por la persona.
