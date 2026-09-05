<!--
INSTRUCCIONES — audit.template.md (Fase 5: Auditoría)

1. Lo completa el auditor-de-cadena: modelo o familia DISTINTA del forjador si es posible (A9).
   Si es la misma, se declara en "Auditor".
2. `veredicto:` toma `apto` o `no-apto`. `astra check --gate mainnet` exige `apto` y ninguna
   fila con severidad crítica/alta en estado abierta.
3. Severidad: crítica · alta · media · baja. Estado: abierta · cerrada · aceptada.
4. Copiar a docs/astra/audit.md (astra init lo hace). Borrar este bloque al completar.
-->

# Auditoría — <nombre del artefacto>

**Fecha**: <YYYY-MM-DD>
**Commit auditado**: <sha>
**Auditor**: <rol / modelo / familia; "misma familia que el forjador" si aplica>
**Checklist recorrido**: guides/audit-checklist.md (universal + <familia> + <L2 si aplica>)
veredicto: <apto | no-apto>

## Hallazgos

| ID | Severidad | Estado | Hallazgo | Fix |
|---|---|---|---|---|
| A-1 | <alta> | <cerrada> | <descripción concreta, con función y línea> | <qué se cambió y qué test lo cubre> |

## Cobertura

- Tests unitarios: <n> · propiedad/fuzz: <n runs sobre invariantes 1, 2> · integración en testnet: <fecha, dirección>
- `astra check`: <OK, fecha>

## Riesgos aceptados

<cada "aceptada" de la tabla con su justificación y quién la aceptó>

## Lo que el auditor no pudo verificar

<dependencias externas, oráculos, componentes off-chain>
