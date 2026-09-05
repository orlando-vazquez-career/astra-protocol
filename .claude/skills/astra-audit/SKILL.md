---
name: astra-audit
description: Fase 5 de ASTRA (Auditoria). Recorre el checklist de seguridad universal y el de la familia de cadena (Soroban, EVM, L2, Syscoin) sobre el codigo, la Carta y el registro de testnet; escribe docs/astra/audit.md con hallazgos, severidad, estado y veredicto. Usar cuando el usuario dice "auditar", "revision de seguridad del contrato", "que puede salir mal", "audit" o antes de cualquier mainnet.
---
<!-- GENERADO por astra skills sync desde skills/astra-audit/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-audit — Fase 5: Auditoría

## Cuándo

Con la entrada testnet registrada. Produce `docs/astra/audit.md` con `veredicto: apto` o `no-apto`. La corre un rol distinto del que construyó (A9): otra familia de modelos si se puede; si no, se declara.

## Pasos

1. Lee `docs/astra/chart.md`, el código, los tests, `.astra/deployments.json` (commit de la entrada testnet), `docs/astra/orbit.md` (amenazas) y `guides/audit-checklist.md` completa.
2. Recorre el checklist universal: control de acceso, reentrancy, aritmética, entradas, oráculos, front-running, upgradeabilidad, pausas, eventos, dependencias, secretos.
3. Recorre la sección de la familia: Stellar (TTL, `require_auth`, recursos, SEP-0041, activos clásicos, upgrade), EVM (`tx.origin`, `delegatecall`, aprobaciones, `receive`, gas, EIP-712), Layer 2 (secuenciador, retiros, paymasters) y Syscoin (bloques lentos, puentes, PoDA).
4. Cruza la interfaz desplegada en testnet con la Carta: toda función pública que no esté en la Carta es hallazgo alta. Cada invariante sin test de propiedad es hallazgo.
5. `astra check` sobre el commit auditado; un error de secretos es hallazgo crítica.
6. Escribe `docs/astra/audit.md` desde `templates/audit.template.md`: auditor y familia, commit, tabla `| ID | Severidad | Estado | Hallazgo | Fix |` con función y línea, riesgos aceptados (los acepta la persona), lo que no se pudo verificar.
7. Veredicto: `apto` solo sin críticas ni altas abiertas. Si hay, `no-apto`, y el sprint vuelve a `astra-build`; después de los fixes, re-prueba en testnet y vuelve a auditar.
8. Confirma con `astra check --gate mainnet` que `auditoria-apta` quede en OK.

## No hacer

- No editar código ni tests (solo lectura). No cerrar hallazgos propios. No aceptar riesgos por cuenta del agente.

## Done

`audit.md` completo, fechado, con veredicto; `auditoria-apta` OK si es apto.
