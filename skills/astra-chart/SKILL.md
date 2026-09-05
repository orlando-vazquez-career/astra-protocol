---
name: astra-chart
description: Fase 2 de ASTRA (Carta) y Gate 1. Escribe el diseño completo del contrato o dApp en docs/astra/chart.md (interfaz, storage, roles, invariantes falsables, economia, estandares, plan de tests, plan de upgrade) y prepara la aprobacion humana. Usar cuando el usuario dice "carta", "diseño del contrato", "gate 1", "aprobar el diseño" o pide especificar un token, contrato o dApp antes de codear.
---

# astra-chart — Fase 2: Carta · Gate 1

## Cuándo

Con `orbit.md` firmado. Produce `docs/astra/chart.md`; la persona lo aprueba escribiendo `aprobada: YYYY-MM-DD`.

## Pasos

1. Lee `docs/astra/orbit.md` y `guides/chains/<familia>.md`. Abre `docs/astra/chart.md` (plantilla `templates/chart.template.md`).
2. Interfaz pública: tabla función / quién puede llamarla / entradas / salida y evento / error. La mínima que resuelve el problema.
3. Storage: clave, tipo, quién escribe; en Soroban además ámbito (persistent / temporary / instance) y estrategia de TTL (`extend_ttl`, quién paga).
4. Roles y permisos: administrador, pausa, upgrade; cómo se rota cada uno; qué alias lo tendrá al lanzar (nombre, nunca clave).
5. Invariantes como frases falsables: cada una tiene que poder ser un test de propiedad.
6. Economía (A10): costo estimado por operación y quién paga; reservas o rent.
7. Estándares (A5): cada uno con enlace, estado, fecha de verificación y decisión (completo / parcial). `astra standards search`.
8. Plan de tests (unitarios, propiedad/fuzz, testnet) y plan de upgrade / pausa / rollback (o inmutable y por qué). Fuera de alcance explícito.
9. Recorre con la persona el checklist de Gate 1 en `guides/gates.md` y pídele que escriba la línea `aprobada: <fecha>` en la cabecera. Tú no la escribes.

## No hacer

- No empezar a codear "mientras se aprueba".
- No dejar marcadores `<...>` ni "TBD".
- No escribir claves ni direcciones de mainnet de terceros.

## Done

`chart.md` completo y `astra check --gate mainnet` muestra `carta-aprobada` en OK.
