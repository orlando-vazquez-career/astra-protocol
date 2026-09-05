# Checklist de auditoría — fase 5

Lo recorre el `auditor-de-cadena` (solo lectura, modelo o familia distinta del `forjador` si es posible, A9) con la Carta, el código, los tests y el registro de testnet a la vista. Cada punto que falla es un hallazgo en la tabla de `docs/astra/audit.md`.

## Formato de `audit.md`

Cabecera con `veredicto: apto` o `veredicto: no-apto`, el modelo o familia que auditó, la fecha y el commit revisado. Tabla:

```
| ID | Severidad | Estado | Hallazgo | Fix |
|---|---|---|---|---|
| A-1 | alta | cerrada | transfer() no valida amount > 0 | require amount > 0; test unitario |
```

Severidad: `crítica` (pérdida de fondos o control total), `alta` (pérdida parcial, bloqueo de fondos, escalada de privilegios), `media` (comportamiento incorrecto sin pérdida directa), `baja` (calidad, gas, claridad). Estado: `abierta`, `cerrada`, `aceptada` (con justificación en la columna Fix).

`astra check --gate mainnet` lee esta tabla: cualquier fila `crítica` o `alta` en estado `abierta` cierra el gate.

## Checklist universal

### Control de acceso
- [ ] Cada función que cambia estado o mueve valor verifica quién la llama (Soroban: `require_auth` en la dirección correcta; EVM: modifier o check explícito).
- [ ] El rol administrador se puede rotar; no hay una sola clave sin plan de sucesión.
- [ ] Las funciones de inicialización no se pueden volver a llamar (Soroban: constructor CAP-0058 o guard de `initialized`; EVM: `initializer` o constructor).
- [ ] Ninguna función privilegiada queda pública por olvido (recorrer la interfaz completa contra la Carta).

### Reentrancy y orden de efectos
- [ ] Checks, efectos, interacciones: el estado se actualiza antes de llamar a otro contrato o transferir.
- [ ] Llamadas a contratos externos (tokens, oráculos, callbacks) tratadas como hostiles: ¿qué pasa si revierten, si consumen todo el gas o si reentran?

### Aritmética y límites
- [ ] Overflow/underflow imposibles (Rust `checked_*` o panics explícitos; Solidity ≥ 0.8 con `unchecked` solo donde se justifica).
- [ ] Redondeo a favor del protocolo, no del usuario; no se puede extraer valor por redondeo repetido.
- [ ] Cantidades cero, máximas y negativas (donde el tipo lo permita) probadas.
- [ ] Decimales consistentes entre tokens (7 en Stellar clásico y SAC; 18 típico en EVM, pero USDC usa 6).

### Validación de entradas
- [ ] Direcciones no nulas, no el propio contrato, no una dirección de otra cadena.
- [ ] Longitudes y tamaños acotados (strings, vectores, mapas) para no agotar recursos.
- [ ] Parámetros de tiempo (deadlines, vencimientos) validados contra el ledger o `block.timestamp`, sabiendo que el operador de la red influye en ellos.

### Oráculos y precios
- [ ] Ninguna decisión económica depende de un precio que una sola transacción pueda mover (pools con poca liquidez, spot sin TWAP).
- [ ] El oráculo tiene edad máxima aceptable y el contrato la verifica.

### Front-running y MEV
- [ ] Las operaciones sensibles al orden (subastas, mints limitados, liquidaciones) tienen mitigación (commit-reveal, límites por bloque, slippage).
- [ ] Ninguna aprobación o permiso puede ser explotada entre dos transacciones del usuario.

### Upgradeabilidad y storage
- [ ] Si es actualizable: quién puede actualizar, con qué demora (timelock) y qué pruebas exige; el layout de storage no se rompe entre versiones.
- [ ] Si es inmutable: la Carta lo dice y el plan de rollback es "nuevo contrato + migración", con quién paga la migración.

### Pausas y límites
- [ ] Existe una pausa de emergencia o la Carta justifica que no.
- [ ] Límites por operación y por período donde hay dinero de usuarios.

### Eventos y observabilidad
- [ ] Cada cambio de estado relevante emite un evento con lo necesario para reconstruirlo off-chain.
- [ ] Los errores son distinguibles (códigos o tipos), no un panic genérico.

### Dependencias y build
- [ ] Versiones fijadas del SDK y de librerías; sin dependencias sin mantener.
- [ ] Build reproducible: el hash desplegado se puede volver a obtener desde el commit.
- [ ] Tests de propiedad o fuzz sobre los invariantes de la Carta corrieron y están en el repo.

### Secretos
- [ ] `astra check` sin errores en el commit auditado.

## Stellar / Soroban

- [ ] **TTL del storage**: cada entrada persistente o de instancia tiene una estrategia de `extend_ttl`; se sabe qué pasa cuando expira (estado archivado: restaurable pero inaccesible hasta restaurar) y quién paga la restauración. El storage temporal no guarda nada que no pueda perderse.
- [ ] `require_auth` en la dirección correcta (la del usuario que paga o autoriza, no la del contrato) en cada función que actúa en nombre de alguien.
- [ ] Límites de recursos: instrucciones, lecturas/escrituras de ledger y tamaño de eventos dentro de los límites de la red; probado con `stellar contract invoke` en testnet y con simulación.
- [ ] Tokens: si implementa SEP-0041, toda la interfaz (`transfer`, `transfer_from`, `approve`, `allowance`, `balance`, `burn`, `decimals`, `name`, `symbol`) con la semántica del estándar; si el activo es clásico, se usa el SAC y no un token custom.
- [ ] Activos clásicos: trustlines requeridas, flags de autorización (`AUTH_REQUIRED`, `AUTH_REVOCABLE`, `CLAWBACK_ENABLED`) coherentes con la Carta, reservas base de las cuentas.
- [ ] Upgrade: `update_current_contract_wasm` restringido al administrador y con proceso (SEP-0049); el hash del WASM nuevo se verifica antes.
- [ ] Verificación pública: hash de WASM en el explorer coincide con el build reproducible (SEP-0055).

## EVM (Base, Syscoin NEVM, Rollux, otras)

- [ ] `msg.sender` versus `tx.origin`: nunca `tx.origin` para autorización.
- [ ] `delegatecall` solo a implementaciones controladas; proxies con slots ERC-1967; sin `selfdestruct`.
- [ ] Aprobaciones ERC-20: `approve` de 0 a N (race), `permit` (ERC-2612) con nonce y deadline verificados.
- [ ] `receive` / `fallback`: no aceptan ETH sin propósito; no ejecutan lógica cara.
- [ ] Gas griefing: bucles acotados, sin dependencia del gas restante para decidir.
- [ ] Firmas: EIP-712 con dominio (chainId incluido, EIP-155) y nonces; sin malleability.
- [ ] Verificación de fuente en el explorer (Blockscout, Basescan) con los mismos flags de compilación.

## Layer 2 (Base, Rollux)

- [ ] Se sabe quién opera el secuenciador y qué pasa si se detiene (retiros forzados, demora).
- [ ] Retiros a L1 con ventana de disputa (7 días en OP Stack): la Carta lo dice si afecta a los usuarios.
- [ ] Diferencias de opcodes o precompilados respecto de Ethereum L1 revisadas en la documentación de la cadena.
- [ ] Paymasters / cuentas inteligentes (ERC-4337, EIP-7702): quién paga el gas, límites, y qué pasa si el paymaster deja de patrocinar.

## Syscoin específico

- [ ] NEVM: bloques de ~2.5 minutos; la UX y los timeouts lo asumen.
- [ ] Puente L1 ↔ NEVM y Rollux ↔ NEVM: fondos en tránsito, finalidad y quién custodia mientras tanto.
- [ ] PoDA como disponibilidad de datos: qué datos se publican y por cuánto tiempo son recuperables.
- [ ] L1 UTXO: direcciones validadas con `astra address syscoin-utxo`, cambio (change) manejado, fees por byte estimadas con el nodo.
