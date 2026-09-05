<!--
INSTRUCCIONES — chart.template.md (Fase 2: Carta · Gate 1)

1. Lo escribe el cartógrafo con el navegante; lo aprueba la persona en el Gate 1.
2. La línea `aprobada:` de la cabecera queda VACÍA hasta que la persona la complete con
   la fecha (YYYY-MM-DD). `astra check --gate mainnet` busca exactamente esa línea.
3. Todo cambio durante la Construcción se anota en la sección "Cambios durante la
   construcción"; la Carta tiene que describir lo que se desplegó, no lo que se planeó.
4. Copiar a docs/astra/chart.md (astra init lo hace). Borrar este bloque al completar.
-->

# Carta — <nombre del artefacto>

**Fecha**: <YYYY-MM-DD>
**Cadena**: <id del registro> · **Red final**: <id mainnet>
aprobada:

## Qué se construye

<dos o tres párrafos: el artefacto, el problema que resuelve, quién lo usa>

## Interfaz pública

| Función | Quién puede llamarla | Entradas | Salida / evento | Error si no |
|---|---|---|---|---|
| <transfer> | <cualquier cuenta con balance> | <from, to, amount> | <evento transfer> | <InsufficientBalance> |

## Modelo de storage

| Clave | Tipo | Ámbito / TTL (Soroban) | Quién escribe |
|---|---|---|---|
| <Balance(addr)> | <i128> | <persistent, extender 30 días> | <transfer, mint> |

## Roles y permisos

| Rol | Puede | Cómo se rota | Quién lo tiene al lanzar |
|---|---|---|---|
| <admin> | <pausar, actualizar> | <set_admin con timelock> | <alias prod-admin> |

## Invariantes (falsables)

1. <el total emitido nunca supera el cap>
2. <la suma de balances es igual al total emitido>

## Economía (A10)

| Operación | Costo estimado | Quién paga |
|---|---|---|
| <deploy> | <X> | <operador> |
| <transfer> | <Y> | <usuario> |

## Estándares (A5)

- <SEP-0041 Soroban Token Interface (Draft, verificado 2026-09-04) — se implementa completo>

## Plan de tests

- Unitarios: <cada función pública, caminos de error>
- Propiedad / fuzz: <invariantes 1 y 2>
- Testnet: <secuencia de integración>

## Plan de upgrade / pausa / rollback

<quién, con qué demora, qué pasa con los fondos mientras tanto; o "inmutable" y por qué>

## Fuera de alcance

- <...>

## Decisiones del Gate 1

<fecha, qué se pidió cambiar, qué se aceptó>

## Cambios durante la construcción

<fecha · qué cambió respecto de lo aprobado · por qué>
