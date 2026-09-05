---
name: astra
description: Punto de entrada del protocolo ASTRA (desarrollo Web3 agentico, multi-cadena). Determina en que fase esta el proyecto (Orbita, Carta, Construccion, Ensayo, Auditoria, Lanzamiento, Bitacora) leyendo .astra/ y docs/astra/, y propone el siguiente paso con el comando exacto. Usar cuando el usuario dice "astra", "en que fase estamos", "que sigue en el protocolo", "quiero desplegar un contrato", "vamos a mainnet" o empieza cualquier trabajo con contratos, tokens, dApps o despliegues en Stellar, Syscoin, Base u otra EVM.
---
<!-- GENERADO por astra skills sync desde skills/astra/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra — ¿en qué fase estamos?

## Cuándo

Al empezar cualquier trabajo que toque una cadena, o cuando no está claro qué sigue. Si el repo no tiene `.astra/`, el siguiente paso es siempre `astra init`.

## Pasos

1. Lee, si existen: `.astra/astra.json`, `.astra/deployments.json`, `docs/astra/orbit.md`, `docs/astra/chart.md`, `docs/astra/audit.md`, `docs/astra/launch.md`.
2. Determina la fase con esta regla, en orden:
   - No hay `.astra/astra.json` → **Inicio**: `astra init --chain <id> --runtimes <runtime>`.
   - No hay `orbit.md` o no tiene go firmado → **Órbita** (skill `astra-orbit`).
   - `chart.md` no existe o no tiene la línea `aprobada: YYYY-MM-DD` → **Carta / Gate 1** (skill `astra-chart`).
   - El registro no tiene ninguna entrada `network: testnet` → **Construcción / Ensayo** (skills `astra-build`, `astra-testnet`).
   - `audit.md` no existe o no dice `veredicto: apto` → **Auditoría** (skill `astra-audit`).
   - `launch.md` no tiene `firmado_por` y `fecha_firma` → **Gate 2** (skill `astra-launch`).
   - El registro no tiene ninguna entrada `network: mainnet` → **Lanzamiento** (skill `astra-launch`).
   - Todo existe → **Bitácora** (skill `astra-logbook`).
3. Corre `astra check` (y `astra check --gate mainnet` si estás en Auditoría o después) y lee la salida.
4. Responde en este formato:

```
Fase: <nombre> · Cadena: <id> · Gate pendiente: <ninguno / G1 / G2>
Falta: <artefacto o condición concreta>
Siguiente paso: <comando exacto o skill>
Quién: <rol> (persona si es un gate)
```

## Reglas que aplican siempre

- Mainnet es irreversible (A1): nunca proponer un comando de mainnet si `astra check --gate mainnet` no está en OK.
- Las claves no existen para el agente (A2): si el usuario pega una clave, decirle que la rote y no repetirla.
- Testnet primero (A3). Los agentes no mueven fondos de mainnet (A7).

## Done

El usuario sabe la fase, lo que falta, el próximo comando y quién lo ejecuta.
