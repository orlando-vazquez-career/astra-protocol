---
name: astra-launch
description: Gate 2 y Fase 6 de ASTRA (Lanzamiento a mainnet). Verifica mecanicamente el gate con astra check --gate mainnet, prepara docs/astra/launch.md (costos, alias, plan de rollback, comandos exactos) para la firma humana, y despues del deploy registra y verifica publicamente la direccion. Usar cuando el usuario dice "mainnet", "lanzar", "producción", "gate 2", "verificar el contrato en el explorer" o "anunciar la dirección".
---
<!-- GENERADO por astra skills sync desde skills/astra-launch/SKILL.md — no editar a mano; editar el canonico y re-correr el sync -->

# astra-launch — Gate 2 · Fase 6: Lanzamiento

## Cuándo

Con `audit.md` en `veredicto: apto`. Mainnet es irreversible (A1): esta skill no ejecuta nada en mainnet por su cuenta.

## Pasos — Gate 2

1. `astra check --gate mainnet`. Los cinco ítems (`carta-aprobada`, `testnet-desplegada`, `auditoria-apta`, `launch-firmado`, `sin-secretos`) tienen que estar en OK; `launch-firmado` estará en FAIL hasta que la persona firme. Cualquier otro FAIL devuelve el sprint a su fase.
2. Completa `docs/astra/launch.md` desde `templates/launch.template.md`: commit a lanzar (igual a la entrada testnet o explica la diferencia), `costo_estimado` (deploy + inicialización + reservas/rent + humo; A10), `alias_mainnet` (nombre, distinto del de testnet; A2), `plan_rollback`, verificación pública prevista, prueba de humo prevista, y los comandos exactos.
3. Recorre con la persona el checklist de Gate 2 en `guides/gates.md`. Dile que complete `firmado_por: <nombre>` y `fecha_firma: <YYYY-MM-DD>`. Tú no las escribes.
4. Vuelve a correr `astra check --gate mainnet`: los cinco en OK.

## Pasos — Lanzamiento

5. La persona ejecuta el deploy con el alias de mainnet (A7). Si te pide ejecutarlo, solo ese comando exacto, con alias, nunca con clave, nunca transferencias de fondos.
6. `astra address <id-mainnet> <dirección>` y `astra deployments add --chain <id-mainnet> --address <dirección> --tx <hash> --commit <sha> --label <nombre>`.
7. Verificación pública (A6): `forge verify-contract ...` (Blockscout / Basescan) o hash de WASM reproducible + explorer (SEP-0055). Cuando el explorer muestra el código verificado: `astra deployments add ... --verified --verification-url <url>` (con `--force` si actualizas la misma dirección).
8. La persona hace la prueba de humo con el monto mínimo. Solo después se anuncia la dirección.
9. Completa la tabla "Resultado" de `launch.md` y sigue con `astra-logbook`.

## No hacer

- No proponer ni ejecutar un comando de mainnet con el gate cerrado. No completar `firmado_por`. No pedir ni leer claves (A2). No anunciar sin verificación pública.

## Done

Entrada `network: mainnet, verified: true` en el registro; `launch.md` con firma y resultado.
