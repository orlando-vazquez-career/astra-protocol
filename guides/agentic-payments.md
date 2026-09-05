# Pagos agénticos — x402 y MPP

Cuando el que paga es una máquina (un agente de IA, un servicio) y el que cobra es una API, el problema no es "aceptar cripto" sino cobrar **por request**, sin cuentas ni tarjetas, con liquidación en cadena. Dos protocolos abiertos resuelven eso; ASTRA los trata como cualquier otra integración en cadena: Órbita → Carta → ... → Gate 2.

## x402

- Reutiliza el código HTTP **402 Payment Required**. El cliente pide un recurso; el servidor responde 402 con los requisitos de pago; el cliente firma un pago y reenvía la request con la cabecera de pago; un **facilitador** verifica y liquida en cadena; el servidor entrega el recurso.
- Origen: Coinbase, con el ecosistema EVM (Base) como cuna. Sitio: https://www.x402.org · código: https://github.com/coinbase/x402.
- En **Stellar** hay guía oficial (https://developers.stellar.org/docs/build/apps/x402): el cliente firma **entradas de autorización** (auth entries) del SAC de USDC en vez de la transacción completa, y el facilitador arma la transacción y paga las fees, así que un agente puede pagar **sin tener XLM**. Identificadores de red en CAIP-2: `stellar:testnet`, `stellar:pubnet`.
- Trade-off: dependes del facilitador (verificación y liquidación). Sin facilitador propio o de confianza, mirar MPP.

## MPP — Machine Payments Protocol

- Estándar abierto para pagos máquina a máquina: https://mpp.dev.
- Dos modos: **por cargo** (una transacción en cadena por request; simple, sin facilitador) y **por canal** (el agente abre un canal, hace commits off-chain de alta frecuencia y liquida al cerrar; para agentes que hacen muchas requests por sesión).
- Trade-off: el modo canal exige desplegar y auditar un contrato de canal (entra completo en ASTRA: es un contrato con dinero).

## Cuál elegir

| Situación | Elegir |
|---|---|
| Quiero cobrar por una API lo antes posible y que los clientes no necesiten gas | x402 con facilitador |
| No quiero depender de terceros y cada request puede pagar una transacción | MPP por cargo |
| El agente hace decenas o cientos de requests por sesión | MPP por canal |
| Ya existe un ecosistema x402 alrededor (Base, otras EVM) | x402 |

## Qué va en la Carta

- Quién paga las fees de red en cada request y qué pasa si el facilitador no responde.
- Moneda y decimales (USDC tiene 6 decimales en EVM y 7 en Stellar clásico/SAC).
- Límite por request y por sesión del agente comprador (un bug en el agente no puede vaciar su cuenta).
- Protección contra replay: cada pago referencia un recurso y un nonce; el servidor no entrega dos veces por el mismo pago.
- Registro: los contratos de canal se registran en `deployments.json` como cualquier otro; las direcciones de cobro se validan con `astra address`.

## Qué mira la Auditoría

- Verificación del pago en el servidor: nunca confiar en la cabecera sin confirmar con el facilitador o con la cadena.
- Canal: fondos bloqueados, cierre unilateral, ventana de disputa, qué pasa si una parte desaparece.
- Claves del agente comprador: alias con fondos acotados, nunca la clave en el prompt (A2).
