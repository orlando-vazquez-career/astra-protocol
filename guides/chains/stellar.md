# Stellar — guía de cadena

**Familia**: `stellar` · **Verificado en vivo**: 2026-09-04 (Horizon mainnet y testnet, RPC testnet, protocolo 28).

## Redes

| id | Red | Passphrase | Horizon | RPC | Explorer |
|---|---|---|---|---|---|
| `stellar-testnet` | testnet | `Test SDF Network ; September 2015` | https://horizon-testnet.stellar.org | https://soroban-testnet.stellar.org | https://stellar.expert/explorer/testnet |
| `stellar-mainnet` | mainnet (pubnet) | `Public Global Stellar Network ; September 2015` | https://horizon.stellar.org | proveedor propio (https://developers.stellar.org/docs/data/apis/rpc/providers) | https://stellar.expert/explorer/public |

- Fondos de testnet: friendbot (https://friendbot.stellar.org) o `stellar keys fund <alias> --network testnet`. La testnet se **resetea periódicamente**: los contratos desaparecen y hay que redesplegar.
- Laboratorio para armar y firmar transacciones a mano: https://lab.stellar.org.
- `astra chain probe stellar-testnet` valida la passphrase contra el registro; un RPC apuntando a otra red falla ahí.

## Toolchain

| Herramienta | Para qué | Instalación |
|---|---|---|
| Rust + cargo | compilar contratos Soroban a WASM | https://rustup.rs |
| target wasm | `rustup target add wasm32v1-none` (Rust ≥ 1.85 y CLI ≥ 23) o `wasm32-unknown-unknown` (anteriores); `stellar contract build` elige el correcto | — |
| Stellar CLI (`stellar`) | build, upload, deploy, invoke, keys, bindings | `cargo install --locked stellar-cli` |
| `soroban-sdk` | SDK de contratos | dependencia del proyecto Rust |
| `@stellar/stellar-sdk` | JS/TS para dApps y scripts | dependencia del proyecto JS |

`astra doctor --chain stellar-testnet` da SAFE cuando `stellar` y `cargo` están (y avisa si falta el target wasm).

## Direcciones (SEP-0023)

| Prefijo | Qué es | Ejemplo válido |
|---|---|---|
| `G...` | cuenta (clave pública ed25519) | `GA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJVSGZ` |
| `C...` | contrato | `CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC` (SAC de XLM en testnet) |
| `M...` | cuenta multiplexada (cuenta + id) | `MA7QYNF7SOWQ3GLR2BGMZEHXAVIRZA4KVWLTJJFC7MGXUA74P7UJUAAAAAAAAAAAACJUQ` |
| `S...` | **semilla secreta** — nunca en un repo, un chat ni un log | — |

`astra address stellar-testnet <dirección>` valida checksum y tipo, y rechaza semillas.

## Ciclo de desarrollo

```bash
stellar contract build                                              # → target/wasm32*/release/<nombre>.wasm
cargo test                                                          # tests unitarios con el entorno de pruebas del SDK
stellar keys generate --global alice --network testnet --fund      # alias de testnet (lo hace la persona)
stellar contract deploy --wasm target/wasm32v1-none/release/hello.wasm --source-account alice --network testnet
#   → imprime el id del contrato C...  → astra deployments add --chain stellar-testnet --address C... --label hello --tx <hash>
stellar contract invoke --id C... --source-account alice --network testnet -- hello --to mundo
stellar contract bindings typescript --contract-id C... --network testnet --output-dir bindings/hello   # cliente TS
stellar contract id asset --asset native --network testnet         # dirección del SAC de un activo (aquí XLM)
```

Deploy en dos pasos cuando se quiere reutilizar código: `stellar contract upload --wasm ...` (devuelve el hash del WASM) y `stellar contract deploy --wasm-hash <hash> ...`. Los constructores (CAP-0058) reciben argumentos en el deploy con `-- --arg valor`.

## Verificación pública (A6)

- El explorer muestra el hash del WASM del contrato; el build tiene que ser reproducible desde el commit (SEP-0055 Contract Build Verification; `stellar contract info meta --id C...` muestra los metadatos SEP-0046 embebidos).
- Registrar con `astra deployments add ... --wasm-hash <hash> --verified --verification-url https://stellar.expert/explorer/public/contract/C...` solo cuando el hash coincide.

## Costos (A10)

- Fee base de transacción: 100 stroops (0.00001 XLM) por operación, más las **fees de recursos** de Soroban (instrucciones, lecturas/escrituras, bytes de eventos) que se conocen por simulación antes de enviar.
- Reserva base: 0.5 XLM por entrada de ledger (una cuenta necesita 1 XLM mínimo; cada trustline, oferta o firmante suma 0.5).
- Storage de contratos: se paga **rent** por el tiempo de vida (TTL); extender el TTL cuesta, y una entrada expirada se archiva hasta que alguien pague la restauración.
- Medir en testnet cada operación de la interfaz y anotar el costo en la Carta.

## Estándares que más se usan

SEP-0041 (interfaz de token; el SAC la implementa), SEP-0023 (direcciones), SEP-0010 / SEP-0045 (auth web, cuentas contrato), SEP-0049 + CAP-0058 (upgrade e inicialización), SEP-0055 (verificación de build), CAP-0053 (TTL), CAP-0059 / 0074 / 0075 (ZK). `astra standards search "<tema>" --family stellar`.

## Riesgos para la Auditoría

- TTL y archivado del storage (ver `audit-checklist.md`): la causa más común de "el contrato dejó de funcionar".
- `require_auth` ausente o sobre la dirección equivocada.
- Límites de recursos por transacción: una función que crece con el estado puede volverse imposible de invocar.
- Activos clásicos: flags de autorización y clawback deben ser deliberados; trustlines faltantes hacen fallar pagos.
- Reset de testnet: un `deployments.json` con entradas de testnet viejas no prueba nada; se re-despliega antes de auditar.

## Enlaces verificados

https://developers.stellar.org · https://developers.stellar.org/docs/build/smart-contracts · https://developers.stellar.org/docs/build/security-docs · https://github.com/stellar/stellar-protocol · https://stellar.expert · https://lab.stellar.org
