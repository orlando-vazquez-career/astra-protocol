<!--
INSTRUCCIONES — orbit.template.md (Fase 1: Órbita)

1. Lo completa el cartógrafo (o el navegante) al inicio del sprint, antes de diseñar nada.
2. Cada dato de red se confirma con `astra chain info <id>` y `astra chain probe <id>`;
   el veredicto de capacidad sale de `astra doctor --chain <id>`.
3. El go/no-go lo firma la persona. Sin go no hay Carta.
4. Copiar a docs/astra/orbit.md (astra init lo hace). Borrar este bloque al completar.
-->

# Órbita — <nombre del proyecto>

**Fecha**: <YYYY-MM-DD>
**Idea en una frase**: <qué se construye y para quién>

## Cadena y red

| Campo | Valor |
|---|---|
| Cadena (id del registro) | <stellar-testnet / base-sepolia / syscoin-tanenbaum / ...> |
| Familia | <stellar / evm / utxo> |
| Red objetivo final | <mainnet: stellar-mainnet / base / ...> |
| Sonda en vivo (`astra chain probe`) | <OK chainId 84532 · bloque N · fecha> |
| Explorer | <url> |
| Faucet (testnet) | <url> |

## Capacidad (`astra doctor --chain <id>`)

| Familia | Veredicto | Falta |
|---|---|---|
| <stellar> | <SAFE / CAUTION / AVOID> | <nada / target wasm / forge> |

Con AVOID no se sigue: instalar primero y volver a correr.

## Estándares candidatos (A5)

| Estándar | Estado (verificado el) | Decisión |
|---|---|---|
| <SEP-0041 Soroban Token Interface> | <Draft, 2026-09-04> | <implementar completo> |

## Modelo de amenazas inicial

- **Activos en juego**: <fondos de usuarios / rol administrador / datos>
- **Atacantes plausibles**: <usuario malicioso / operador comprometido / front-runner / oráculo manipulado>
- **Superficies**: <funciones públicas / upgrade / claves / puente>
- **Supuestos que si fallan lo cambian todo**: <...>

## Costos preliminares (A10)

<deploy estimado, reservas o rent, fees por operación; se afinan en Ensayo>

## Decisión

**go / no-go**: <go>
**Justificación**: <una línea>
**Firmado por**: <nombre> · <YYYY-MM-DD>
